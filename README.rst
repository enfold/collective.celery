.. image:: https://secure.travis-ci.org/collective/collective.celery.png?branch=master
    :target: http://travis-ci.org/collective/collective.celery
    :alt: Travis CI badge

.. image:: https://pypip.in/d/collective.celery/badge.png
    :target: https://pypi.python.org/pypi/collective.celery/
    :alt: Downloads

``collective.celery`` provides the necessary bits to use `Celery <http://celery.readthedocs.org/en/latest/>`_ within `Plone <http://plone.org/>`_.

Celery Integration
==================
The collective.celery package requires adding the celery and
collective.celery eggs to the main buildout section eggs. Example:

eggs =
    celery
    Plone
    collective.celery

The celery part is required, but is simpler:

[celery-broker]
host = 127.0.0.1
port = 6379

[celery]
recipe = zc.recipe.egg
environment-vars = ${buildout:environment-vars}
eggs =
    ${buildout:eggs}
    flower
scripts = pcelery flower

The celery part depends on having some variables added to the main
environment-vars section:

environment-vars =
    CELERY_BROKER_URL redis://${celery-broker:host}:${celery-broker:port}
    CELERY_RESULT_BACKEND redis://${celery-broker:host}:${celery-broker:port}
    CELERY_TASKS collective.es.index.tasks

Additional Zope configuration
-----------------------------

There's now a hook in collective.celery for carrying out additional zope
configuration before running the tasks. If the tasks module contains an
'extra_config' method, it is passed the zope startup object at worker
initialization time. 

Monitoring celery tasks
-----------------------

Celery needs to be started as an independent process. It's recommended to
use supervisord for this. To try it out from the command line, you can run
"bin/pcelery worker" from the buildout directory. Note that the script is
now named 'pcelery' and it needs a path to the zope configuration. Example:

$ bin/pcelery worker parts/client1/etc/zope.conf

Flower is included in this setup. Run "bin/flower" from the buildout
directory and consult the dashboard at http://localhost:5555 using a
browser. Note that the broker is now a requried parameter:

$ bin/flower --broker redis://127.0.0.1:6379

Supervisord Configuration
=========================
Below are some example supervisord configuration files which could be used.

Celery Worker
-------------

Celery workers perform the task runs.

[program:celery_worker]
command = {{ zope_buildout_dir }}/bin/pcelery worker parts/{{ desktop_instances[0]['name'] }}/etc/zope.conf
directory={{ zope_buildout_dir }}
startretries = 3
redirect_stderr = true
autostart = true
autorestart = true
priority = 101
user = {{ zope_user }}
stopasgroup = true
numprocs=1
startsecs=10
stopwaitsecs=600

Celery Beat
-----------

Celery beat provides a scheduling mechanism for celery if you want to use cron-like functionality.

[program:celery_beat]
command = {{ zope_buildout_dir }}/bin/celery beat --schedule {{ celery_dir }}/beat.db
directory={{ zope_buildout_dir }}
startretries = 3
redirect_stderr = true
autostart = true
autorestart = true
priority = 101
user = {{ zope_user }}
stopasgroup = true
numprocs=1
startsecs=10

Flower
------

Provides a web based tool for monitoring and administrating Celery clusters.

[program:flower]
command = {{ zope_buildout_dir }}/bin/flower --address={{ flower_address }} --port={{ flower_port }} --broker redis://127.0.0.1:6379
directory={{ zope_buildout_dir }}
startretries = 3
redirect_stderr = true
autostart = true
autorestart = true
priority = 201
user = {{ zope_user }}
stopasgroup = true
numprocs=1
startsecs=10
