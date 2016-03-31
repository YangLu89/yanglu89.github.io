---
layout: post
title:  "Deploy Django Project to Openshift"
date:   2016-03-29 14:09:21
author: Yang
---
OpenShift is Red Hat's Platform-as-a-Service (PaaS) that allows developers to quickly develop, host, and scale applications in a cloud environment. I recently went througth the deployment process of a Django project on Openshift and would like to share the summarized the steps.

1.Set up an application on openshift using **python 2.7 cartridge**.

2.Clone your repository via git (you must first add the ssh key of your local machine to the openshift), and you will have a project directory like this

```
YourProject /
  wsgi.py
  setup.py
  .openshift /
```

3.Create a directory **wsgi/static** with an empty dummy file **.gitkeep** under the **YourProject folder**, copy your django project into the **YourProject** folder, and you have your project directory like this

```
YourProject /
  wsgi.py
  setup.py
  .openshift /
  wsgi /
    static/
      .gitkeep
  yourdjangoproject /
  static /
  manage.py
```

4.Include all the dependencies in the setup.py file

5.Edit the openshift-given WSGI script as follows.

```python
#!/usr/bin/python
import os
virtenv = os.environ['OPENSHIFT_PYTHON_DIR'] + '/virtenv/'
virtualenv = os.path.join(virtenv, 'bin/activate_this.py')
try:
    execfile(virtualenv, dict(__file__=virtualenv))
except IOError:
    pass

from yourdjangoproject.wsgi import application

```

6.Edit the hooks in **.openshift/action_hooks** to automatically perform db sincronization and media management, create two files named **build** and **deploy**.

build hook

```
#!/bin/bash
#this is .openshift/action/hooks/build
#remember to make it +x so openshift can run it.
if [ ! -d ${OPENSHIFT_DATA_DIR}media ]; then
    mkdir -p ${OPENSHIFT_DATA_DIR}media
fi
ln -snf ${OPENSHIFT_DATA_DIR}media $OPENSHIFT_REPO_DIR/wsgi/static/media
######################### end of file

```

deploy hook

```
#!/bin/bash
#this one is the deploy hook .openshift/action_hooks/deploy
source $OPENSHIFT_HOMEDIR/python/virtenv/bin/activate
cd $OPENSHIFT_REPO_DIR
echo "Executing 'python manage.py migrate'"
python manage.py migrate
echo "Executing 'python manage.py collectstatic --noinput'"
python manage.py collectstatic --noinput
########################### end of file

```

7.Edit the django setting.py file to tell the location of the static files

```python
STATIC_URL = '/static/'
MEDIA_URL = '/media/'
STATIC_ROOT = os.path.join(BASE_DIR, 'wsgi', 'static')
MEDIA_ROOT = os.path.join(BASE_DIR, 'wsgi', 'static', 'media')
= (os.path.join(BASE_DIR, 'static'),)
TEMPLATE_DIRS = (os.path.join(BASE_DIR, 'templates'),)
```

8.Add 'django.contrib.staticfiles' to the installed apps in Django settings.py file, and run manage.py collectstatic, then you’ll have the static files in wsgi/static folder, this is the folder where openshift looks for static files.

9.Finally push the changes to github, and your application should mount.

10.If you get errors you can always ssh to the openshift application instance and check the python-log under the app-root folder.
