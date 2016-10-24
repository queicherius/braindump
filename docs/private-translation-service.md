# Private translation service

This sets up a private service for translation, using [Pootle](http://pootle.translatehouse.org/). This assumes a pre-configured Ubuntu 16 system, e.g. by following [this guide](./configure-a-ubuntu-16-server.md)

TODO use pootle 2.6 instead of the garbage that is 2.8

### Install requirements

Install the whole bunch of requirements for the application to run in a production environment.

```bash
# Build tools
apt-get install build-essential libxml2-dev libxslt-dev python-dev python-pip zlib1g-dev
pip -V

# Redis
apt-get install redis-server
service redis-server status

# Node.JS
curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
sudo apt-get install -y nodejs
node -v

# MySQL
apt-get install mysql-server libmysqlclient-dev
sudo /usr/bin/mysql_secure_installation # "yes" to all questions
service mysql status
mysql -u root -p
# -> run in mysql
CREATE DATABASE pootle CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL PRIVILEGES ON pootle.* TO pootle@localhost IDENTIFIED BY 'secretpassword';
FLUSH PRIVILEGES;
exit
# -> exit mysql


# Nginx
apt-get install nginx
service nginx status
```

### Setup the virtual environment

Setup the virtual environment where you want to install Pootle to.

```bash
apt-get install python-virtualenv
mkdir -p ~/pootle
cd ~/pootle
virtualenv env
```

### Install Pootle

From this stop on, we are working with the virtual environment, which you can activate with the first command below.

```bash
source env/bin/activate

# Make sure that pip is updated
pip install --upgrade pip

# Install the DB bindings
pip install MySQL-python

# Install Pootle
pip install --pre Pootle
pootle --version
```

### Configure Pootle

(Inside of the virtual environment)

```bash
pootle init
```

**`~/.pootle/pootle.conf`**

```
# Site title
POOTLE_TITLE = 'My translation server'

# ...

# Allowed hosts
ALLOWED_HOSTS = [
  '*'
]

# ...

# Database backend settings
DATABASES = {
    'default': {
        'ENGINE': 'transaction_hooks.backends.mysql',
        'NAME': 'pootle',
        'USER': 'pootle',
        'PASSWORD': 'secretpassword',
        'HOST': '',
        'PORT': '',
        'ATOMIC_REQUESTS': True
    }
}

# Cache Backend settings
CACHES = {
    'default': {
        'BACKEND': 'django_redis.cache.RedisCache',
        'LOCATION': 'redis://127.0.0.1:6379/1',
        'TIMEOUT': 60,
    },
    'redis': {
        'BACKEND': 'django_redis.cache.RedisCache',
        'LOCATION': 'redis://127.0.0.1:6379/2',
        'TIMEOUT': None,
    },
    'stats': {
        'BACKEND': 'django_redis.cache.RedisCache',
        'LOCATION': 'redis://127.0.0.1:6379/3',
        'TIMEOUT': None,
    },
    'exports': {
        'BACKEND': 'django.core.cache.backends.filebased.FileBasedCache',
        'LOCATION': working_path('exports/'),
        'TIMEOUT': 259200
    },
}

### ...

# You should probably setup the email server too, but I'll skip that, 
# because I personally don't need it. Feel free to PR if you get it working!
```

#### Setup the backend worker

(Outside of the virtual environment)

*`~/pootle-worker.sh`*

```
#!/bin/bash
cd /root/pootle/
source env/bin/activate
pootle rqworker
```

```bash
chmod +x pootle-worker.sh
```

*`/etc/systemd/system/pootle-worker.service`*

```
[Unit]
Description=Pootle RQ Worker
After=network.target

[Service]
ExecStart=/root/pootle-worker.sh
Restart=always
WorkingDirectory=/root/

[Install]
WantedBy=multi-user.target
```

```bash
systemctl daemon-reload
service pootle-worker start
```

#### Initialize the database

(In env)

```bash
# Migrate the db
pootle migrate
pootle initdb

# Create the admin user (e.g. "david") and very it
pootle createsuperuser
pootle verify_user david
```

#### Setup the proxy server

chown -R "$USER":www-data /root/pootle/env/lib/python2.7/site-packages/pootle/assets

chown -R www-data:www-data /root/pootle/env/lib/python2.7/site-packages/pootle/assets
chmod -R 0755 /root/pootle/env/lib/python2.7/site-packages/pootle/assets
chmod -R og+x /root // TODO not cool, install in other directory instead of /root/pootle

TODO nginx config + letsencrypt

server {
   listen  80;
   gzip on;
   charset utf-8;

   location /assets {
       alias /root/pootle/env/lib/python2.7/site-packages/pootle/assets;
       expires 14d;
       access_log off;
   }

   location / {
     proxy_pass         http://localhost:8000;
     proxy_redirect     off;

     proxy_set_header   Host             $host;
     proxy_set_header   X-Real-IP        $remote_addr;
     proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
   }
 }

#### Setup the backend server

TODO systemd script

#### Setup the firewall

```bash
ufw allow "Nginx HTTP"
ufw allow "Nginx HTTPS"
```