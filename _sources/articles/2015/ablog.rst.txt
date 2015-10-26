.. post:: 12 Aug, 2015 12:06
   :tags: Python, reStructuredText, Sphinx
   :category: Python
   :author: Uralbash
   :language: ru

.. _ablog:

Блог в reStructuredText разметке
================================

.. image:: /_static/2015/uralbash.ru.png
   :align: left
   :width: 500px

Перевел тут блог с https://www.blogger.com/ на :github:`abakan/ablog`. Блоггер
это конечно хорошо, но у него крайне ограниченный WYSIWYG, тем более только из
браузера. :l:`Ablog` по сути это :pypi:`sphinx` движок заточенный на блог со
всеми плюшками, rST разметкой, RSS и disqus комментами.

.. raw:: html

   <br clear="both"/>
   <test/>

Профит
------

* :pypi:`Sphinx` позволяет автоматизировать многие вещи
* ``reStructuredText`` позволяет точно и красиво оформлять статьи, это как
  писать "курсач" в LaTeX заместо MS Office
* писать в :l:`Vim` :)
* хоститься где угодно и хранить информацию у себя локально

Установка
---------

.. code-block:: bash
   :caption: Стабильная версия с PyPi

   $ pip install ablog

.. code-block:: bash
   :caption: Свежая версия из мастера

   $ pip install git+https://github.com/abakan/ablog

Инициализация нового проекта
----------------------------

.. seealso::

   http://ablog.readthedocs.org/manual/ablog-quick-start/

Можно добавить настройки в ``conf.py`` уже существующего проекта:

.. code-block:: python

   # 1. Add 'ablog' to list of extensions
   extensions = [
       '...',
       'ablog'
   ]

   # 2. Add ablog templates path
   import ablog

   # 2a. if `templates_path` is not defined
   templates_path = [ablog.get_html_templates_path()]

   # 2b. if `templates_path` is defined
   templates_path.append(ablog.get_html_templates_path())

Или создать новый проект:

.. code-block:: bash

   $ ablog start

Сборка и запуск
---------------

Что бы собрать блог нужно выполнить:

.. code-block:: bash

   $ ablog build

HTML страницы создаются в папке ``_website``

Что запустить блог достаточно выполнить:

.. code-block:: bash

   $ ablog serve

теперь он доступен по адресу http://127:0.0.1:8000/

Статьи
------

Главная страница блога это ``index.rst``. У меня она выглядит так:

.. code-block:: rst
   :caption: index.rst

   .. postlist:: 100500
      :excerpts:
      :date: %d-%m-%Y
      :format: {date} | {title}

Статьи могут находится в любом месте проекта, :l:`Ablog` их находит
автоматически. Я отвожу для этого отдельную папку:

.. code-block:: bash

   articles/
   ├── 2011
   │   ├── breadcrumbs.rst
   │   ├── capslock.rst
   │   ├── content_assigment.rst
   │   ├── couchdb.rst
   │   ├── demo.rst
   │   ├── fa_keyerror.rst
   │   ├── fa_rest_controller.rst
   │   ├── formalchemy.rst
   │   ├── java_iceweasel.rst
   │   ├── js_rrd.rst
   │   ├── peyote.rst
   │   ├── psycopg2.rst
   │   ├── pylons_backslash.rst
   │   ├── pylons_jscss_minificator.rst
   │   ├── repoze.what.rst
   │   ├── rrd_cut2.rst
   │   ├── rrd_cut.rst
   │   ├── rrdtool_ds18b20.rst
   │   ├── snmp.rst
   │   ├── sqlalchemy_mixin.rst
   │   ├── sqlalchemy_trigger.rst
   │   ├── sqlalchemy_uml.rst
   │   ├── sqlalchemy_why_postgres.rst
   │   ├── thinkpad_trackpoint.rst
   │   ├── wtforms.rst
   │   ├── wtform_validation.rst
   │   ├── xfce_button.rst
   │   └── xfce_display.rst
   ├── 2012
   │   ├── 3com.rst
   │   ├── ajax_triple_select.rst
   │   ├── arch_book.rst
   │   ├── boyan.rst
   │   ├── cms_pyramid2.rst
   │   ├── cms_pyramid.rst
   │   ├── debian_bc.rst
   │   ├── fortune.rst
   │   ├── http_header_pyramid.rst
   │   ├── jinja_pylons_i18n.rst
   │   ├── jinja_pyramid_i18n.rst
   │   ├── jinja_silent_none.rst
   │   ├── lisp_book.rst
   │   ├── lisp.rst
   │   ├── matplotlib_numpy_and_virtualenv.rst
   │   ├── osm_example.rst
   │   ├── osm_geotagging.rst
   │   ├── paramiko.rst
   │   ├── penguin_troll.rst
   │   ├── penguin_vs_leo.rst
   │   ├── pg_book.rst
   │   ├── proxmox.rst
   │   ├── proxy_apt.rst
   │   ├── pycrypto_virtualenv.rst
   │   ├── pylons_console.rst
   │   ├── pylons_csv.rst
   │   ├── pylons_yapsy.rst
   │   ├── pypi.rst
   │   ├── pyramid_as_django.rst
   │   ├── pyramid_blog.rst
   │   ├── pyramid_formalchemy.rst
   │   ├── python_labirint.rst
   │   ├── python_snmp.rst
   │   ├── pyyandexmap.rst
   │   ├── qtile.rst
   │   ├── redactor-js.rst
   │   ├── rrd_faketime.rst
   │   ├── rrdtool.rst
   │   ├── sa_big_table.rst
   │   ├── shared_net.rst
   │   ├── sphinx_github.rst
   │   ├── sran.rst
   │   ├── vi_book.rst
   │   └── x220t_buttons.rst
   ├── 2013
   │   ├── car_differ.rst
   │   ├── dbsession_includeme.rst
   │   ├── flag_jinja2.rst
   │   ├── hdaps_music.rst
   │   ├── hstore_plpython.rst
   │   ├── jinja2_lorem_ipsum.rst
   │   ├── linux_attack.rst
   │   ├── matrix.rst
   │   ├── penguin.rst
   │   ├── python_vim.rst
   │   ├── reason_names_of.rst
   │   ├── sacrud_alfa_0_1_0.rst
   │   ├── sacrud_demo.rst
   │   ├── sacrud_howto.rst
   │   ├── sacrud_release_0_0_3.rst
   │   ├── sacrud_release_0_1_0.rst
   │   ├── sacrud.rst
   │   ├── vi_book.rst
   │   ├── vim_book.rst
   │   ├── wat.rst
   │   └── where_to_start.rst
   ├── 2014
   │   ├── alembic_pyramid.rst
   │   ├── chameleon_deform.rst
   │   ├── ci.rst
   │   ├── cornice.rst
   │   ├── django-hyango.rst
   │   ├── go.rst
   │   ├── ia32-libs-multiarch:i386.rst
   │   ├── king_penguin.rst
   │   ├── man_month_book.rst
   │   ├── panel_pyramid_dt.rst
   │   ├── sacrud_release_0.1.1.rst
   │   ├── sacrud_release_0.1.2.rst
   │   ├── scrum.rst
   │   ├── sphinx.rst
   │   ├── sqlalchemy_mptt_0_0_5.rst
   │   ├── sqlalchemy_mptt.rst
   │   └── sqlite_to_pg.rst
   └── 2015
       └── ablog.rst

   5 directories, 111 files

