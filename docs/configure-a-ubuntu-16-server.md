# Configure a Ubuntu 16 server

This is a general setup to configure a raw Ubuntu 16.04.1 LTS system. It is a mix of security and comfort and adresses most configuration issues that exist within a stock install.

### Root login

I want to login as root, and the raw install does not have a password for it set, so you can't log in. It's true that this is less secure than giving superuser priviledges to the original user, but having to prepend `sudo` before every command defeats the security purpose as well. This also makes port-binding easier.

```bash
# Set a password for root
sudo passwd root

# Update the SSH file to allow login with passwords (this will get changed below)
# -> PermitRootLogin yes
sudo nano /etc/ssh/sshd_config
sudo service ssh restart

# Logout and log back in as "root" with the password you set
logout

# Remove the (now obsolete) user
userdel -r david
```

---

### Hostname

We can configure the hostname to match what we want. I usually use a naming scheme of `application.server`, so e.g. `gw2efficiency.mongo`:

```bash
nano /etc/hostname
reboot
```

---

### Network interface names

Since Ubuntu 15 the network interfaces changed their names from `eth0`, `etc1`, ... to something else. In my opinion that's confusing, since the names can be slightly unpredictable on virtual hardware, so this changes it back.

```bash
nano /etc/default/grub
# -> change GRUB_CMDLINE_LINUX="net.ifnames=0"
update-grub
reboot
ifconfig -a
```

---

### Static IP

You should get a static IP provided with your server, which we will use here. Alongside we'll configure the server to use Google's nameservers.

**`/etc/network/interfaces`**

```
auto eth0
iface eth0 inet static
address x.x.x.x
netmask 255.255.255.255
gateway x.x.x.254
dns-nameservers 8.8.8.8 8.8.4.4
```

After the configuration, we apply it by restarting the interface. Then we should be able to still ping the internet!

```bash
ipdown -a && ifup -a
ping google.com
```

---

### Update the system

Now, that we are connected to the internet, we'll download a bunch of updates and install some essentials.

```bash
# Update everything!
apt-get update && apt-get upgrade

# Install essentials
apt-get install nano build-essential htop git
```

> **Note:** Since we changed the "grub" file, it might ask you if you want to keep your version or update it. Just check the diff to see if everything is in order.

---

### SSH-Key only login

Since password logins are super uncomfortable, we'll use an SSH key to login to the server. First we'll have to generate the key and let the server know about it:

```bash
# These commands are to be executed on the client

# Generate a keyfile (or use an existing keyfile and skip this)
ssh-keygen -t rsa -C "my@email.com"

# Copy the keyfile onto the server
ssh-copy-id -i ~/.ssh/id_rsa root@serverip

# Login with the SSH key (should not promt for a password!)
ssh root@serverip
```

At this point you should be able to log into the server without a password. Now we disable password logins for security purposes:

```bash
# Disable password login
# -> PermitRootLogin without-password
# -> PasswordAuthentication no
nano /etc/ssh/sshd_config
service ssh restart

# Install fail2ban for banning too many login tries
apt-get install fail2ban
```

---

### Timezone

The timezone should be set to UTC (for your own sanity).

```bash
rm /etc/localtime
ln -s /usr/share/zoneinfo/UTC /etc/localtime

# Now these two commands should say the same
date
date -u
```

---

### Firewall

We'll install and use `ufw` to block any connections we don't want. Since we currently only run SSH publically, we'll only allow connections to that:

```bash
ufw allow OpenSSH
ufw enable
```

> *Note:* Use `nmap -F x.x.x.x` on your client to test this!