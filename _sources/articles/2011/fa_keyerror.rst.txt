.. post:: 18 Dec, 2011 22:51
   :tags: python, FormAlchemy, WebOb
   :category: Python
   :author: Uralbash
   :language: ru

FormAlchemy KeyError: "Key not found: "
=======================================

Ошибка в :l:`FormAlchemy` типа ``KeyError: "Key not found:
u'Task--super_task_id'"`` лечится обновлением:

.. code-block:: bash

   $ pip install formalchemy --upgrade

При этом новая версия подтянет бетта версию :l:`webob` 1.2 с которым
:l:`pylons` 1 еще не работает из-за:

``DeprecationWarning: decode_param_names is deprecated and will not be supported starting with WebOb 1.2``

Откатимся:

.. code-block:: bash

   $ pip install webob==1.1.1

Ошибка должна исчезнуть, вот обсуждение `google groups
<http://groups.google.com/group/formalchemy/browse_thread/thread/19348cc5faae8711/3aaf49efffd9f160?lnk=gst&q=key+not+found#3aaf49efffd9f160>`_
