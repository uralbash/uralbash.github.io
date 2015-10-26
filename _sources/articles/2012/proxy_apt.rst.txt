.. post:: May 23, 2012 15:45
   :tags: Linux, apt, proxy
   :category: Linux
   :author: Uralbash
   :language: ru

Настройка прокси для apt-get
============================

В файле ``/etc/apt/apt.conf.d/proxy`` (его небыло) добавить строки:

.. code-block:: bash

   Acquire::http::Proxy "http://мой логин@ип моего прокси:порт";
   Acquire::ftp::Proxy "http://мой логин@ип моего прокси:порт";
   Acquire::::Proxy "true";
