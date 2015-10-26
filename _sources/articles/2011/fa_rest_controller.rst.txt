.. post:: 13 Nov, 2011 21:10
   :tags: CRUD, REST, Pylons, FormAlchemy, SQLalchemy
   :category: Pylons
   :author: Uralbash
   :language: ru

Pylons + FormAlchemy REST Controller
====================================

Для своих :l:`REST` контроллеров можно использовать, формы :l:`FormAlchemy`.

Создаем контроллер:

.. code-block:: bash

   /path/to/youproj$ paster restcontroller comment comments
   Creating yourproj/yourproj/controllers/comments.py
   Creating yourproj/yourproj/tests/functional/test_comments.py

Или если нужно в отдельной директории:

.. code-block:: bash

   /path/to/yourproj$ paster restcontroller admin/tracback admin/trackbacks
   Creating yourproj/controllers/admin
   Creating yourproj/yourproj/controllers/admin/trackbacks.py
   Creating yourproj/yourproj/tests/functional/test_admin_trackbacks.py

В файле нашего :l:`REST` контроллера добавим:

.. code-block:: python

   from formalchemy.ext.pylons.controller import RESTController

И в конце файла обернем его так:

.. code-block:: python

   # wrap with formalchemy RESTController
   CommentsController = RESTController(CommentsController, 'comment', 'comments')

Теперь если закомментировать какой-нибудь из стандартных методов в
контроллере (``index``, ``new``, ``update``, ``delete``, ``show``, ``edit``) он
будет браться из контроллера :l:`FormAlchemy` со стандартными формами. Довольно
удобно в разработке.
