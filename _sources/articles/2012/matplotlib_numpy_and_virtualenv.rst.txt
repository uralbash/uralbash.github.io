.. post:: Mar 12, 2012 16:59
   :tags: Python, Virtualenv, NumPy, Matplotlib
   :category: Python
   :author: Uralbash
   :language: ru

Установка Matplotlib и Numpy в virtualenv на Debian 
====================================================

Взято от `сюда
<http://coreygoldberg.blogspot.com/2012/01/python-matplotlib-and-numpy-on.html>`_.
Если вы хотите в :pypi:`virtualenv`, через ``pip install`` установить
:pypi:`Numpy` и :pypi:`Matplotlib`, то необходимо вначале поставить в систему:

.. code-block:: bash

   $ sudo apt-get install build-essential python-dev libfreetype6-dev libpng-dev python-virtualenv

Дальше устанавливаем пакеты в окружении:

.. code-block:: bash

   $ virtualenv env
   $ cd env
   $ source bin/activate
   (env)$ pip install numpy matplotlib
 
   ...
   ...
   Successfully installed numpy matplotlib
   Cleaning up...