Что бы файл ``*.rst`` стал статьей в нем должна находится директива ``post``.
Например:

.. code-block:: rst
   :caption: foo.rst

   .. post:: 15 Apr, 2013
      :tags: tips, ablog, directive
      :category: Example, How To
      :author: Ahmet, Durden
      :location: Pittsburgh, SF
      :redirect: blog/old-page-name-for-the-post
      :excerpt: 2
      :image: 1

.. seealso::

   http://ablog.readthedocs.org/manual/posting-and-listing/#directive-post

Пример кода этой статьи:

.. literalinclude:: /articles/2015/ablog.rst
   :language: rst
   :caption: articles/2015/ablog.rst

Тема
----

Тема по умолчанию `Alabaster <https://github.com/bitprophet/alabaster>`_, но
можно поменять стандартными способами в ``conf.py``:

.. no-code-block:: python
   :caption: conf.py установка темы

   import itcase_sphinx_theme

   ...

   # -- Options for HTML output ----------------------------------------------

   # The theme to use for HTML and HTML Help pages.  See the documentation for
   # a list of builtin themes.
   html_theme = 'itcase'

   # Theme options are theme-specific and customize the look and feel of a theme
   # further.  For a list of options available for each theme, see the
   # documentation.
   html_theme_options = {
       'github_button': False,
   }

   # Add any paths that contain custom themes here, relative to this directory.
   html_theme_path = [itcase_sphinx_theme.get_html_themes_path()]

   # The name for this set of Sphinx documents.  If None, it defaults to
   # "<project> v<release> documentation".
   html_title = "Ural penguins"

Я использую :github:`ITCase/itcase_sphinx_theme` с переопределенными шаблонами.
Для этого нужно указать где искать шаблоны и в каком порядке:

.. code-block:: python
   :caption: conf.py указание пути по шаблонов темы

   # Add any paths that contain templates here, relative to this directory.
   templates_path = ['_templates', ablog.get_html_templates_path()]

Добавляем в папку ``_templates`` нашего проекта шаблон ``layout.html``:

.. literalinclude:: /_templates/layout.html
   :language: html+jinja
   :caption: _templates/layout.html

Комменты
--------

.. seealso::

   http://ablog.readthedocs.org/manual/ablog-configuration-options/#disqus-integration

DISQUS ставится просто:

.. code-block:: python
   :caption: настройки DISQUS

   # -- disqus integration -------------------------------------------------------

   # you can enable disqus_ by setting ``disqus_shortname`` variable.
   # disqus_ short name for the blog.
   disqus_shortname = "uralbash"

Правда у меня завелось только после переопределения шаблона ``page.html``:

.. literalinclude:: /_templates/page.html
   :language: html+jinja
   :caption: _templates/page.html
   :emphasize-lines: 37

Деплой
------

.. note::

   http://ablog.readthedocs.org/manual/deploy-to-github-pages/

#. Создать репу с именем ``<username>.github.io``
#. Добавить имя в настройки, например:

   .. code-block:: python
      :caption: conf.py настройки github pages

      github_pages = "uralbash"

#. Сбилдить проект:

   .. code-block:: bash

      $ ablog build

#. Выложить на гитхаб:

   .. code-block:: bash

      $ ablog deploy

Итог
----

На мой взгляд лучшее решения для ``reStructuredText`` блога, местами сырой, но
активно развивается.
