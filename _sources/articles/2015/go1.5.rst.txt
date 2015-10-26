.. post:: 20 Aug, 2015 10:59
   :tags: Go
   :category: Go
   :author: Uralbash
   :language: ru

Обновление до Go1.5
===================

Для управления версиями я пользуюсь :github:`moovweb/gvm`.

Установка gvm
-------------

.. code-block:: bash

   $ bash < <(curl -s -S -L https://raw.githubusercontent.com/moovweb/gvm/master/binscripts/gvm-installer)

Установка Go 1.4
----------------

.. code-block:: bash

   $ gvm install go1.4
   $ gvm use go1.4

Установка Go 1.5
----------------

Теперь компилим ``Go`` при помощи ``Go``:

.. code-block:: bash

   $ GOROOT_BOOTSTRAP=~/.gvm/gos/go1.4 gvm install go1.5
   Installing go1.5...
    + Compiling...

Выберем новую версию:

.. code-block:: bash

   $ gvm use go1.5
   Now using version go1.5
   $ go version
   go version go1.5 linux/amd64

Убедимся в ее наличии:

.. no-code-block:: bash

   $ gvm list --all

   gvm gos (installed)

      go1.4
   => go1.5
      system
