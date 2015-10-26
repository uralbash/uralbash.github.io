.. post:: Dec 04, 2012 15:27
   :tags: Python, Pyramid, Pylons
   :category: Pyramid
   :author: Uralbash
   :language: ru

Меняем заголовок ответа в Pyramid/Pylons
========================================

Обычно в заголовке находится что то типа:

.. code-block:: bash

   Content-Length 6657
   Content-Type text/html; charset=UTF-8
   итд

Изменить заголовок можно например в настройках :l:`Apache` или :l:`nginx` или в
самом приложении.

Для :l:`Pylons` пишем ``WSGI middleware``:

.. code-block:: python

   class HeaderMiddleware(object):
       '''WSGI middleware'''

       def __init__(self, application):

           self.app = application

       def __call__(self, environ, start_response):

           request = Request(environ)
           resp = request.get_response(self.app)
           resp.headers['X-Frame-Options'] = 'DENY'

           return resp(environ, start_response)

Для :l:`Pyramid` можно обойтись без ``WSGI``:

.. code-block:: python
   :caption: view.py

   from pyramid.events import subscriber
   from pyramid.events import NewRequest

   @subscriber(NewRequest)
   def add_header(event):
       response = event.request.response
       response.headers.add('X-Frame-Options', 'DENY')
