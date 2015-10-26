.. post:: 09 Mar, 2013 2:55
   :tags: python, sacrud, CRUD, Pyramid, Jinja
   :category: Pyramid
   :author: Uralbash
   :language: ru

CRUD интерфейс для SQLAlchemy и подключение к Pyramid
=====================================================

Запилил Yet another CRUD интерфейс для :l:`SQLAlchemy`. По сути это аналог
:l:`Django` админки или :l:`FormAlchemy`, но ОЧЕНЬ сильно упрощенный, ничего
лишнего. Есть поддержка большинства полей + кастомные поля типа файл (для
загрузки файлов, изображений) и ``GUID``. Довольно просто подключить к
:l:`Pyramid` проекту и сразу начать работать по адресу
http://localhost:6543/sacrud

Проект доступен на github https://github.com/uralbash/sacrud

В след. релизах планирую добавить новые типы полей, кастомные поля типа
``tree`` и ``btree`` с ``AJAX`` обработкой в интерфейсе, расширение для других
фреймворков (например :l:`flask`), кастомные фильтры, пагинацию итд

Установка
---------

.. code-block:: bash
   :caption: PyPi

   $ pip install sacrud

.. code-block:: bash
   :caption: source

   $ python setup.py install

Пример использования в Pyramid
------------------------------

Add to your project config:

.. code-block:: python

   # pyramid_jinja2 configuration
   config.include('pyramid_jinja2')
   config.add_jinja2_search_path("myprojectname:templates")

   from .models import (Model1, Model2, Model3,)
   # add sacrud and project models
   config.include('sacrud.pyramid_ext')
   settings = config.registry.settings
   settings['sacrud_models'] = (Model1, Model2, Model3)

go to http://localhost:6543/sacrud

Скриншоты
---------

.. figure:: /_static/2013/sacrud_index.png
   :align: center

   список таблиц

.. figure:: /_static/2013/sacrud_rows.png
   :align: center

   список записей в таблице

.. figure:: /_static/2013/sacrud_edit.png
   :align: center

   редактирование записи
