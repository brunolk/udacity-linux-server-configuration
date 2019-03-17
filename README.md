# Linux Server Configuration

This project is part of Udacity's Backend Nanodegree. It is a list of instructions explaining how to host the [Restaurant Menu App](https://github.com/brunolk/udacity-menu-app) in a Ubuntu Linux server on an Amazon Lightsail instance. You can temporalily see the app deployed on: http://3.90.161.205/

## Get your server

### 1- Create Ubuntu Linux Server on Amazon Lightsail
* Login into [Amazon Lightsail](https://lightsail.aws.amazon.com/)
* Click Create instance button
* Select Linux/Unix platform (OS Only > Ubuntu 16.04)
* Select an instance plan
* Rename your instance
* Click Create button
* Wait for the instance to start

### 2- Create your development environment
* Download and install [VirtualBox](https://www.virtualbox.org/wiki/Download_Old_Builds_5_2)
* Download and install [Vagrant](https://www.vagrantup.com/)
* Create a new folder on your computer where you’ll store the application, then open that folder within your terminal
* Type `vagrant init ubuntu/trusty64` to tell Vagrant what kind of Linux virtual machine you would like to run
* Type `vagrant up` to download and start running the virtual machine (VM)

### 3- SSH into the server
* Access your [Amazon Lightsail](https://lightsail.aws.amazon.com/) account page, click on SSH KEYS tab and download the private key
* Move the downloaded key pairs file to the application folder
* Type `vagrant ssh` to access your VM
* Within the VM move the key pairs file to the .ssh folder, and rename it as lightsail.rsa
```
cd /vagrant/
mv LightsailDefaultKey-us-east-1.pem ~/.ssh/lightsail.rsa
```
* Protect private key file so only the owner can read and write
```
chmod 600 ~/.ssh/lightsail.rsa
```
* SSH into the server
```
ssh -i ~/.ssh/lightsail.rsa ubuntu@3.90.161.205
```

## Secure the server

### 4- Update all currently installed packages 
* Check for updates
```
sudo apt-get update
```
* Install updates
```
sudo apt-get upgrade
```
* Set for future updates
```
sudo apt-get dist-upgrade
```

### 5- Change the SSH port from 22 to 2200
* Open configuration file
```
sudo nano /etc/ssh/sshd_config
```
* Within the file change the port number from 22 to 2200
* Save and exit using `CTRL+X`, confirm with `Y` and press `Enter` key
* Restart SSH
```
sudo service ssh restart. 
```

### 6- Configure the Uncomplicated Firewall (UFW)
* Configure UFW to only allow incoming connections for SSH(port 2200), HTTP(port 80), and NTP(port 123)
```
sudo ufw status                  # The UFW should be inactive.
sudo ufw default deny incoming   # Deny any incoming traffic.
sudo ufw default allow outgoing  # Enable outgoing traffic.
sudo ufw allow 2200/tcp          # Allow incoming tcp packets on port 2200.
sudo ufw allow 80/tcp            # Allow HTTP traffic in.
sudo ufw allow 123/udp           # Allow incoming udp packets on port 123.
sudo ufw deny 22                 # Deny incoming request for port 22
```
* Activate firewall
```
sudo ufw enable
```
* Confirm firewall status
```
sudo ufw status
```
* Exit SSH connection
```
Exit
```
* Access [Amazon Lightsail](https://lightsail.aws.amazon.com/), go to your instance page and click on the NETWORKING tab. Delete default SSH port 22 and add ports 80, 123, 2200
* SSH into the server
```
ssh -i ~/.ssh/lightsail.rsa ubuntu@3.90.161.205 -p 2200
```

## Give grader access

### 7- Create a new user named grader
* Create grader user
```
sudo adduser grader
```
Enter a password and fill out further information

### 8- Grant grader sudo permission
* Create grader file
```
sudo nano /etc/sudoers.d/grader
```
* Write into the file
```
grader    ALL=(ALL:ALL) ALL
```   
* Save and exit using `CTRL+X`, confirm with `Y` and press `Enter` key
* Exit SSH connection
```
Exit
```

### 9- Create an SSH key pair for grader using the ssh-keygen tool
* On your development environment (VM) create a ssh key pair using the ssh-keygen
```
ssh-keygen
```
* Enter the location in which to save the key pair
```
/home/vagrant/.ssh/grader
```
* Open key file and copy its content
```
cat /home/vagrant/.ssh/grader.pub
```
* SSH into the server
```
ssh -i ~/.ssh/lightsail.rsa ubuntu@3.90.161.205 -p 2200
```
* Login as grader user
```
su - grader
```
* Create .ssh directory
```
sudo mkdir .ssh
```
* Create authorized_keys files and past the copied content
```
sudo nano .ssh/authorized_keys and paste the content in the file
```
* Save and exit using `CTRL+X`, confirm with `Y` and press `Enter` key
* Make sure grader is the owner of .ssh directory
```
sudo chown -R grader.grader /home/grader/.ssh
```
* Set the permissions: 
```
sudo chmod 700 .ssh
sudo chmod 644 .ssh/authorized_keys
```
* Open configuration file
```
sudo nano /etc/ssh/sshd_config
```
* Within the file make sure PasswordAuthentication is set to no
* Save and exit using `CTRL+X`, confirm with `Y` and press `Enter` key
* Restart SSH
```
sudo service ssh restart. 
```
* Exit SSH connection
```
Exit
```
* Test SSH connection via key pair
```
ssh -i ~/.ssh/grader grader@3.90.161.205 -p 2200
```

## Prepare to deploy your project

### 10- Configure local timezone to utc
* Configure local timezone to utc
```
sudo dpkg-reconfigure tzdata
```
* Select your continent and country/city

### 11- Install and configure Apache to serve a Python mod_wsgi application.
* Install Apache
```
sudo apt-get install apache2
```
* Enter public IP (http://3.90.161.205/) into your browser. Apache2 Ubuntu Default Page must show up.
* Install the mod_wsgi package
```
sudo apt-get install libapache2-mod-wsgi python-dev
```
* Enable mod_wsgi
```
sudo a2enmod wsgi
```   
* Restart Apache:
```
sudo service apache2 restart
```

### 12- Install and configure PostgreSQL
* Install PostgeSQL
```
sudo apt-get install postgresql
```
* Open .conf file
```
sudo nano /etc/postgresql/9.5/main/pg_hba.conf
```
* In order to not allow remote connections, make sure the only uncommented lines are:
```
local   all             postgres                                peer
local   all             all                                     peer
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5
```
* Save and exit using `CTRL+X`, confirm with `Y` and press `Enter` key
* Switch to the postgres user
```
sudo su - postgres
```
* Connect to PostgreSQL
```
psql
```
* Create catalog user with a password
```
CREATE ROLE catalog WITH LOGIN PASSWORD 'catalog';
```
* Grant catalog user the ability to create databases 
```        
ALTER USER catalog CREATEDB;
```
* Create catalog database
```
CREATE DATABASE catalog WITH OWNER catalog;
```
* Connect to catalog database
```
\c catalog
```
* Revoke public rights: 
```
REVOKE ALL ON SCHEMA public FROM public;
```
* Grant access to catalog user: 
``` 
GRANT ALL ON SCHEMA public TO catalog;
```
* Exit PostgreSQL
```
\q
```
* Switch back to grader user
```
exit
```
* Create linux catalog user
```
sudo adduser catalog
```
* Create catalog file
```
sudo nano /etc/sudoers.d/grader
```
* Write into the file
```
catalog    ALL=(ALL:ALL) ALL
```   
* Save and exit using `CTRL+X`, confirm with `Y` and press `Enter` key
* Exit SSH connection
```
Exit
```
* Switch to catalog user
```
sudo su - catalog
```
* Create catalog database
```
createdb catalog
```
* Switch back to grader user
```
exit
```

### 13- Install git
* Install git
```
sudo apt-get install git
```

## Deploy the Item Catalog project.

### 14- Clone and setup the [Restaurant Menu App](https://github.com/brunolk/udacity-menu-app) project from the Github
* Create application directory
```
sudo mkdir /var/www/catalog/
```
* Change to the just created directory and clone the catalog project:
```
cd /var/www/catalog/
```
* Clone github project
```
sudo git clone https://github.com/brunolk/udacity-menu-app.git catalog
```
* Give grader user the ownership of `/var/www/catalog/`
```
cd /var/www/catalog/
sudo chown -R grader:grader catalog
```
* Open project.py file
```
sudo nano cd /var/www/catalog/catalog/project.py
```    
* Make the following changes
    * Substitute `engine = create_engine('sqlite:///restaurantmenu.db')` for `engine = create_engine('postgresql://catalog:PASSWORD@localhost/catalog')`
    * Substitute `open('client_secrets.json', 'r').read())['web']['client_id']` for `open('/var/www/catalog/catalog/client_secrets.json', 'r').read())['web']['client_id']` 
    * Substitute `app.run(host='0.0.0.0', port=8000)` for `app.run()`
* Save and exit using `CTRL+X`, confirm with `Y` and press `Enter` key
* Open database_setup.py file
```
sudo nano cd /var/www/catalog/catalog/database_setup.py
```
* Make the following changes
    * Substitute `engine = create_engine('sqlite:///restaurantmenu.db')` for `engine = create_engine('postgresql://catalog:PASSWORD@localhost/catalog')`
* Save and exit using `CTRL+X`, confirm with `Y` and press `Enter` key
* Open lotsofmenus.py file
```
sudo nano cd /var/www/catalog/catalog/lotsofmenus.py
```
* Make the following changes
    * Substitute `engine = create_engine('sqlite:///restaurantmenu.db')` for `engine = create_engine('postgresql://catalog:PASSWORD@localhost/catalog')`
* Save and exit using `CTRL+X`, confirm with `Y` and press `Enter` key
* Rename the project.py file to __init__.py
```
mv project.py __init__.py
```

### 16- Set up for deploying the application 
* Install pip
```
sudo apt-get install python-pip
```
* Install the following dependencies
```
sudo pip install httplib2
sudo pip install requests
sudo pip install --upgrade oauth2client
sudo pip install sqlalchemy
sudo pip install flask
sudo apt-get install libpq-dev
sudo pip install psycopg2
sudo pip install psycopg2-binary
```

### 17- Set up and enable virtual host
* Create a .conf file 
```
sudo nano /etc/apache2/sites-available/catalog.conf
```   
* Paste the following code
```    
<VirtualHost *:80>
    ServerName XX.XX.XX.XX
    ServerAdmin admin@xx.xx.xx.xx
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi
    <Directory /var/www/catalog/catalog/>
        Order allow,deny
        Allow from all
        Options -Indexes
    </Directory>
    Alias /static /var/www/catalog/catalog/static
    <Directory /var/www/catalog/catalog/static/>
        Order allow,deny
        Allow from all
        Options -Indexes
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
* Save and exit using `CTRL+X`, confirm with `Y` and press `Enter` key
* Enable virtual host
```
sudo a2ensite catalog
```
* Reload Apache
```
sudo service apache2 reload
```

### 18- Set up Flask application
* Create .wsgi file
```
sudo nano /var/www/catalog/catalog.wsgi
``` 
* Paste the following code
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
application.secret_key = "secret"
```    
* Save and exit using `CTRL+X`, confirm with `Y` and press `Enter` key
* Set grader user as owner
```
sudo chown -R grader.grader /var/www/catalog/catalog.wsgi
```
* Restart Apache:
```
sudo service apache2 restart
```

### 19- Create and populate database schema
* Create application database schemas
```
python /var/www/catalog/catalog/database_setup.py/
```
* Populate application database schema with fictitious data
```
python /var/www/catalog/catalog/lotsofmenus.py
```
* Restart Apache:
```
sudo service apache2 restart
```

### 20- Launch the application
* The application should be running on your Lightsail instance public IP's address: http://3.90.161.205/


### 21- Resources
* https://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt/138442
* https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps

* https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps

* http://flask.pocoo.org/docs/0.12/installation/
