.. post:: Apr 11, 2012 11:56
   :tags: Python, SQLalchemy
   :category: Python
   :author: Uralbash
   :language: ru

SQLAlchemy и большие таблицы (``Memory error``)
===============================================

:l:`SQLAlchemy` в больших таблицах при таком запросе ``s.query(TableName).all()``
зависает и выдает ошибку ``Memory error``. Для решения проблемы нужно
использовать метод :meth:`~sqlalchemy.orm.query.Query.yield_per`. Если
необходимо еще изменять данные, то нужно делать коммиты каждые N записей.

Пример:

.. code-block:: python

   def fdbarp(self):
       ''' Функция переноса арпов из одной таблицы в другую.
           Около 3млн записей.
       '''
       for i, arp in enumerate(s.query(Arp).yield_per(100)):
           new_arp = newArp()
           new_arp.equipment_id = arp.equipment
           new_arp.mac = arp.mac
           new_arp.ip = arp.ip
           new_arp.first_seen = arp.first_seen
           new_arp.last_seen = arp.last_seen
           new_arp.times = arp.times

           s.add(new_arp)
           if not i%10000:
               s.commit()
               s.close()
           print "arp ", i

       s.commit()
