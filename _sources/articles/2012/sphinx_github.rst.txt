.. post:: Apr 12, 2012 18:55
   :tags: Python, Sphinx, github, reStructuredText
   :category: Python
   :author: Uralbash
   :language: ru

Хостим документацию Sphinx на github
====================================

Немного допиленная версия этого
http://djangonaut.blogspot.com/2009/05/sphinx-documentation-github-pages-3.html

По порядку:

* создать репозитарий на :l:`github`
* добавить туда свой локальный репозитарий с проектом ``Sphinx``
* захостить репозитарий через сервис `Pages <https://pages.github.com/>`_
* поправить ``Makefile`` в ``Sphinx`` проекте

.. code-block:: make

  docs_dir = doc
  ghdocs:
   rm -rf $(docs_dir)
   $(MAKE) clean
   $(MAKE) html
   cp -r build/html $(docs_dir)
   mv $(docs_dir)/_static $(docs_dir)/static
   mv $(docs_dir)/_sources $(docs_dir)/sources
   perl -pi -e "s/_sources/sources/g;" $(docs_dir)/*.html
   perl -pi -e "s/_static/static/g;" $(docs_dir)/*.html
   git add .
   git commit -a -m "Updates $(project)."
   git checkout gh-pages
   cp -rf $(docs_dir)/* .
   git add .
   git commit -a -m 'Updates $(project) documentation.'
   git checkout master
   rm -rf $(docs_dir)
   git push origin gh-pages

Теперь при запуске ``make ghdocs`` будет автоматом генерироваться новая
документация и пушаться на :l:`github`.

Пример здесь http://uralbash.github.com/python-web-lectures/

Для автоматической замены директорий ``_static`` и ``_sources``, можно
установить модуль :pypi:`sphinxtogithub`.

.. code-block:: bash

   $ pip install sphinxtogithub

И добавить в проекте в ``config.py``:

.. code-block:: python
   :caption: config.py

   extensions = ["sphinxtogithub", ]

В этом случае нужно закомментировать в скрипте строчки:

``mv $(docs_dir)/_static $(docs_dir)/static``

и

``mv $(docs_dir)/_sources $(docs_dir)/sources``

Это необходимо делать потому-что :l:`github` не понимает директории начинающиеся
с подчеркивания.
