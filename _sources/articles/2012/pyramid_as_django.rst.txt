.. post:: Jul 05, 2012 10:42
   :tags: Python, Pyramid, Django
   :category: Pyramid
   :author: Uralbash
   :language: ru

Структура Pyramid приложений как в Django
=========================================

Одной из причин отказа развивать ветку :l:`Pylons` стала его архитектура
проекта.  Все контроллеры хранятся в директории ``controllers``, модели в
``models``, шаблоны в ``templates``. Это очень удобно когда у вас маленький
проект, но если он разрастается до десятков и сотен сущностей, то становится
крайне сложно скакать по этим папкам выискивая нужный файл, относящийся именно
к этой сущности. В :l:`Django` сделано по другому, в проекте хранятся
приложения (``application``) - это такие маленькие подпрограммы которые
отвечают за конкретный функционал проекта (например фотогалерея
:pypi:`django-photologue` или дерево сайта :pypi:`django-sitetree` и прочее).
Каждое такое приложение имеет свою папку и уже в ней хранятся контроллеры
(``views`` в данном случае) и модели (``models``). Т.е. вместо такой
архитектуры :l:`Pylons`:

.. code-block:: bash

   Project/
       controllers/
           controllers1.py
           controllers2.py
       model/
           models1.py
           models2.py
       templates/
           templates1/
           templates2/
       routes.py

мы получаем:

.. code-block:: bash

   Project/
       app1/
           models.py
           routes.py
           views.py
       app2/
           models.py
           routes.py
           views.py
       templates/
           templates1/
           templates2/

Разработчики не стали менять архитектуру :l:`Pylons` и создавать Pylons 2.0, а
начали развивать новый проект :l:`Pyramid` с похожей на :l:`Django` структурой
проекта. При этом :l:`Pylons` не считается устаревшим, а просто имеет немного
другой подход к разработке.

Стоит ли сейчас выбирать :l:`Pylons`? Да, если вы его хорошо знаете и не
планируете создавать слишком масштабное приложение, иначе лучше все таки
выбрать :l:`Pyramid`.

Посмотрим как в нем организовать структуру проекта:

После создания проект выглядит так:

.. code-block:: bash

   MyProject/
   ├── CHANGES.txt
   ├── development.ini
   ├── MANIFEST.in
   ├── myproject
   │   ├── __init__.py
   │   ├── models.py
   │   ├── scripts/
   │   ├── static/
   │   ├── templates/
   │   ├── tests.py
   │   └── views.py
   ├── production.ini
   ├── README.txt
   ├── setup.cfg
   └── setup.py

Наша задача перенести ``models.py``, ``views.py``, ``tests.py`` в ``app1``.
Создаем папку ``app1`` и переносим файлы.

Должно получится так:

.. code-block:: bash

   MyProject/
   .
   ├── CHANGES.txt
   ├── development.ini
   ├── MANIFEST.in
   ├── myproject
   │   ├── app1
   │   │   ├── __init__.py
   │   │   ├── models.py
   │   │   ├── tests.py
   │   │   └── views.py
   │   ├── __init__.py
   │   ├── scripts
   │   │   ├── initializedb.py
   │   │   └── __init__.py
   │   ├── static
   │   │   ├── favicon.ico
   │   │   ├── footerbg.png
   │   │   ├── headerbg.png
   │   │   ├── ie6.css
   │   │   ├── middlebg.png
   │   │   ├── pylons.css
   │   │   ├── pyramid.png
   │   │   ├── pyramid-small.png
   │   │   └── transparent.gif
   │   └── templates
   │       └── mytemplate.pt
   ├── production.ini
   ├── README.txt
   ├── setup.cfg
   └── setup.py

Меняем ``__init__.py`` в проекте:

Вместо

.. code-block:: python

   from pyramid.config import Configurator
   from sqlalchemy import engine_from_config

   from .models import DBSession

   def main(global_config, **settings):
       """ This function returns a Pyramid WSGI application.
       """
       engine = engine_from_config(settings, 'sqlalchemy.')
       DBSession.configure(bind=engine)
       config = Configurator(settings=settings)
       config.add_static_view('static', 'static', cache_max_age=3600)
       config.add_route('home', '/')
       config.scan()
       return config.make_wsgi_app()

Делаем

.. code-block:: python

   from pyramid.config import Configurator
   from sqlalchemy import engine_from_config

   from .models import DBSession

   def main(global_config, **settings):
       """ This function returns a Pyramid WSGI application.
       """
       engine = engine_from_config(settings, 'sqlalchemy.')
       DBSession.configure(bind=engine)
       config = Configurator(settings=settings)
       config.add_static_view('static', 'static', cache_max_age=3600)

       # add config for each of your subapps
       config.include('myproject.app1')

       # pyramid_jinja2 configuration
       config.include('pyramid_jinja2')
       config.add_jinja2_search_path("myproject:templates")

       return config.make_wsgi_app()

Теперь в этом ``__init__.py`` мы будем хранить глобальные настройки, а в инитах
апликайшинах настройки самих апликайшинов.

Добавляем ``models.py`` в основной проект:

