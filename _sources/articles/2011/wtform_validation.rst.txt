.. post:: Nov 27, 2011 22:37
   :tags: python, WTForm
   :category: Pylons
   :author: Uralbash
   :language: ru

WTForm валидация
================

Продолжение статьи :ref:`wtforms`. Рассмотрим как создать свой
класс для валидации. Здесь можно найти стандартные валидаторы
http://wtforms.simplecodes.com/docs/0.6/validators.html

Добавим файл ``equipments.py`` в папку ``validators``:

.. code-block:: bash

   |~forms/
   | |~mycontrollers/
   | | |-__init__.py
   | | `-equipments.py
   | |~validators/
   | | |-__init__.py
   | | `-equipments.py
   | `-__init__.py

Напишем валидатор IP адресов, вообще в :l:`WTForm` есть класс
:class:`wtforms.validators.IPAddress`, но он работает только с IPv4 адресами
(на момент написания статьи).

.. code-block:: python

   import ipaddr
   from wtforms.validators import ValidationError

   class IPNetwork(object):
       """
       Validates an IP(v4 and v6) address.

       :param message:
           Error message to raise in case of a validation error.
       """
       def __init__(self, message=None):
           self.message = message

       def __call__(self, form, field):
           data = field.data
           # Use PEP3144 for validation
           if not ipaddr.IPNetwork(data) and data:
               self.message = field.gettext(u'Invalid IP address.')
               raise ValidationError(self.message)

Сначала получаем значение поля, а потом при помощи модуля `ipaddr
<https://pypi.python.org/pypi/ipaddr>`_ проверяем. Если проверка не проходит то
генерируем исключение ``ValidationError``. Теперь напишем похожий валидатор для
MAC адреса.

.. code-block:: python

   ...
   from wtforms.validators import Regexp
   ...
   class MACAddress(Regexp):
       """
       Validates an MAC address.

       :param message:
           Error message to raise in case of a validation error.
       """
       def __init__(self, message=None):
           super(MACAddress, self).__init__(r'^([a-f\d]{1,2}\:){5}[a-f\d]{1,2}$',\
                                           message=message)

       def __call__(self, form, field):
           if self.message is None:
               self.message = field.gettext(u'Invalid MAC address.')

           if field.data:
               super(MACAddress, self).__call__(form, field)

Здесь мы использовали встроенный класс :class:`~wtforms.validators.Regexp`. В
файле формы это используется так:

.. code-block:: python

   ...
   from myapp.forms.validators.mycontroller import IPNetwork, MACAddress
   ...
   class EditForm(Form):

       ip = TextField('IP address', [IPNetwork()])
       mac = TextField('MAC address', [MACAddress()])

.. figure:: /_static/2011/wtforms_validation.png
   :align: center

   WTForm валидация
