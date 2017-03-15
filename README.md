Udacity-Linux Server Configuration
================================

**Live Demo**
http://52.4.191.30/

**Project Requirements**

> You will take a baseline installation of a Linux distribution on a
> virtual machine and prepare it to host your web applications, to
> include installing updates, securing it from a number of attack
> vectors and installing/configuring web and database servers.

**Info**
IP Address: 52.4.191.30
SSH Port: 2200

## Configurations: ##

 
 I. **Create user "grader":**
1. Logged into Amazon Lightsail VM as root user via SSH
2. Added a new user called "grader"
3. Created a new file under the sudoers directory: `$ sudo nano /etc/sudoers.d/grader`. Created file with the following content: `grader ALL=(ALL:ALL) ALL`.

II. **Update packages**
1. `sudo apt-get update` to pull updates
2. `sudo apt-get upgrade` to upgrade the packages

III. **Configure TimeZone**
1. `sudo dpkg-reconfigure tzdata` to set UTC and EST time

IV. **Cofigure UFW Firewall**
1. `sudo ufw allow 2200`
2. `sudo ufw allow 80`
3. `sudo ufw allow 123`
4. `sudo ufe enable`

V. **Key-based authentication**
1.  Downloaded Amazon Lightsails's generated key
2.  Placed file  locally in `~/.ssh/authorized_keys.pem`
3. Created blank file on server to store key in `/home/grader/.ssh/authorized_keys`
4. Copied the key to the `authorized_keys file`
5. Changed permssions on the sever:
    * `sudo chmod 700 /home/grader/.ssh` 
    * `sudo chmod 644 /home/grader/.ssh/authorized_keys`
6. Now you can log in remotely as grader via `ssh grader@52.4.191.30 -i ~/.ssh/authorized_keys.pem`

VI. **Enforce key-based authentication**
1. `sudo nano /etc/ssh/sshd_config`
2. `sudo service ssh restart`

VII. **Change SSH Port**
1. `sudo nano /etc/ssh/sshd_config`
2. Change `Port` to `2200`
3. `sudo ssh service restart`
4.  Login using port 2200 `ssh grader@52.4.191.30 -p 2200 -i ~/.ssh/authorized_keys.pem`

VIII. **Disable ssh login for root**
1. `sudo nano /etc/ssh/ssh_config`
2. Change `PermitRootLogin` to `no`
3. `sudo service ssh restart`

IX: **Setup Apache**
1. `sudo apt get install apache2`
2. `sudo apt-get install libapache2-mod-wsgi python-dev`
3. `sudo a2enmod wsgi`
4. `sudo service apache2 start`

X: **Install git to clone project repo**
1. `sudp apt-get install git`
2. login to github account
3. `git config --global user.name <username>`
4. `git config --global user.email <email>`

XI: **Clone Catalog project App**
1. `cd /var/www`
2. `sudo mkdir FlaskApp`
3. `cd FlaskApp`
4. `git clone https://github.com/glc3344/udacity-itemcatalog.git`
5. Create blank file called 'flaskpapp.wsgi'
6. Add to 'flaskapp.wsgi'
```#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/FlaskApp/")

from FlaskApp import app as application
application.secret_key = 'super secret key'
```
XII: **Configure Virtual Host**
1. `sudo nano /etc/apache2/sites-available/FlaskApp`
```
<VirtualHost *:80>
    ServerName 52.4.191.30
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
```
XIII: **Install and configure PostgreSQL**

1. Install some necessary Python packages for working with PostgreSQL: `$ sudo apt-get install libpq-dev python-dev`.
2. Install PostgreSQL: `$ sudo apt-get install postgresql postgresql-contrib`.
3. Change the user with: `$ sudo su - postgres`, then connect to the database system with `$ psql`.
4. Create a new user called 'catalog' with his password: `# CREATE USER catalog WITH PASSWORD 'sillypassword';`.
5. Give *catalog* user the CREATEDB capability: `# ALTER USER catalog CREATEDB;`.
6. Create the 'catalog' database owned by *catalog* user: `# CREATE DATABASE catalog WITH OWNER catalog;`.
7. Connect to the database: `# \c catalog`.
8. Revoke all rights: `# REVOKE ALL ON SCHEMA public FROM public;`.
9. Lock down the permissions to only let *catalog* role create tables: `# GRANT ALL ON SCHEMA public TO catalog;`.
10. Log out from PostgreSQL: `# \q`. Then return to the *grader* user: `$ exit`.
11. Inside the Flask application, the database connection is now performed with: 
```python
engine = create_engine('postgresql://catalog:password@localhost/catalog')
```
XIV: **Install requirement packages and change Google authroized URI**
- Install pip with `sudo apt-get install python-pip`
- Install Flask `pip install Flask`
- Install other project dependencies `sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils`

XV: **Restart apache for changes to effect**
1. `sudo service apache2 restart`

**DONE!**

thanks to [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps) on deploying my flask app

thanks to [rrjoson](https://github.com/rrjoson/udacity-linux-server-configuration/blob/master/README.md) and [iiliketomatoes](https://github.com/iliketomatoes/linux_server_configuration) for help with the readme

