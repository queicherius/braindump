# Setting up common applications

These are tiny guides to install and configure programs that are not installed by default. This assumes a pre-configured Ubuntu 16 system, e.g. by following [this guide](./configure-a-ubuntu-16-server.md) 

---

### Node.js

```bash
curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
sudo apt-get install -y nodejs
```

> **Note**: Confirm with `node -v` that it actually installed the right version (6.x). If the installer fails it may install the version of the official repository (which is 4.x).

---

### Redis

> Basically just a short version of [this guide](http://shokunin.co/blog/2014/11/11/operational_redis.html).

```bash
apt-get install redis-server
```

**`/etc/sysctl.conf`**

```conf
vm.swappiness=0                       # turn off swapping
net.ipv4.tcp_sack=1                   # enable selective acknowledgements
net.ipv4.tcp_timestamps=1             # needed for selective acknowledgements
net.ipv4.tcp_window_scaling=1         # scale the network window
net.ipv4.tcp_congestion_control=cubic # better congestion algorythm
net.ipv4.tcp_syncookies=1             # enable syn cookied
net.ipv4.tcp_tw_recycle=1             # recycle sockets quickly
net.ipv4.tcp_max_syn_backlog=65536    # backlog setting
net.core.somaxconn=65536              # up the number of connections per port
```

**`/etc/redis/redis.conf`**

```conf
bind 0.0.0.0
requirepass <password>
tcp-backlog 65536
maxclients 10000
maxmemory <memory>
```

**Allow access through the firewall**

```bash
ufw allow 6379
```

---

### MongoDB

**Install MongoDB**

```bash
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv EA312927
echo "deb http://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.2.list
sudo apt-get update
sudo apt-get install -y mongodb-org
```

**`/lib/systemd/system/mongod.service`**

```conf
[Unit]
Description=High-performance, schema-free document-oriented database
After=network.target
Documentation=https://docs.mongodb.org/manual

[Service]
Restart=always
User=mongodb
Group=mongodb
ExecStart=/usr/bin/mongod --quiet --config /etc/mongod.conf

[Install]
WantedBy=multi-user.target
```

```bash
systemctl enable mongod
```

**`/etc/rc.local`**

```bash
if test -f /sys/kernel/mm/transparent_hugepage/enabled; then
   echo never > /sys/kernel/mm/transparent_hugepage/enabled
fi

if test -f /sys/kernel/mm/transparent_hugepage/defrag; then
   echo never > /sys/kernel/mm/transparent_hugepage/defrag
fi
```

**Create the admin user**

```bash
mongo

-> use admin
-> db.createUser({user: 'myuser', pwd: 'somepassword', roles: ['root']})
```

Since this will force you to log in with the passwod, you'll have to login with the following after you enable auth:

```bash
mongo host/mydb -u myuser -p somepassword --authenticationDatabase admin
```

**`/etc/mongod.conf`**

```conf
net:
  port: 27017
  bindIp: 0.0.0.0
  
  http:
    enabled: false
    JSONPEnabled: false
    RESTInterfaceEnabled: false

security:
  authorization: enabled
```

**Allow access through the firewall**

```bash
ufw allow 27017
```

**Apply settings**

```bash
reboot
service mongod status
```

---

### Elasticsearch

```bash
# Install the needed dependencies
apt-get install default-jre

# Add the distribution
wget https://download.elastic.co/elasticsearch/release/org/elasticsearch/distribution/deb/elasticsearch/2.3.3/elasticsearch-2.3.3.deb

# Install the distribution
sudo dpkg -i elasticsearch-2.3.3.deb

# Enable the service and start it
sudo systemctl enable elasticsearch.service
service elasticsearch start

# Allow access through the firewall
ufw allow 9200
```

---

### Python

```bash
apt-get install python
```

---

### Mysql

**Install the server**

```bash
apt-get install mysql-server
sudo mysql_install_db
sudo /usr/bin/mysql_secure_installation # "yes" to all questions
```

**Create a administrator user**

```sql
CREATE USER 'username'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON *.* TO 'username'@'localhost';
CREATE USER 'username'@'%' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON *.* TO 'username'@'%';
FLUSH PRIVILEGES;
```

**Allow access through the firewall**

```bash
ufw allow 3306
```

---

### Ruby & Compass

```bash
# Install
apt-get install ruby ruby-dev

# Install compass
gem install compass
```

---

### nginx and PHP

```bash
# Import ppa
add-apt-repository ppa:ondrej/php
apt-get update

# Install nginx, php fpm, git and htop
apt-get install nginx php5.6-fpm php5.6-cli php5.6-mcrypt php5.6-mysqlnd php5.6-curl php5.6-mbstring

# Update php config (change fix_pathinfo=0)
nano /etc/php/5.6/fpm/php.ini

# Enable mcyrypt for php
sudo phpenmod mcrypt
sudo service php5.6-fpm restart

# Install composer
wget https://getcomposer.org/composer.phar
mv composer.phar /usr/local/bin/composer
chmod +x /usr/local/bin/composer

# Generate the directory for the sourcecode
mkdir -p /var/www/html

# Edit the nginx config (see below)
nano /etc/nginx/sites-enabled/default

# Restart nginx for changes to take effect
sudo service nginx restart

# Change into directory and clone code
cd /var/www/html
git clone http://github.com/...
```

**`/etc/nginx/sites-enabled/default`**

```
server {
	listen 443 ssl spdy;

	root /var/www/html/gw2-api/public;
	index index.php;
	server_name gw2-api.com;
	error_page 404 /404.html;

	location / {
		try_files $uri $uri/ /index.php?$query_string;
	}

	location ~ \.php$ {
		try_files $uri /index.php =404;
		fastcgi_split_path_info ^(.+\.php)(/.+)$;
		fastcgi_pass unix:/var/run/php5-fpm.sock;
		fastcgi_index index.php;
		fastcgi_intercept_errors on;
		fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
		include fastcgi_params;
	}

	ssl on;
	ssl_certificate /etc/nginx/ssl/www_gw2api_com_bundle.crt;
	ssl_certificate_key /etc/nginx/ssl/www_gw2api_com.key;
	ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
	ssl_ciphers "ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA256:ECDHE-RSA-AES256-SHA:ECDHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES128-SHA256:DHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES256-GCM-SHA384:AES128-GCM-SHA256:AES256-SHA256:AES128-SHA256:AES256-SHA:AES128-SHA:DES-CBC3-SHA:HIGH:!aNULL:!eNULL:!EXPORT:!DES:!MD5:!PSK:!RC4";
	ssl_prefer_server_ciphers on;
}
```

**Restarting**

```bash
sudo service php5-fpm restart && sudo service nginx restart
```

> **Note** Sometimes the php5-fpm process doesn't want to restart, in which case you should try and kill the process via it's process id.
