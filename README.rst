The PyOhio 2015 website being built by Caktus Consulting Group, based on Pinax
Symposion.

Rather than use this as the basis for your conference site directly, you should
instead look at https://github.com/pinax/symposion which was designed for reuse.

PyOhio 2015
============

PyOhio 2015 is built on top of Pinax Symposion but may have customizations that
may make things more difficult for you.

To get running locally
----------------------

This documentation assume you have the following installed::

- `pip >= 1.2.1 <http://www.pip-installer.org/>`_
- `virtualenv >= 1.11 <http://www.virtualenv.org/>`_
- `virtualenvwrapper >= 3.6 <http://pypi.python.org/pypi/virtualenvwrapper>`_

Create a new virtualenv and install the necessary requirements::

    mkvirtualenv --distribute pyohio --python=python2.7
    $VIRTUAL_ENV/bin/pip install -r $PWD/requirements/dev.txt

(For production, install -r requirements/base.txt).

Then create a local settings file and set your ``DJANGO_SETTINGS_MODULE`` to use it::

    cp pyohio/settings/local.py.example pyohio/settings/local.py
    echo "export DJANGO_SETTINGS_MODULE=pyohio.settings.local" >> $VIRTUAL_ENV/bin/postactivate
    echo "unset DJANGO_SETTINGS_MODULE" >> $VIRTUAL_ENV/bin/postdeactivate

Exit the virtualenv and reactivate it to activate the settings just changed::

    deactivate
    workon pyohio

Setup the postgres database and load fixtures::

    createdb pyohio2015
    python manage.py syncdb
    python manage.py migrate
    python manage.py loaddata fixtures/*

Create a user account::

    python manage.py createsuperuser


Run local server::

    python manage.py runserver

For Gondor
--------------
Copy database instance from previous year locally::

    gondor sqldump primary > pyohio-<year>-db.sql

Copy instance variables::

    gondor env:get

Copy site media (user images, sponsor logos, etc.)::

    scp -r <old-instance-id>@ssh.gondor.io:site_media/media /tmp/media

Update site to new year::

    vi gondor.yml

Create new instances for the new site::

    gondor create --kind=production primary
    gondor create --kind=dev dev

Deploy database dump to new year instance::

    gondor manage dev database:load pyohio-<year>-db.sql 
    gondor manage primary database:load pyohio-<year>-db.sql 

Set instance variables to new year instance::

    gondor end:set dev SITE_ID=2
    ...
    gondor env:set primary SITE_ID=3
    ...

Deploy site media::

    scp -r /tmp/media/* <new-instance-id>@ssh.gondor.io:site_media/media

Deploy code to new year instance::

    gondor deploy <primary|dev> <HEAD|master|git commit id>

To run tests
------------

::

    python manage.py test pyhio
