.. post:: Oct 17, 2011 23:13
   :tags: CapsLock, Linux
   :category: KeyBoard
   :author: Uralbash
   :language: ru

Отключить CapsLock
==================

Отключение:

.. code-block:: bash

   xmodmap -e "clear Lock"
 
Включение:

.. code-block:: bash

   xmodmap -e "add Lock = Caps_Lock"

или в ``/usr/share/X11/xkb/keycodes/xfree86`` заменить ``CAPS`` код 66 на 100500 
