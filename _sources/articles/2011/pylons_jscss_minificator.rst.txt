.. post:: Oct 25, 2011 00:36
   :tags: Pylons, JavaScript, CSS
   :category: Pylons
   :author: Uralbash
   :language: ru

Pylons javascript и css link
============================

В :l:`Pylons`, в шаблоны есть возможность вставить :l:`CSS` или JS при помощи
:func:`~webhelpers.pylonslib.minify.stylesheet_link` и
:func:`~webhelpers.pylonslib.minify.javascript_link`. Но существует расширение
:l:`MinificationWebHelpers` которое позволяет также удобно добавлять javascript
файлы.

Установка:

.. code-block:: bash

   $ pip install MinificationWebHelpers

Пример использования:

.. code-block:: mako

   ${h.javascript_link('/js/file1.js',
                       '/js/file2.js',
                       minified=True,
                       combined=True,
                       combined_filename='all_javascript_files')}

   ${h.stylesheet_link('/css/style1.css',
                       '/css/style2.css',
                       minified=True,
                       combined=True,
                       beaker_kwargs=dict(invalidate_on_startup=False))}

Очень удобная штука, особенно когда нужно добавить много файлов. 
