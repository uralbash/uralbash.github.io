.. post:: 23 Feb, 2014 18:02
   :tags: Python, Pyramid, CRUD, sacrud
   :category: Pyramid
   :author: Uralbash
   :language: ru

Обновление sacrud. Версия 0.1.1
===============================

.. image:: /_static/2014/sacrud_0_1_1.png
   :align: left

| Код: https://github.com/uralbash/sacrud
| Описание: http://sacrud.readthedocs.org/

Что нового?

* в расширении для pyramid параметр "sacrud_models" переименован в "sacrud.models"
* "sacrud.models" теперь словарь, а не список:

   .. code-block:: python

      settings['sacrud.models'] = {
          'Company': [Company, User, EmployeeType],
          'Auth': [Group, GroupPermission, UserGroup,
          GroupResourcePermission, Resource, UserPermission,
          UserResourcePermission, ExternalIdentity],
          '': [Company]
      }

* в словаре можно поделить модели на группы
* тесты исправлены для :l:`Pyramid` ``1.5``:

  .. deprecated:: pyramid 1.5

      Removed the ability to influence and query a pyramid.request.Request object as
      if it were a dictionary. Previously it was possible to use methods like
      __getitem__, get, items, and other dictlike methods to access values in the
      WSGI environment. This behavior had been deprecated since Pyramid 1.1. Use
      methods of request.environ (a real dictionary) instead. 
