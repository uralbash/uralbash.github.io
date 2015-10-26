.. post:: Oct 17, 2011 23:22
   :tags: ThinkPad, TrackPoint, Linux
   :category: ThinkPad
   :author: Uralbash
   :language: ru

ThinkPad TrackPoint настройка в Linux
=====================================

Вообще всю информацию по настройке :l:`trackpoint` можно взять здесь: `How to
configure the TrackPoint
<http://www.thinkwiki.org/wiki/How_to_configure_the_TrackPoint>`_

Но для ленивых ниже скрипт для автозагрузки:

.. code-block:: bash

   #!/bin/sh
   xinput list | sed -ne 's/^[^ ][^V].*id=\([0-9]*\).*/\1/p' | while read id
   do
           case `xinput list-props $id` in
           *"Middle Button Emulation"*)
                   xinput set-int-prop $id "Evdev Wheel Emulation" 8 1
                   xinput set-int-prop $id "Evdev Wheel Emulation Button" 8 2
                   xinput set-int-prop $id "Evdev Wheel Emulation Timeout" 8 200
                   xinput set-int-prop $id "Evdev Wheel Emulation Axes" 8 6 7 4 5
                   xinput set-int-prop $id "Evdev Middle Button Emulation" 8 1
                   xinput set-int-prop $id "Evdev Middle Button Timeout" 8 50
                   ;;
           esac
   done

   # disable middle button
   #xmodmap -e "pointer = 1 9 3 4 5 6 7 8 2"
