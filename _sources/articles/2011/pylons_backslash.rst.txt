.. post:: Oct 27, 2011 02:07
   :tags: Pylons, routes
   :category: Pylons
   :author: Uralbash
   :language: ru

Pylons + Routes фикс слэша в конце URL
======================================

Если мапить URL'ы вот так:

.. code-block:: python

   map.connect('/logs', controller='logs', action='logs')

То при попытке открыть URL ``/logs/`` вместо ``/logs`` появится страница 404.

Можно конечно делать так:

.. code-block:: python

   map.connect('/logs/', controller='logs', action='logs')
   map.connect('/logs', controller='logs', action='logs')

Но это жутко неудобно.

По совету `stackoverflow
<http://stackoverflow.com/questions/235191/trailing-slashes-in-pylons-routes>`_
можно обойти эту проблему простым редиректом:

.. code-block:: python

   map.redirect('/*(url)/', '/{url}',
                _redirect_code='301 Moved Permanently')

Теперь все запросы оканчивающиеся на слэш будут перенаправляться на адрес без слэша.
