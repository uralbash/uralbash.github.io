.. post:: 22 Nov, 2011 02:51
   :tags: Jinja2, WTForm, Pylons
   :category: Pylons
   :author: Uralbash
   :language: ru

.. _wtforms:

готовим Pylons + WTForms
========================

:l:`WTForm` простая, но довольно удобная библиотека для создания форм. И еще
:l:`WTForm` очень похожа на формы в :l:`Django` - одно из немногого что в
джанге сделано хорошо. Посмотрим как это работает с :l:`Pylons`. Для удобства
будем хранить формы отдельно:

::

   |+config/
   |+controllers/
   |~forms/
   | |~mycontroller/
   | | |-__init__.py
   | | `-equipments.py
   | |+validators/
   | `-__init__.py

Создаем форму для редактирования оборудования ``equipments.py``:

.. code-block:: python
   :caption: equipments.py

   from myapp.model.meta import Session as s
   from myapp.model.mymodel import EquipmentType
   from wtforms import Form, TextField, validators
   from wtforms.ext.sqlalchemy.fields import QuerySelectField

   # Выбор всех разновидностей оборудования для списка type в форме
   def all_equipment_types():
       return s.query(EquipmentType).all()

   class EditForm(Form):
       ip = TextField('ip address')
       netmask = TextField('network mask')
       mac = TextField('mac address')
       type = QuerySelectField('type of equipment',
                               query_factory=all_equipment_types)

``QuerySelectField`` это поле из расширения :l:`WTForm` для :l:`SQLAlchemy`,
которое позволяет создавать списки на основе выборок. Также есть расширения для
:l:`Django` и GAE. Инициализируем нашу форму в контроллере и передадим шаблону:

.. code-block:: python

   from myapp.forms.mycontroller.equipments import EditForm

   class EquipmentsController(BaseController):
       ...
       def edit(self, id, format='html'):
           """GET /myapp/equipments/id/edit: Form to edit an existing item"""
           # url('myapp_edit_equipment', id=ID)
           c.equipment = s.query(Equipment).filter(id==id)
           c.form = EditForm(ip=c.equipment.ip, mac=c.equipment.mac,
                           type=c.equipment.equipmenttype,
                           netmask=c.equipment.netmask)

           return render('/myapp/equipments/edit.html')

После передачи одноименных полям параметров в форму, значения этих параметров
будут по умолчанию выведены в форме. Напишем шаблон на :l:`Jinja`:

.. code-block:: html+jinja

   {% extends "base.html" %}

   {% block content %}
      <form action="{{ url(controller='myapp/equipments', action='update', id=c.equipment.id) }}" method="POST">
         <table class="simpletable">
            <caption>
               Edit equipment of {{ c.equipment.equipmenttype }}
                ({{ c.equipment.ip }})
            </caption>
            <tbody>
               {% for field in c.form.data %}
                  <tr><td>{{ c.form[field].label }}</td>
                      <td>{{ c.form[field] }}</td>
                  </tr>
               {% endfor %}
               <tr><td colspan="5">
               <button><img src="/img/common/save.png" />Save</button>
               <input type="hidden" name="_method" value="PUT" />
               </td></tr>
            </tbody>
         </table>
      </form>
   {% endblock %}

В примере поля выводятся в цикле из списка ``form.data``. Я использую :l:`REST`
контроллеры поэтому обновление происходит в методе ``update``:

.. code-block:: python

   from myapp.forms.mycontroller.equipments import EditForm

   class EquipmentsController(BaseController):
       ...
       def update(self, id):
           """PUT /myapp/equipments/id: Update an existing item"""
           # Forms posted to this method should contain a hidden field:
           #    <input name="_method" value="PUT" type="hidden">
           # Or using helpers:
           #    h.form(url('myapp_equipment', id=ID),
           #           method='put')
           # url('myapp_equipment', id=ID)
           equipment = s.query(Equipment).filter(id=id)
           if request.POST['ip']:
               equipment.ip = request.POST['ip']
           if request.POST['netmask']:
               equipment.netmask = request.POST['netmask']
           if request.POST['mac']:
               equipment.mac = request.POST['mac']
           type = EquipmentType.by_id(request.POST['type'])
           equipment.equipmenttype = type
           s.commit()

           came_from = str(request.environ.get('HTTP_REFERER', '')) or url('/')

           redirect(came_from)

Последние две строки возвращают пользователя обратно на страницу
редактирования. Выглядит это как-то так:

.. figure:: /_static/2011/wtform_eq.png
   :align: center

   Пример :l:`WTForm` и :l:`Pylons`
