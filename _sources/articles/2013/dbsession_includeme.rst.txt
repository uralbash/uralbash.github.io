.. post:: 17 Feb, 2013 17:43
   :tags: python, Pyramid, SQLalchemy
   :category: Pyramid
   :author: Uralbash
   :language: ru

Использование SQLAlchemy в дополнениях к Pyramid
================================================

У меня есть дополнение к pyramid которое я включаю почти в каждый проект при
помощи ``includeme`` (:meth:`~pyramid.config.Configurator.include`). Приложение
это дает мне простой ``CRUD`` интерфейс с :l:`Jinja` шаблонами малой кровью.
Что позволяет избавиться от монстра :l:`FormAlchemy`.  Естественно каждый
проект имеет свое название поэтому пришлось применить немного магии что бы
создать универсальный механизм получения ``DBSession`` в своем подключаемом
дополнении.

Примерная структура папок дополнения:

.. code-block:: bash

   pyramid_ext_plugin
   |-- __init__.py
   `-- action.py

В ``__init__.py`` находятся все настройки ``includeme``, а в ``action.py``
действия с базой. Для получения ``DBSession`` в ``action.py`` исправим сначала
``__init__.py``:

.. code-block:: python
   :caption: __init__.py

   import sqlalchemy
   import sqlalchemy.orm as orm
   from zope.sqlalchemy import ZopeTransactionExtension

   DBSession = None


   def includeme(config):
       global DBSession
       engine = sqlalchemy.engine_from_config(config.registry.settings)
       if DBSession is None:
           DBSession = orm.scoped_session(
               orm.sessionmaker(extension=ZopeTransactionExtension()))
       DBSession.remove()
       DBSession.configure(bind=engine)

Все Ж) теперь можно в ``action.py`` писать так:

.. code-block:: python
   :caption: action.py

   from pyramid_ext_pugin import DBSession
