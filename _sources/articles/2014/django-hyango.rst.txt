.. post:: 2 Feb, 2014 20:43
   :tags: Python, Django, trololo, ненависть, ненормальное, программирование
   :category: Django
   :author: Uralbash
   :language: ru

Django - кот в мешке.
=====================

.. image:: /_static/2014/wat.jpg
   :align: right
   :width: 420px

:l:`Django` это фреймворк, при помощи которого быстро пишутся сайты. Так ли это?

Фичи джанги:

* Админка
* куча аппликэйшинов
* Шаблоны
* ORM
* DebugToolbar

.. raw:: html

   <br clear="both"/>

Админка
-------

Из коробки у вас есть админка. Для простых сайтов её нужно кастомайзить долго и
упорно, правда, обычно, это делается один раз. В сайте среднего уровня
жангоадминка уже не прокатит. Вывод админка даёт профит в простых сайтах.

Аппликашины
-----------

Обычно очень посредственного качества и не совместимы друг с другом, поэтому
танцы с бубном и велосипеды наше всё + ``djangosnippets``.

Шаблоны
-------

* нету break, continue
  ``jinja2.ext.loopcontrols This extension adds support for break and continue in loops.``

* рекурсия Jinja

  .. code-block:: html+jinja

   <ul class="sitemap">
   {% for item in sitemap recursive %}
       <li><a href="{{ item.href|e }}">{{ item.title }}</a>
       {% if item.children -%}
           <ul class="submenu">{{ loop(item.children) }}</ul>
       {% endif %}</li>
   {% endfor %}
   </ul>

  а в джанге это видимо не нужно

* не работает вызов функций типа ``item()`` (лолшто?)
* ``[:-1]`` не работает, нужно писать ``|latest``
* :strike:`для js выражение типа:`

  :strike:`var foo = [{% for item in items %}{{ item.paren }}{% endfor %}]`
  :strike:`не работает, потому что js выполняется до рендеринга таких {%%} конструкций.`

