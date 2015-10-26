.. post:: Feb 10, 2012 13:17
   :tags: python, Jinja2
   :category: Python
   :author: Uralbash
   :language: ru

Jinja замена ``None``, ``Null``, итд на пустую строку
=====================================================

В питоне пустые значения возвращаются как ``None``. Поэтому в шаблонах
:l:`Jinja` вместо пустых значений отображаются ``None``. Что бы поправить это
нужно изменить метод ``finalize``. Пример из google groups:

.. code-block:: python

   def silent_none(value):
        if value is None:
            return ''
        return value

   from jinja2 import Environment
   env = Environment()
   env.finalize = silent_none

Теперь вместо None будет писаться пустая строка ''. В pylons нужно править файл
environment.py:

.. code-block:: python
   :caption: environment.py

   def silent_none(value):
       """ Jinja fix output None
       For more details:
       http://groups.google.com/group/pocoo-libs/browse_thread/thread/490f6e6e8fca6a6c
       """
       if value is None:
           return ''
       return value

   def load_environment(global_conf, app_conf):
       """Configure the Pylons environment via the ``pylons.config``
       object
       """
       bla bla bla...
       # Create the Jinja2 Environment
       config['pylons.app_globals'].jinja2_env = Environment(loader=ChoiceLoader(
               [FileSystemLoader(path) for path in paths['templates']]))
       # replace None output to ''
       config['pylons.app_globals'].jinja2_env.finalize = silent_none
       # Jinja2's unable to request c's attributes without strict_c
       config['pylons.strict_c'] = True
       bla bla bla...
