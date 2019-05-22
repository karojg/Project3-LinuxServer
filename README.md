# README

## Development Environment Information

- Public IP Address: 3.94.87.12
- Private Key: Shared via Udacity platform

## SSH Access

- Copy the downloaded key to `~/.ssh` folder
- Change the permissions to the key 
  ` chmod 600 [keyname]`

- In your terminal execute the following command:
  `ssh -i ~/.ssh/aws.pem ubuntu@3.94.87.12`

## Update all currently installed packages

- Download package lists with `sudo apt update`
- Fetch new versions of packages with `sudo apt upgrade`

## Secure server

### Connections
- Block all incoming, then only allow what you need.
  `sudo ufw default deny incoming`

- Establish default rule for outgoing connections
  `sudo ufw default deny outgoing`

### Change Port
- Change the SSH port from **22** to **2200**. 
  - Run `sudo nano /etc/ssh/sshd_config` 
  - Change the port from 22 to 2200
  - Make sure to configure the Lightsail firewall to allow it.  
  - Confirm by accessing via port 2200
    `ssh -i ~/.ssh/aws.pem -p 2200 ubuntu@3.94.87.12`
  
### Update UFW
- Configure the Uncomplicated Firewall (UFW)
  - `sudo ufw allow 2200/tcp`
  - `sudo ufw allow 80/tcp`
  - `sudo ufw allow 123/udp` 
  - `sudo ufw enable`
 
### Prevent the `error sudo: unable to resolve host`
 - Run `sudo nano /etc/hosts`
 - Add `127.0.1.1 ip-172-26-13-6`

## Give `grader` access
- Create a new user account named `grader`
  `sudo adduser grader`
- Give `grader` the permission to `sudo`
	`sudo vim /etc/sudoers.d/grader`
	type in `grader ALL=(ALL:ALL) ALL`

### Set ssh login using keys

- Locally generate keys 
    `ssh-keygen`

- Deploy public key on developement enviroment

    ```
    su - grader
    mkdir .ssh
    touch .ssh/authorized_keys
    vim .ssh/authorized_keys
    ```

- Copy the public key previously generated authorized_keys
- Update .ssh permissions

    ```
    chmod 700 .ssh
    chmod 644 .ssh/authorized_keys
    ```

- Reload SSH 
    `sudo service ssh restart`
- Grader should be able to login with ssh at port 2200
    `ssh -i ~/.ssh/udacity -p 2200 grader@3.94.87.12`
    
### Remove remote access as root
 - `sudo nano /etc/ssh/sshd_config`
 - Set `PermitRootLogin` to `no   `

## Prepare to deploy your project

- Configure the local timezone to UTC.
	`sudo dpkg-reconfigure tzdata`

### Install and configure Apache to serve a Python mod_wsgi application
- Install Apache
	`sudo apt-get install apache2`
- Install mod_wsgi 
	`sudo apt-get install libapache2-mod-wsgi python-dev`
- Restart Apache 
	`sudo service apache2 restart`

## Clone the Catalog app from Github
- Install git
    `sudo apt-get install git`
- Create folder where app will be saved
    `cd /var/www`
    `sudo mkdir catalog`
    `cd catalog/`
- Clone project from github 
    `sudo git clone https://github.com/karojg/Project3-ItemCatalog-Ubuntu.git catalog`
- Create a wsgi file

  `sudo nano catalog.wsgi`

````
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
application.secret_key = 'supersecretkey'
````

## Install virtual environment

- Install pip with `sudo apt-get install python-pip`
- Install the virtual environment `sudo pip install virtualenv`
- Create a new virtual environment with `sudo virtualenv venv`
- Activate the virtual environment `source venv/bin/activate`

## Install Flask and other dependencies

- Install Flask `sudo pip install Flask`
- Install dependencies `sudo apt-get install libpq-dev python-dev`
- Install other project dependencies `sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils requests`

## Configure and enable a new virtual host

- Run this: `sudo nano /etc/apache2/sites-available/catalog.conf`
- Paste this code:

```
<VirtualHost *:80>
	ServerName 3.94.87.12
	ServerAdmin admin@gmail.com
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
    WSGIProcessGroup catalog
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi
    <Directory /var/www/catalog/catalog/>
        Order allow,deny
        Allow from all
    </Directory>
    Alias /static /var/www/catalog/catalog/static
    <Directory /var/www/catalog/catalog/static/>
        Order allow,deny
        Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

- Enable the virtual host `sudo a2ensite catalog`

- Reload Apache `service apache2 reload`

## Install and configure PostgreSQL

```
sudo apt-get install postgresql postgresql-contrib
sudo su - postgres
psql
CREATE USER catalog WITH PASSWORD 'password';
ALTER USER catalog CREATEDB;
CREATE DATABASE catalog WITH OWNER catalog;
\c catalog
REVOKE ALL ON SCHEMA public FROM public;
GRANT ALL ON SCHEMA public TO catalog;
\q
exit
```

- Populate DB 
    `sudo python /var/www/catalog/catalog/catalogdb_setup.py`
- Make sure no remote connections to the database are allowed
    `sudo nano /etc/postgresql/9.5/main/pg_hba.conf`


## Restart Apache
- Restart Apache
`sudo service apache2 restart`

## Resources used to accomplish project

- https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps

- https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps

- https://github.com/jungleBadger/-nanodegree-linux-server
