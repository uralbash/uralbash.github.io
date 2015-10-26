.. post:: 19 Feb, 2013 3:12
   :tags: python, Jinja
   :category: python
   :author: Uralbash
   :language: ru

Jinja2 Lorem ipsum dolor sit amet
=================================

Иногда в шаблоне нужно зафигачить какую-нибудь рыбу типа ``"Lorem ipsum dolor
sit amet"``, часто в цикле итд. Для этого существует функция `lipsum()
<http://jinja.pocoo.org/docs/dev/templates/#lipsum>`_.

Вот пример: 

.. code-block:: html+jinja

   {% for x in range(5) %}
     {{ lipsum()|truncate(150)|safe }}
     <hr>
   {% endfor %}


И результат:

.. raw:: html

   Justo aliquam faucibus lacus pulvinar commodo nisl, quisque est fusce venenatis mattis magnis arcu, hac felis parturient suspendisse a. Vitae ...
   <hr>

   Porta tellus turpis leo suspendisse rutrum metus blandit, montes dis lacinia felis non, vehicula vivamus condimentum luctus massa, vehicula ...
   <hr>

   Convallis molestie blandit viverra imperdiet eros dolor nam, ridiculus tortor duis blandit duis enim, cursus bibendum lobortis faucibus dui ...
   <hr>

   Diam placerat risus porta litora consequat vel, accumsan tempus ligula laoreet a mollis rutrum, aptent est tortor pulvinar senectus, litora etiam. ...
   <hr>

   Cursus non morbi non proin, porttitor consequat eget ac eget netus penatibus, egestas primis suspendisse ad, iaculis eu cursus urna blandit, leo. ...
   <hr>
