# Linux Server for "Best Entertainment of 2017"

## Introduction
This repo serves only as documentation for how to deploy ["The Best Entertainment of 2017"](https://github.com/colbeezy/fullstack-nanodegree-vm/blob/master/vagrant/catalog/) app to a live Linux server, as part of the Udacity Fullstack Nanodegree.

## Basic Info (as of time of commit)
URL: http://ec2-13-58-125-203.us-east-2.compute.amazonaws.com

Public IP Address: 13.58.125.203

SSH Port: 2200

## Configuration Steps

### Get Your Server
Create instance of Ubuntu server on [Amazon Lightsail](https://amazonlightsail.com/)

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

### Give Grader Access
1. Create a new user account named grader.

```
sudo adduser grader
```

2. Give grader the permission to sudo.

```
sudo nano /etc/sudoers.d/grader
```

Add text: `grader ALL=(ALL) NOPASSWD:ALL`

3. Create an SSH key pair for grader using the ssh-keygen tool.

```
ssh-keygen
```

Make directories to store the key on the server:

```
mkdir /home/grader/.ssh
nano /home/grader/.ssh/authorized_keys
chown grader /home/grader/.ssh
chown grader /home/grader/.ssh/authorized_keys
```

Copy-paste the public key into /home/grader/.ssh/authorized_keys.

Set permissions:

```
chmod 700 /home/grader/.ssh
chmod 600 /home/grader/.ssh/authorized_keys
```

Grader can access the machine with SSH:

```
ssh grader@13.58.125.203 -p 2200 -i FILEPATHTOKEY
```

### Prepare to deploy your project.
1. Configure the local timezone to UTC.

```
sudo dpkg-reconfigure tzdata
```

When prompted to select a timezone, pick 'None of the Above' and then UTC.

2. Install and configure Apache to serve a Python mod_wsgi application.

```
sudo apt-get install apache2
sudo apt-get install libapache2-mod-wsgi
```

3. Install and configure PostgreSQL.

```
sudo apt-get install PostgreSQL
```

To prevent remote connections, create a new database user named catalog that has limited permissions to your catalog application database.

```
sudo adduser catalog
sudo -u postgres -i
postgres:~$ creatuser catalog
postgres:~$ createdb catalog
postgres:~$ psql
postgres=# ALTER DATABASE catalog OWNER TO catalog;
postgres=# ALTER USER catalog WITH PASSWORD 'catalog';
```

4. Install git.

```
sudo apt-get install git
```

### Deploy the Item Catalog project.
1. Clone and setup your Item Catalog project from the Github repository you created earlier in this Nanodegree program.

```
cd /var/www
git clone https://github.com/colbeezy/fullstack-nanodegree-vm.git
```

Within this directory, all items below refer to /vagrant/catalog.
Open application.py, database_setup.py, and sample_db.py and update the db to postgres:

```
engine = create_engine('postgresql://catalog:PASSWORD@localhost/catalog')
```

2. Set it up in your server so that it functions correctly when visiting your server’s IP address in a browser. Make sure that your .git directory is not publicly accessible via a browser! Install the dependencies:

```
sudo apt-get -y install python-pip
sudo pip install SQLAlchemy
sudo pip install psycopg2
sudo pip install flask
sudo pip install oauth2client
sudo pip install requests
```

Create schema and populate the catalog database with sample data

```
python /var/www/fullstack-nanodgree-vm/vagrant/catalog/sample_db.py
```

Configure Apache to serve the web application using WSGI.
First, Create the web application WSGI file.

```
sudo nano /var/www/fullstack-nanodegree-vm/app.wsgi 
```

Add text to the file:

```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/fullstack-nanodegree-vm/vagrant/catalog/")
from application import app as application
application.secret_key = 'SECRETKEY'

```

Update the Apache configuration file to serve the web application with WSGI.

```
sudo nano /etc/apache2/sites-enabled/000-default.con
```

Add the following text inside the <VirtualHost *:80> element:

```
WSGIScriptAlias / /var/www/fullstack-nanodegree-vm/app.wsgi
```

Restart Apache:

```
sudo apache2ctl restart
```

### Final Tweaks
You might think you're done now. I did, but I was wrong. You can check at http://ec2-13-58-125-203.us-east-2.compute.amazonaws.com/. If there are any errors, check the logs at `sudo tail /var/log/apache2/error.log`

There are still 2 issues to fix with the app:

1. Update application.py to use absolute path of JSON file:

The relative paths in client_secrets.json were not being found, so I changed the relative paths to absolute paths.

  i. Add `import os`
  
  ii. Create path variable `path = os.path.dirname(__file__)`
  
  iii. Update CLIENT_ID variable CLIENT_ID = `json.loads(open(os.path.dirname(__file__)+'/client_secrets.json', 'r').read())['web']['client_id']`
  
  iv. Update gconnect method: `oauth_flow = flow_from_clientsecrets(path+'/client_secrets.json', scope='')`
  
```
sudo apache2ctl restart
```

2. Update OAuth Google Credentials:

Google requires that you add the origin URI of the client application to their "Authorized JavaScript Origins" in their console.


### References
* [Udacity's Configuring Linux Web Severs Course](https://www.udacity.com/course/configuring-linux-web-servers--ud299)
* [Digital Ocean Posts](https://www.digitalocean.com/community/tutorials/how-to-setup-a-firewall-with-ufw-on-an-ubuntu-and-debian-cloud-server)
* [Ask Ubuntu Forum](https://askubuntu.com/)
* [Official Ubuntu Documentation](https://help.ubuntu.com/community/UbuntuTime)
* [nixCraft](https://www.cyberciti.biz/faq/linux-resetting-a-users-password/)
* [Apache Docs](https://httpd.apache.org/docs/2.4/)
* [A lot of Stackoverflow pages, such as this one](https://stackoverflow.com/questions/16850350/got-origin-mismatch-error-in-google-share-api)
