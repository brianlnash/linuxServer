<h1>Get your server.</H1>

<h3>1. Start a new Ubuntu Linux server instance on Amazon Lightsail. There are full details on setting up your Lightsail instance on the next page.</h3>

Created using Amazon Lightsail

<h3>2. Follow the instructions provided to SSH into your server.</H3>

ssh -i ubuntu@35.163.87.91

<h1>Secure your server.</h1>
<h3>3. Update all currently installed packages.</h3>

sudo apt-get update
sudo apt-get upgrade

<h3>4. Change the SSH port from 22 to 2200. Make sure to configure the Lightsail firewall to allow it.</h3>h3>

Changed initially on Amazon Lightsail
Additional change made through sudo nano /etc/ssh/sshd_config

<h3>5. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).</h3>

sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow www
sudo ufw allow 2200/tcp
sudo ufw allow ntp

sudo ufw enable

Warning: When changing the SSH port, make sure that the firewall is open for port 2200 first, so that you don't lock yourself out of the server. Review this video for details! When you change the SSH port, the Lightsail instance will no longer be accessible through the web app 'Connect using SSH' button. The button assumes the default port is being used. There are instructions on the same page for connecting from your terminal to the instance. Connect using those instructions and then follow the rest of the steps.


<h1>Give grader access.</h1>
In order for your project to be reviewed, the grader needs to be able to log in to your server.

<h3>6. Create a new user account named grader.</h3>

sudo adduser grader
add password and left remaining fields blank

<h3>7. Give grader the permission to sudo.</h3>

sudo nano /etc/sudoers.d/grader ALL= (ALL:ALL) ALL

<h3>8. Create an SSH key pair for grader using the ssh-keygen tool.</h3>

ssh-keygen
cp ~/linuxCourse.pub /home/grader/.ssh/authorized_keys

<h1>Prepare to deploy your project.</h1>
<h3>9. Configure the local timezone to UTC.</h3>

time  (this was already set properly)

<h3>10. Install and configure Apache to serve a Python mod_wsgi application.</h3>

sudo apt-get install apache2
sudo apt-get install libapache2-mod-wsgi python-dev

<h3>11. Install and configure PostgreSQL:</h3>

sudo apt-get install poatgresql postgresql-contrib

<h1>Do not allow remote connections</h1>

Create a new database user named catalog that has limited permissions to your catalog application database.

Login as user "postgres" sudo su - postgres

Get into postgreSQL shell psql

Create a new database named catalog and create a new user named catalog in postgreSQL shell

postgres=# CREATE DATABASE catalog;
postgres=# CREATE USER catalog;
Set a password for user catalog

postgres=# ALTER ROLE catalog WITH PASSWORD 'password';
Give user "catalog" permission to "catalog" application database

postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
Quit postgreSQL postgres=# \q

Exit from user "postgres"

exit

<h3>12. Install git.</h3>

sudo apt-get install git

<h1>Deploy the Item Catalog project.</h1>
<h3>13. Clone and setup your Item Catalog project from the Github repository you created earlier in this Nanodegree program.</h3>

Use cd /var/www to move to the /var/www directory
Create the application directory sudo mkdir FlaskApp
Move inside this directory using cd FlaskApp
Clone the Catalog App to the virtual machine git clone https://github.com/brianlnash/item_catalog.git
Rename the project's name sudo mv ./item_catalog ./FlaskApp
Move to the inner FlaskApp directory using cd FlaskApp
Rename item_catalog.py to __init__.py using sudo mv website.py __init__.py
Edit item_database_config.py and database_operations.py  and change engine = create_engine('item_catalog.db') to engine = create_engine('postgresql://catalog:(catalog password)@localhost/catalog')

install Flask
sudo apt-get install python-virtualenv

install PIP

sudo apt-get install pip

<h3>14. Set it up in your server so that it functions correctly when visiting your server’s IP address in a browser. Make sure that your .git directory is not publicly accessible via a browser!</h3>

Create FlaskApp.conf:

<VirtualHost *:80>
        ServerName 35.163.87.91
        ServerAdmin brianlnash@gmail.com
        ServerAlias ec2-35-163-87-91.us-west-2.compute.amazonaws.com
        WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
        <Directory /var/www/FlaskApp/FlaskApp/>
                Order allow,deny
                Allow from all
        </Directory>
        Alias /static /var/www/FlaskApp/FlaskApp/static
        <Directory /var/www/FlaskApp/FlaskApp/static/>
                Order allow,deny
                Allow from all
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

Enable the virtual host:

sudo a2ensite FlaskApp

Add WSGI file:

#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/FlaskApp/")

from FlaskApp import app as application

application.secret_key = '(enter secret key here)'

# linuxServer


<h3> Resources</h3>
https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
http://wsgi.readthedocs.io/en/latest/
