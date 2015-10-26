.. post:: Feb 10, 2012 11:47
   :tags: openstreetmap, osm, javascript, geotagging
   :category: JavaScript
   :author: Uralbash
   :language: ru

.. _osm_geotagging:

OpenStreetMap, Геокодирование и автодополнение адреса в строке поиска (как у гугла) с помощью Яндекс API :)
===========================================================================================================

`Геокоди́рование` — процесс назначения географических идентификаторов (таких как
`географические координаты
<http://ru.wikipedia.org/wiki/%D0%93%D0%B5%D0%BE%D0%B3%D1%80%D0%B0%D1%84%D0%B8%D1%87%D0%B5%D1%81%D0%BA%D0%B8%D0%B5_%D0%BA%D0%BE%D0%BE%D1%80%D0%B4%D0%B8%D0%BD%D0%B0%D1%82%D1%8B>`_,
выраженные в виде `широты
<http://ru.wikipedia.org/wiki/%D0%A8%D0%B8%D1%80%D0%BE%D1%82%D0%B0>`_ и
`долготы
<http://ru.wikipedia.org/wiki/%D0%94%D0%BE%D0%BB%D0%B3%D0%BE%D1%82%D0%B0>`_)
объектам карты и записям данных.

Например, геокодированием является назначение координат записям, описывающим
адрес (улица/дом) или фотографиям (где было сделано фото) или IP-адресам, или
любой другой информации, имеющей географический компонент. Геокодированные
объекты могут быть использованы в `геоинформационных системах
<http://ru.wikipedia.org/wiki/%D0%93%D0%B5%D0%BE%D0%B8%D0%BD%D1%84%D0%BE%D1%80%D0%BC%D0%B0%D1%86%D0%B8%D0%BE%D0%BD%D0%BD%D0%B0%D1%8F_%D1%81%D0%B8%D1%81%D1%82%D0%B5%D0%BC%D0%B0>`_.

Создадим карту и строку поиска объекта по адресу, через API Яндекс Карты.
Служба Яндекс.Карт предлагает своим пользователям сервис геокодирования. Он
позволяет определять координаты и получать сведения о географическом объекте
по его названию или адресу и наоборот, определять адрес объекта на карте по
его координатам (обратное геокодирование).

Например, по запросу «Москва, ул. Малая Грузинская, д. 27/13» геокодер
возвратит географические координаты этого дома: "37.571309, 55.767190"
(долгота, широта). И, наоборот, если в запросе указать географические
коордианты дома "37.571309, 55.767190", то геокодер вернет его адрес.
К геокодеру можно обращаться как по HTTP-протоколу, так и с помощью `JavaScript
API <http://api.yandex.ru/maps/jsapi>`_. При обращении к геокодеру по
HTTP-протоколу ответ может быть сформирован либо в виде XML-документа формата
`YMapsML <http://api.yandex.ru/maps/ymapsml>`_, либо в формате `JSON <http://ru.wikipedia.org/wiki/JSON>`_.

При обращении к геокодеру по HTTP-протоколу в параметрах запроса требуется
указывать API-ключ. Ключ можно получить, заполнив соответствующую `форму
<http://api.yandex.ru/maps/form.xml>`_.
В данном документе описаны `параметры HTTP-запроса
<http://api.yandex.ru/maps/geocoder/doc/desc/concepts/input_params.xml>`_ к
геокодеру, его `ответ
<http://api.yandex.ru/maps/geocoder/doc/desc/concepts/response_structure.xml>`_,
а также приведены примеры использования.

Как добавить карту на сайт можно посмотреть здесь :ref:`osm_example`.

Я приведу пример использования в окружении :l:`Pylons` и шаблонизатора
:l:`Jinja`. При желании переделать скрипты под другие фреймворки не составит
труда. Главное понять принципы.

Добавим поле для поиска

.. code-block:: html

   <input id="searchAddress" name="address" size="73" type="text" />

Напишем AJAX функцию для выпадающего списка при автодополнении
(http://www.devbridge.com/projects/autocomplete/jquery/):

.. code-block:: html+jinja

   <script src="/jquery/jquery.min.js" type="text/javascript"></script>
   <script src="/jquery/jquery.autocomplete.js" type="text/javascript"></script>
   <script>
       // Автокомплит для адреса
       //var options, a;
        jQuery(function(){
         options = {
             serviceUrl:"{{ url(controller='common', action='geocodding_ajax') }}",
           width:900,
           deferRequestBy: 100, //miliseconds
         };
         a = $('#searchAddress').autocomplete(options);
       });
   </script>

Создадим контроллер который будет отдавать данные по AJAX запросу:

.. code-block:: python
   :caption: common.py

   # coding=utf8
   import logging
   import pylons

   from pyandexmap import geotagging
   from pylons import request, response, session, tmpl_context as c, url
   from pylons.controllers.util import abort, redirect

   from myapp.lib.base import BaseController, render

   log = logging.getLogger(__name__)

   class CommonController(BaseController):

       def geocodding_ajax(self):
           """ for autocomplete search in Yandex API
           """
           # Ключ для Яндекс карт из development.ini
           key = pylons.config['yandexKey']
           address = ''
           for addr in geotagging.listGeoObject(request.GET['query'], key):
               if address:
                   address = address +", '"+addr+"'"
               else:
                   address = "['"+addr+"'"

           resp = '''{
            query:'%s',
            suggestions: %s,
            data:[]
           }''' % (request.GET['query'], address+']')
           return resp

Формат ответа выглядит так:

.. code-block:: javascript

   {
    query:'Li',
    suggestions:['Liberia','Libyan Arab Jamahiriya','Liechtenstein','Lithuania'],
    data:['LR','LY','LI','LT']
   }

:pypi:`pyandexmap` устанавливаем из :l:`pypi` (про этот модуль я напишу позже).

.. code-block:: bash

   $ pip install pyandexmap

Все должно заработать, остались только стили;

.. code-block:: html

   <style type="text/css">
   .autocomplete-w1 { background:url(img/shadow.png) no-repeat bottom right; position:absolute; top:0px; left:0px; margin:6px 0 0 6px; /* IE6 fix: */ _background:none; _margin:1px 0 0 0; }
   .autocomplete { border:1px solid #999; background:#FFF; cursor:default; text-align:left; max-height:350px; overflow:auto; margin:-6px 6px 6px -6px; /* IE6 specific: */ _height:350px;  _margin:0; _overflow-x:hidden; }
   .autocomplete .selected { background:#F0F0F0; }
   .autocomplete div { padding:2px 5px; white-space:nowrap; overflow:hidden; }
   .autocomplete strong { font-weight:normal; color:#3399FF; }
   </style>

Результат:

.. figure:: /_static/2012/osm_geotagging.png
   :align: center

   Яндекс Карты автодополнение через геокодинг
