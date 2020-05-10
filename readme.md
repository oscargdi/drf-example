# Django Rest Framework - Sample project

Rest API that returns the data sent in request. 

## Project setup

In order to create this project, you should perform the following steps:

### Install Python 3.7.x

You should install **Python 3.7.x** in your machine, to do so go to [download page](https://www.python.org/downloads/) and install any of the 3.7.x releases for your Operating System.

### Install VirtualEnv

VirtualEnv allows you to create isolated Python environments for the different projects you work in. Once Python is installed, use **pip** (package manager) to install **virtualenv** by executing the following:

In Mac OS:

```bash
$ pip3 install virtualenv
```

In Windows, there should not be any issue with different **pip** versions:
```bash
$ pip install virtualenv
```

> Mac OS brings Python 2 out of the box. This cause that using only **pip** command executes the Python 2 package manager. To avoid this, once Python 3 is installed you must use **pip3** command to manage the packages for this version of Python.


### Git repository initialization

```bash
$ git init # initialize git repository
$ touch .gitignore # create gitignore file
$ nano .gitignore # source: http://gitignore.io/api/linux,macos,django,python,pycharm,windows,virtualenv,visualstudiocode
$ git add .gitignore # add to stage
$ git commit -m "project init" # commit changes
$ git remote add origin https://github.com/oscargdi/drf-example.git # set remote
$ git push -u origin master # push to remote
```


### Create virtual environment
Once **virtualenv** is installed, within the project folder execute the command:

```bash
$ virtualenv .venv
```

It will create a folder called **.venv** that contains all the python packages and dependencies out of the box. It is mandatory to activate the virtual environment, to do so you should run:

In Mac OS (within project folder):
```bash
$ source .venv/bin/activate
```

In Windows (within project folder):
```bash
$ .venv\Scripts\activate
```

Right after you activate your virtual environment, you should see and indicator that it is active like this:

```bash
(.venv) $
```

>Once you are in the virtual environment, you no longer need to use pip3 (default pip within a virtual environment is python 3 version).

### Install python packages

Use the package manager **pip** to install project python packages.

```bash
# Install dependencies
(.venv) $ pip install djangorestframework 
# Gunicorn
(.venv) $ pip install "gunicorn[gevent]"
# Useful modules
(.venv) $ pip install pylint
(.venv) $ pip install autopep8
(.venv) $ pip install rope
# Freeze dependencies
(.venv) $ pip freeze > requirements.txt
```

Commit changes to repository
```bash
(.venv) $ git add .
(.venv) $ git commit -m "add requirements.txt"
(.venv) $ git push"
```

### Django Project Setup
```bash
$ django-admin startproject drf . # start project
$ django-admin startapp sample # start app
$ export SECRET_KEY="YOUR_SECRET_KEY" # setup SECRET_KEY in environment
$ ./manage.py migrate # run initial migrations
```

### Project Code

1. adding `rest_framework` and `sample` apps to `INSTALLED_APPS`:
```python
# drf/settings.py
INSTALLED_APPS = [
    ...
    'rest_framework',
    'sample.apps.SampleConfig',
    ...
]
```

2. Add production settings
```bash
# drf/settings.py

SECRET_KEY = os.environ['SECRET_KEY']

DEBUG = False

ALLOWED_HOSTS = ['drf.oscargdi.dev']
```

3. Creating view:
```python
# sample/views.py
from rest_framework.decorators import api_view
from rest_framework.response import Response


@api_view(['POST'])
def sample_view(request):
    return Response(request.data)
```
4. Adding url for the view above:
```python
# sample/urls.py
from django.urls import path

from . import views

urlpatterns = [
    path('', views.sample_view)
]
```
5. Adding `sample.urls` to project urls:
```python
# drf/urls.py
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('sample.urls')),
]
```
6. execute project:
```bash
$ ./manage.py runserver
```

### Pushing code to Github repository
```bash
$ git add . # adding changes to stage
$ git commit -m "adding django project with working endpoint" # committing changes
$ git push # push changes to remote
```

### Deploy AWS instance

1. Create AWS EC2 Ubuntu 18.04 Instance:

> Inbound rules: SSH (22), HTTP (80), HTTPS (443).
> Outbound rules: HTTP (80), HTTPS (443)

Connect to your instance through SSH with key pair.

2. Install software
```bash
$ sudo apt update && sudo apt upgrade -y
$ sudo apt install build-essential nginx supervisor virtualenv python3.7 python3.7-dev -y
```

3. Clone Github Repository
```bash
$ git clone https://github.com/oscargdi/drf-example.git
```

4. Create and activate Virtual Environment
```bash
$ cd drf-example
$ virtualenv -p python3.7 .venv
$ source .venv/bin/activate
```

5. Install Python dependencies
```bash
(.venv) $ pip install -r requirements.txt
```

6. Prepare supervisor to start application

Create configuration for app launch
```bash
(.venv) $ cd /var/log/
(.venv) $ mkdir gunicorn
(.venv) $ cd /etc/supervisor/conf.d/
(.venv) $ touch supervisor.conf
(.venv) $ sudo nano supervisor.conf
```
Set supervisor configuration:
```ini
[program:drf]
directory=/home/ubuntu/drf-example # project root directory
command=/home/ubuntu/drf-example/.venv/bin/gunicorn --workers 3 --worker-class gevent --timeout 30 --graceful-timeout 20 --bind unix:/home/ubuntu/drf-example/app.sock drf.wsgi:application # run project using gunicorn (exact configuration depends on hardware)
autostart=true # launch app on start up
autorestart=true # relaunch app if fails
stderr_logfile=/var/log/gunicorn/drf.err.log # logs
stdout_logfile=/var/log/gunicorn/drf.out.log # logs

[group:guni]
programs:drf # run program
```
Refresh supervisor to run new configuration
```bash
(.venv) $ sudo supervisorctl reread
(.venv) $ sudo supervisorctl update
(.venv) $ sudo supervisorctl reload
```
To check status of execution, run:
```bash
(.venv) $ sudo supervisorctl status
guni:drf                         RUNNING   pid 9231, uptime 0:28:44
```

7. Prepare nginx configuration:

Create configuration for web server
```bash
(.venv) $ cd /etc/nginx/sites-available/
(.venv) $ touch nginx.conf
(.venv) $ sudo nano nginx.conf
```
Set configuration for webserver
```ini
server {
        listen 80;
        server_name drf.oscargdi.dev;

        location / {
                include proxy_params;
                proxy_pass http://unix:/home/ubuntu/drf-example/app.sock;
        }
}
```
Enable site in web server
```bash
(.venv) $ cd ../sites-enabled/
(.venv) $ sudo ln -s ../sites-available/nginx.conf
```

Restart nginx
```bash
(.venv) $ sudo service nginx restart
```

8. Enable HTTPS

Follow CertBot installation guide in this [article](https://certbot.eff.org/lets-encrypt/ubuntubionic-nginx) to enable HTTPS in server.


## Test API

Open Terminal / Command Prompt and run:
```bash
curl -L -X POST 'https://drf.oscargdi.dev' -H 'Content-Type: application/json' --data-raw '{"application": "Django Rest Framework - Sample project", "github": "https://github.com/oscargdi/drf-example.git"}'
{"application":"Django Rest Framework - Sample project","github":"https://github.com/oscargdi/drf-example.git"}
```