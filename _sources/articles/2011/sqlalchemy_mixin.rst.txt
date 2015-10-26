.. post:: Oct 27, 2011 00:07
   :tags: SQLalchemy, Mixin, Django
   :category: SQLalchemy
   :author: Uralbash
   :language: ru

Pylons + SQLalchemy расширенная модель (Mixin)
==============================================

Часто при проектировании структуры БД появляется необходимость повторять одни
и те же действия с таблицами. Добавлять одинаковые поля, ссылки, счетчики и
т.д. Модели в :l:`Pylons` (:l:`SQLAlchemy`), как и в большинстве других фреймворках
использующих паттерн MVC, являются классом и соответственно могут быть
унаследованы от других классов. Это позволяет нам избежать рутинной работы с
повторяющимися действиями.

Все расширения для наших моделей будем добавлять в ``models/common.py``.
Создадим базовую модель в которой будет поле ``id``, автоматическая генерация
названия таблицы (``__tablename__``) и метод выбора элемента по ``id`` (``SELECT * FROM
table WHERE id=integer``):

.. code-block:: python

   class Base(object):
       """Базовая модель. Добавляет во всех наследников поле id и атрибут
       __tablename__ который заполняется автоматически. Имя таблицы берется из
       названия класса и переводится в нижний регистр. Таблица наследник имеет по
       умолчанию название и поле id, сильно облегчая жизнь.
       """

       @declared_attr
       def __tablename__(cls):
           if (has_inherited_table(cls) and
               Tablename not in cls.__bases__):
               return None
           return cls.__name__.lower()

       # Method "byId" for use in code like this:
       #   session.query(Table).byId(5)
       #
       # SQL statement like:
       #   SELECT * FROM Table WHERE id = 5;
       @classmethod
       def byId(cls, id) :
           return Session.query(cls).filter_by(id = id).first()

       id =  Column(Integer, autoincrement=True, primary_key=True)

Метод ``byId`` сильно сокращает запись в коде например:

| before: ``Session.query(net).filter_by(id = id).first()``
| after: ``net.byId(id)``

Теперь создадим нашу модель унаследовав все плюшки с базовой модели:

.. code-block:: python

   class Net(Base, DeclarativeBase):
       """Net or subnet."""

       cidr = Column(postgresql.CIDR, index = True)
       description = Column(UnicodeText())

       def __init__(self, cidr=''):
           self.cidr = cidr

       def __repr__(self):
           return "%s" % self.cidr

Наша модель связанна с БД при помощи наследования от ``DeclarativeBase``, имеет
название, поле ``id`` и метод ``byId`` благодаря наследованию от базовой модели
``Base`` из файла ``common.py``.

Для более наглядного примера создадим типовую модель для таблиц которые должны
содержать служебную информацию. Модель будет добавлять в другие модели поля:

| ``created_by`` - кто создал
| ``updated_by`` - последний кто обновил
| ``created_at`` - дата создания
| ``updated_at`` - дата последнего обновления

Поля заполняются автоматически. Кто создал и обновил ссылаются на модель
``auth.User``. Пользователь берется из текущей сессии, при помощи библиотеки
``lib.auth`` и метода ``get_user``. Откуда взялась модель ``User`` можно узнать из этой
статьи :ref:`repoze-what`.

.. code-block:: python

   class CreatedMixin(object):
       """Абстрактная примесь которая добавляет в другие модели поля:
           created_by - кто создал
           updated_by - последний кто обновил
           created_at - дата создания
           updated_at - дата последнего обновления
       Поля заполняются автоматически. Кто создал и обновил ссылаются на модель
       auth.User. Пользователь берется из текущей сессии, при помощи библиотеки
       lib.auth и метода get_user
       """

       @declared_attr
       def created_by(cls):
           return Column(Integer, ForeignKey('user.user_id',
                         onupdate="cascade", ondelete="restrict"))

       @declared_attr
       def updated_by(cls):
           return Column(Integer, ForeignKey('user.user_id',
                         onupdate="cascade", ondelete="restrict"))

       created_at = Column(DateTime, nullable=False, default=dt.now())
       updated_at = Column(DateTime, nullable=False, default=dt.now(),
                           onupdate=dt.now())

Внешние ссылки и другие атрибуты отличающиеся от обычных полей нужно добавлять
при помощи декоратора ``declare_attr``. Теперь меняем нашу модель ``Net`` просто
добавив ``CreateMixin``:

.. code-block:: python

   class Net(Base, DeclarativeBase, CreatedMixin):

Вот полный листинг ``common.py``:

