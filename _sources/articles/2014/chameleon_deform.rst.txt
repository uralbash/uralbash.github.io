.. post:: 19 Aug, 2014 19:29
   :tags: Python, Chameleon, Colander, deform
   :category: Python
   :author: Uralbash
   :language: ru

Chameleon, deform и маленькая хитрость
======================================

:pypi:`Deform` - это такая штука которая генерит формы, а шаблоны для виджетов
в нем написаны в формате Chameleon шаблонизатора.

Простой пример формы (http://deformdemo.xo7.de/nonrequiredfields/):

.. code-block:: python

   class Schema(colander.Schema):
       required = colander.SchemaNode(
           colander.String(),
           description='Required Field'
           )
       notrequired = colander.SchemaNode(
           colander.String(),
           missing=unicode(''),
           description='Unrequired Field')
   schema = Schema()
   form = deform.Form(schema, buttons=('submit',))
   return self.render_form(form)

.. image:: /_static/2014/chameleon_deform.png

К полям можно добавлять описание, но если вставить html то он
заэскапируется
(https://github.com/Pylons/deform/blob/bb4fc86913884deafa9350de86d87fb5232263fa/deform/templates/form.pt#L42).
Можно воспользоваться хитрой возможностью :pypi:`Chameleon`'а что бы вывести html.

.. code-block:: python

   class HTMLText(object):
       def __init__(self, text):
           self.text = text

       def __html__(self):
           return unicode(self.text)

   notrequired = colander.SchemaNode(
                   colander.String(),
                   missing=unicode(''),
                   description=HTMLText('Hello <hr color="red" /> World!!! <hr />'))

:pypi:`Chameleon` по умолчанию ищет метод ``__html__`` у объектов и если он
есть то выводит его результат в чистом виде. Вот.
