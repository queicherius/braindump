# Https & caching for Node.JS

Here, we're using [Let's Encrypt](https://letsencrypt.org/) for certificates, [Nginx](https://www.nginx.com/) as a [TLS Termination proxy](https://en.wikipedia.org/wiki/TLS_termination_proxy) and [Varnish](https://www.varnish-cache.org) for caching. Assuming the domain name `mydomain.com` (you have to change that) and your Node.JS app is running on port `8080` (but you can change that!).

## Setting up SSL

```bash
# Create the certificate with letsencrypt
apt-get install letsencrypt
letsencrypt certonly --standalone -d mydomain.com

# Generate DH-Parameter for nginx
openssl dhparam -out /etc/letsencrypt/live/mydomain.com/dhparam.pem 2048

# Install nginx
apt-get install nginx
```

**`/etc/nginx/sites-enabled/default`**

```conf
# Don't send the nginx version number in error pages and Server header
server_tokens off;

# Redirect traffic to SSL always
server {
  listen 80;
  server_name mydomain.com;

  # Enable HTTP Strict Transport Security to avoid ssl stripping
  add_header Strict-Transport-Security "max-age=31622400; includeSubDomains; preload" always;

  return 301 https://$server_name$request_uri;
}

server {
  listen 443 ssl;
  server_name mydomain.com;

  # --- SSL CONFIGURATION ------------------------------------

  # SSL certificate
  ssl_certificate /etc/letsencrypt/live/mydomain.com/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/mydomain.com/privkey.pem;

  # Enable session resumption to improve https performance
  ssl_session_cache shared:SSL:50m;
  ssl_session_timeout 5m;

  # Diffie-Hellman parameter for DHE ciphersuites
  ssl_dhparam /etc/letsencrypt/live/mydomain.com/dhparam.pem;

  # Enables server-side protection from BEAST attacks
  ssl_prefer_server_ciphers on;

  # Disable SSLv3 since it's less secure then TLS
  ssl_protocols TLSv1 TLSv1.1 TLSv1.2;

  # Ciphers chosen for forward secrecy and compatibility
  ssl_ciphers "ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA256:ECDHE-RSA-AES256-SHA:ECDHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES128-SHA256:DHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES256-GCM-SHA384:AES128-GCM-SHA256:AES256-SHA256:AES128-SHA256:AES256-SHA:AES128-SHA:DES-CBC3-SHA:HIGH:!aNULL:!eNULL:!EXPORT:!DES:!MD5:!PSK:!RC4";

  # Enable OCSP stapling
  resolver 8.8.8.8;
  ssl_stapling on;
  ssl_trusted_certificate /etc/letsencrypt/live/mydomain.com/fullchain.pem;

  # Enable HTTP Strict Transport Security to avoid ssl stripping
  add_header Strict-Transport-Security "max-age=31622400; includeSubDomains; preload" always;

  # --- SERVICE CONFIGURATION ------------------------------------

  gzip on;

  location / {
    proxy_pass http://127.0.0.1:8080;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto https;
    proxy_set_header X-Forwarded-Port 443;
    proxy_set_header Host $host;
  }
}
```

```bash
service nginx restart
```

At this point the page should be reachable with https and redirect to https when you try and use http. You should also get a shiny A+ on https://www.ssllabs.com.

## Setting up Varnish

```bash
apt-get install varnish
```

**`/etc/nginx/sites-enabled/default`**

```conf
# Ensure that the default values are in here
backend default {
    .host = "127.0.0.1";
    .port = "8080";
}
```

**`/etc/nginx/sites-enabled/default`**

```conf
# Make sure that "Alternative 2" is used and 
# Varnish is listening on the default port 6081
DAEMON_OPTS="-a :6081
 ...
```

**`/etc/nginx/sites-enabled/default`**

```conf
  location / {
    proxy_pass http://127.0.0.1:6081;
```

```bash
service varnish restart
service nginx restart
```

Now Nginx is listening on port 80 and 433, ["unwrapping" the SSL request](https://en.wikipedia.org/wiki/TLS_termination_proxy), giving it to Varnish, which caches the responses from our Node.JS app.

## Firewall rules

Now the only thing missing is setting up the firewall rules so that you can only reach the public nginx server, and not the internal ports, e.g. using `ufw`:

```bash
ufw allow OpenSSH
ufw allow 'Nginx HTTP'
ufw allow 'Nginx HTTPS'
ufw enable
```