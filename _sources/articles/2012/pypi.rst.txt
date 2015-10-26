.. post:: Feb 10, 2012 13:17
   :tags: python, pypi, setuptools
   :category: Python
   :author: Uralbash
   :language: ru

Размещаем свои python проекты на https://pypi.python.org 
=========================================================

Я периодически использую свои скрипты в разных местах, для того что бы не
копипастить их постоянно и следить за обновлением удобно добавить их в
https://pypi.python.org и устанавливать/обновлять через ``pip install``. Для
этого необходимо подготовить наш модуль.

.. seealso::

   * https://packaging.python.org/en/latest/distributing.html
   * http://pythonhosted.org/setuptools/

Структура файлов:

.. code-block:: bash

   .
   ├── myproject
   │   ├── mymodel.py
   │   └── __init__.py
   ├── README.rst
   └── setup.py

.. code-block:: python
   :caption: setup.py

   from setuptools import setup
 
   def read(fname):
       return open(os.path.join(os.path.dirname(__file__), fname)).read()
 
   setup(
       name='pyandexmap',
       version='0.0.2',
       description='Scripts for get data from yandex map API',
       author='',
       author_email='',
       url='http://github.com/uralbash/pyandexmap/',
       keywords = "yandex map api search ajax geocode geocodding directions\
           navigation json",
       install_requires=[''],
       license='GPL',
       packages=['pyandexmap'],
       long_description=read('README.rst'),
       classifiers=[
           'Development Status :: 3 - Alpha',
           'Environment :: Console',
           'Intended Audience :: Developers',
           'License :: OSI Approved :: GNU General Public License (GPL)',
           'Natural Language :: English',
           'Natural Language :: Russian',
           'Operating System :: OS Independent',
           'Programming Language :: Python',
           'Topic :: Scientific/Engineering :: GIS',
           ],
   )

`classifiers <https://pypi.python.org/pypi?:action=list_classifiers>`_ - это
список разделов куда попадет ваш пакет, взять существующие можно `здесь
<https://pypi.python.org/pypi?:action=browse>`_

Затем регаем свой модуль так:

.. code-block:: bash

   $ python setup.py register
   $ python setup.py sdist upload

Отвечаем на вопросы если вы еще не зарегистрированы и все :)
