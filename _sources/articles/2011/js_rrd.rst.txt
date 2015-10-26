.. post:: 13 Nov, 2011 22:02
   :tags: rrdtool, javascript
   :category: JavaScript
   :author: Uralbash
   :language: ru

Красивые графики javascriptRRD + Float
======================================

В продолжение статьи :ref:`py_rrdtool_temp`: температура на улице и в
серверной рассмотрим как можно рисовать :l:`RRD` используя только
javascript на стороне клиента. Результат будет такой:

.. image:: /_static/2011/js_rrd.png
   :align: center

Используем библиотеку `javascriptrrd
<http://sourceforge.net/projects/javascriptrrd/>`_. Качаем и добавляем
следующие файлы из нее в наш шаблон в ``<head>``:

.. code-block:: html

   <script type="text/javascript" src="media/js/lib/binaryXHR.js"></script>
   <script type="text/javascript" src="media/js/lib/rrdFile.js"></script>

   <!-- rrdFlot class needs the following four include files !-->
   <script type="text/javascript" src="media/js/lib/rrdFlotSupport.js"></script>

   <script type="text/javascript" src="media/js/lib/rrdFlot.js"></script>
   <script type="text/javascript" src="media/flot/jquery.js"></script>
   <script type="text/javascript" src="media/flot/jquery.flot.js"></script>
   <script type="text/javascript" src="media/flot/jquery.flot.selection.js"></script>
   <script type="text/javascript" src="media/flot/jquery.flot.tooltip.js"></script>

И саму js функцию для отрисовки:

.. code-block:: html

   <h2>javascriptRRD пример</h2>
       <table id="infotable" border=1>
           <tr><td colspan="21"><b>Javascript needed for this page to work</b></td></tr>
    <tr><td><b>RRD file</b></td><td id="fname" colspan="5">None</td></tr>
       </table>

       <div id="mygraph"></div>

       <script type="text/javascript">

         // Remove the Javascript warning
         document.getElementById("infotable").deleteRow(0);

         // fname and rrd_data are the global variable used by all the functions below
         fname="media/temperature.rrd";
         rrd_data=undefined;

         // This function updates the Web Page with the data from the RRD archive header
         // when a new file is selected
         function update_fname() {
           // Finally, update the file name and enable the update button
           document.getElementById("fname").firstChild.data=fname;

           var graph_opts={legend: { noColumns:4}};
           var ds_graph_opts={'Oscilator':{ color: "#ff8000",
                                            lines: { show: true, fill: true, fillColor:"#ffff80"} },
                              'Idle':{ label: 'IdleJobs', color: "#00c0c0",
                                       lines: { show: true, fill: true} },
                              'Running':{color: "#000000",yaxis:2}};



           // the rrdFlot object creates and handles the graph
           var f=new rrdFlot("mygraph",rrd_data,graph_opts,ds_graph_opts);
         }

         // This is the callback function that,
         // given a binary file object,
         // verifies that it is a valid RRD archive
         // and performs the update of the Web page
         function update_fname_handler(bf) {
             var i_rrd_data=undefined;
             try {
               var i_rrd_data=new RRDFile(bf);
             } catch(err) {
               alert("File "+fname+" is not a valid RRD archive!");
             }
             if (i_rrd_data!=undefined) {
               rrd_data=i_rrd_data;
               update_fname()
             }
         }

         // this function is invoked when the RRD file name changes
         function fname_update() {
           fname="media/temperature.rrd";
           try {
             FetchBinaryURLAsync(fname,update_fname_handler);
           } catch (err) {
              alert("Failed loading "+fname+"\n"+err);
           }
         }

       </script>

Для автоматического запуска функции при загрузки страницы добавим событие
``onLoad`` в ``body``:

.. code-block:: html

   <body bgcolor="white" text="black" onLoad="fname_update()">

Всё, радуемся результату.
