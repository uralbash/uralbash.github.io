.. post:: Apr 17, 2012 15:15
   :tags: Python, SNMP, ненависть, негодование
   :category: Python
   :author: Uralbash
   :language: ru

Что юзать в Python'е для SNMP...
================================

Для `SNMP <https://en.wikipedia.org/wiki/Simple_Network_Management_Protocol>`_
в питоне есть несколько библиотек но у всех свои недостатки.

:pypi:`yapsnmp` - быстрая, простая библиотека, но не обновлялась с 2004 года и
имеет некоторые глюки. Для установки во всякие там Линуксы требуется бубен. Не
работают многие флаги, например PRINT_NUMERIC_OIDS
(http://sourceforge.net/tracker/?func=detail&aid=1119247&group_id=21077&atid=121077).

:pypi:`PySNMP` - хорошая документация, но работает ОООооочень медленно, причем
даже в тредах. Для большого объема данных не подходит.

`Net-SNMP <http://net-snmp.sourceforge.net/wiki/index.php/Python_Bindings>`_ -
обертка на питоне для netsnmp. Работает, синтаксис сложный.

Для просмотра таблицы коммутации используется команда типа:

.. code-block:: bash

   snmpwalk -v 2c -c mycomm 192.168.1.100 1.3.6.1.2.1.17.7.1.2.2.1.2 -O n

Флаг ``-0 n`` означает вывод в цифровом виде (вместо бинарного).

Если Вы хотите запрограммировать эту команду на питоне, то ``yapsnmp`` не
подходит из-за флагов, ``pysnmp`` неподойдет если вам нужно например пройти
пару сотен/тысяч свичей. ``Net-SNMP`` слишком много букв писать в коде для
такой простой команды.

Поэтому как решение можно делать так:

.. code-block:: python

   for ip in hosts:
       fdb = os.popen("snmpwalk -v 2c -c mycomm %s 1.3.6.1.2.1.17.7.1.2.2.1.2 -O n" % ip)
       try:
           tmp = map(int, e.replace("= INTEGER:", ".").replace("\n", "").split(".")[1:])
       except Exception, e:
           print e
           continue

       mac = "%02x:%02x:%02x:%02x:%02x:%02x" % tuple(i for i in tmp[-7:-1])
       port = tmp[-1]
       vlan = tmp[-8]

Вообщем я негодуэ!
