# Hosting Webpage on DigitalOcean

Updated documentation of DigitalOcean webpage hosting using Ubuntu server.

## Contents

- Migrate Domain DNS to DigitalOcean
- DigitalOcean Droplet Setup
- Basic SSH Security Setup
- Basic Firewall Setup
- Install Git
- Install [node.js](https://nodejs.org/en/)
- Install [PM2](https://www.npmjs.com/package/pm2) (process manager similar to nodemon but without console output)
- Free SSL Certificate With [Certbot](https://certbot.eff.org/)

## Change Domain DNS to point to DigitalOcean. 
We'll be using DigitalOcean to manage any DNS updates.

Enter your own Nameservers:
* ns1.digitalocean.com
* ns2.digitalocean.com
* ns3.digitalocean.com

This now tells the service you purchased your domain from that you don't want them to handle the nameservers.
This also means that any changes you want to make, you'll have to do them from DigitalOcean.

## Create a DigitalOcean droplet. 
* Create droplet with Ubuntu 22
* Disable Password Access
* Enable SSH Access ONLY

## Access server and add basic security.
```sh
# Access ubuntu server (change 1.1.1.1 to your droplet's IP Address)
$ ssh root@1.1.1.1

# Create a user to avoid using the root user. (Which has unrestricted sudo rights):
$ adduser pedronunezcode

# Add that new user to the 'sudo' group.
$ usermod -aG sudo pedronunezcode

# Switch to new user and add SSH key to the new user. Since the ability to sign into root will be disabled later.
$ su - custom_user
$ mkdir ~/.ssh
$ chmod 700 ~/.ssh

# Paste SSH key in here: (Yes... Vim > Emacs || nano)
$ vim ~/.ssh/authorized_keys

# You can test to see if it worked by exiting the server and trying to connect to your new user.
$ exit 
$ ssh pedronunezcode@1.1.1.1 
```

## Disable RootLogin and PasswordAuthentication

```sh
$ sudo vim /etc/ssh/sshd_config
```

Change `PermitRootLogin yes` to `PermitRootLogin no`

Change `#PasswordAuthentication yes` to `PasswordAuthentication no`

Save file then restart the SSH service: `$sudo systemctl reload sshd`

## Setup Basic Firewall:
```sh
# Enable OpenSSH Connections:
$ sudo ufw allow OpenSSH

# Enable HTTP Traffic:
$ sudo ufw allow http

# Enable HTTPS Traffic:
$ sudo ufw allow https

# Turn on Firewall
$ sudo ufw enable
```

## Install Node.js, Git and PM2 (process manager)
I mainly use React and Node.js for my projects. This is what I'll be documenting on here. I assume whatever you 
use for web development will be similar.

```sh
# Install git to clone repository. (I don't develop from the webserver)
$ sudo apt-get install git

# Instruct apt-get which version of Node.js to download. (I use the current v16 stable build)
$ curl -sL https://deb.nodesource.com/setup_16.x | sudo -E bash -

# Install the version you asked apt-get to install:
$ sudo apt-get install nodejs

# To check if the version you wanted installed is correct:
$ node --version
```

## Install PM2 (Proccess Manager)

```sh
# I use npm here since I installed node earlier
$ sudo npm install -g pm2
```

PM2 is used very similarly to nodemon or node to start a process. The sytax is as follows

```sh
# index.js is where I usually have an express server. This will serve my app to localhost.
$ pm2 start index.js 
```

## Install Certbot (SSL Certificate Configurator)

```sh
# Make sure snapd is up to date
$ sudo snap install core; sudo snap refresh core

# Install Certbot
$ sudo snap install --classic certbot

# Prepare the Certbot command
$ sudo ln -s /snap/bin/certbot /usr/bin/certbot

# Create SSL Certificate (have domain in hand)
sudo certbot certonly --standalone

# Auto Renew SSL Certificate
$ sudo certbot renew --dry-run
```

## Install and configure nginx

```sh
# Install nginx
$ sudo apt-get install nginx

# Delete everything inside /etx/nginx/sites-enabled/default
$ sudo vim /etc/nginx/sites-enabled/default

# Paste the following:
server {
    server_name *change to your domain*;
    listen 80 default_server;
    listen [::]:80 default_server ipv6only=on; 

     # Enable HTTP/2
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name *Change to your domain*;
    # Use letsencrypt certificate
    ssl_certificate /etc/letsencrypt/live/*Change to your domain*/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/*Change to your domain*/privkey.pem;
    include snippets/ssl-params.conf;
    location / {
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-NginX-Proxy true;
        proxy_pass http://localhost:3000/;
        proxy_ssl_session_reuse off;
        proxy_set_header Host $http_host;
        proxy_cache_bypass $http_upgrade;
        proxy_redirect off;
    }
}

```

## Create a secure Diffie-Hellman Group (Makes SSL Even Safer)
```sh
$ sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048

# Create a configuration file for SSL
$ sudo vim /etc/nginx/snippets/ssl-params.conf

# Paste the following inside the file:

# See https://cipherli.st/ for details on this configuration
ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
ssl_prefer_server_ciphers on;
ssl_ciphers "EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH";
ssl_ecdh_curve secp384r1; # Requires nginx >= 1.1.0
ssl_session_cache shared:SSL:10m;
ssl_session_tickets off; # Requires nginx >= 1.5.9
ssl_stapling on; # Requires nginx >= 1.3.7
ssl_stapling_verify on; # Requires nginx => 1.3.7
resolver 8.8.8.8 8.8.4.4 valid=300s;
resolver_timeout 5s;
add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload";
add_header X-Frame-Options DENY;
add_header X-Content-Type-Options nosniff;

# Add our strong Diffie-Hellman group
ssl_dhparam /etc/ssl/certs/dhparam.pem;
```

## Test the nginx configuration
```sh
$ sudo nginx -t

# If the output of the command above is successful:
$ sudo systemctl start nginx
```
