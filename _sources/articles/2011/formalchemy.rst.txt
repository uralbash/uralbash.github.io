.. post:: Oct 27, 2011 03:31
   :tags: Pylons, routes
   :category: Pylons
   :author: Uralbash
   :language: ru

Админка FormAlchemy для Pylons и Jinja
======================================

Для :l:`SQLAlchemy` существуют интерфейсы управления моделями(что-то типа админки).
Как минимум их 2, это `Rum
<http://brickenstein.bitbucket.org/rum-docs/user/tutorial.html>`_ и
`formalchemy.ext.pylons <http://docs.formalchemy.org/formalchemy/ext/pylons.html>`_.

Рассмотрим :l:`FormAlchemy`. Вообще как обычно можно прочитать документацию, но я
опишу еще как это все состыковать с шаблонами на :l:`Jinja` и поддержкой полей
:l:`Postgres` таких как ``MAC``, ``CIDR``, ``NET`` и т.д..

Создаем контроллер ``admin``:

.. code-block:: bash

   $ paster controller admin

Далее редактируем его:

.. code-block:: python
   :caption: controllers/admin.py

   # coding=utf-8
   import logging

   from pylons import request, response, session, tmpl_context as c, url

   from formalchemy.ext.pylons.controller import ModelsController

   from webhelpers.paginate import Page

   from repoze.what.predicates import has_permission
   from repoze.what.plugins.pylonshq import ControllerProtector

   from myapp.lib.base import BaseController
   from myapp import model
   from myapp import forms
   from myapp.model import meta

   log = logging.getLogger(__name__)

   class AdminControllerBase(BaseController):
       model = model # where your SQLAlchemy mappers are
       forms = forms # module containing FormAlchemy fieldsets definitions
       def Session(self): # Session factory
           return meta.Session

       ## customize the query for a model listing
       # def get_page(self):
       #     if self.model_name == 'Foo':
       #         return Page(meta.Session.query(model.Foo).order_by(model.Foo.bar)
       #     return super(AdminControllerBase, self).get_page()

   AdminController = ModelsController(AdminControllerBase,
                                        prefix_name='admin',
                                        member_name='model',
                                        collection_name='models',)

Дальше добавляем пути:

.. code-block:: python
   :caption: config/routing.py

   # ADMIN
   # Map the /admin url to FA's AdminController
   # Map static files
   map.connect('fa_static', '/admin/_static/{path_info:.*}',
               controller='admin', action='static')
   # Index page
   map.connect('admin', '/admin', controller='admin', action='models')
   map.connect('formatted_admin', '/admin.json', controller='admin',
               action='models', format='json')
   # Models
   map.resource('model', 'models', path_prefix='/admin/{model_name}',
                controller='admin')

И добавим в шаблоны папку ``myapp/templates/forms`` можно взять с `github
<https://github.com/FormAlchemy/formalchemy/tree/master/pylonsapp/pylonsapp/templates>`_.

Правим следующим образом:

.. code-block:: python
   :caption: myapp/forms/__init__.py

   from pylons.templating import render_mako
   from pylons import config
   from myapp import model
   from myapp.lib.base import render
   from formalchemy import config as fa_config
   from formalchemy import templates
   from formalchemy import validators
   from formalchemy import fields
   from formalchemy import forms
   from formalchemy import tables
   from formalchemy.ext.fsblob import FileFieldRenderer
   from formalchemy.ext.fsblob import ImageFieldRenderer

   fa_config.encoding = 'utf-8'

   class TemplateEngine(templates.TemplateEngine):
       def render(self, name, **kwargs):
           return render_mako('/forms/%s.mako' % name, extra_vars=kwargs)
   fa_config.engine = TemplateEngine()

   class FieldSet(forms.FieldSet):
       pass

   class Grid(tables.Grid):
       pass

   ## Initialize fieldsets

   #Foo = FieldSet(model.Foo)
   #Reflected = FieldSet(Reflected)

   ## Initialize grids

   #FooGrid = Grid(model.Foo)
   #ReflectedGrid = Grid(Reflected)