.. code-block:: python

   # coding=utf-8
   """Модуль с типовыми моделями
   """

   from sqlalchemy import Column, ForeignKey
   from sqlalchemy.orm import relation, relationship
   from sqlalchemy.types import Integer, String, DateTime
   from sqlalchemy.ext.declarative import declared_attr
   from sqlalchemy.ext.declarative import has_inherited_table

   from gottlieb.model.auth import User
   from gottlieb.lib import auth

   from datetime import datetime as dt

   class Base(object):
       """Базовая модель. Добавляет во всех наследников поле id и атрибут
       __tablename__ который заполняется автоматически. Имя таблицы берется из
       названия класса и переводится в нижний регистр. Таблица наследник имеет по
       умолчанию название и поле id, сильно облегчая жизнь.
       """

       @declared_attr
       def __tablename__(cls):
           if (has_inherited_table(cls) and
               Tablename not in cls.__bases__):
               return None
           return cls.__name__.lower()

       # Method "byId" for use in code like this:
       #   session.query(Table).byId(5)
       #
       # SQL statement like:
       #   SELECT * FROM Table WHERE id = 5;
       @classmethod
       def byId(cls, id) :
           return Session.query(cls).filter_by(id = id).first()

       id =  Column(Integer, autoincrement=True, primary_key=True)

   class CreatedMixin(object):
       """Абстрактная примесь которая добавляет в другие модели поля:
           created_by - кто создал
           updated_by - последний кто обновил
           created_at - дата создания
           updated_at - дата последнего обновления
       Поля заполняются автоматически. Кто создал и обновил ссылаются на модель
       auth.User. Пользователь берется из текущей сессии, при помощи библиотеки
       lib.auth и метода get_user
       """

       @declared_attr
       def created_by(cls):
           return Column(Integer, ForeignKey('user.user_id',
                         onupdate="cascade", ondelete="restrict"))

       @declared_attr
       def updated_by(cls):
           return Column(Integer, ForeignKey('user.user_id',
                         onupdate="cascade", ondelete="restrict"))

       created_at = Column(DateTime, nullable=False, default=dt.now())
       updated_at = Column(DateTime, nullable=False, default=dt.now(),
                           onupdate=dt.now())

Для полной картины приведу аналогичный пример на :l:`Django` + DjangoORM.

.. code-block:: python
   :caption: myapp/accompaniment/models.py

   from django.db import models
   from django.contrib.auth.models import User

   # Create your models here.
   class ExtendedModel(models.Model):
       created_by = models.ForeignKey(User, null=True, blank=True,
                    editable=False, related_name='%(class)s_creator')
       created_time = models.DateTimeField(auto_now_add=True, editable=False)
       modified_by = models.ForeignKey(User, null=True, blank=True,
                    editable=False, related_name='%(class)s_modifier')
       modified_time = models.DateTimeField(auto_now=True, editable=False)

       class Meta:
           abstract = True

В папке accompaniment я привык держать всякие такие хелперы для проекта. Теперь
используем эту модель в нашем проекте ``myapp/projectname/models.py``:

.. code-block:: python

   from django.db import models
   from accompaniment.models import ExtendedModel

   class Ticket(ExtendedModel):

       OPEN_STATUS = 1
       REOPENED_STATUS = 2
       RESOLVED_STATUS = 3
       CLOSED_STATUS = 4
       DUPLICATE_STATUS = 5

       STATUS_CHOICES = (
           (OPEN_STATUS, _('Open')),
           (REOPENED_STATUS, _('Reopened')),
           (RESOLVED_STATUS, _('Resolved')),
           (CLOSED_STATUS, _('Closed')),
           (DUPLICATE_STATUS, _('Duplicate')),
       )

       PRIORITY_CHOICES = (
           (1, _('1. Critical')),
           (2, _('2. High')),
           (3, _('3. Normal')),
           (4, _('4. Low')),
           (5, _('5. Very Low')),
       )

       title = models.CharField(
           _('Title'),
           max_length=200,
           )

       queue = models.ForeignKey(
           Queue,
           verbose_name=_('Queue'),
           )

       assigned_to = models.ForeignKey(
           User,
           related_name='assigned_to',
           blank=True,
           null=True,
           verbose_name=_('Assigned to'),
           )

       status = models.IntegerField(
           _('Status'),
           choices=STATUS_CHOICES,
           default=OPEN_STATUS,
           )

       description = models.TextField(
           _('Description'),
           blank=True,
           null=True,
           help_text=_('The content of the customers query.'),
           )

       priority = models.IntegerField(
           _('Priority'),
           choices=PRIORITY_CHOICES,
           default=3,
           blank=3,
           )

       class Meta:
           get_latest_by = "created"
           verbose_name = u'Заявки'
           verbose_name_plural = u'Заявки'

       def __unicode__(self):
           return u'%s' % self.title

       def save(self, force_insert=False, force_update=False):
           if not self.priority:
               self.priority = 3

           super(Ticket, self).save(force_insert, force_update)

Такой несложный метод освобождает нас от размножения кучи одинаковых полей в
моделях.

Update: в моделях Mixin был атрибут ``__abstract__ = True`` это неправильно,
так-как все таблицы стают абстрактными. НО! Это вполне прокатит на версиях
меньше 0.7, там этот атрибут почему-то не учитывается. На 0.7 версии работает
как надо. Вот описание проблемы: `stackoverflow
<http://stackoverflow.com/questions/7990790/sqlalchemy-0-7-mapperevent-error>`_
