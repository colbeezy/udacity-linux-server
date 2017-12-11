# Linux Server for "Best Entertainment of 2017"

## Introduction
This repo serves only as documentation for how to deploy ["The Best Entertainment of 2017"](https://github.com/colbeezy/fullstack-nanodegree-vm/blob/master/vagrant/catalog/) app to a live Linux server, as part of the Udacity Fullstack Nanodegree.

## Basic Info (as of time of commit)
URL: http://ec2-13-58-125-203.us-east-2.compute.amazonaws.com
Public IP Address: 13.58.125.203
SSH Port: 2200

## Configuration Steps

### Get Your Server
Create instance of Ubuntu server on [Amazon Lightsail] (https://amazonlightsail.com/)

### Secure Your Server
1. SSH in as default ubuntu user via web
2. Update all currently installed packages

```
sudo apt-get update
sudo apt-get upgrade
```

3. Change the SSH port from 22 to 2200. Make sure to configure the Lightsail firewall to allow it.

```
sudo nano /etc/ssh/sshd_config
```
Change port 22 to port 2200.

4. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).

```
sudo ufw allow 2200/tcp
sudo ufw allow 80/tcp
sudo ufw allow 123/tcp
sudo ufw enable
```

Then, in the AWS console, go to the instance's "Networking" tab and add the ports into the "Firewall" section.

Warning: When changing the SSH port, make sure that the firewall is open for port 2200 first, so that you don't lock yourself out of the server. Review this video for details! When you change the SSH port, the Lightsail instance will no longer be accessible through the web app 'Connect using SSH' button. The button assumes the default port is being used. There are instructions on the same page for connecting from your terminal to the instance. Connect using those instructions and then follow the rest of the steps.
