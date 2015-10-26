.. post:: Mar 27, 2012 23:38
   :tags: Python, Pylons, CSV
   :category: Pylons
   :author: Uralbash
   :language: ru

Pylons обработка csv с веб формы (метод POST)
=============================================

Небольшой "хинт" как при помощи :l:`Pylons` обрабатывать CSV файлы,
отправленные из формы.

Создаем шаблон:

.. code-block:: html+jinja

   <form action="" method="POST" enctype="multipart/form-data">
      CSV file: <input name="csvfile" type="file">
      <button>OK</button>
   </form>
   <table class="simpletable">
      {% for data in c.data %}
         <tr>
            <td>{{ data[0] }}</td>
            <td>{{ data[1] }}</td>
         </tr>
      {% endfor %}
   </table>

Метод контроллера выглядит как то так:

.. code-block:: python

   # coding=utf8
   import pylons
   import csv

   from cStringIO import StringIO
   from pylons import request, response, session, tmpl_context as c, url
   from myproject.lib.base import BaseController, render

   class CsvController(BaseController):

       def index(self):
           data = request.POST.get('csvfile', '')
           if data:
               data = data.value
           else:
               return render("/example/csv/index.html")

           f = StringIO(data)

           c.data = []
           for row in csv.reader(f):
               c.data.append((row[0], row[5]))

           return render("/example/csv/index.html")
