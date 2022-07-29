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

## Install
