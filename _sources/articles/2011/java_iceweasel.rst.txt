.. post:: 19 Nov, 2011 19:01
   :tags: Iceweasel, Java, jre
   :category: Java
   :author: Uralbash
   :language: ru

Установка Java Environment на Debian для Iceweasel (FireFox)
============================================================

Часто веб интерфейс какого-нибудь устройства требует :l:`Java`. Что бы ее
поставить на :l:`Debian` необходимо выполнить команды:

.. code-block:: bash

   $ sudo apt-get install sun-java6-jre sun-java6-plugin
   $ update-alternatives --config java

После перезагрузки браузера настройки применяться.
