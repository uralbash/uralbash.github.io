.. post:: 16 Nov, 2011 00:42
   :tags: CIDR, INET, PostgreSQL, SQLalchemy
   :category: SQLalchemy
   :author: Uralbash
   :language: ru

SQLAlchemy почему PostgreSQL?
=============================

Потому что я могу делать так:

.. code-block:: python

   c.nets = s.query(Net)
   ip_type = request.GET.get('ip_type', '')
   if ip_type=='4':
       c.nets = s.query(Net).filter("family(cidr)=4")
   elif ip_type=='6':
       c.nets = s.query(Net).filter("family(cidr)=6")

   c.nets = c.nets.order_by(Net.cidr).all()

Здесь если GET переменная ``ip_type=4`` то выбираются все строки где ``cidr``
IPv4, если 6 то все стоки где ``cidr`` IPv6, иначе просто отдается все. Для
фильтрации используется стандартная функция :l:`Postgres` ``family``. Пример
был так расписан для наглядности, теперь сократим:

.. code-block:: python

   c.nets = s.query(Net)
   ip_type = request.GET.get('ip_type', '')
   if ip_type:
       c.nets = s.query(Net).filter("family(cidr)=%s" % ip_type)

   c.nets = c.nets.order_by(Net.cidr).all()

По моему это просто шЫкарно.

.. image:: /_static/2011/awesome.jpeg
