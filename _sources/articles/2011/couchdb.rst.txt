.. post:: 28 Nov, 2011 10:53
   :tags: CouchDB, NoSQL
   :category: NoSQL
   :author: Uralbash
   :language: ru

.. _wtforms:

Основы CouchDB
==============

.. image:: /_static/2011/couchdb.jpg
   :align: left

:l:`CouchDB` - документо-ориентированная СУБД, в которой удобно хранить разную
информацию с изменяющимся количеством параметров. Например сущность товары
могут иметь бесконечное количество параметров. Создавать в РСУБД миллионы
таблиц или одну огромную не вариант. В :l:`couchdb` хранятся документы, т.е.
простые записи. В записи может быть сколько угодно разных параметров. А для
унификации записей обычно создают параметр type который заменяет имя таблицы в
РСУБД.

.. raw:: html

   <br clear="both"/>

Установка
---------

Debian
~~~~~~

.. code-block:: bash

   $ sudo apt-get install couchdb

From source
~~~~~~~~~~~

.. code-block:: bash

   $ wget http://erlang.org/download/otp_src_R12B-5.tar.gz
   $ tar -xvzf otp_src_R12B-5.tar.gz
   $ cd otp_src_R12B-5
   $ ./configure
   $ sudo make && make install

   $ wget http://download.filehat.com/apache/incubator/couchdb/0.8.1-incubating/apache-couchdb-0.8.1-incubating.tar.gz
   $ tar -xvzf apache-couchdb-0.8.1-incubating.tar.gz
   $ cd apache-couchdb-0.8.1-incubating
   $ ./configure
   $ sudo make && make install

MacOS, Android, iOS
~~~~~~~~~~~~~~~~~~~

http://www.couchbase.com/downloads

:l:`CouchDB` можно установить на смартфон, причем он может работать как
автономно, так и в режиме репликации когда появляется связь с основной базой.

Futon
-----

Futon_ это веб интерфейс для управления :l:`CouchDB`, что то типа :l:`phpmyadmin`. После
запуска :l:`CouchDB` появится доступ к Futon_ по http://127.0.0.1:5984/. Для того что
бы увидеть инструменты Futon нужно перейти на http://127.0.0.1:5984/_utils/

.. figure:: /_static/2011/couchdb1.png
   :align: center

   Couchdb Futon

Все манипуляции с базой происходят через http при помощи :l:`REST` интерфейса.
Поэтому можно пользоваться как браузером, так и curl или например модулем urllib в
python. Futon_ использует для запросов плагин на :l:`jQuery`. Посмотреть его можно
здесь: http://127.0.0.1:5984/_utils/script/jquery.couch.js

Пользователи
------------

По умолчанию к :l:`CouchDB` открыт доступ всем, но его можно ограничить нажав
на "Welcome to Admin Party! Everyone is admin. Fix this" в нижнем правом углу.

Создание документа
------------------

Вначале создадим базу (тоже кстати документ), нажимаем "Create Database..." и
вводим название, дальше нажимаем "New document" и заводим значения:

.. figure:: /_static/2011/couchdb2.png
   :align: center

   CouchDB создание документа

Для сохранения нажимаем "Save document". Как видно можно вводить русские
названия параметров. Поле _id формируется автоматически при создании. Поле _rev
формируется автоматически при каждом изменении, первая цифра означает
количество изменений. _rev используется для внутренних служб системы при
репликации.

Создание документа с помощью cURL
---------------------------------

Данные в :l:`CouchDB` обрабатываются в формате JSON. Создадим документ ``giant.json``:

.. code-block:: json
   :caption: giant.json

   {
       "manufacture": "Giant",
       "model":  "REVEL 2",
       "цвет": "синий",
       "цена": 16000,
       "type": "bike"
   }

Для вставки этого документа введем в консоле:

.. code-block:: bash

   $ curl -X POST http://127.0.0.1:5984/test/ -d @giant.json -H "Content-Type: application/json"

В базе появится документ типа:

.. code-block:: json

   {"ok":true,"id":"08601f0b9d3574a116ee390bf0000f44","rev":"1-67d8f0ad3100d85952dbb17ba68b08eb"}

Принятые правила в :l:`REST`:

| ``POST`` – создать новую запись
| ``GET`` – получить значения
| ``PUT`` – обновить записи
| ``DELETE`` – удалить

Выбрать все записи
------------------

В :l:`CouchDB` есть view которые по определенным условиям фильтруют данные.
Выбор всех данных это стандартная ``view _all_docs``.

.. code-block:: bash

   $ curl -X GET http://127.0.0.1:5984/mycouchshop/_all_docs

Получим:

.. code-block:: json

   {"total_rows":2,"offset":0,"rows":[
   {"id":"08601f0b9d3574a116ee390bf0000f44","key":"08601f0b9d3574a116ee390bf0000f44","value":{"rev":"1-67d8f0ad3100d85952dbb17ba68b08eb"}},
   {"id":"2f6904cbb5d279fbdbb9e9eef40015ec","key":"2f6904cbb5d279fbdbb9e9eef40015ec","value":{"rev":"3-6d0577a0a515178f0cd939b5caa19572"}}
   ]}

Создание представлений
----------------------

Представления работают по принципу ``map/reduce``:

| ``map`` - это выбор и фильтрация документов
| ``reduce`` - это последующая обработка данных

Простая ``map`` функция:

.. code-block:: javascript

   function (doc) {
           if (doc.type === "bike" && doc.manufacture) {
               emit(doc.manufacture, doc);
           }
       }

Функции создаются в правом верхнем углу в ``Temporary View``:

.. figure:: /_static/2011/couchdb3.png
   :align: center

Сохраним view по нажатию "Save As..."

.. figure:: /_static/2011/couchdb4.png
   :align: center

Вызовем из консоли:

.. code-block:: bash

   $ curl -X GET http://127.0.0.1:5984/test/_design/product/_view/bike

Результат:

.. code-block:: json

   {"total_rows":1,"offset":0,"rows":[
   {"id":"08601f0b9d3574a116ee390bf0000f44","key":"Giant","value":{"_id":"08601f0b9d3574a116ee390bf0000f44","_rev":"1-67d8f0ad3100d85952dbb17ba68b08eb","manufacture":"Giant","model":"REVEL 2","\u0446\u0432\u0435\u0442":"\u0441\u0438\u043d\u0438\u0439","\u0446\u0435\u043d\u0430":"16000","type":"bike"}}
   ]}

Reduce функции
--------------

Map функция:

.. code-block:: javascript

   function (doc) {
           if (doc.type === "bike" && doc.цена) {
               emit(doc._id, doc.цена);
           }
       }

Получим:

.. code-block:: json

   {"total_rows":2,"offset":0,"rows":[
   {"id":"08601f0b9d3574a116ee390bf0000f44","key":"08601f0b9d3574a116ee390bf0000f44","value":12000},
   {"id":"08601f0b9d3574a116ee390bf0001130","key":"08601f0b9d3574a116ee390bf0001130","value":16000}
   ]}

Reduce функция:

.. code-block:: javascript

   function(keys, prices) {
       return sum(prices);
   }

Получаем сумму:

.. code-block:: json

   {"rows":[
   {"key":null,"value":28000}
   ]}

В заключение можно сказать что NoSQL в виде :l:`CouchDB` мне показался
очень простым и удобным механизмом.

Для дополнительного чтения:
http://net.tutsplus.com/tutorials/getting-started-with-couchdb/

.. _Futon: https://couchdb.readthedocs.org/en/1.4.x/intro.html
