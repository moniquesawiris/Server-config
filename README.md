# Server-config
IP Address:52.56.54.92

SSH Port:2200
URL:52.56.54.92

Steps Followed to Configure the server
Create instance on amazon Lightsail 

1. Update all packages
sudo apt-get update
sudo apt-get upgrade
sudo apt-get dist-upgrade
Enable automatic security updates

sudo apt-get install unattended-upgrades
sudo dpkg-reconfigure --priority=low unattended-upgrades

Optional 2. Change timezone to UTC and Fix language issues
sudo timedatectl set-timezone UTC
sudo update-locale LANG=en_US.utf8 LANGUAGE=en_US.utf8 LC_ALL=en_US.utf8

3. Create a new user grader and Give him sudo access
sudo adduser grader
sudo nano /etc/sudoers.d/grader 
#Then add the following text grader ALL=(ALL) ALL

4. Setup SSH keys for grader
#On local machine ssh-keygen Then choose the path for storing public and private keys
#On remote machine home as user grader
sudo su - grader
mkdir .ssh
touch .ssh/authorized_keys 
sudo chmod 700 .ssh
sudo chmod 600 .ssh/authorized_keys 
nano .ssh/authorized_keys 
#Then paste the contents of the public key created on the local machine

5. Change the SSH port from 22 to 2200 | Enforce key-based authentication | Disable login for root user
sudo nano /etc/ssh/sshd_config
#Then change the following:

#Find the Port line and edit it to 2200.
#Find the PasswordAuthentication line and edit it to no.
#Find the PermitRootLogin line and edit it to no.

#Save the file and run sudo service ssh restart

6. Configure the Uncomplicated Firewall (UFW)
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2200/tcp
sudo ufw allow www
sudo ufw allow ntp
sudo ufw allow 8000/tcp  `serve another app on the server`
sudo ufw enable

7. Install Apache2 and mod-wsgi for python3 and Git
sudo apt-get install apache2 libapache2-mod-wsgi-py3 git

8. Install and configure PostgreSQL
sudo apt-get install libpq-dev python3-dev
sudo apt-get install postgresql postgresql-contrib
sudo su - postgres
psql

#Then

CREATE USER catalog WITH PASSWORD 'password';
CREATE DATABASE catalog WITH OWNER catalog;
\c catalog
REVOKE ALL ON SCHEMA public FROM public;
GRANT ALL ON SCHEMA public TO catalog;
\q
exit

#Note: In your catalog project you should change database engine to

engine = create_engine('postgresql://catalog:password@localhost/catalog')

9. Clone the Catalog app from GitHub and Configure it
cd /var/www/
sudo mkdir catalog
sudo chown grader:grader catalog
git clone <your_repo_url> catalog
cd catalog
git checkout production # If you have a diffrent branch!
nano catalog.wsgi
Then add the following in catalog.wsgi file

#!/usr/bin/python3
import sys
sys.stdout = sys.stderr

activate_this = '/var/www/catalog/env/bin/activate_this.py'
with open(activate_this) as file_:
    exec(file_.read(), dict(__file__=activate_this))
# -------

sys.path.insert(0,"/var/www/catalog")

from app import app as application


10. Configure apache server
sudo nano /etc/apache2/sites-enabled/000-default.conf
#Then add the following content:

# serve catalog app
<VirtualHost *:80>
  ServerName <52.56.54.92>
  ServerAdmin <monique.sawiris@gmail.com>
  DocumentRoot /var/www/catalog
  WSGIDaemonProcess catalog user=grader group=grader
  WSGIScriptAlias / /var/www/catalog/catalog.wsgi

  <Directory /var/www/catalog>
    WSGIProcessGroup catalog
    WSGIApplicationGroup %{GLOBAL}
    Require all granted
  </Directory>

  ErrorLog ${APACHE_LOG_DIR}/error.log
  LogLevel warn
  CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>


11. Reload & Restart Apache Server
sudo service apache2 reload
sudo service apache2 restart

#Resources
Amazon EC2 Linux Instances
Flask mod_wsgi (Apache)
Apache Server Configuration Files
Deploy a Flask Application on an Ubuntu VPS
Set Up Apache Virtual Hosts on Ubuntu
mod_wsgi documentation
Automatic Security Updates
Protect SSH with Fail2Ban
UFW with Fail2ban
Fix locale issue
Ask Ubuntu
Stack Overflow
