.. post:: Feb 10, 2012 12:32
   :tags: python, pylons, console
   :category: Pylons
   :author: Uralbash
   :language: ru

Скрипты работающие в окружении проекта на Pylons
=================================================

Иногда необходимо написать скрипт который выполняется из консоли и использует
окружение проекта на Pylons. Копипастю простой пример с `pylonshq
<http://pylonshq.com/snippets/running_commands_with_the_app_environment>`_.
Так-как там есть привычка периодически удалять информацию.

.. code-block:: python

   import optparse

   import pylons
   from paste.deploy import appconfig

   from YOURAPP.config.environment import load_environment


   if __name__ == '__main__':
       option_parser = optparse.OptionParser()
       option_parser.add_option('--ini',
           help='INI file to use for pylons settings',
           type='str',
           default='development.ini')
       options, args = option_parser.parse_args()

       # Initialize the Pylons app
       conf = appconfig('config:' + options.ini, relative_to='.')
       load_environment(conf.global_conf, conf.local_conf)

       # Now code can be run, the SQLalchemy Session can be used, etc.
       ....
