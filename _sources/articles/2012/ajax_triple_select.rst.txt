.. post:: Feb 22, 2012 13:58
   :tags: javascript
   :category: JavaScript
   :author: Uralbash
   :language: ru

AJAX тройной выпадающий список (triple select)
==============================================

Опишу очень простую реализацию трех выпадающих списков, данные которых связанны
последовательно. Выглядит это примерно так ``"Город" -> "Улица" -> "Дом"``.
После выбора дома, на основе ``AJAX`` запроса, генерируется список улиц этого
города, а после выбора улицы генерируется список домов на этой улице. Создадим
html файл:

.. code-block:: html
   :caption: index.html

   <table>
      <tbody>
         <tr>
            <td>City:</td>
            <td>
               <select id="city" name="city" onchange="getStreet(this);">
                  <option selected="selected" value="None">-----------</option>
                  <option value="Екатеринбург">Екатеринбург</option>
                  <option value="Березовский">Березовскийг</option>
                  <option value="Артемоский">Артемовский</option>
               </select>
            </td>
         </tr>
         <tr><td>Street:</td><td><span id="streetAJAX"></span></td></tr>
         <tr><td>Build:</td><td><span id="buildAJAX"></span></td></tr>
      </tbody>
   </table>

При выборе города срабатывает функция ``getStreet``, которая генерирует код
списка улиц. Добавим ниже javascript:

.. code-block:: html

   <script language="JavaScript">
   //Gets the browser specific XmlHttpRequest Object
   function getXmlHttpRequestObject() {
       if (window.XMLHttpRequest) {
           return new XMLHttpRequest(); //Not IE
       } else if(window.ActiveXObject) {
           return new ActiveXObject("Microsoft.XMLHTTP"); //IE
       } else {
           //Display your error message here.
           //and inform the user they might want to upgrade
           //their browser.
           alert("Your browser doesn't support the XmlHttpRequest object.  Better upgrade to Firefox.");
       }
   }

   //Get our browser specific XmlHttpRequest object.
   var receiveReq = getXmlHttpRequestObject();
   //Initiate the asyncronous request.
   function getStreet(sel) {
       //If our XmlHttpRequest object is not in the middle of a request, start the new asyncronous call.
       document.getElementById('buildAJAX').innerHTML = ''
       var value = sel.options[sel.selectedIndex].value;
       if (receiveReq.readyState == 4 || receiveReq.readyState == 0) {
           //Setup the connection as a GET call to SayHello.html.
           //True explicity sets the request to asyncronous (default).
           receiveReq.open("GET", "/ajax/get_ajax_street?city="+value, true);
           //Set the function that will be called when the XmlHttpRequest objects state changes.
           receiveReq.onreadystatechange = handleStreet;
           //Make the actual request.
           receiveReq.send(null);
       }
   }

   //Called every time our XmlHttpRequest objects state changes.
   function handleStreet() {
       //Check to see if the XmlHttpRequests state is finished.
       if (receiveReq.readyState == 4) {
           //Set the contents of our span element to the result of the asyncronous call.
           //document.getElementById('span_result').innerHTML = receiveReq.responseText;
           //alert(receiveReq.responseText);
           document.getElementById('streetAJAX').innerHTML = receiveReq.responseText;
           getBuild(document.getElementById('street'));
       }
   }
   </script>

Теперь проделаем то же само для выбора улицы, добавим функции:

.. code-block:: javascript

   function getBuild(sel) {
       //If our XmlHttpRequest object is not in the middle of a request, start the new asyncronous call.
       var value = sel.options[sel.selectedIndex].value;
       if (receiveReq.readyState == 4 || receiveReq.readyState == 0) {
           //Setup the connection as a GET call to SayHello.html.
           //True explicity sets the request to asyncronous (default).
           receiveReq.open("GET", "/ajax/get_ajax_build?street="+value, true);
           //Set the function that will be called when the XmlHttpRequest objects state changes.
           receiveReq.onreadystatechange = handleBuild;
           //Make the actual request.
           receiveReq.send(null);
       }
   }

   //Called every time our XmlHttpRequest objects state changes.
   function handleBuild() {
       //Check to see if the XmlHttpRequests state is finished.
       if (receiveReq.readyState == 4) {
           //Set the contents of our span element to the result of the asyncronous call.
           //document.getElementById('span_result').innerHTML = receiveReq.responseText;
           //alert(receiveReq.responseText);
           document.getElementById('buildAJAX').innerHTML = receiveReq.responseText;
           summAddress();
       }
   }

