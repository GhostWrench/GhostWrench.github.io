Setup Instructions for GhostNet Server
================================================================================

Overview
--------------------------------------------------------------------------------
This contains basic instructions for setting up a Linode Debian 10 instance
to serve a static website with https and email capabilities. Along with any 
other nice to have features and security.

Update Server Software
--------------------------------------------------------------------------------
```bash
root@localhost:~$ apt-get update && apt-get upgrade
```

Setting the Hostname
--------------------------------------------------------------------------------
Our hostname for this guide will be casper, the name of a friendly ghost. Also 
assume that the ip4 address of the server is 55.55.55.55 and the ip6 address of
the server is 5555:5555::5555:5555:5555:5555.
```bash
root@localhost:~$ hostnamectl set-hostname casper
```

Then add the following to */etc/hosts*:
```
127.0.0.1 localhost.localdomain localhost
55.55.55.55 casper.ghostwrench.net casper
5555:5555::5555:5555:5555:5555 casper.ghostwrench.net casper
```

Reconfigure Server Time Zone (Optional)
--------------------------------------------------------------------------------
```bash
root@localhost:~$ dpkg-reconfigure tzdata
```

Add a Limited User Account
--------------------------------------------------------------------------------
Doing things logged in as root is bad practice, we need to make a user account
as the main administrative account. We will call this user 'daniel'.

```bash
root@localhost:~$ adduser daniel
root@localhost:~$ adduser daniel sudo
```
Now log back in through SSH as the new user

Create a User SSH Key For Login Rather Than Password
--------------------------------------------------------------------------------
Create a key on your local machine
```bash
daniel@mycomputer:~$ ssh-keygen -b 4096
daniel@mycomputer:~$ ssh-copy-id daniel@55.55.55.55
```
Now disallow login to the root account over SSH by setting the option 
'PermitRootLogin' and 'PasswordAuthentication' in 
*/etc/ssh/sshd_config* to 'no'
```bash
daniel@casper:~$ sudo sed -i 's/^#\?PermitRootLogin.*$/PermitRootLogin no/' /etc/ssh/sshd_config
daniel@casper:~$ sudo sed -i 's/^#\?PasswordAuthentication.*$/PasswordAuthentication no/' /etc/ssh/sshd_config
```

Restart the ssh daemon
```bash
daniel@casper:~$ sudo systemctl restart sshd
```

Install fail2ban to Throttle SSH connection attempts
--------------------------------------------------------------------------------
The default settings for fail2ban are OK. Linode has a good tutorial on 
setting it up if finer grained control is desired.
```bash
daniel@casper:~$ sudo apt-get install fail2ban
```

Setup a Firewall
--------------------------------------------------------------------------------
Setup defaults
```bash
daniel@casper:~$ sudo ufw default allow outgoing
daniel@casper:~$ sudo ufw default deny incoming
```

Allow SSH on port 22
```bash
daniel@casper:~$ sudo ufw allow ssh
```

Start the firewall service
```bash
sudo systemctl start ufw
sudo ufw enable
```

Install NGINX to Serve Static Content
--------------------------------------------------------------------------------
Install
```bash
daniel@casper:~$ sudo apt-get install nginx
```

Setup a folder where the website files will live
```bash
daniel@casper:~$ sudo mkdir /var/www/ghostwrench.net
```

Remove the default server symlink from sites-available
```bash
daniel@casper:~$ sudo rm /etc/nginx/sites-enabled/default
```

Create a server configuration file in sites-available
```bash
daniel@casper:~$ sudo touch /etc/nginx/sites-available/ghostwrench.net
```

```
server {
    listen 80;
    listen [::]:80;
    server_name 50.116.25.111;

    root /var/www/50.116.25.111;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Make a symlink to the new server configuration in sites-enabled
```bash
daniel@casper:~$ sudo ln -s /etc/nginx/sites-available/ghostwrench.net /etc/nginx/sites-enabled/
```

Open up firewall on port 80
```bash
daniel@casper:~$ sudo ufw allow 80/tcp
```

Test NGINX to see if everything is set up correctly
```bash
daniel@casper:~$ sudo nginx -t
```

Reload the server
```bash
daniel@casper:~$ sudo nginx -s reload
```

Point Domain Name Provider's DNS Records to Server
--------------------------------------------------------------------------------

