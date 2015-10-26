.. post:: Feb 10, 2012 10:33
   :tags: openstreetmap, osm, javascript
   :category: JavaScript
   :author: Uralbash
   :language: ru

.. _osm_example:

Пример карты OpenStreetMap на своем сайте
=========================================

:l:`OpenStreetMap` — это свободно редактируемая карта всего мира. Она сделана
такими же людьми, как и вы. :l:`OpenStreetMap` позволяет совместно
просматривать, изменять и использовать географические данные в любой точке
Земли. Также позволяет накладывать разные слои в том числе и карты Яндекса с
Гуглом. У гугла проблема с детализацией карт в России у Яндекса с API поэтому
:l:`OpenStreetMap` смотрится наиболее красивым решением, при том что софт и
сами карты распространяются по свободным лицензиям.

Добавим js:

.. code-block:: html

   <script src="http://www.openlayers.org/api/OpenLayers.js">
   </script>
   <script src="http://www.openlayers.org/api/OpenLayers.js">
   </script>
   <script type="text/javascript">
   function GetMap() {
       map = new OpenLayers.Map("OSMap");//инициализация карты
       var mapnik = new OpenLayers.Layer.OSM();//создание слоя карты
       map.addLayer(mapnik);//добавление слоя
       map.zoomToMaxExtent();
       // Широта/долгота
       var lonlat = new OpenLayers.LonLat(66.666, 77.777);
       map.setCenter(lonlat.transform(
               new OpenLayers.Projection("EPSG:4326"), // переобразование в WGS 1984
               new OpenLayers.Projection("EPSG:900913") // переобразование проекции
             ), 17 // масштаб 17 крут
           );

       //создаем новый слой оборудования
       var layerMarkers = new OpenLayers.Layer.Markers("Equipments");
       map.addLayer(layerMarkers);//добавляем этот слой к карте

       // Маркер текущего еквипмента
       var size = new OpenLayers.Size(21, 25);//размер картинки для маркера
       //смещение картинки для маркера
       var offset = new OpenLayers.Pixel(-(size.w / 2), -size.h);
       //картинка для маркера
       var icon = new OpenLayers.Icon('/img/common/switch_svg.gif', size, offset);
       layerMarkers.addMarker(//добавляем маркер к слою маркеров
           new OpenLayers.Marker(lonlat, //координаты вставки маркера
           icon));//иконка маркера

       // шкала для выбора заранее настроенного масштаба
       //map.addControl(new OpenLayers.Control.PanZoomBar());

       // панель инструментов (сдвиг и масштабирование)
       //map.addControl(new OpenLayers.Control.MouseToolbar());

       // переключатель видимости слоев
       map.addControl(new OpenLayers.Control.LayerSwitcher({'ascending':false}));

       // ссылка внизу карты на текущее положение/масштаб
       //map.addControl(new OpenLayers.Control.Permalink());
       //map.addControl(new OpenLayers.Control.Permalink('permalink'));

       // координаты текущего положения мыши
       // преобразование из метров в градусы с помощью proj4js
       map.addControl(
           new OpenLayers.Control.MousePosition({
               displayProjection: new OpenLayers.Projection('EPSG:4326')
           })
       );

       // обзорная карта
       //map.addControl(new OpenLayers.Control.OverviewMap());

       // горячие клавиши
       //map.addControl(new OpenLayers.Control.KeyboardDefaults());
   }
   </script>

На карте добавлен слой ``"Equipments"`` и маркер с картинкой ``switch_svg.gif``.

На странице вставляем тег карты:

.. code-block:: html

   <div id="OSMap"></div>

И в тег ``<body>`` добавляем ``onload="GetMap();"``.

Результат:

.. figure:: /_static/2012/osm.png
   :align: center

   OpenStreeMap пример

`Хорошая статья на Хабре <http://habrahabr.ru/post/145117/>`_
