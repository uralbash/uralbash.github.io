.. post:: 13 Aug, 2015 17:24
   :tags: Python, reStructuredText, Sphinx
   :category: Python
   :author: Uralbash
   :language: ru

Vim (и не только) для reStructuredText
======================================

:l:`reStructuredText` - очень удобная разметка, которая используется при
написании документации (:pypi:`sphinx`), README для гитхаба, битбакета, PyPi и
всего прочего вплоть до научных статей (`пример
<http://hplgit.github.io/teamods/writing_reports/_static/report_solarized2_dark.html>`_)
или же статей для :ref:`блога <ablog>` :)

Так-как документация отымает бОльшую часть времени от разработки, то хорошо бы
настроить окружение под нее. В конечном счете это сэкономит кучу времени.

TagBar
------

:github:`majutsushi/tagbar` - это плагин который показывает тэги документа в
отдельном окне. Для настройки нам понадобится добавить его в :l:`Vim` и
установить клавишу вызова (например :kbd:`F8`).

.. code-block:: vim
   :caption: .vimrc добавляем tagbar

   Plug 'majutsushi/tagbar'
   nmap <F8> :TagbarToggle<CR>

Все что ``tagbar`` не умеет делать "из коробки" можно подключить в виде
отдельных расширений (`полный список тут
<https://github.com/majutsushi/tagbar/wiki>`_) или написать свое.

Для :l:`reStructuredText` есть :github:`jszakmeister/rst2ctags`. Подключается
он довольно просто:

* клонируем ``git`` репу в локальную папку
* и указываем путь до нее в ``.vimrc``

.. code-block:: vim
   :caption: .vimrc конфиг rst2ctags
   :emphasize-lines: 3

   let g:tagbar_type_rst = {
       \ 'ctagstype': 'rst',
       \ 'ctagsbin' : '/home/uralbash/.vim/config/common/rst2ctags/rst2ctags.py',
       \ 'ctagsargs' : '-f - --sort=yes',
       \ 'kinds' : [
           \ 's:sections',
           \ 'i:images'
       \ ],
       \ 'sro' : '|',
       \ 'kind2scope' : {
           \ 's' : 'section',
       \ },
       \ 'sort': 0,
   \ }

После этого по нажатию :kbd:`F8` будет открываться "содержание" вашего
документа, как показано на рисунках ниже.

.. figure:: /_static/2015/rst2ctags.png
   :align: center

   Пример :github:`jszakmeister/rst2ctags` в :l:`Vim`

.. figure:: /_static/2015/rst2ctags2.png
   :align: center

   Пример :github:`jszakmeister/rst2ctags` в :l:`Vim` с более простой структурой
   разделов

Таким образом упрощается навигация по документу и структурирование разделов.

Riv.vim
-------

:github:`Rykka/riv.vim` - это что то на подобии ``python-mode`` для
:l:`reStructuredText`. Всякие ништяки для rST.

.. code-block:: vim
   :caption: Добавляем :github:`Rykka/riv.vim` в :l:`Vim`

   Plug 'Rykka/riv.vim', { 'for': 'rst' }

В командном режиме появятся следующие функции:

.. code-block:: bash

    Riv2BuildPath           RivCreateHyperLink      RivHelpTodo
    RivListType2            RivShiftRight           RivTitle2
    Riv2HtmlAndBrowse       RivCreateInterpreted    RivInstruction
    RivListType3            RivSpecification        RivTitle3
    Riv2HtmlFile            RivCreateLink           RivIntro
    RivListType4            RivSuperCEnter          RivTitle4
    Riv2HtmlProject         RivCreateLiteralBlock   RivItemClick
    RivNormEqual            RivSuperEnter           RivTitle5
    Riv2Latex               RivCreateLiteralInline  RivItemToggle
    RivNormLeft             RivSuperMEnter          RivTitle6
    Riv2Odt                 RivCreateStrong         RivLinkNext
    RivNormRight            RivSuperSEnter          RivTodoAsk
    Riv2Pdf                 RivCreateTime           RivLinkOpen
    RivPrimer               RivTableCreate          RivTodoDate
    Riv2S5                  RivCreateTransition     RivLinkPrev
    RivProjectHtmlIndex     RivTableFormat          RivTodoDel
    Riv2Xml                 RivDeleteFile           RivLinkShow
    RivProjectIndex         RivTableNextCell        RivTodoPrior
    RivCheatSheet           RivDirectives           RivListDelete
    RivProjectList          RivTablePrevCell        RivTodoToggle
    RivCreateContent        RivFoldAll              RivListNew
    RivQuickStart           RivTestFold0            RivTodoType1
    RivCreateDate           RivFoldToggle           RivListSub
    RivReload               RivTestFold1            RivTodoType2
    RivCreateEmphasis       RivFoldUpdate           RivListSup
    RivScratchCreate        RivTestObj              RivTodoType3
    RivCreateExplicitMark   RivGetLatest            RivListToggle
    RivScratchView          RivTestTest             RivTodoType4
    RivCreateFoot           RivHelpFile             RivListType0
    RivShiftEqual           RivTitle0               RivTodoUpdateCache
    RivCreateGitLink        RivHelpSection          RivListType1
    RivShiftLeft            RivTitle1               RivVimTest

