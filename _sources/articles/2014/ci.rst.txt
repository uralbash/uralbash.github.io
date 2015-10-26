.. post:: 13 Apr, 2014 4:40
   :tags: Go, CI, python, TDD
   :category: QA
   :author: Uralbash
   :language: ru

Локальный Continuous Integration сервер
=======================================

.. image:: /_static/2014/ci1.jpg
   :width: 400px
   :align: center

Идея непрерывной интеграции заключается в том, что при любом изменении проекта
он пересобирается в условиях приближенных к реальной эксплуатации и каждый раз
запускает тесты. Это позволяет моментально отловить баги и исправить их не
отходя от кассы, пока ещё помнишь что понаписал.

 Принцип работы у всех примерно один:

* скачать код
* создать окружение
* установить (собрать) код
* запустить и протестировать
* отправить уведомление

Это можно сделать самостоятельно, например при помощи ``fabric``, :man:`cron`,
:man:`chroot` или :l:`docker` или при помощи готовых ``CI`` серверов:

* `jenkins <http://jenkins-ci.org/>`_
* `buildbot <http://buildbot.net/>`_
* `travis-ci <https://travis-ci.org/>`_
* `StriderCD <http://stridercd.com/>`_
* `drone <https://github.com/drone/drone>`_

Облачные ``CI`` сервера:

* https://travis-ci.org/
* https://drone.io

Облачный ``CI`` отлично подойдет для ``OpenSource`` проектов, а остальное можно
тестировать на локальном сервере.

Посмотрим как это работает на примере python + pyramid приложения.

.. image:: /_static/2014/ci2.JPG
   :width: 400px
   :align: center

Начнём с Travis-ci
------------------

.. image:: /_static/2014/travis.png
   :align: left
   :target: https://travis-ci.org/

Следит за вашими репозитариями на :l:`github`, выкачивает при каждом коммите,
собирает окружение в индивидуальном контейнере каждый раз, имеет простой
конфиг. Работает только с :l:`github`, закрытые репы за деньги.

.. raw:: html

   <br clear="both"/>

Авторизация через гитхаб аккаунт, дальше указываем что будем тестировать

.. image:: /_static/2014/travis1.png

Теперь положим в корень репы на гитхабе файл с настройками ``.travis.yml``

.. image:: /_static/2014/travis3.png

В этом файле находятся инструкции, какое окружение нужно для вашего проекта,
как его собирать, запускать тесты и куда слать уведомления.

.. code-block:: yaml
   :caption: .travis.yml

   language: python

   notifications:
     email: "sacrud@uralbash.ru"
     email: "arkadiy@bk.ru"

   python:
     1. "2.7"
     2. "2.6"

   install:
     1. pip install nose coverage coveralls
     2. pip install pyramid pyramid_jinja2 pyramid_beaker
     3. pip install -r requirements.txt

   script:
     1. nosetests --with-coverage --cover-package sacrud --cover-erase --with-doctest

   after_success:
     coveralls

Здесь я думаю и так все понятно. :pypi:`coveralls` нужен для сервиса
https://coveralls.io/ (про него ниже).

После каждого коммита создается задание в трависе которое заканчивается
примерно таким выводом https://travis-ci.org/ITCase/sacrud/jobs/22811094

Иногда СЕОшники всё портят как здесь
https://travis-ci.org/ITCase/sacrud/jobs/22688250 т.к. не умеют запускать
тесты, но благодаря ``CI`` эти проблемы сразу обнаруживаются.
Более сложный конфиг с установкой postgres
https://github.com/ITCase/pyramid_sacrud_example/blob/master/.travis.yml

drone.io
--------

.. image:: /_static/2014/drone.io.png
   :align: center

``Drone`` похож на ``travis-ci`` но он дешевле, умеет ``bitbucket`` и исходный
код https://github.com/drone/drone. Очень удобно ``OpenSource`` в облаке,
приватные репы на локальном сервере и все это имеет одинаковый конфиг. Конфиг
``.drone.yml``, формат очень похож на ``travis``.

