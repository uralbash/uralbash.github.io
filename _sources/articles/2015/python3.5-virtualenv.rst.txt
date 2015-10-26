.. post:: 10 Oct, 2015 13:51
   :tags: python, python3, virtualenv
   :category: Python
   :author: Uralbash
   :language: ru

Установка python 3.5 в virtualenv
=================================

Все уже слышали про новый pyhton версии 3.5
(https://docs.python.org/3/whatsnew/3.5.html). Я постараюсь описать как начать
им пользоваться в вашем виртуальном окружении.

Скачиваем
---------

.. note::

   В `оф. документации
   <https://docs.python.org/devguide/setup.html#getting-the-source-code>`_
   предлагают скачать ртутью с фирменного сайта:

   .. code-block:: bash

      $ hg clone https://hg.python.org/cpython
      $ hg update 3.5

Скачиваем с гитхаба :github:`python/cpython`:

.. code-block:: bash

   git clone https://github.com/python/cpython.git

Выбираем ветку ``3.5``:

.. code-block:: bash

   git checkout 3.5

Собираем
--------

Укажем локальную директорию для сборки:

.. code-block:: bash

   ./configure --prefix=$HOME/Projects/bin/python3.5

Скомпилируем:

.. code-block:: bash

   make && make install

Теперь можно запускать:

.. code-block:: ipython

  $ $HOME/Projects/bin/python3.5/bin/python3
  Python 3.5.0+ (default, Oct 10 2015, 13:35:25)
  [GCC 4.9.2] on linux
  Type "help", "copyright", "credits" or "license" for more information.
  >>>

.. code-block:: ipython

  >>> {*range(4), 4, *(5, 6, 7)}
  {0, 1, 2, 3, 4, 5, 6, 7}
  >>> import asyncio
  >>> async def foo(bar): await asyncio.sleep(42)

virtualenv
----------

Укажем виртуальному окружению где находится интерпретатор cpython:

.. code-block:: bash

   $ mkvirtualenv --python=$HOME/Projects/bin/python3.5/bin/python3 python35_env
   Running virtualenv with interpreter /home/uralbash/Projects/bin/python3.5/bin/python3
   Using base prefix '/home/uralbash/Projects/bin/python3.5'
   New python executable in aiohttp/bin/python3
   Also creating executable in aiohttp/bin/python
   Installing setuptools, pip, wheel...done.
