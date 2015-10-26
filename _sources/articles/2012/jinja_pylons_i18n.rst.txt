.. post:: Sep 27, 2012 20:07
   :tags: Python, Pylons, Jinja
   :category: Pylons
   :author: Uralbash
   :language: ru

Перевод шаблонов Jinja в Pylons
===============================

Все точно также как и с :l:`mako`, "Но есть нюансы"©

Добавьте в ``setup.py``:

.. code-block:: python

   package_data={'myproject': ['i18n/*/LC_MESSAGES/*.mo']},
   message_extractors={'myproject': [
           ('**.py', 'python', None),
           ('templates/**.html', 'jinja2', None),
           ('public/**', 'ignore', None)]},

Добавьте в ``lib/base.py``:

.. code-block:: python

   from pylons.i18n.translation import _, ungettext

И что то типа того в ``config/environment.py``:

.. code-block:: python

   # Create the Jinja2 Environment
   config['pylons.app_globals'].jinja2_env = Environment(loader=ChoiceLoader(
           [FileSystemLoader(path) for path in paths['templates']]),
           autoescape=True,
           extensions=['jinja2.ext.do', 'jinja2.ext.i18n'])
   config['pylons.app_globals'].jinja2_env.install_gettext_translations(pylons.i18n)
   # Jinja2's unable to request c's attributes without strict_c
   config['pylons.strict_c'] = True

Теперь можно переводить ``{{ _('Translate me!') }}``.
