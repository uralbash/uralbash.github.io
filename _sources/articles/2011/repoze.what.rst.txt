.. post:: Oct 26, 2011 23:35
   :tags: Pylons, repoze, jinja
   :category: Pylons
   :author: Uralbash
   :language: ru

.. _repoze-what:

Авторизация в Pylons за 5 мин при помощи repoze.what
====================================================

Статья по сути вольный перевод `PylonsTemplates: extra Paster templates for
Pylons apps <http://countergram.com/open-source/PylonsTemplates>`_ с моими
дополнениями.

PylonsTemplates_ дает вам дополнительные шаблоны в paster для приложений на
:l:`Pylons`. После установки PylonsTemplates_ можно создать новый проект на
:l:`Pylons` примерно так:

.. code-block:: bash

   $ paster create -t [templatename] [projectname]

pylons_repoze_what
------------------


Шаблон ``pylons_repoze_what`` добавляет систему авторизации основанную на
:mod:`repoze.what` и :mod:`repoze.what.plugins.quickstart`. (При этом аутентификация на
:mod:`repoze.who` устанавливается автоматически.)

Шаблон включает в себя:

* Модели ``User``, ``Group`` и ``Permission`` для :l:`SQLALchemy`
* Контроллер login (& logout)
* Простой шаблон для входа
* Зависимость от :mod:`repoze.what.plugins.pylonshq`, включающая декораторы которые можно
  использовать в контроллерах и действиях (action).
* Закоментированный код в websetup.py который создает user, group и permission.

.. _PylonsTemplates: https://github.com/countergram/PylonsTemplates/

Список всех шаблонов:

.. code-block:: bash

   paster create --list-templates

Создание проекта:

.. code-block:: bash

   paster create -t pylons_repoze_what myapp

Пример с repoze.what-pylons
---------------------------

После создания проекта используя ``paster create -t pylons_repoze_what``, вам нужно
защитить контроллеры и их действия от несанкционированного доступа. Ниже
простой пример:

.. code-block:: python

   from repoze.what.predicates import has_permission
   from repoze.what.plugins.pylonshq import ActionProtector

   class HelloWorldController(BaseController):
       @ActionProtector(has_permission('be_cool'))
       def index(self):
           return 'Hello World'

Здесь запрещен доступ к действию ``index`` всем у кого нет прав ``be_cool``. Что бы
запретить весь контроллер есть декоратор
:class:`repoze.what.plugins.pylonshq.protectors.ControllerProtector`. Более
подробно об этом можно почитать в документации на :mod:`repoze.what.plugins.pylonshq`.

У меня в шаблонах отображается постоянно кто сейчас залогирован.
Что бы это сделать, создадим ``lib/auth.py``

.. code-block:: python

   from pylons import request

   def get_user(default):
       """Return the user object from the `repoze.who` Metadata Plugin

       :param default: default item to send back if user not logged in

       Since we might not be logged in and template choke on trying to output
       None/empty data we can pass in a blank User object to get back as a default
       and the templates should work ok with default empty values on that

       """
       if 'repoze.who.identity' in request.environ:
           return request.environ['repoze.who.identity']['user']
       else:
           return default

И добавим контекстную переменную во все контроллеры при помощи ``lib/base.py``

.. code-block:: python

   from gottlieb.model.auth import User
   from gottlieb.lib import auth


       def __call__(self, environ, start_response):
           """Invoke the Controller"""
           # WSGIController.__call__ dispatches to the Controller method
           # the request is routed to. This routing information is
           # available in environ['pylons.routes_dict']

           # if there's no user set, just setup a blank instance
           c.current_user = auth.get_user(User())

           try:
               return WSGIController.__call__(self, environ, start_response)
           finally:
               meta.Session.remove()

Теперь можно в шаблоны вставлять что то вроде того (примеры на :l:`Jinja`):

.. code-block:: html+jinja

   <div class="userBox">
      {% if c.current_user.user_name %}
          {{ c.current_user.user_name }} <a href="/logout">(logout)</a>
      {% else %}
          <a href="/login">login</a>
      {% endif %}
   </div>

или

.. code-block:: html+jinja

   {% for group in c.current_user.groups %}
       {% if group.group_name == 'admin' %}
           <li>
              <a href="/admin" target="_self">Admin</a>
           </li>
       {% endif %}
   {% endfor %}

.. figure:: /_static/2011/header_login_logout.png
   :align: center

   отображение залогированного пользователя

.. figure:: /_static/2011/login_form.png
   :align: center

   форма авторизации
