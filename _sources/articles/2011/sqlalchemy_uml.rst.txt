.. post:: Oct 24, 2011 19:27
   :tags: SQLalchemy, UML, Django, graphviz
   :category: SQLalchemy
   :author: Uralbash
   :language: ru

SQLalchemy UML диаграмма
========================

.. note::

   UPD: :l:`sadisplay` - замечательная штука!

Для визуализации своей базы в :l:`SQLAlchemy` удобно использовать :l:`graphviz`
и библиотеку :l:`sqlalchemy_schemadisplay`.

Установка:

.. code-block:: bash

   $ apt-get install graphviz
   $ pip install sqlalchemy_schemadisplay

Далее читаем доки `SQLAlchemy Schema Display
<http://www.sqlalchemy.org/trac/wiki/UsageRecipes/SchemaDisplay>`_

Для Ъ:

Схема БД строится на основании данных базы.

.. code-block:: python

   from sqlalchemy import MetaData
   from sqlalchemy_schemadisplay import create_schema_graph

   # create the pydot graph object by autoloading all tables via a bound metadata object
   graph = create_schema_graph(
      metadata=MetaData('postgres://user:pwd@host/database'),
      show_datatypes=False,   # The image would get nasty big if we'd show the datatypes
      show_indexes=False,     # ditto for indexes
      rankdir='LR',           # From left to right (instead of top to bottom)
      concentrate=False       # Don't try to join the relation lines together
   )
   graph.write_png('dbschema.png') # write out the file

.. figure:: /_static/2011/dbschema.png
   :align: center

   Схема БД Postgres

Схема UML строится по моделям проекта.

.. code-block:: python

   from myapp import model
   from sqlalchemy_schemadisplay import create_uml_graph
   from sqlalchemy.orm import class_mapper

   # lets find all the mappers in our model
   mappers = []
   for attr in dir(model):
       if attr[0] == '_': continue
       try:
           cls = getattr(model, attr)
           mappers.append(class_mapper(cls))
       except:
           pass

   # pass them to the function and set some formatting options
   graph = create_uml_graph(mappers,
       show_operations=False,       # not necessary in this case
       show_multiplicity_one=False  # some people like to see the ones, some don't
   )
   graph.write_png('schema.png') # write out the file

.. figure:: /_static/2011/schema.png
   :align: center

   Схема моделей в Pylons

Для :l:`Django` кодеров есть модуль :l:`django-extension` который добавляет
много полезных команд для ``manage.py``. Вот мой вариант скрипта для :l:`Django`:

.. code-block:: bash
   :caption: project_dir/_visualozation/visualized.sh

   curent_d="`date +%H%M_%d%m%y`"
   exec python ../manage.py graph_models -a -g -o scheme_of_$curent_d.png

.. figure:: /_static/2011/django_uml.png
   :align: center

   пример django-extension + graphviz
