.. post:: 15 Mar, 2014 17:53
   :tags: Python, SQLalchemy, Django, PostgreSQL, sqlite
   :category: Python
   :author: Uralbash
   :language: ru

Перенос БД с sqlite на postgres
===============================

``ORM`` позволяет быстро переключатся между БД не учитывая их диалект
(практически).  Но данные хранятся физически в разных местах и естественно их
надо переносить, например при переключении с :l:`sqlite` на :l:`PostgreSQL`. В
:l:`Django` есть встроенный функционал в виде:

.. code-block:: bash

   # Выгрузка в JSON
   $ python manage.py dumpdata myapp.A > a.json

   # Загрузка из JSON
   $ python manage.py loaddata a.json

Т.е. мы выгружаем данные из :l:`sqlite` в ``JSON`` формат, затем меняем строку
подключения на :l:`postgres` и выполняем загрузку из ``JSON``. Очень удобно, но
почему то этот метод не работает, либо работает только при переносе из ``sqlite
-> sqlite``, что в принципе не очень интересно, точнее бессмысленно. Есть какие
то решений с бубном как это вот:

| http://macrotoma.blogspot.ru/2012/10/solved-move-django-from-sqlite-to-mysql.html,
| http://blog.abourget.net/2009/7/7/exporting-sql-schemas-from-sqlalchemy-table-definitions/.

Эти методы не универсальны, потому что имеют привязку к моделям (ORM), требуют
для переноса проект на :l:`Django` и ручные действия вроде создания схемы и
выполнения миграций (далеко не всегда миграции созданы правильно).

Я написал небольшой пример как можно перевести данные не имея фреймворков, не
привязываясь к моделям, указав только две строки подключения откуда переносить
и куда (вроде было что то похожее на руби). За основы взят пример из этой статьи
http://www.tylerlesmann.com/2009/apr/27/copying-databases-across-platforms-sqlalchemy/.
Где предлагается указать дополнительно названия таблиц.

В :l:`sqlalchemy` с весии 9.1 появилась встроенная возможность автоматического
определения схемы БД
http://docs.sqlalchemy.org/en/rel_0_9/orm/extensions/automap.html. Правда
намного раньше появились сторонние решения:

* http://turbogears.org/2.1/docs/main/Utilities/sqlautocode.html
* https://sqlsoup.readthedocs.org/en/latest/.

Первое что мы сделаем, это получим схему БД:

.. code-block:: python

   from sqlalchemy.ext.automap import automap_base

   def get_metadata(self, engine):
      # produce our own MetaData object
      metadata = MetaData()

      # we can reflect it ourselves from a database, using options
      # such as 'only' to limit what tables we look at...
      # only = ['news_news', 'pages_page']
      if self.only:
          metadata.reflect(engine, only=self.only)
      else:
          metadata.reflect(engine)

      # we can then produce a set of mappings from this MetaData.
      Base = automap_base(metadata=metadata)

      # calling prepare() just sets up mapped classes and relationships.
      Base.prepare()
      return metadata, Base

Дальше получаем все таблицы:

``tables = Base.classes``

Создаем такую же структуру БД в :l:`postgres`:

``metadata.create_all(self.engine_dst)``

По очереди проходим каждую таблицу и переносим из нее данные в новую БД:

.. code-block:: python

   for table in tables:
       columns = table.__table__.c.keys()
       print 'Transferring records to %s' % table.__table__.name
       for record in self.session.query(table).all():
           data = dict(
               [(str(column), getattr(record, column)) for column in columns]
           )
           NewRecord = quick_mapper(table.__table__)
           self.session_dst.merge(NewRecord(**data))
           self.session_dst.commit()

Если возникли конфликты можно их решить переопределением типов полей. Например
при переносе из :l:`sqlite` в :l:`postgres` тип полей ``DATETIME`` нужно
заменить на ``DateTime``:

.. code-block:: python

   dialect = self.engine.dialect.name
   dialect_dst = self.engine_dst.dialect.name
   if dialect == dialect_dst:
       return
   for table in self.tables:
       columns = table.__table__.c
       for column in columns:
           if dialect_dst == 'postgresql':
               # DATETIME->DateTime
               if isinstance(column.type, DATETIME):
                   column.type = DateTime()

Пример запуска:

.. code-block:: bash

   $ python convertdbdata.py -f "sqlite:///fromMydb.sqlite" -t "postgresql://postgres:postgres@localhost/toMydb" -i "auth_user,news_news"

``-i`` параметр указывает какие таблицы нужно запускать первыми, например в такой ситуации:

.. code-block:: bash

   DETAIL:  Ключ (user_id)=(1) отсутствует в таблице "auth_user".
    'INSERT INTO django_admin_log (id, action_time, user_id, content_type_id, object_id, object_repr, action_flag,
    change_message) VALUES (%(id)s, %(action_time)s, %(user_id)s, %(content_type_id)s, %(object_id)s, %(object_repr)s,
    %(action_flag)s, %(change_message)s)' {'action_flag': 1, 'action_time': datetime.datetime(2014, 2, 5, 13, 15, 27, 948000),
    'user_id': 1, 'content_type_id': 39, 'object_repr': u'dfgsdfg', 'object_id': u'1', 'change_message': u'', 'id': 1}

Код полностью https://github.com/ITCase/convertdbdata 

Сейчас скрипт не рассчитан на большие объемы данных и учитывает только разницу
в ``DateTime`` поле, но думаю это легко исправить по мере поступления задач.
