.. post:: May 10, 2012 11:40
   :tags: Python, pycrypto, Virtualenv
   :category: Python
   :author: Uralbash
   :language: ru

Обновление rrd задней датой при помощи faketime
===============================================

Иногда скрипты, выполнение которых зависит от времени, не срабатывают. Менять
время и запускать их повторно или менять код приложения неправильно. Что бы
выполнить их задним числом есть утилита ``faketime``.

Установка:

.. code-block:: bash

   $ sudo apt-get install faketime

Пример использования:

.. code-block:: bash

   $ faketime 'last Friday 5 pm' /bin/date
   $ faketime '2008-12-24 08:15:42' /bin/date

Также работает с ``wine``:

.. code-block:: bash

   $ faketime '2006-09-20' wine myprogramm.exe

У меня есть скрипт на питоне который добавляет записи в rrd каждый день.
Пересчитаем rrd за последние 3 дня.

.. code-block:: bash

   $ faketime '2012-05-05 5 am' python updatemyrrd.py
   $ faketime '2012-05-04 5 am' python updatemyrrd.py
   $ faketime '2012-05-03 5 am' python updatemyrrd.py