Все должно работать :) Осталось написать функции на стороне сервера, для AJAX
запросов. Тут их две ``get_ajax_build`` и ``get_ajax_street``. Обе возвращают
обычный HTML список формата:

.. code-block:: xml

   <select id="street" name="street" onchange="getBuild(this);">
   <option value="1750">8 Марта</option>
   <option value="1718">Академика Королева</option>
   <option value="1734">Анучина</option>
   <option value="1733">БЗСК пос</option>
   <option value="1715">Больничный городок</option>
   <option value="1744">Брусницына</option>
   <option value="1704">Воротникова</option>
   <option value="1747">Восточная</option>
   <option value="1709">Гагарина</option>
   <option value="1701">Героев труда</option>
   <option value="1714">Горького</option>
   <option value="1721">Декабристов</option>
   <option value="1735">Загвозкина</option>
   <option value="1746">Заречная (Шиловка)</option>
   <option value="1738">Исакова</option>
   <option value="1731">Кирова</option>
   <option value="1707">Коммуны</option>
   <option value="1708">Комсомольская</option>
   <option value="1706">Косых</option>
   <option value="1737">Красноармейская</option>
   <option value="1702">Красных героев</option>
   <option value="1745">Ленина</option>
   <option value="1742">ЛЕНИНСКИИ (ШИЛОВКА)</option>
   <option value="1705">Ленинский (Шиловка)</option>
   <option value="1736">Мамина-Сибиряка</option>
   <option value="1740">Машинистов</option>
   <option value="1732">Маяковского</option>
   <option value="1741">Мебельщиков</option>
   <option value="1717">Мира</option>
   <option value="1725">Мичурина</option>
   <option value="1722">Новая (Шиловка)</option>
   <option value="1723">Овощное отделение</option>
   <option value="1728">Парковая (Шиловка)</option>
   <option value="1720">Первомайская</option>
   <option value="1739">Первомайский пос</option>
   <option value="1703">Пролетарская</option>
   <option value="1743">Смирнова</option>
   <option value="1710">Совхозная (Шиловка)</option>
   <option value="1713">Спортивная</option>
   <option value="1727">Старых Большевиков</option>
   <option value="1729">Строителей</option>
   <option value="1716">Театральная</option>
   <option value="1748">Толбухина</option>
   <option value="1730">Февральская</option>
   <option value="1749">Фурманова</option>
   <option value="1724">Циолковского</option>
   <option value="1711">Чапаева</option>
   <option value="1712">Чкалова</option>
   <option value="1726">Шиловская</option>
   <option value="1755">Энергостроителей</option>
   </select>

У меня на питоне они выглядят так:

.. code-block:: python

   def get_ajax_street(self):
           city = request.GET['city']
           streets = s.query(Streets).filter(Streets.city==city).\
                   order_by(Streets.name).all()
           html = ''
           for street in streets:
               html += "<option value="%s">" % str(street.id) +\
                       "%s</option>\n" % (street.name or 'No name')

           if html:
               html = "<select "="" "onchange="getBuild(this);" +\="" id="street" name="street">\n" + html + "</select>"

           return html

   def get_ajax_build(self):
       street = request.GET['street']
       builds = s.query(Builds).filter(Builds.streetid==street).\
               order_by(Builds.building).all()
       html = ''
       for build in builds:
           html += "<option value="%s">" % str(build.id) +\
                   "%s</option>\n" % (build.building or 'No name')

       if html:
           html = "<select "="" "onchange="summAddress();" +\="" id="build" name="build">\n" + html + "</select>"

       return html

Выглядит это как-то так:

.. image:: /_static/2012/triple_select.png
   :align: center

Думаю понятно, что функции должны быть доступны по адресам
``/ajax/get_ajax_street`` и ``/ajax/get_ajax_build`` соответственно.

Пользуйтесь ) 
