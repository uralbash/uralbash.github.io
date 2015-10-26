.. post:: Apr 26, 2012 00:24
   :tags: Python, pycrypto, Virtualenv
   :category: Python
   :author: Uralbash
   :language: ru

Установка pycrypto в virtualenv
===============================

Если появляется ошибка типа ``"RuntimeError: autoconf error"``, то необходимо
установить ``C`` компилятор: 

.. code-block:: bash

   $ apt-get install gcc

или ``"src/MD2.c:31:20: fatal error: Python.h: Нет такого файла или каталога"`` 

.. code-block:: bash

   $ apt-get install python-dev

Далее стандартно: 

.. code-block:: bash

   $ source mydevenv/bin/activate
   $ pip install pycrypto
