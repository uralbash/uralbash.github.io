.. post:: Nov 12, 2012 0:23
   :tags: ThinkPad, ACPI
   :category: ThinkPad
   :author: Uralbash
   :language: ru

x220 tablet и доп кнопки в Linux
================================

.. image:: /_static/2012/x220t.jpg
   :align: left

В принципе мой :l:`Debian` Wheezy определил почти все системные клавиши кроме 2х
справа на картинке (``Ctr+Alt+Del`` и ``поворот экрана``), микрофон и zoom по
``Fn+пробел``. ``Ctr+Alt+Del`` не нужен, микрофон и зум это повод написать
отдельную тему, а про поворот экрана я напишу подробно.

.. raw:: html

   <br clear="both"/>

Вначале определим код кнопки:

| 1 - запускаем команду ``xev`` и нажимаем кнопку если появляется код то гуд иначе ``п.2``
| 2 - можно посмотреть код этой кнопки в файле ``/lib/udev/keymaps/lenovo-thinkpad_x200_tablet`` называется ``"rotate screen"`` код ``0x6c``, или можно погуглить например http://www.thinkwiki.org/wiki/Tablet_Hardware_Buttons или если не Ъ то запускаем ``tail -f | dmesg`` нажимаем кнопку и видим:

.. raw:: html

   <br/>

.. code-block:: bash

   [ 3650.281439] atkbd serio0: Unknown key released (translated set 2, code 0x6c on isa0060/serio0).
   [ 3650.281443] atkbd serio0: Use 'setkeycodes 6c <keycode>' to make it known.

Тут же нам подсказывают что надо задать ``keycode`` этой кнопке.

3 - ищем свободный код командой ``getkeycodes|grep <код>``, например: ``getkeycodes|grep 220``

Если вывода не последовало то код свободен

| 4 - назначаем код кнопке, в файле ``/etc/rc.local`` пишем ``exec sudo setkeycodes 6c 220 &``
| 5 - рестартимся или выполняем ``setkeycodes`` вручную, запускаем ``xev`` и нажимаем кнопку поворота, должны появится надписи

Дальше я привязал кнопку к действию при помощи ``xbindkeys`` (``apt-get install
xbindkeys``):

| 1 - создаем ``~/.xbindkeysrc``
| 2 - вызываем ``xbindkeys -k``
| 3 - нажимаем кнопку и копируем вывод
| 4 - записываем в ``xbindkeysrc``:

.. raw:: html

   <br/>

.. code-block:: bash

   "think-rotate"
   m:0x10 + c:224

Выполняем ``xbindkeys`` и вуаля!

| ``think-rotate`` это скрипт поворота экрана из директории ``/usr/bin/``.
| Сам скрипт я скачал с github'a: https://github.com/martin-ueding/think-rotate

.. raw:: html

   <br/>

| P.S.: Если вдруг у Вас не завелись другие кнопки то можно по аналогии привязать к ним скрипты от сюда:
| 1 - http://www.debian-fr.org/x220-install-configuration-t36152.html#p364748
| 2 - https://github.com/aconbere/thinkpad-x201-acpi-scripts

.. raw:: html

   <br/>

| UPD: вот еще нашел хорошую инструкцию http://kubuntu.ru/book/export/html/3491
| UPDUPD: Если кому то нужна кнопка посередине на экране то код ``0x67``
| UPDUPDUPD: Кнопка может быть привязана через ``acpi`` поэтому неплохо бы еще вначале проверить ``acpi_listen`` и если гут, то все таки сделать через ``acpi``.

