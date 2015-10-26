.. post:: 22 Sep, 2014 15:15
   :tags: Python, Sphinx, reStructuredText
   :category: Python
   :author: Uralbash
   :language: ru

Документация python проекта на практике
=======================================

Документация в :l:`python` проектах пишется при помощи :pypi:`sphinx`, он умеет
используя расширение ``automodule`` читать докстринги и формировать
документацию из кода.

Создать проект можно ответив на вопросы через ``sphinx-quickstart`` или
использовать уже подготовленный шаблон который генерит дополнительно API
пакета:

.. code-block:: bash

   $ sphinx-apidoc -F -o docs sacrud

   Creating file docs/sacrud_deform.rst.
   Creating file docs/sacrud_deform.tests.rst.
   Creating file docs/conf.py.
   Creating file docs/index.rst.
   Creating file docs/Makefile.
   Creating file docs/make.bat.

Автогенератор API обычно создает много лишнего и далеко не идеально генерит
названия, поэтому удалим API тестов и попереимеуем все остальное.

.. image:: /_static/2014/sphinx.png
   :align: center

Для сборки доков  нужно в папке docs выполнить make html. Ниже структура
получившихся файлов:

.. code-block:: bash

   .
   ├── _build
   │   ├── doctrees
   │   │   ├── environment.pickle
   │   │   ├── index.doctree
   │   │   ├── readme.doctree
   │   │   ├── sacrud_deform.doctree
   │   │   └── sacrud_deform.tests.doctree
   │   └── html
   │       ├── genindex.html
   │       ├── index.html
   │       ├── _modules
   │       │   ├── index.html
   │       │   ├── sacrud_deform
   │       │   │   ├── tests
   │       │   │   │   └── test_form.html
   │       │   │   └── widgets.html
   │       │   └── sacrud_deform.html
   │       ├── objects.inv
   │       ├── py-modindex.html
   │       ├── readme.html
   │       ├── sacrud_deform.html
   │       ├── sacrud_deform.tests.html
   │       ├── search.html
   │       ├── searchindex.js
   │       ├── _sources
   │       │   ├── index.txt
   │       │   ├── readme.txt
   │       │   ├── sacrud_deform.tests.txt
   │       │   └── sacrud_deform.txt
   │       └── _static
   │           ├── ajax-loader.gif
   │           ├── basic.css
   │           ├── comment-bright.png
   │           ├── comment-close.png
   │           ├── comment.png
   │           ├── default.css
   │           ├── doctools.js
   │           ├── down.png
   │           ├── down-pressed.png
   │           ├── file.png
   │           ├── jquery.js
   │           ├── minus.png
   │           ├── plus.png
   │           ├── pygments.css
   │           ├── searchtools.js
   │           ├── sidebar.js
   │           ├── underscore.js
   │           ├── up.png
   │           ├── up-pressed.png
   │           └── websupport.js
   ├── conf.py
   ├── index.rst
   ├── make.bat
   ├── Makefile
   ├── readme.rst
   ├── sacrud_deform.rst
   ├── _static
   └── _templates

В директории ``build/html`` находится наша готовая документация в формате html.
Добавим описание проекта на основную страницу. Для этого создаем ``readme.rst``
в директории ``docs`` и включаем его в ``index.rst``, а в дальнейшем этот же
``readme.rst`` будет использоваться в ``README.rst`` в корне проекта для
:l:`github` и :l:`PyPi`.

.. code-block:: rst
   :caption: readme.rst

   |Build Status| |Coverage Status| |Stories in Progress| |PyPI|

   .. |Build Status| image:: https://travis-ci.org/ITCase/sacrud_deform.svg?branch=master
      :target: https://travis-ci.org/ITCase/sacrud_deform
   .. |Coverage Status| image:: https://coveralls.io/repos/ITCase/sacrud_deform/badge.png?branch=master
      :target: https://coveralls.io/r/ITCase/sacrud_deform?branch=master
   .. |Stories in Progress| image:: https://badge.waffle.io/ITCase/sacrud_deform.png?label=in%20progress&title=In%20Progress
      :target: http://waffle.io/ITCase/sacrud_defrom
   .. |PyPI| image:: http://img.shields.io/pypi/dm/sacrud_deform.svg
      :target: https://pypi.python.org/pypi/sacrud_deform/

   sacrud_deform
   ==============

   Form generotor for SQLAlchemy models.

   Install
   =======

   develop version from source

   .. code-block:: bash

     pip install git+git://github.com/ITCase/sacrud_deform@develop

   from pypi

   .. code-block:: bash

     pip install sacrud_deform

   Use
   ===

   .. code-block:: python

           data = form_generator(dbsession=DBSession,
                                      obj=obj_of_model, table=MyModel,
                                      columns=columns_of_model)
           form, js_list = data.render()

Здесь вроде все понятно, но есть нюансы, если например в директиве ``..
code-block`` не указать язык или не дописывать "=" в заголовках то документация
будет нормально генериться, но :l:`PyPi` её не поймет и выведет что-то типа
этого:

.. image:: /_static/2014/sphinx2.png
   :align: center

Нужно быть внимательным и проверять как :l:`PyPi` подхватил ``README.rst``.
Дальше в наш ``index.rst`` добавим ``readme.rst`` что бы он был по презентабельнее.

