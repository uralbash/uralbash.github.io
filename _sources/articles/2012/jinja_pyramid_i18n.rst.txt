.. post:: Sep 28, 2012 17:58
   :tags: Python, Pyramid, Jinja
   :category: Pyramid
   :author: Uralbash
   :language: ru

Перевод шаблонов Jinja в Pyramid
================================

Основная документация как это делать `здесь
<http://pyramid.readthedocs.org/en/1.2-branch/narr/i18n.html>`_. Но как обычно
есть нюансы.

Рабочий пример можно посмотреть установив шаблон. Устанавливаем :pypi:`pyramid_jinja2`
и пишем: 

.. code-block:: bash

   $ pcreate -t pyramid_jinja2_starter yoyoyoyo

| Если лень устанавливать пример, то нужно сделать следующее:
| Добавить файл ``message-extraction.ini``

.. code-block:: ini

   [python: **.py]
   [jinja2: **.jinja2]
   encoding = utf-8

Добавить в ``setup.cfg``:

.. code-block:: ini

   [extract_messages]
   add_comments = TRANSLATORS:
   output_file = myprojectname/locale/myprojectname.pot
   width = 80
   mapping_file = message-extraction.ini


далее в ``__init__.py``:

.. code-block:: python

   settings = dict(settings)
   settings.setdefault('jinja2.i18n.domain', 'myprojectname')
   ...
   config.add_translation_dirs("myprojectname:locale/")
   config.include('pyramid_jinja2')
   config.add_jinja2_search_path("myprojectname:templates")

В шаблоне: 

.. code-block:: jinja

   {{ gettext('Logo') }}
   {% trans %}Home{% endtrans %}

все остальное по документации... Я для упрощения использую скрипт ``i18n.sh``:

.. code-block:: bash

   #!/usr/bin/env bash
 
   py=python
 
   $py setup.py extract_messages
   $py setup.py update_catalog
   $py setup.py compile_catalog
   # vim:set et sts=4 ts=4 tw=80:
