.. post:: Jul 04, 2012 11:51
   :tags: Python, FormAlchemy, Pyramid
   :category: Pyramid
   :author: Uralbash
   :language: ru

pyramid_formalchemy
===================

:pypi:`FormAlchemy` это ``CRUD`` для :l:`SQLAlchemy`. Для :l:`Pylons`
существовало расширение прямо в самом модуле в разделе ext. Для Pyramid создали
отдельный пакет :pypi:`pyramid_formalchemy`. Посмотрим как это работает:

Создаем проект:

.. code-block:: bash

   $ pcreate -s alchemy -s pyramid_fa myapp

добавляем в проект файл ``forms.py``:

.. code-block:: python
   :caption: forms.py

   from formalchemy import FieldSet, Grid

структура файлов должна выглядеть так:

.. code-block:: bash

   | |+static/
   | |+templates/
   | |-__init__.py
   | |-faforms.py
   | |-fainit.py
   | |-faroutes.py
   | |-forms.py
   | |-models.py
   | |-tests.py
   | `-views.py

изменяем ``__init__.py``:

.. code-block:: python
   :caption: __init__.py

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

       # pyramid_formalchemy's configuration
       config.include('pyramid_fanstatic')
       config.include('pyramid_formalchemy')
       config.include('fa.jquery')

       # register an admin UI
       config.formalchemy_admin('/admin', package='youAppName',
               view='fa.jquery.pyramid.ModelView')

    return config.make_wsgi_app()

Заходим в http://0.0.0.0:6543/admin/ и радуемся.

.. image:: /_static/2012/pyramid_formalchemy.png
   :align: center

Online демо находится `здесь <http://docs.formalchemy.org/demo/admin/>`_.
