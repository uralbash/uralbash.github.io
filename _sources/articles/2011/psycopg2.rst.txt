.. post:: 16 Dec, 2011 18:45
   :tags: python, virtualenv, psycopg2, PostgreSQL
   :category: Python
   :author: Uralbash
   :language: ru

Установка psycopg2 в virtualenv для Postgres 9.1
================================================

При установке :l:`psycopg` в :l:`virtualenv` может возникнуть ошибка:

.. code-block:: bash

   (mypythonenv)uralbash@server:~$ pip install psycopg
   Downloading/unpacking psycopg
     Downloading psycopg2-2.4.3.tar.gz (687Kb): 687Kb downloaded
     Running setup.py egg_info for package psycopg
       Error: You need to install postgresql-server-dev-X.Y for building a server-side extension or libpq-dev for building a client-side application.

       Complete output from command python setup.py egg_info:
       running egg_info

   creating pip-egg-info/psycopg2.egg-info

   writing pip-egg-info/psycopg2.egg-info/PKG-INFO

   writing top-level names to pip-egg-info/psycopg2.egg-info/top_level.txt

   writing dependency_links to pip-egg-info/psycopg2.egg-info/dependency_links.txt

   writing manifest file 'pip-egg-info/psycopg2.egg-info/SOURCES.txt'

   warning: manifest_maker: standard file '-c' not found

   Error: You need to install postgresql-server-dev-X.Y for building a server-side extension or libpq-dev for building a client-side application.



   ----------------------------------------
   Command python setup.py egg_info failed with error code 1
   Storing complete log in /home/uralbash/.pip/pip.log

Это возникает из-за утилиты ``pg_config`` которая требуется для установки :l:`psycopg`.
В случае если у Вас :l:`Postgres` 8.4 нужно добавить ``pg_config`` в ``PATH`` (как то
так ``PATH=$PATH:/usr/bin/``). Путь до ``pg_config`` можно узнать:

.. code-block:: bash

   $ whereis pg_config
   pg_config: /usr/bin/pg_config

И установить пакеты:

.. code-block:: bash

   $ sudo apt-get install libpq-dev python-dev

Но если стоит 9.1 версия то может возникнуть ошибка:

.. code-block:: bash

   Reading package lists... Done
   Building dependency tree
   Reading state information... Done
   Some packages could not be installed. This may mean that you have
   requested an impossible situation or if you are using the unstable
   distribution that some required packages have not yet been created
   or been moved out of Incoming.
   The following information may help to resolve the situation:

   The following packages have unmet dependencies:
    libpq-dev : Depends: libpq5 (= 8.4.9-0squeeze1) but 9.1.1-1~bpo60+1 is to be installed
   E: Broken packages

В этом случае устанавливаем :l:`psycopg` не в окружении через:

.. code-block:: bash

   $ sudo pip install psycopg2

Смотрим куда это встало:

.. code-block:: bash

   $ sudo python -c "import psycopg2;print psycopg2.__file__"
   /usr/lib/python2.6/dist-packages/psycopg2/__init__.pyc

И копируем к себе в lib'ы в :l:`virtualenv`:

.. code-block:: bash

  $ sudo cp -r /usr/lib/python2.6/dist-packages/psycopg2  mypythonenv/lib/python2.6/site-packages/

Копипастим яйцо:

.. code-block:: bash

   $ sudo cp /usr/lib/python2.6/dist-packages/psycopg2-2.4.4.egg-info  mypythonenv/lib/python2.6/site-packages/

Еще надо будет скопировать ``mx``:

.. code-block:: bash

   $ sudo cp -r /usr/lib/python2.6/dist-packages/mx  mypythonenv/lib/python2.6/site-packages/

Жеский дьютихак но главное что работает малой кровью.
