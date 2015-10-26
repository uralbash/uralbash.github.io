.. post:: 19 Nov, 2011 20:47
   :tags: SQLAlchemy, event
   :category: SQLAlchemy
   :author: Uralbash
   :language: ru

Альтернатива SQL триггерам в SQLAlchemy
=======================================

Когда необходимо выполнять какие-то действия при записи в базу, обычно создают
триггеры ``before``, ``after INSERT UPDATE``. В :l:`SQLAlchemy` можно
реализовать аналогичный механизм но работающий на стороне питона а не БД. В
версии 0.6 это делается при помощи
:class:`~sqlalchemy.orm.interfaces.MapperExtension`, в 0.7
:class:`~sqlalchemy.orm.interfaces.MapperExtension` заменили на :func:`sqlalchemy.event.listen`.

MapperExtension
---------------

Пример :class:`~sqlalchemy.orm.interfaces.MapperExtension` с `сайта nagare.org
<http://www.nagare.org/trac/blog/SQLAlchemy MapperExtension>`_.

 Создадим таблицу пользователи:

.. code-block:: python

   class User(object):

       def __init__(self, name):
           self.name = name

   from sqlalchemy import Table, Column, Integer, Text

   users_table = Table('users',
                        Column('id', Integer, primary_key=True),
                        Column('name', Text))

Допустим мы хотим переводить первую букву имени пользователя в верхний регистр
при каждом SQL запросе ``UPDATE`` или ``INSERT``. Для этого в
:class:`~sqlalchemy.orm.interfaces.MapperExtension` есть методы аналогичные
триггерам SQL: ``before_insert``, ``before_update``, ``after_insert``,
``after_update`` и другие:

.. code-block:: python

   from sqlalchemy.orm import MapperExtension

   class CapitalizeNameMapperExtension(MapperExtension):

       def _capitalize_name(self, instance):
           if instance.name is not None:
               instance.name = instance.name.capitalize()

       def before_insert(self, mapper, connection, instance):
           self._capitalize_name(instance)

       def before_update(self, mapper, connection, instance):
           self._capitalize_name(instance)

Теперь надо привязать наше расширение к модели:

.. code-block:: python

   from sqlalchemy.orm import mapper

   user_mapper = mapper(User, users_table,
                        extension=CapitalizeNameMapperExtension())

Капитализация в действии:

.. code-block:: ipython

   >>> user1 = User("mynameis")
   >>> print user1.name
   mynameis

   >>> from nagare import database
   >>> database.session.add(user1)
   >>> print user1.name
   mynameis

   >>> database.session.flush()
   >>> print user1.name
   Mynameis

Вначале было введено имя в нижнем регистре, а после записи автоматом сработала
наша функция и перевела первую букву имени в верхний регистр.

Event
-----

Теперь рассмотрим пример с event для :l:`SQLAlchemy` 0.7 (текущей версией на данный момент)

.. code-block:: python

   class Host(Base):
       """Hosts of net."""
       __tablename__="host"

       ip = Column(postgresql.INET, unique=True)
       mac = Column(postgresql.MACADDR)
       description = Column(UnicodeText())

       created_by = Column(Integer, ForeignKey('user.user_id', onupdate="cascade", ondelete="restrict"))

       updated_by = Column(Integer, ForeignKey('user.user_id', onupdate="cascade", ondelete="restrict"))

       def __repr__(self):
           return "%s" % self.cidr

Создадим функции-триггеры:

.. code-block:: python

   def host_before_insert_listener(mapper, connection, target):
       # execute a stored procedure upon INSERT,
       # apply the value to the row to be inserted
       #target.calculated_value = connection.scalar(
       #                            "select my_special_function(%d)"
       #                            % target.special_number)
       identity = request.environ.get('repoze.who.identity')
       id = identity['user'].user_id
       target.created_by = id

   def host_before_update_listener(mapper, connection, target):
       # execute a stored procedure upon INSERT,
       # apply the value to the row to be inserted
       #target.calculated_value = connection.scalar(
       #                            "select my_special_function(%d)"
       #                            % target.special_number)
       identity = request.environ.get('repoze.who.identity')
       id = identity['user'].user_id
       target.updated_by = id

Здесь мы получаем из переменных окружения ``id`` текущего пользователя и
присваиваем его полю ``updated_by`` или ``created_by``. Привяжем наши функции к
модели.

.. code-block:: python

   # associate the listener function with CreatedMixin,
   # to execute during the "before_insert" hook
   event.listen(Host, 'before_insert', host_before_insert_listener)
   event.listen(Host, 'before_update', host_before_update_listener)

Всё, теперь при добавлении или обновлении Host будут автоматически заполняться
поля ``created_by`` или ``updated_by`` соответственно. На мой взгляд намного удобней
держать логику в одном месте, так как в будущем ее будет проще поддерживать и
проводить тестирование.
