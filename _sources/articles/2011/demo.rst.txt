.. post:: 19 Dec, 2011 11:48
   :tags: paster, Python, pylons, virtualenv
   :category: Pylons
   :author: Uralbash
   :language: ru

Как поднять demo версию проекта на paster + virtualenv в Debian
===============================================================

Иногда необходимо поднять демо версию своего проекта для тестов. Склонируем
наш ``git`` проект на сервер где будет демо:

.. code-block:: bash

   $ git clone --bare ~/myproject ssh://uralbash@myserver/~/my_project.git

Проект склонируется в домашнюю директорию сервера ``myserver``. Ключ ``--bare``
означает что клон предназначен только для ``push`` или ``pull`` т.е. все
коммиты мы будем делать у себя локально а потом пушить на сервак. Далее напишем
скрипт который будет из нашего bare репозитария создавать проект для запуска демки:

.. code-block:: bash

   $ rm -r /home/uralbash/my_project
   $ git clone /home/uralbash/my_project.git /home/uralbash/my_project

После этого создаем :l:`virtualenv` окружение (``/home/uralbash/mypythonenv/``)
и добавляем скрипт запуска в ``/etc/init.d/my_project.sh``:

.. code-block:: bash

   #! /bin/sh
  
   ### BEGIN INIT INFO
   # Required-Start:    $all
   # Default-Start:     2 3 4 5
   # Default-Stop:      0 1 6
   # Short-Description: starts the paster server
   # Description:       starts paster 
   ### END INIT INFO
  
  
   PROJECT=/home/uralbash/my_project
   PID_DIR=/var/run/my_project/
   PID_FILE=/var/run/my_project/paster.pid
   LOG_FILE=/home/uralbash/logs/my_project/paster.log
   USER=root
   GROUP=root
   PROD_FILE=/home/uralbash/demo.ini
   RET_VAL=0
  
   cd $PROJECT
  
   case "$1" in
   start)
   ../mypythonenv/bin/paster  serve \
   --daemon \
   --pid-file=$PID_FILE \
   --log-file=$LOG_FILE \
   --user=$USER \
   --group=$GROUP \
   $PROD_FILE \
   start
  
   ;;
   stop)
   ../mypythonenv/bin/paster  serve \
   --daemon \
   --pid-file=$PID_FILE \
   --log-file=$LOG_FILE \
   --user=$USER \
   --group=$GROUP \
   $PROD_FILE \
   stop
  
   ;;
   restart)
   ../mypythonenv/bin/paster  serve \
   --daemon \
   --pid-file=$PID_FILE \
   --log-file=$LOG_FILE \
   --user=$USER \
   --group=$GROUP \
   $PROD_FILE \
   restart
  
  
   ;;
   *)
   echo $"Usage: $0 {start|stop|restart}"
   exit 1
   esac

Для того что бы не хранить пароли в репозитарии файл с настройками вынесен
отдельно ``/home/uralbash/demo.ini``:

И добавляем в крон перезапуск:

.. code-block:: bash

   # выполнять каждый четный час в 00 мин
   0 */2 * * *       /etc/init.d/my_project.sh stop > /dev/null
   # выполнять каждый четный час в 01 мин
   1 */2 * * *       /etc/init.d/my_project.sh start > /dev/null
   # выполнять в 9:00
   0 9 * * *       /home/uralbash/demo_update.sh > /dev/null

После этого каждый день в 9:00 будет обновляться код и каждые 2 часа
перезапускаться сервер (на случай если он полег по какойто причине). Останется
только периодически отправлять коммиты. 