.. code-block:: bash

   MyProject/
   .
   ├── CHANGES.txt
   ├── development.ini
   ├── MANIFEST.in
   ├── myproject
   │   ├── app1
   │   │   ├── __init__.py
   │   │   ├── models.py
   │   │   ├── tests.py
   │   │   └── views.py
   │   ├── __init__.py
   │   ├── scripts
   │   │   ├── initializedb.py
   │   │   └── __init__.py
   │   ├── models.py
   │   ├── static
   │   │   ├── favicon.ico
   │   │   ├── footerbg.png
   │   │   ├── headerbg.png
   │   │   ├── ie6.css
   │   │   ├── middlebg.png
   │   │   ├── pylons.css
   │   │   ├── pyramid.png
   │   │   ├── pyramid-small.png
   │   │   └── transparent.gif
   │   └── templates
   │       └── mytemplate.pt
   ├── production.ini
   ├── README.txt
   ├── setup.cfg
   └── setup.py

В нем будет хранится сессия :l:`SQLAlchemy` общая для всех проектов:

.. code-block:: python

   from zope.sqlalchemy import ZopeTransactionExtension
   from sqlalchemy.ext.declarative import declarative_base
   from sqlalchemy.orm import (
       scoped_session,
       sessionmaker,
       )

   DBSession = scoped_session(sessionmaker(extension=ZopeTransactionExtension()))
   Base = declarative_base()

Меняем ``__init__.py`` в ``app1``:

.. code-block:: python

   def includeme(config):
       config.add_route('home', '/')
       config.scan()

Меняем ``models.py`` в ``app1``:

.. code-block:: python

   from sqlalchemy import (
       Column,
       Integer,
       Text,
       )

   from sqlalchemy.ext.declarative import declarative_base

   from sqlalchemy.orm import (
       scoped_session,
       sessionmaker,
       )

   from ..models import (
       DBSession,
       Base,
       )

   class MyModel(Base):
       __tablename__ = 'models'
       id = Column(Integer, primary_key=True)
       name = Column(Text, unique=True)
       value = Column(Integer)

       def __init__(self, name, value):
           self.name = name
           self.value = value

Меняем ``tests.py``:

.. code-block:: python

   import unittest
   import transaction

   from pyramid import testing

   from ..models import DBSession

   class TestMyView(unittest.TestCase):
       def setUp(self):
           self.config = testing.setUp()
           from sqlalchemy import create_engine
           engine = create_engine('sqlite://')
           from .models import (
               Base,
               MyModel,
               )
           DBSession.configure(bind=engine)
           Base.metadata.create_all(engine)
           with transaction.manager:
               model = MyModel(name='one', value=55)
               DBSession.add(model)

       def tearDown(self):
           DBSession.remove()
           testing.tearDown()

       def test_it(self):
           from .views import my_view
           request = testing.DummyRequest()
           info = my_view(request)
           self.assertEqual(info['one'].name, 'one')
           self.assertEqual(info['project'], 'MyProject')

Меняем ``views.py``:

.. code-block:: python

   from sqlalchemy.exc import DBAPIError

   from ..models import DBSession
   from .models import MyModel

   @view_config(route_name='home', renderer='../templates/mytemplate.pt')
   def my_view(request):
       try:
           one = DBSession.query(MyModel).filter(MyModel.name=='one').first()
       except DBAPIError:
           return Response(conn_err_msg, content_type='text/plain', status_int=500)
       return {'one':one, 'project':'MyProject'}

   conn_err_msg = """\
   Pyramid is having a problem using your SQL database.  The problem
   might be caused by one of the following things:

   A.  You may need to run the "initialize_MyProject_db" script
       to initialize your database tables.  Check your virtual
       environment's "bin" directory for this script and try to run it.

   B.  Your database server may not be running.  Check that the
       database server referred to by the "sqlalchemy.url" setting in
       your "development.ini" file is running.

   After you fix the problem, please restart the Pyramid application to
   try it again.
   """

Меняем ``initializedb.py`` в ``scripts``:

.. code-block:: python

   import os
   import sys
   import transaction

   from sqlalchemy import engine_from_config

   from pyramid.paster import (
       get_appsettings,
       setup_logging,
       )

   from ..models import (
       DBSession,
       Base,
       )

   from ..app1.models import MyModel

   def usage(argv):
       cmd = os.path.basename(argv[0])
       print('usage: %s <config_uri>\n'
             '(example: "%s development.ini")' % (cmd, cmd))
       sys.exit(1)

   def main(argv=sys.argv):
       if len(argv) != 2:
           usage(argv)
       config_uri = argv[1]
       setup_logging(config_uri)
       settings = get_appsettings(config_uri)
       engine = engine_from_config(settings, 'sqlalchemy.')
       DBSession.configure(bind=engine)
       Base.metadata.create_all(engine)
       with transaction.manager:
           model = MyModel(name='one', value=1)
           DBSession.add(model)

| Выполняем ``python setup.py install``, запускаем проект, профит!
| Остальные апликайшины добавляются по аналогии.
| Навеянно этим http://stackoverflow.com/questions/6012991/pyramid-project-structure
