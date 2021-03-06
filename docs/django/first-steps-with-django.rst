=========================
 First steps with Django
=========================

Using Celery with Django
========================

To use Celery with your Django project you must first define
an instance of the Celery library.

If you have a modern Django project layout like

    - proj/
      - proj/__init__.py
      - proj/settings.py
      - proj/urls.py
    - manage.py

then the recommended way is to create a new `proj/proj/celery.py` module
that defines the Celery instance:

:file: `proj/proj/celery.py`

.. code-block:: python

    from celery import Celery
    from django.conf import settings

    celery = Celery('proj.celery')
    celery.config_from_object(settings)
    celery.autodiscover_tasks(settings.INSTALLED_APPS, related_name='tasks')

    @celery.task(bind=True)
    def debug_task(self):
        print('Request: {0!r}'.format(self.request))

Let's explain what happens here.
First we create the Celery app instance:

.. code-block:: python

    celery = Celery('proj')

Then we add the Django settings module as a configuration source
for Celery.  This means that you don't have to use multiple
configuration files, and instead configure Celery directly
from the Django settings.

.. code-block:: python

    celery.config_from_object(settings)

Next, a common practice for reusable apps is to define all tasks
in a separate ``tasks.py`` module, and Celery does have a way to
autodiscover these modules:

.. code-block:: python

    celery.autodiscover_tasks(settings.INSTALLED_APPS, related_name='tasks')

With the line above Celery will automatically discover tasks in reusable
apps if you follow the ``tasks.py`` convention::

    - app1/
        - app1/tasks.py
        - app2/models.py
    - app2/
        - app2/tasks.py
        - app2/models.py

This way you do not have to manually add the individual modules
to the :setting:`CELERY_IMPORTS` setting.


Finally, the ``debug_task`` example is a task that dumps
its own request information.  This is using the new ``bind=True`` task option
introduced in Celery 3.1 to easily refer to the current task instance.


The `celery` command
--------------------

To use the :program:`celery` command with Django you need to
set up the :envvar:`DJANGO_SETTINGS_MODULE` environment variable:

.. code-block:: bash

    $ DJANGO_SETTINGS_MODULE='proj.settings' celery -A proj worker -l info

    $ DJANGO_SETTINGS_MODULE='proj.settings' celery -A proj status

If you find this inconvienient you can create a small wrapper script
alongside ``manage.py`` that automatically binds to your app, e.g. ``proj/celery.py`

:file:`proj/celery.py`

.. code-block:: python

    #!/usr/bin/env python
    import os

    from proj.celery import celery


    if __name__ == '__main__':
        os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'proj.celery')
        celery.start()

Then you can use this command directly:

.. code-block:: bash

    $ ./celery.py status


Using the Django ORM/Cache as a result backend.
-----------------------------------------------

The ``django-celery`` library defines result backends that
uses the Django ORM and Django Cache frameworks.

To use this with your project you need to follow these three steps:

    1. Install the ``django-celery`` library:

        .. code-block:: bash

            $ pip install django-celery

    2. Add ``djcelery`` to ``INSTALLED_APPS``.

    3. Create the celery database tables.

        If you are using south_ for schema migrations, you'll want to:

        .. code-block:: bash

            $ python manage.py migrate djcelery

        For those who are not using south, a normal ``syncdb`` will work:

        .. code-block:: bash

            $ python manage.py syncdb

.. _south: http://pypi.python.org/pypi/South/

.. admonition:: Relative Imports

    You have to be consistent in how you import the task module, e.g. if
    you have ``project.app`` in ``INSTALLED_APPS`` then you also
    need to import the tasks ``from project.app`` or else the names
    of the tasks will be different.

    See :ref:`task-naming-relative-imports`

Starting the worker process
===========================

In a production environment you will want to run the worker in the background
as a daemon - see :ref:`daemonizing` - but for testing and
development it is useful to be able to start a worker instance by using the
``celery worker`` manage command, much as you would use Django's runserver:

.. code-block:: bash

    $ DJANGO_SETTINGS_MODULE='proj.settings' celery -A proj worker -l info

For a complete listing of the command-line options available,
use the help command:

.. code-block:: bash

    $ celery help

Where to go from here
=====================

If you want to learn more you should continue to the
:ref:`Next Steps <next-steps>` tutorial, and after that you
can study the :ref:`User Guide <guide>`.
