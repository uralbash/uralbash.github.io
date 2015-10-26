.. post:: 26 Aug, 2013 17:06
   :tags: python, pyramid, sacrud
   :category: Pyramid
   :author: Uralbash
   :language: ru

Запись в БД через sacrud используя SQLAlchemy session.
======================================================

Для простых ``CRUD`` действий с БД, можно воспользоваться модулем
:mod:`~sacrud.action` из :PyPi:`sacrud`. Это немного сократит код и добавит
некоторой универсальности в ПО со сложной логикой.

.. code-block:: python

   from sacrud import action
   from models import (
       DBSession,
       TestTable,
   )

   hstore_data = str({'param1': 'bla bla bla',
               'param2': 'bla bla bla2',
               'param3': '7389a498-9347-48e3-835d-c3900dcd2566',
               'patam4': 'dddddddd'})

   param = {'value': ('123',),
            'description': ('test description',),
            'myhash': [hstore_data, ],
           }
   # записывает транзакцию в БД
   action.create(DBSession, TestTable, param)

параметры в виде списка сделаны для того что бы можно было принимать
множественные значения поля с HTML формы.

UPD: в новой версии можно делать так:

.. code-block:: python

   param = {'value': '123',
            'description': 'test description',
            'myhash': hstore_data,
           }
   # записывает транзакцию в БД
   action.create(DBSession, TestTable, param)
