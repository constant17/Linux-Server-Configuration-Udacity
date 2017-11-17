# Linux-Server-Configuration-Udacity
docs: steps to configure a Linux server and deploy a python app to the server
# Linux Server Configuration

Live at [13.58.234.82][1] and [ec2-13-58-234-82.us-west-2.compute.amazonaws.com/][2]
Credentials for user `Grader` :
	user: grader
	password: grader17
	port: 2200

#### 1. Install & upgrade packages:
Ubuntu Docs: 
 -  `apt-get update`
 - `sudo apt-get upgrade`
 - `apt-get install unattended-upgrades`
 

#### 2. Install & configure apache
Udacity & Ubuntu: 
 - `apt-get install apache2`
 - `apt-get install python-setuptools libapache2-mod-wsgi`
 - `sudo nano /etc/apache2/conf-available/fqdn.conf` - add ServerName 18.58.234.82
 - `sudo a2enconf fqdn`
 - `sudo apt-get install libapache2-mod-wsgi python-dev`
 - `sudo a2enmod wsgi`
 - `sudo service apache2 restart`

#### 3. Clone and configure application
 - `cd /var/www/`
 - `sudo mkdir app`
 - `cd app`
 - `mkdir app`
 - `cd app`
 - `git clone https://github.com/constant17/itemcatalog.git`
 - `rm README.md`
 - `mv project.py __init__.py`
 - `mv * ../`
 - `cd ../`

#### 4. Install python packages
 - `sudo apt-get install python-pip`
 - `sudo pip install virtualenv`
 - `sudo python -m virtualenv  venv` 
 - `sudo chmod -R 777 venv`
 - `sudo pip install Flask-Seasurf`
 - `sudo pip install requests`
 - `sudo pip install --upgrade oauth2client`
 - `sudo pip install sqlalchemy`
 - `sudo apt-get install python-psycopg2`
 - `sudo pip install bleach`

#### 5. Configure application config file: 
 - `cd ../../../`
 - `sudo nano /etc/apache2/sites-available/app.conf`
 Copy and Paste the following code in the file: 
  "<VirtualHost *:80>
	      ServerAdmin admin@13.58.234.82
	      ServerAlias ec2-13-58-234-82.us-west-2.compute.amazonaws.com
	      WSGIScriptAlias / /var/www/app/app.wsgi
	      <Directory /var/www/app/app/>
	          Order allow,deny
	          Allow from all
	      </Directory>
	      Alias /static /var/www/app/app/static
	      <Directory /var/www/app/app/static/>
	          Order allow,deny
	          Allow from all
	      </Directory>
	      ErrorLog ${APACHE_LOG_DIR}/error.log
	      LogLevel warn
      CustomLog ${APACHE_LOG_DIR}/access.log combined
	</VirtualHost>"
 - `sudo a2ensite app`
 - `sudo nano app.wsgi`
 Copy and paste the following code in the file:
`#!/usr/bin/python
 import sys
 import logging
 logging.basicConfig(stream=sys.stderr)
 sys.path.insert(0,"/var/www/app/") 
 from app import app as application
 application.secret_key = "Add your secret key"`
 - `sudo service apache2 restart`

#### 6. Install & config postgres & instantiate db
Ubuntu Docs & Digital Ocean:
 - `sudo apt-get install postgresql postgresql-contrib`
 - `cd /etc/postgresql/9.3/main/`
 - `sudo su - postgres`
 - `psql`
 - `CREATE USER catalog WITH PASSWORD 'catalogpw';`
 - `ALTER USER catalog CREATEDB;`
 - `CREATE DATABASE appdb WITH OWNER catalog;`
 - `\c appdb`
 - `REVOKE ALL ON SCHEMA public FROM public;`
 - `GRANT ALL ON SCHEMA public TO catalog;`
 - `\q`
 - `ctrl + D`
 - `cd /var/www/app/app && mv /var/www/app/app/run.py var/www/app/app/__init__.py`
 - `sudo nano __init__.py` Change the following line
 engine = create_engine('sqlite:///category_items.db') to engine = create_engine('postgresql://catalog:password@localhost/appdb')
 Do the same thing for database_setup.py and lotsofitems.py
 - `python database_setup.py`
 - `python lotsofitems.py`
 - `python __init__.py`

#### 7. Monitor & ban abuse
Glances & Fail2ban: [1][13] & [2][14]
 - `apt-get install python-pip build-essential python-dev`
 - `pip install Glances`
 - `apt-get install lm-sensors`
 - `pip install PySensors`
 - `apt-get install fail2ban`
 - `cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local`
 - `vim /etc/fail2ban/jail.local`
  - set bantime  = 900
  - destemail = grader@localhost
 - `apt-get install sendmail iptables-persistent`
 - `service fail2ban restart`

#### 8. Add grader User
 -  `sudo adduser grader`
 -  `sudo nano /etc/sudoers.d/grader` add "grader ALL=(ALL:ALL) ALL"
 
#### 9. Generate SSH key for user grader
 - `sudo nano /etc/ssh/sshd_config` (Enable password login)
 - On local machine: `ssh-keygen` set path to `/home/ubuntu/.ssh/graderAccess`
 - `sudo su - grader`
 - `mkdir ~/.ssh`
 - `chmod 700 ~/.ssh`
 - `cat ~/graderAccess.pub >> ~/.ssh/authorized_keys`
 - `rm ~/graderAccess.pub`
 - `chmod 600 ~/.ssh/authorized_keys`
 - `ctrl + D`
 - `sudo nano /etc/ssh/sshd_config` (Change ssh to 2200, Enforce ssh key login, Don't permit root login)
 - `service ssh restart`

#### 10. Firewall configuration
Ufw Docs & Digital Ocean: 
 - `sudo ufw enable`
 - `sudo ufw allow 2200/tcp`
 - `sudo ufw allow 80/tcp`
 - `sudo ufw allow 123/udp`
 - `sudo service ufw restart`
 
 #### 11. Launch app
 - `cd /var/www/app/app`
 - `python __init__.py`
