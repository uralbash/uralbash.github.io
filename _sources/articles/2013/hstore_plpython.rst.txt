.. post:: 22 Jul, 2013 18:17
   :tags: python, plpython, PostgreSQL
   :category: Python
   :author: Uralbash
   :language: ru

Опиум hstore в plpython
=======================

Приведу просто пример триггера на ``plpython`` который использует данные из
поля ``hstore``:

.. code-block:: python

   if TD["event"] in ("INSERT", "UPDATE"):
       obj_id = TD["new"]["id"]
       data = TD["new"]["data"]
       prepare_data = plpy.prepare("SELECT hstore($1)->$2 as val;", ["character varying", "character varying"])
       param1 = plpy.execute(prepare_data, [data, "param1"])[0]['val']
       param2 = plpy.execute(prepare_data, [data, "param2"])[0]['val']
       param3 = plpy.execute(prepare_data, [data, "param3"])[0]['val']
       param4 = plpy.execute(prepare_data, [data, "param4"])[0]['val']

Вот так я получаю параметры из ``hstore``, вместо
``TD["new"]["data"]["param1"]``. Может я что то делаю не так?
