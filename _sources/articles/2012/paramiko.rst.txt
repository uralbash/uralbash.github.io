.. post:: Feb 10, 2012 12:32
   :tags: python, ssh, paramiko
   :category: Python
   :author: Uralbash
   :language: ru

Python + ssh или основы :pypi:`paramiko`
========================================

Для работы с ssh в питоновских скриптах идеально подходит модуль
:pypi:`paramiko`.

.. code-block:: bash

   $ pip install paramiko

В сети много примеров как подключиться при помощи пароля, я приведу пример как
подключаться при помощи ключа.

.. code-block:: python

   import paramiko
   from cStringIO import StringIO

   ssh = paramiko.SSHClient()
   # Убираем логи
   # в случае если вы используете их, например в Pylons окружении
   paramiko.util.logging.disable(ssh)
   # Подтверждаем ключи от хостов автоматически
   ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
   # Наш ключ для доступа без пароля
   key = StringIO('''-----BEGIN RSA PRIVATE KEY-----\n
   DIxdStYlzBCzySh60ewXurqUnv7KdPLzxXBeBN8LEpbekMkdXdhNBskT4JtDV3N8
   /43yEoNtjAN6iCXhcsCJNNkNHqMvI5jeiv64oZA/LgG3JLjiQNvG5IujxY1B8fNI
   bla bla bla...
   DIxdStYlzBCzySh60ewXurqUnv7KdPLzxXBeBN8LEpbekMkdXdhNBskT4JtDV3N8
   8GqlxiXnUhwQBk12G3QbyN6sxqKlH4wVpyN3lbV4LWFD9rj7Xr6avrAPxsIO1LF4
   W4IpuJT9+ABPCmxqWvVzj/YTTluHsvgGaDc+VhHSYmT1ti8plA==\n
   -----END RSA PRIVATE KEY-----\n''')

   # если нужно читать ключ из файла (например из ~/.ssh/mykey)
   # используйте функцию "from_private_key_file"
   privkey = paramiko.RSAKey.from_private_key(key)

   # Соединяемся
   # SSH connect cli like:
   # ssh -o StrictHostKeyChecking=no -i /xxx/.ssh/mykey <user>@<ip>
   ssh.connect('192.168.0.113', username='admin', pkey=privkey)
   # Выполняем команду
   stdin, stdout, stderr = ssh.exec_command('ls -la')
   # проверяем на ошибки
   if not stderr.readlines():
       # считываем вывод
       res = stdout.readlines()
