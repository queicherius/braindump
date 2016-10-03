# Simple mail redirection server

This sets up a simple mail server, which can redirect mail to a personal mail address. This does not cover sending emails, but if you need to do that at an application level (e.g. for password resets) take a look at [Mailgun](https://mailgun.com).

### Install postfix

```bash
apt-get install postfix
```

When the installer asks you for the type of mail server you want to run, choose "Internet Size" and fill in your domain as the "Fully qualified domain name".

### Configure postfix

**`/etc/postfix/main.cf`**

```
myhostname = mydomain.com
virtual_alias_maps = hash:/etc/postfix/virtual
mynetworks = 127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128
```

**`/etc/postfix/virtual`**

You can add a list of any emails you want to redirect in this file

```
support@mydomain.com mymail@someprovider.com
mail@mydomain.com mymail@someprovider.com
```

To finish the configuration, we have to create a postmap of the list of virtual emails and restart Postfix:

```bash
postmap /etc/postfix/virtual
service postfix restart
service postifx status
```

### Firewall access

If you are using a firewall like `ufw` (which you should), you can easily allow access with the following:

```bash
ufw allow Postfix
```
