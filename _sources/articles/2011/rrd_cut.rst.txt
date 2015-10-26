.. post:: 09 Dec, 2011 10:47
   :tags: RRDTool, python
   :category: RRDTool
   :author: Uralbash
   :language: ru

.. _rrd_cut:

Срезаем пики в RRD при помощи python
====================================

В некоторых случаях на :l:`rrd` графиках появляются пики, как например после
перезагрузки сетевой карты:

.. figure:: /_static/2011/rrd_pick.png
   :align: center

   пики в rrd

Для их удаления нам нужно воспользоваться утилитой ``dump`` из :l:`rrdtool`,
которая переводит файл :l:`rrd` в формат xml. Затем обнулить значение пиков и
записать изменения обратно в :l:`rrd` файл при помощи утилиты restore. На оф.
сайте :l:`rrdtool` можно найти скрипт на :l:`perl` ``removespikes`` и на
:l:`php` для ``cacti`` ``spikekill``.  ``removespike`` работает не всегда
корректно, поэтому рассмотрим как можно обрезать графики при помощи скрипта на
:l:`python`:

.. code-block:: python

   #!/usr/bin/env python
   import os
   import shutil
   from xml.etree import ElementTree as ET
   from optparse import OptionParser

   __version__ = '0.1'

   def get_configuration():
       parser = OptionParser(version="%prog v" + __version__,
                             usage="%prog <filename>")
       parser.add_option("-v", "--verbose", action="count", default=0,
                         help="produce more output")
       parser.add_option('-b', '--border', action ='store', default=10**10,
                    help='Min value of peak. Default 100000')
       options, args = parser.parse_args()

       if not os.path.isfile(args[0]):
           parser.error("Not found file")

       return [ options ] + args

   def main():
       options, filename = get_configuration()
       try:
           rrd_dump = os.system("rrdtool dump '%s' > 'old.xml'" % (filename))
       except:
           print "Error RRD file type"
           raise

       if not os.path.exists('rrd_bak'):
           os.makedirs('rrd_bak')
       shutil.copy2(filename, 'rrd_bak')

       tree = ET.parse('old.xml')

       def iterparent(tree):
           for parent in tree.getiterator():
               for child in parent:
                   yield parent, child

       for parent, child in iterparent(tree):
           if child.tag == 'v':
               if float(child.text) > options.border:
                   print 'peak detect in %s' % filename
                   child.text = 'NaN'

       tree.write('new.xml')
       os.unlink(filename)
       os.system("rrdtool restore 'new.xml' '%s'" % filename)
       os.unlink('new.xml')
       os.unlink('old.xml')

   if __name__ == '__main__':
       import sys
       sys.exit(main())

Для того что бы парсить параметры командной строки воспользуемся модулем
:mod:`optparse`.

.. code-block:: python

   def get_configuration():
       parser = OptionParser(version="%prog v" + __version__,
                             usage="%prog <filename>")
       parser.add_option("-v", "--verbose", action="count", default=0,
                         help="produce more output")
       parser.add_option('-b', '--border', action ='store', default=10**10,
                    help='Min value of peak. Default 100000')
       options, args = parser.parse_args()

       if not os.path.isfile(args[0]):
           parser.error("Not found file")

       return [ options ] + args

Дампим :l:`rrd` файл в xml:

.. code-block:: python

   try:
      rrd_dump = os.system("rrdtool dump '%s' > 'old.xml'" % (filename))
   except:
      print "Error RRD file type"
      raise

На всякий случай копируем старый :l:`rrd` в папку ``rrd_bak``:

.. code-block:: python

   if not os.path.exists('rrd_bak'):
           os.makedirs('rrd_bak')
       shutil.copy2(filename, 'rrd_bak')

Функция для получения списка всех элементов из xml:

.. code-block:: python

   def iterparent(tree):
           for parent in tree.getiterator():
               for child in parent:
                   yield parent, child

Записи в xml выглядят так:

.. code-block:: xml

   <database>
      <row><v>1.2531404050e+04</v><v>1.1760614140e+03</v><v>1.0547667362e+01</v><v>0.0000000000e+00</v><v>0.0000000000e+00</v><v>0.0000000000e+00</v></row>
      <row><v>6.2535788107e+03</v><v>6.5077305135e+02</v><v>5.9844172295e+00</v><v>0.0000000000e+00</v><v>0.0000000000e+00</v><v>0.0000000000e+00</v></row>
      <row><v>7.4349123852e+03</v><v>7.2086196285e+02</v><v>6.7862914917e+00</v><v>0.0000000000e+00</v><v>0.0000000000e+00</v><v>0.0000000000e+00</v></row>
      <row><v>6.6044470194e+03</v><v>6.1316221529e+02</v><v>6.2081965076e+00</v><v>0.0000000000e+00</v><v>0.0000000000e+00</v><v>0.0000000000e+00</v></row>
      <row><v>1.6377377735e+04</v><v>2.8213539887e+03</v><v>1.4662221016e+01</v><v>0.0000000000e+00</v><v>0.0000000000e+00</v><v>0.0000000000e+00</v></row>
      <row><v>1.4991881969e+04</v><v>1.9741971231e+03</v><v>1.3208469357e+01</v><v>0.0000000000e+00</v><v>0.0000000000e+00</v><v>0.0000000000e+00</v></row>
   ...
   </database>

Пробегаем по всем тегам и ищем значения которые больше порогового 10 в степени
10 и записываем нулевое значение (``NaN``):

.. code-block:: python

   for parent, child in iterparent(tree):
           if child.tag == 'v':
               if float(child.text) > options.border:
                   print 'peak detect in %s' % filename
                   child.text = 'NaN'

Сохраняем xml и импортируем изменения в :l:`rrd`:

.. code-block:: python

   tree.write('new.xml')
   os.system("rrdtool restore 'new.xml' '%s'" % filename)

Полученный результат:

.. figure:: /_static/2011/rrd_cut_pick.png
   :align: center

   rrd график без пиков

Скрипт на ``bash`` для удаления пиков во всей директории:

.. code-block:: bash

   #!/bin/bash
   if [ -a $1 ]
   then
       for file in $1*
       do
           echo $file
           python killerpeak.py $file
       done
   fi
   echo "Don't forget delete directory rrd_bak if all is ok"
   exit 0

Запуск скрипта ``killerpeak.py``:

.. code-block:: bash

   $ sudo ./killerpeak.py data/rrdfile/7890.rrd
   peak detect in data/rrdfile/7890.rrd
   peak detect in data/rrdfile/7890.rrd
   peak detect in data/rrdfile/7890.rrd
   peak detect in data/rrdfile/7890.rrd

Запуск скрипта на ``bash`` ``killpeak.sh``:

.. code-block:: bash

   $ ./killpeak.sh data/rrdfile

Взять код или поправить можно на :l:`github` https://github.com/uralbash/rrd_killerpeak
