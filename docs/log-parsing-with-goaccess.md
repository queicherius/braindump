# Log parsing with GoAccess

[GoAccess](http://goaccess.io/) is a real-time web log analyzer and interactive viewer. We'll use it to parse nginx and haproxy logs.

### Install

```bash
apt-get install goaccess
```

### Parse nginx logs

```bash
goaccess -f /var/log/nginx/access.log
```

When asked for a format, choose `NCSA Combined Log Format`.

### Parse haproxy logs

Since goaccess does not support haproxy's log format natively, this command builds the structure for us.

```bash
goaccess -f haproxy.log --log-format='%^  %^ %^:%^:%^ %^ %^[%^]: %h:%^ [%d:%t.%^] %^ %^ %^/%^/%^/%^/%L %s %b %^ %^ %^ %^/%^/%^/%^/%^ %^/%^ "%r"' --date-format='%d/%b/%Y' --time-format='%H:%M:%S' -q
```
