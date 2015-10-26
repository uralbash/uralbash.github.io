.. post:: 28 Feb, 2013 22:49
   :tags: python, Jinja
   :category: python
   :author: Uralbash
   :language: ru

Объявление флага в цикле Jinja2
===============================

Понадобилось мне тут создать флаг в цикле, который можно использовать где
нибудь потом в шаблоне. По логике все должно выглядеть примерно так:

.. code-block:: jinja

   {% set exists = 0 %}
   {% for i in range(5) %}
         {% if True %}
             {% set exists = 1 %}
         {% endif %}
   {% endfor %}
   {% if exists %}
       <!-- exists is true -->
   {% endif %}

Но такой код не фурычит! ``exist`` всегда будет ``0``. Это особенность области
видимости переменных в :l:`Jinja` при присваивании.

Поэтому есть небольшой хак как это поправить:

.. code-block:: jinja

   {% set exists = [] %}
   {% for i in range(5) %}
         {% if True %}
             {% do exists.append(1) %}
         {% endif %}
   {% endfor %}
   {% if exists %}
       <!-- exists is true -->
   {% endif %}

Решение взято от `сюда
<http://stackoverflow.com/questions/4870346/can-a-jinja-variables-scope-extend-beyond-in-an-inner-block>`_.