* коменты ``{# #}`` неработают в несколько строк
* вместо ``{{ list[5] }}`` -> ``{{ arr.5 }}``
* для словарей ``{{ dict.foo }}`` не работает, да здравствует ``template tag``
* для :mod:`itertools.chain` можно сосать лапу с таким фокусом ``{{ list[5] }}``
* о джанга ты прекрасна ``{{ myval|add:"-5" }}`` вместо уродского ``{{ myval - 5 }}``
* нельзя сделать так ``{{ dir(foo) }}``
* как узнать длину :mod:`itertools`? в :l:`jinja` делается так ``{{ list(foo).length }}``

ORM
---

* ``Page.objects.filter(parent__in=objects)`` вместо ``DBSession.query(Page).filter(parent in objects)``
* великолепный ``foomodel_set__barmodel__name`` лолшто?!
* ``Entry.objects.all()[:1].get()`` вместо ``DBSession.query(Entry).first()`` или ``one()``
* добавляем в модель: ``class Meta: app_label = 'asdasda'`` и переходим к разделу дебаг!

Debug
-----

.. raw:: html

   <style>
    .red {
       color: red;
       font-weight: bold;
    }
    .red-white {
       background-color: red;
       font-weight: bold;
       color: white;
   </style>

.. role:: red-white
    :class: red-white

.. role:: red
   :class: red

:red-white:`Дебаг джанго кормит вас!` Потому что вы :l:`Django` программист и
половину рабочего времени пытаетесь понять что за !@#$% произошла, получая за
это зарплату.

``class Meta: app_label = 'asdasda'`` генерит трейс:

:red:`django.core.management.base.CommandError: One or more models did not validate:
gallery.galleryimage: 'gallery' has a relation with model <class
'gallery.models.Gallery'>, which has either not been installed or is abstract.`

Шыкарные красные ошибки джанги. Очень информативно, ищется аникейством и
вспоминанием чё правил.

Я запилил `небольшой пример <https://github.com/uralbash/django-hyango>`_ что
бы вы могли поднять его и посмотреть на эту магию.

Квик старт
----------


1. Установка по ``pip install -r requirements.txt``
2. Далее введим ``python manage.py syncdb`` и видим :red:`ImportError: cannot import name SEOModel` Супер информативный вывод, все сразу стало понятно.

    2.1 Прошел все модели в проекте, ``SEOModel`` не нашел, при учете что у меня ``gallery`` стоит в :pypi:`virtualenv`

    2.2 может во вью? Во вью то же нету

    2.3 ищем по проекту тупо по всем файлам, нету.

    2.4 идем в ``settings/apps.py`` и смотрим какое г может это содержать (так на угад), смотрим ``common`` опа вот он в моделях, осталось найти где он импортируется неправильно.

    2.5 ради интереса запускаю сервер ``python manage.py runserver`` и получаю:

       :red:`File ".../local/lib/python2.7/site-packages/gallery/models.py", line 12, in <module> from website.models import SEOModel, VisibleModel ImportError: cannot import name SEOModel`

т.е. в некоторых случаях джангу надо дебажить runserver'ом

Pages
-----

* захожу в pages в админке
* добавляю дерево
* тыкаю по дереву (вложенность 2го уровня) что бы развернуть список и получаю:

``Error while loading the data from the server.``

.. image:: /_static/2014/pages_tree.png

и трэйс:

.. code-block:: bash

   Internal Server Error: /admin/pages/page/tree_json/
   Traceback (most recent call last):
     File "/home/uralbash/.virtualenvs/django-hyango/local/lib/python2.7/site-packages/django/core/handlers/base.py", line 115, in get_response
       response = callback(request, *callback_args, **callback_kwargs)
     File "/home/uralbash/.virtualenvs/django-hyango/local/lib/python2.7/site-packages/django_mptt_admin/admin.py", line 50, in wrapper
       return self.admin_site.admin_view(view)(*args, **kwargs)
     File "/home/uralbash/.virtualenvs/django-hyango/local/lib/python2.7/site-packages/django/utils/decorators.py", line 91, in _wrapped_view
       response = view_func(request, *args, **kwargs)
     File "/home/uralbash/.virtualenvs/django-hyango/local/lib/python2.7/site-packages/django/views/decorators/cache.py", line 89, in _wrapped_view_func
       response = view_func(request, *args, **kwargs)
     File "/home/uralbash/.virtualenvs/django-hyango/local/lib/python2.7/site-packages/django/contrib/admin/sites.py", line 202, in inner
       return view(request, *args, **kwargs)
     File "/home/uralbash/.virtualenvs/django-hyango/local/lib/python2.7/site-packages/django_mptt_admin/admin.py", line 157, in tree_json_view
       node = self.model.objects.get(id=node_id)
     File "/home/uralbash/.virtualenvs/django-hyango/local/lib/python2.7/site-packages/django/db/models/manager.py", line 143, in get
       return self.get_query_set().get(*args, **kwargs)
     File "/home/uralbash/.virtualenvs/django-hyango/local/lib/python2.7/site-packages/django/db/models/query.py", line 404, in get
       self.model._meta.object_name)
   DoesNotExist: Page matching query does not exist.
   [16/Dec/2013 23:42:28] "GET /admin/pages/page/tree_json/?node=5&_=1387215746550 HTTP/1.1" 500 22668

| бесполезный трейс и непонятная ошибка КРУТО!
| Если я зайду на адрес ``/admin/pages/page/tree_json/?node=5&_=1387215746550``
то получу статический trace и хер знает куда bp вставить что бы хоть за че-то
зацепиться.

Ставлю :pypi:`django-extensions` через :pypi:`pip` и :pypi:`Werkzeug` что бы
получить хоть какой то интерактив на том же трэйсе.

Запускаю командой ``python manage.py runserver_plus``, отваливается
``debug_toolbar``. Но и интерактивный трейс мало чем помогает в этом случае.
Джанга каким то чудом обходит то место которое все ломает, ГРЕБАННЫЙ СТЫД!
Ошибка ищется тупым эникейством. Предполагая что :l:`django` идеальна и в ней
нет ошибок, :pypi:`django_mptt_admin` на ``example`` работает хорошо, может
быть что то в Pages??? Ад же какойто?