Сейчас, как и пишут в документации, вы действительно по адресу
``http://localhost:5000/admin`` попадете в админку! Но только если вы не
используете особенностей БД, например :mod:`sqlalchemy.databases.postgresql`.
Вторая проблема в том что шаблоны админки не встроены в ваши шаблоны.

Для поддержки :l:`Postgres` полей в :l:`FormAlchemy` необходимо добавить:

.. code-block:: python
   :caption: lib/base.py

   from formalchemy.fields import FieldRenderer
   from formalchemy.tests import FieldSet

   from sqlalchemy.databases import postgresql

   class PostgresFieldRenderer(FieldRenderer):
       """render a field as a postgres field"""
       def render(self, **kwargs):
           return h.text_field(self.name, value=self.value, **kwargs)

   # fix postgres field in FormAlchemy
   # tnx for http://code.google.com/p/formalchemy/issues/detail?id=167
   FieldSet.default_renderers[postgresql.CIDR] = PostgresFieldRenderer
   FieldSet.default_renderers[postgresql.MACADDR] = PostgresFieldRenderer
   FieldSet.default_renderers[postgresql.INET] = PostgresFieldRenderer

Здесь мы создали 3 правила для :l:`FormAlchemy` как рендерить поля
``postgresql.CIDR``, ``postgresql.MACADDR`` и ``INET``. Этот трюк можно
проделать со всем чем угодно :) Более подробно здесь
http://code.google.com/p/formalchemy/issues/detail?id=167

И наконец встраиваем админку в ваш шаблон.

Для того что бы это сделать придется переписать все шаблоны на :l:`jinja` и
возможно поменять некоторые файлы (как минимум ``forms/__init__.py``). Я
приведу более ленивый и простой метод. У меня в шаблонах есть ``header.html``
который я инклудом добавляю в ``base.html`` в самом начале. Так же и здесь
добавим его в начало админки :l:`FormAlchemy`.

.. code-block:: python
   :caption: lib/base.py

   class BaseController(WSGIController):

       def __call__(self, environ, start_response):
           """Invoke the Controller"""
           # WSGIController.__call__ dispatches to the Controller method
           # the request is routed to. This routing information is
           # available in environ['pylons.routes_dict']

           # Костыль с mako шаблонами в админке FormAlchemy
           # FIXME: для чистоты кода, переписать шаблоны с mako на jinja в папке forms
           c.jinja_menu = render('/common/header.html')

И в шаблоне ``templates/form/restfieldset.mako``

.. code-block:: html+mako
   :caption: templates/form/restfieldset.mako

   <body>
      ${c.jinja_menu}
      <div id="content" class="ui-admin ui-widget">
      ...
      </div>
      ...
   </body>

Сразу после ``<body>`` добавляем ``${c.jinja_menu}``. Тем самым в админку уже
отдается отрендеренное меню.

В документации показан интерфейс ``fa.jquery`` на самом деле будет обычный. Для
:l:`jquery` его еще надо дополнительно пилить, добавлять в ``controller/admin``
``from fa.jquery.pylons import ModelsController as ModelsControllerJQ`` и
дальше как сказано в исходниках
https://github.com/FormAlchemy/fa.jquery/blob/master/fa/jquery/pylons.py
обернуть наш контроллер. Что то вроде этого:

.. code-block:: python

   AdminController = ModelsController(AdminControllerBase,
                                      prefix_name='admin',
                                      member_name='model',
                                      collection_name='models',)
   AdminController = ModelsControllerJQ(AdminController,
                                        prefix_name='admin',
                                        member_name='model',
                                        collection_name='models',)

В целом :l:`FormAlchemy` довольно мощная штука, но очень сильно привязана к
:l:`mako`, поэтому пользоваться ей не удобно и к сожалению придется искать что
то другое.