Плагин имеет множество разных функций которые лучше
изучить при помощи команды ``:RivQuickStart``.

Например команда ``:Riv2HtmlAndBrowse`` рендерит текущий документ и открывает
его в браузере. А сочетание клавиш :kbd:`Ctrl+e` :kbd:`s` :kbd:`1` делает
текущую строку заголовком 1-го уровня, т.е. выполняет функцию ``RivTitle1``.

rstcheck
--------

:github:`myint/rstcheck` - это отличный линтер для разметки
:l:`reStructuredText`, также помимо ``docutils`` он понимает :l:`sphinx-doc`
директивы и роли.

Установка
~~~~~~~~~

.. code-block:: bash

    $ pip install --upgrade rstcheck

Проверка синтаксиса в директиве ``code-block``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``rstcheck`` умеет проверять синтаксис следующих языков в директиве ``..
code-block::``:

* Bash
* Doctest
* C (C99)
* C++ (C++11)
* JSON
* Python
* reStructuredText

Например я допустил ошибку в коде на :l:`Python`:

.. code-block:: rst

    ====
    Test
    ====

    .. code-block:: python

        print(


В этом случае линтер ее обнаружит:

.. code-block:: bash

    $ rstcheck bad_python.rst
    bad_python.rst:7: (ERROR/3) (python) unexpected EOF while parsing

Еще пример с ``C++``:

.. code-block:: rst

    ====
    Test
    ====

    .. code-block:: cpp

        int main()
        {
            return x;
        }

.. code-block:: bash

    $ rstcheck bad_cpp.rst
    bad_cpp.rst:9: (ERROR/3) (cpp) error: 'x' was not declared in this scope

И пример с опечаткой в rST:

.. code-block:: rst

    ====
    Test
    ===

.. code-block:: bash

   $ rstcheck bad_rst.rst
   bad_rst.rst:1: (SEVERE/4) Title overline & underline mismatch.

Я добавляю ``rstcheck`` в ``travis-ci`` и после каждого коммита проверяю
документацию на наличие ошибок. Пример можно посмотреть `здесь
<https://github.com/ustu/lectures.www/blob/master/test.sh>`_.

Конфиг
~~~~~~

Про некоторые кастомные роли и директивы ``rstcheck`` ничего не знает, что бы
они не попадали в поток ошибок их можно заигнорировать при помощи конфига. Файл
конфига с именем ``.rstcheck.cfg`` должен располагаться в текущей директории,
например так:

.. code-block:: bash

    docs
    ├── foo
    │   └── bar.rst
    ├── index.rst
    └── .rstcheck.cfg

Можно указать какие игнорировать директивы или роли, например для блога я
использую некоторые кастомные вещи из модуля расширения
:github:`ITCase/sphinx-links` и :l:`ABlog`:

.. code-block:: ini

    [rstcheck]
    ignore_directives=post,postlist
    ignore_roles=github,l,man,pypi

Модуль для Python
~~~~~~~~~~~~~~~~~

Т.к. :pypi:`rstcheck` это :l:`Python` модуль, его можно использовать в своих
скриптах:

.. code-block:: ipython

    >>> import rstcheck
    >>> list(rstcheck.check('Example\n==='))
    [(2, '(INFO/1) Possible title underline, too short for the title.')]

Vim
~~~

Для вима используйте :github:`scrooloose/syntastic`:

.. code-block:: vim

    let g:syntastic_rst_checkers = ['rstcheck']

.. image:: /_static/2015/rstcheck_vim.png
   :align: center

Итог
----

Пишите документацию в целом и пишите больше в :l:`reStructuredText` он
прекрасен.