.. code-block:: rst
   :caption: index.rst

   Welcome to sacrud_deform's documentation!
   =========================================

   .. include:: readme.rst

   Contents:

   .. toctree::
      :maxdepth: 4

      sacrud_deform


   Indices and tables
   ==================

   + :ref:`genindex`
   + :ref:`modindex`
   + :ref:`search`

Теперь главная страница документации выглядит так:

.. image:: /_static/2014/sphinx3.png
   :align: center

В принципе уже хорошо, добавим это описание ещё для :l:`github` и :l:`PyPi`.
Они по умолчанию берут файл ``README.rst`` из корня проекта и ничего про папку
``docs`` не знают. Можно было бы заинклудить ``readme.rst`` в корневой
``README.rst`` (``..include:: docs/readme.rst``), но :l:`github` и :l:`PyPi`
инклуды не понимают, поэтому придется писать свой велосипед для копирования.
Либо тупо делать это вручную. Я написал на коленке простой скрипт который это
делает и заменяет директивы ``include`` содержанием их файлов.

.. code-block:: python
   :caption: make_README.py

   import fileinput
   import os
   from shutil import copyfile

   src = "readme.rst"
   src_path = os.path.dirname(os.path.realpath(src))
   dst = "../README.rst"
   copyfile(src, dst)


   def read_file(path):
       with open(path, 'r') as f:
           return f.read()

   for line in fileinput.input(dst, inplace=1):
       splitted = line.rstrip().split('.. include:: ')
       if len(splitted) == 2:
           line = read_file(os.path.join(src_path, splitted[1]))
           print line
       else:
           print line.rstrip()

Теперь если запустить из папки ``docs`` команду ``python make_README.py``, то в
корне проекта появится или перезапишется файл ``README.rst``.

Что бы это делалось автоматически при каждой сборке документации добавим в
``Makefile``:

.. code-block:: bash

   ...
 
   readme:
    python make_README.py
 
   readme_html: html readme

При вызове ``make readme_html`` у нас будет собираться документация html и
копипаститься ``readme``.

Конфиг для ``sphinx`` лежит в папке документации с названием ``conf.py``. В нем
можно настраивать документацию как угодно, к примеру оформим тему как у проекта
:l:`Pyramid`.

Добавим в ``conf.py``:

.. code-block:: python
   :caption: conf.py

   import sys
   import os
 
   # Add and use Pylons theme
   if 'sphinx-build' in ' '.join(sys.argv):  # protect against dumb importers
       from subprocess import call, Popen, PIPE
 
       p = Popen('which git', shell=True, stdout=PIPE)
       git = p.stdout.read().strip()
       cwd = os.getcwd()
       _themes = os.path.join(cwd, '_themes')
 
       if not os.path.isdir(_themes):
           call([git, 'clone', 'git://github.com/Pylons/pylons_sphinx_theme.git',
                 '_themes'])
       else:
           os.chdir(_themes)
           call([git, 'checkout', 'master'])
           call([git, 'pull'])
           os.chdir(cwd)
 
       sys.path.append(os.path.abspath('_themes'))
 
       parent = os.path.dirname(os.path.dirname(__file__))
       sys.path.append(os.path.abspath(parent))
       wd = os.getcwd()
       os.chdir(parent)
       os.chdir(wd)
 
       for item in os.listdir(parent):
           if item.endswith('.egg'):
               sys.path.append(os.path.join(parent, item))

Этот код я скопировал из проекта :l:`Pyramid`, он просто выкачивает их тему с
:l:`github` в директорию ``_themes``. Далее укажем в конфиге название темы и где их искать.

.. code-block:: python

   # -- Options for HTML output ----------------------------------------------
 
   # The theme to use for HTML and HTML Help pages.  See the documentation for
 
   # a list of builtin themes.
 
   html_theme = 'pyramid'
 
   # Add any paths that contain custom themes here, relative to this directory.
 
   html_theme_path = ['_themes']

После сборки проект будет выглядеть так:

.. image:: /_static/2014/sphinx4.png
   :align: center

Вообщем конфиг умеет много чего, есть еще много разных расширений, можно писать
свои, либо переопределять/добавлять директивы итд. Вот например как вставлять
ссылки на сторонние проекты:

.. code-block:: python

   # Add any Sphinx extension module names here, as strings. They can be
   # extensions coming with Sphinx (named 'sphinx.ext.*') or your custom
   # ones.
   extensions += [
       'sphinx.ext.intersphinx'
   ]
 
   intersphinx_mapping = {
       'sacrud': ('http://sacrud.readthedocs.org/en/latest/', None),
   }

Теперь можно писать так:

.. code-block:: rst

   Use :py:class:`sacrud.common.TableProperty` decorator.

И наверно последний этап это добавление документации на readthedocs. Сложностей
там особо нету, регаешься, добавляешь гитхаб профиль, синхронизируешь репы и
выбираешь нужные. Хук на коммиты в таком случае вешается автоматически,
единственное что нужно отметить что если в документации используется automodule
нужно ставить галку virtualenv (в настройках readthedocs) иначе он просто будет
пустым.

.. image:: /_static/2014/sphinx5.png
   :align: center

Готовый пример можно посмотреть здесь
http://sacrud-deform.readthedocs.org/en/develop/ 
