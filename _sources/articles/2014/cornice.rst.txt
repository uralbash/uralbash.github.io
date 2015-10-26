.. post:: 05 Sep, 2014 16:33
   :tags: Python, Pyramid, REST, sacrud
   :category: Pyramid
   :author: Uralbash
   :language: ru

REST API для Pyramid при помощи Cornice и SACRUD
================================================

:github:`Mozilla` использует в своих проектах :l:`Pyramid` и у них есть
отличный модуль для создания ``REST API``
https://cornice.readthedocs.org/en/latest/

``REST API`` обычно меняет, создает и удаляет записи, которые хранятся в БД.
Что бы не писать много кода на :l:`SQLAlchemy` я использую заготовленные
функции из :pypi:`sacrud`.

Итак поехали, представим сервис ``REST API`` для платежных карт с моделью типа:

.. code-block:: python

   class Card(Base):
       __tablename__ = 'card'

       id = Column(Integer, primary_key=True)
       number = Column(BigInteger, nullable=False, unique=True)
       uid = Column(BYTEA, nullable=False, unique=True)
       balance = Column(Numeric(10, 2), nullable=False, default=0)
       preference = Column(GUID())

       def __json__(self):
           return {'id': self.id,
                   'number': self.number,
                   'uid': str(self.uid),
                   'balance': str(self.balance)
                   }

Создадим отдельную папку в :l:`pyramid` проекте с названием ``rest``, где будут
храниться функции api, и пропишем это в основном конфиге.

.. code-block:: python

   # REST API
   config.include("cornice")
   config.include("myapp.rest", route_prefix="/api")

В pyramid'е есть такая штука как ``config.scan()`` она смотрит все файлы в
проекте с расширением ``*.py`` (от текущей директории) и ищет вьюхи обернутые
декоратором ``@view_config``. Я предпочитаю включать в основной конфиг папочки
через инклуд как в примере выше, а локально в каждой папке уже вызывать
``config.scan()``.

Структура папки ``rest``:

.. code-block:: bash

   rest/
   ├── card.py
   ├── __init__.py
   └── validators.py

Файл ``__init__.py``:

.. code-block:: python
   :caption: __init__.py

   def includeme(config):
       config.scan()

В ``card.py`` сама реализация ``REST API``:

GET
---

.. code-block:: python

   import json

   from cornice import Service

   from myapp.models import DBSession
   from myapp.models import Card
   from myapp.rest.validators import _400, _404, valid_hex

   from sacrud.action import CRUD
   from sqlalchemy.exc import DataError
   from sqlalchemy.orm.exc import NoResultFound

   card_api = Service(name='card', path='/card/{UID}',
                      description="REST API for card")


   @card_api.get(validators=valid_hex)
   def get_card(request):
       """``GET``: возвращает данные о карте.

       .. code-block:: bash

           $ curl -D - http://0.0.0.0:6543/api/card/07a8e29d

           HTTP/1.1 200 OK
           Content-Length: 69
           Content-Type: application/json; charset=UTF-8
           Date: Sat, 02 Aug 2014 11:22:52 GMT
           Server: waitress

           {"uid": "07a8e29d", "balance": 507.05, "id": 3, "number": 8224548674}
       """
       key = request.validated['UID']
       card = DBSession.query(Card)
       if key:
           try:
               card = card.filter_by(uid=key).one()
           except NoResultFound:
               raise _404()
           except DataError:
               raise _400()
           return card.__json__()

Декоратор ``card_api`` превращает функцию ``get_card`` во вьюху, ожидающею HTTP
метод GET, и ``config.scan()`` её автоматически подхватит. Внутри можно
реализовывать все что угодно. Про метод validated ниже.

POST
----

.. code-block:: python

   @card_api.post(validators=valid_hex)
   def set_card(request):
       """``POST``: передает параметры для редактирования карты.

       .. code-block:: bash

           $ curl -H 'Accept: application/json'\\
               -H 'Content-Type: application/json'\\
               http://0.0.0.0:6543/api/card/07a8e29d\\
               -d '{"number": "8224548674", "balance": "507.05"}'

           {"uid": "07a8e29d", "balance": 507.05, "id": 3, "number": 8224548674}

       Если карты нету, то создается новая

       .. code-block:: bash

           $ curl -H 'Accept: application/json'\\
               -H 'Content-Type: application/json'\\
               http://0.0.0.0:6543/api/card/550e8400\\
               -d '{"balance": "100.11", "preference": "956bfc40"}'

           {"uid": "550e8400", "balance": 100.11, "id": 7, "number": 91328937986}
       """
       key = request.validated['UID']
       data = {'request': json.loads(request.body)}

       card = DBSession.query(Card).filter_by(uid=key).first()
       if card:
           data['pk'] = {'id': card.id}
       else:
           data['request']['uid'] = key

       try:
           CRUD(DBSession, Card, **data).add()
       except Exception as e:
           raise _400(msg=str(e.message))
       card = DBSession.query(Card).filter_by(uid=key).one()
       return card.__json__()

Метод POST меняет параметры карты или создает новую если такой нету. Карта
создается при помощи мега модуля :pypi:`sacrud`. Подробнее о создании записей
через :pypi:`sacrud` здесь
http://sacrud.readthedocs.org/en/latest/plain_usage.html#create-action

DELETE
------

.. code-block:: python

   @card_api.delete(validators=valid_hex)
   def del_card(request):
       """``DELETE``: удаляет карту по UID

       .. code-block:: bash

           $ curl -H 'Accept: application/json'\\
               -H 'Content-Type: application/json'\\
               http://0.0.0.0:6543/api/card/550e8400 -X DELETE

           {"Goodbye": "550e8400"}
       """
       key = request.validated['UID']
       card = DBSession.query(Card).filter_by(uid=key).first()
       if not card:
           raise _404()
       try:
           CRUD(DBSession, Card, pk={'id': card.id}).delete()
       except DataError as e:
           raise _400(msg=str(e.message))
       return {'Goodbye': key}

Удаление происходит при вызове HTTP метода DELETE. :pypi:`sacrud` опять же
несколько упрощает эту операцию
http://sacrud.readthedocs.org/en/latest/plain_usage.html#delete-action

В файле ``validators.py`` хранятся всякие исключения и сами проверки. Если в
декораторе card_api указан валидатор, то валидные данные нужно будет выбирать
не через ``request.matchdict`` а через ``request.validated``.

Пример файла ``validators.py``

.. code-block:: python

   import json
   import string

   from webob import exc, Response


   class _404(exc.HTTPError):
       def __init__(self, msg='Not Found'):
           body = {'status': 404, 'message': msg}
           Response.__init__(self, json.dumps(body))
           self.status = 404
           self.content_type = 'application/json'


   class _400(exc.HTTPError):
       def __init__(self, msg='Bad Request'):
           body = {'status': 400, 'message': msg}
           Response.__init__(self, json.dumps(body))
           self.status = 400
           self.content_type = 'application/json'


   def valid_hex(request):
       key = request.matchdict['UID']

       if not all(c in string.hexdigits for c in key):
           raise _400(msg="Not valid UID '%s'" % key)

       request.validated['UID'] = str(key)

Таким образом можно довольно просто создать API для вашего проекта на пирамиде.
Cornice делает много за вас, создает вьюхи, пути, валидацию, может генерить
автоматически :pypi:`Sphinx` документацию, а :pypi:`SACRUD` упрощает работу с БД.
