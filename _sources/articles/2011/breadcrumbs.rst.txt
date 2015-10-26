.. post:: Oct 26, 2011 13:59
   :tags: breadcrumbs, JavaScript
   :category: JavaScript
   :author: Uralbash
   :language: ru

Простые хлебные крошки для Pylons и не только...
================================================

В Интернете множество советов как делать хлебные крошки. Для :l:`Pylons` эти советы
довольно запутанные, сложные в реализации и рутины в поддержке на мой взгляд.
Поэтому я рассмотрю простой способ парсить URI. Нас будет интересовать только
URN (та часть которая идет после названия сайта).

Пример:

URI: ``http://example.ru/js/scripts/test``
URN: ``js/scripts/test``
breadcrumbs: ``js >> scripts >> test``

Есть хороший пример http://www.webreference.com/js/scripts/breadcrumbs/, но он
отображает в последней крошке(текущей) не распарсенный URN, а название сайта
document.title. Это не очень удобно в некоторых случаях.

Вот мой вариант этого решения:

.. code-block:: javascript
   :caption: добавляем public/js/breadcrumbs.js

   function breadcrumbs(){
     sURL = new String;
     bits = new Object;
     var x = 0;
     var stop = 0;
     var output = "<a href=\"/\">Home</a> &nbsp;&#187;&nbsp; ";
     var end = "";
     sURL = location.href;
     sURL = sURL.slice(8,sURL.length);
     chunkStart = sURL.indexOf("/");
     sURL = sURL.slice(chunkStart+1,sURL.length)
     while(!stop){
       chunkStart = sURL.indexOf("/");
       if (chunkStart != -1){
         bits[x] = sURL.slice(0,chunkStart)
         sURL = sURL.slice(chunkStart+1,sURL.length);
       }else{
         chunkStart = sURL.length;
         end = sURL.slice(0,chunkStart)
         stop = 1;
       }
       x++;
     }

     for(var i in bits){
       output += "<a href=\"";
       for(y=1;y<x-i;y++){
         output += "../";
       }
       output += bits[i] + "/\">" + bits[i] + "</a> &nbsp;&#187;&nbsp;";
     }

     document.write(output + end);
   }

В шаблон вставляем следующий код:

.. code-block:: html

   <script language="JavaScript">
      breadcrumbs();
   </script>

.. figure:: /_static/2011/breadcrumbs.png
   :align: center

   breadcrumbs + javascript
