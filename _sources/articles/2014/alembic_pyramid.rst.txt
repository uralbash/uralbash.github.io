.. post:: 16 Feb, 2014 2:04
   :tags: Python, Pyramid, alembic
   :category: Pyramid
   :author: Uralbash
   :language: ru

миграции в Pyramid
==================

Пример миграций в пирамиде:

В :l:`Django` :pypi:`south` в :l:`Pyramid` :pypi:`alembic`. Создаём файл
``alembic.ini`` и указываем путь до настроек :, т.е. ``<имя проекта>:<название
папки>`` например ``hlp:alembic``:

.. code-block:: ini
   :caption: alembic.ini
   :emphasize-lines: 23-25

   pyramid.includes =
       pyramid_debugtoolbar
       pyramid_tm
       ziggurat_foundations.ext.pyramid.sign_in

   ziggurat_foundations.model_locations.User = hlp.models.auth:User
   #sqlalchemy.url = sqlite:///%(here)s/hlp.sqlite
   sqlalchemy.url = postgresql://postgres:postgres@localhost:5432/hlp

   # By default, the toolbar only appears for clients from IP addresses
   # '127.0.0.1' and '::1'.
   # debugtoolbar.hosts = 127.0.0.1 ::1

   ###
   # wsgi server configuration
   ###

   [server:main]
   use = egg:waitress#main
   host = 0.0.0.0
   port = 6543

   [alembic]
   # path to migration scripts
   script_location = hlp:alembic

в проекте создаем папку ``alembic`` со следующей структурой:

.. code-block:: bash

    alembic/$ tree
   .
   ├── env.py
   ├── script.py.mako
   └── versions
       └── 29ac938c56f_starting.py

.. code-block:: python
   :caption: env.py

   from __future__ import with_statement
   from alembic import context
   from sqlalchemy import engine_from_config
   from sqlalchemy.engine.base import Engine
   from pyramid.paster import setup_logging

   # this is the Alembic Config object, which provides
   # access to the values within the .ini file in use.
   config = context.config

   from hlp.models import Base

   setup_logging(config.config_file_name)

   engine = engine_from_config(config.get_section('app:main'), 'sqlalchemy.')

   target_metadata = Base.metadata


   def run_migrations_offline():
       """Run migrations in 'offline' mode.

       This configures the context with just a URL
       and not an Engine, though an Engine is acceptable
       here as well. By skipping the Engine creation
       we don't even need a DBAPI to be available.

       Calls to context.execute() here emit the given string to the
       script output.

       """
       context.configure(url=engine.url)

       with context.begin_transaction():
           context.run_migrations()


   def run_migrations_online():
       """Run migrations in 'online' mode.

       In this scenario we need to create an Engine
       and associate a connection with the context.

       """

       if isinstance(engine, Engine):
           connection = engine.connect()
       else:
           raise Exception(
               'Expected engine instance got %s instead' % type(engine)
           )

       context.configure(
           connection=connection,
           target_metadata=target_metadata
       )

       try:
           with context.begin_transaction():
               context.run_migrations()
       finally:
           connection.close()

   if context.is_offline_mode():
       run_migrations_offline()
   else:
       run_migrations_online()

.. code-block:: mako
   :caption: script.py.mako

   """${message}

   Revision ID: ${up_revision}
   Revises: ${down_revision}
   Create Date: ${create_date}

   """

   # revision identifiers, used by Alembic.
   revision = ${repr(up_revision)}
   down_revision = ${repr(down_revision)}

   from alembic import op
   import sqlalchemy as sa
   ${imports if imports else ""}

   def upgrade():
   ${upgrades if upgrades else "pass"}


   def downgrade():
   ${downgrades if downgrades else "pass"}

Создаем первую миграцию:

.. code-block:: bash

   $ alembic revision --autogenerate -m "create table"

переходим к последней миграции:

.. code-block:: bash

   $ alembic upgrade head

Есть ешё ништиковые команды типа:

.. code-block:: bash

   $ alembic upgrade +2
   $ alembic downgrade -1
   $ alembic upgrade 29ac
   $ alembic history
   $ alembic current

| пример исключения таблиц из миграций http://blog.utek.pl/2013/ignoring-tables-in-alembic/
| подробнее здесь https://alembic.readthedocs.org/en/latest/tutorial.html