.. code-block:: yaml
   :caption: .drone.yml

   image: python2.7
   script:
     1. pip install nose coverage pyramid pyramid_jinja2 pyramid_beaker
     2. pip install -r requirements.txt
     3. nosetests --cover-package=sacrud --cover-erase --with-coverage --with-doctest
   notify:
     email:
       recipients:
         1. sacrud@uralbash.ru

С облаком всё понятно, установим ``drone`` локально. В качестве платформы
используется VM с Ubuntu server 12.04 с одним ядром и 2Гб ОЗУ, что вполне
достаточно для небольшой команды программистов.

Т.к. drone собирает проекты в легковесных контенерах при помощи :l:`Docker`
вначале установим его
http://docs.docker.io/en/latest/installation/ubuntulinux/#ubuntu-precise

Сам drone устанавливается очень просто через ``deb`` пакет
http://drone.readthedocs.org/en/latest/install.html, теперь он у вас висит на
80 порту или на том который вы указали. Запускается и конфигурируется через
upstart (``sudo start drone``, ``sudo stop drone``). Можно проверить локально,
если перейти в репозитарий проекта с файлом ``.drone.yml``  и запустить ``drone
build``.

Для того что бы github или bitbucket слал уведомление вашему drone серверу
нужен статический IP. Пробросим порты к виртуалке на роутере :) и укажем IP в
настройках:

.. image:: /_static/2014/drone1.png
   :align: center

добавим репу:

.. image:: /_static/2014/drone2.png
   :align: center

После добавления в гитхаб появится новый аппликайшин:

.. image:: /_static/2014/drone3.png
   :align: center

``Client ID`` и ``Client Secret`` нужно указать в настройках drone. Теперь
комитим и чиним.

.. image:: /_static/2014/drone4.png
   :align: center

Для приватных реп drone автоматически  прописывает RSA ключ. Его можно
посмотреть в настройках репы и скопировать вручную например или поменять.

Вывод похож на travis:

.. image:: /_static/2014/drone5.png
   :align: center


Пару слов почему не другие системы. Во первых ``drone`` это и облако и локалхост,
дальше bitbucket+github, контейнеры docker из коробки, иконка-статус сборки в
Markdown, написан на Go как и Docker. Из недостатков пока мало свистелок и
перделок, первое что бросается в глаза отсутствие кнопки REBUILD (пересобрать
вручную). Но т.к. проект молодой то всё обещают запилить в следующей версии,
судя по issue на github'е.

``Jenkins`` страшный, сложный, всё пилить руками, докера нет, написан на Java.

``Buildbot`` написан на python, хорошая архитектура master-slave, можно запилить
slave в контейнеры, но написан на старой версии twisted и sqlalchemy аж 0.7
версии, код ужасен, инструкции из документации устаревшие, нужно додумывать,
конфиг сложный, будущего у системы нет.

``StriderCD`` написан на nodejs, много чего есть из коробки, принцип плагинов
через npm, docker пилить самому, глючный :( хотя выглядит неплохо.

Есть ещё альтернативы типа gitlab-ci и наверняка ещё что-то, но я их не смотрел.

Coveralls
---------

.. figure:: /_static/2014/coverrals1.png
   :align: center

   https://coveralls.io/r/ITCase/sacrud

.. figure:: /_static/2014/coverrals2.png
   :align: center

Coveralls отличное дополнение, в котором можно визуально отследить что ещё не
покрыто тестами. Из минусов, нет bitbucket, мне показался он дороговатым для
приватных реп и ещё лежит отдавая 503 на момент написания этой статьи, но
локальных альтернатив я не нашёл, к сожалению.

AnyKey
------

| http://jmcvetta.github.io/blog/2013/08/30/continuous-integration-for-go-code/
| http://lucapette.com/go/go-docker-and-a-ci-server/

И напоследок...
---------------

В этом обзоре показан простой пример как можно настроить непрерывную интеграцию
проекта на python и pyramid. Но по аналогии можно поднять любой другой проект.
Думаю вам эта статья поможет, хотя бы начать писать тесты :)

.. image:: /_static/2014/ci_end.jpg
   :align: center
