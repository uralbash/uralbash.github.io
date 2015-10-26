.. post:: 10 Mar, 2014 14:34
   :tags: Python, Pyramid, CRUD, sacrud
   :category: Pyramid
   :author: Uralbash
   :language: ru

Обновление sacrud. Версия 0.1.2
===============================

.. image:: /_static/2014/sacrud_0_1_2.png
   :align: left

| Код: https://github.com/uralbash/sacrud
| Описание: http://sacrud.readthedocs.org/

В этой версии делался упор на кастомизацию интерфейса.

Что нового?

* добавлена пагинация
* теперь pk показывается по умолчанию в форме создания/редактирования
* новая опция ``sacrud_detail_col`` где можно задать отображаемые поля в форме редактирования
* новая опция ``sacrud_list_col`` где можно задать отображаемые поля в списке записей
* новая опция ``verbose_name`` для полей и таблиц
* опция ``sacrud_css_class``, назначает ``CSS`` стили полям
* новый атрибут колонки ``sacrud_position``: "inline" (см. реализацию ``horizontal_fields``)
* новая функция ``horizontal_fields``
* новый тип ``exttype.GUID``
* для переопределения ``base.html`` создан шаблон ``redefineme.html``
* вывод флеш уведомлений если задана ``sesion_factory``
* исправлены названия классов в шаблонах у полей
* исправлен шаблон ``ForeignKey.jinja2``
* ``sa_create``, ``sa_read``, ``sa_update``, ``sa_delete`` view вынесены в общий класс ``CRUD``
* в example добавлены :pypi:`pyramid_beaker`, примеры с кастомизацией, FileField и все
  остальные поля которые есть в sacrud/templates/sacrud/types.

Пример кастомизации:

.. code-block:: python
   :caption: models.py

   class TestCustomizing(Base):
       __tablename__ = "test_customizing"

       id = Column(Integer, primary_key=True)
       name = Column(String)
       date = Column(Date, info={"verbose_name": 'date JQuery-ui'})
       name_ru = Column(String, info={"verbose_name": u'Название', })
       name_fr = Column(String, info={"verbose_name": u'nom', })
       name_bg = Column(String, info={"verbose_name": u'Име', })
       name_cze = Column(String, info={"verbose_name": u'název', })
       description = Column(Text)
       description2 = Column(Text)

       visible = Column(Boolean)
       in_menu = Column(Boolean, info={"verbose_name": u'menu?', })
       in_banner = Column(Boolean, info={"verbose_name": u'on banner?', })

       # SACRUD
       verbose_name = u'Customizing table'
       sacrud_css_class = {'tinymce': [description, description2],
                           'content': [description],
                           'name': [name], 'Date': [date]}
       sacrud_list_col = [name, name_ru, name_cze]
       sacrud_detail_col = [name,
                            hosrizontal_field(name_ru, name_bg, name_fr, name_cze,
                                              sacrud_name=u"i18n names"),
                            description, date,
                            hosrizontal_field(in_menu, visible, in_banner,
                                              sacrud_name=u"Расположение"),
                            description2]

.. code-block:: html+jinja
   :caption: templates/sacrud/redefineme.jinja2

   {% extends "sacrud/base.jinja2" %}

   {% block userspace %}
     {{ super() }}

     <!-- Date field -->
     <script src="http://code.jquery.com/jquery-1.10.2.js"></script>
     <script src="http://code.jquery.com/ui/1.10.4/jquery-ui.js"></script>

     <link href="//code.jquery.com/ui/1.10.4/themes/smoothness/jquery-ui.css" rel="stylesheet"></link>
     <style>
       .content {
         height: 550px;
       }
       .name {
         width: 400px;
       }
     </style>

     <script>
       $(function() {
         $(".Date").datepicker({ dateFormat: 'yy-mm-dd' });
       });
     </script>

     <script src="//tinymce.cachefly.net/4.0/tinymce.min.js"></script>
     <script>
       tinymce.init({
         selector:'textarea.tinymce',
         plugins: "image link",
         file_browser_callback: function(field_name, url, type, win) {
           tinymce.activeEditor.windowManager.open({
               title: "SACRUD file browser",
               url: "/image/filebrowser",
               width: 600,
               height: 400,
             }, {
               oninsert: function(url) {
                 win.document.getElementById(field_name).value = url;
               }
           });
         },
       });
     </script>
   {% endblock %}

   {% block sa_body %}
     {{ super() }}
   {% endblock %}

Результат:

.. figure:: /_static/2014/sacrud_0.1.2_example.png
   :align: center

   Пример кастомного CRUD интерфейса для SQLAlchemy

Демо приложение: https://github.com/uralbash/pyramid_sacrud_example
