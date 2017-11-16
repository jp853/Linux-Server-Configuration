# __Project Goals__

The purpose of this project was to take a baseline installation of a Linux distribution on a virtual machine and prepare it to host a web application. It shall include installing updates, security from a number of attack vectors and installing/configuring web and database servers.

# __Info for the grader__

IP Address: 34.215.68.109

Accessible SSH Port: 2200

App URL: <http://ec2-34-215-68-109.us-west-2.compute.amazonaws.com>

# __Step by Step Walthrough__

### 1. Login to AWS instance and pdate and Upgrade Installation Packages
Download AWS Key Pair and move it to local .ssh directory:

    mv ~/Downloads/'yourKeyPair.pem' ~/.ssh/yourKeyPair.pem

Alter yourKeyPair.pem permissions:

    chmod 400 ~/.ssh/yourKeyPair.pem

Login to AWS instance console:

    $ cd ~/.ssh

    $ ssh ubunut@34.215.68.109 -i yourKeyPair.pem


Will upgrade `apt-get` automatically if the update is succesful:

    $ sudo apt-get update && sudo apt-get upgrade

Automatically mangage packet updates

    $ sudo apt-get install unattended-upgrades

enable the unattended upgrades

    $ sudo dpkg-reconfigure --priority=low unattended-upgrades

Install Finger:

    $ sudo apt-get install finger

Source: [Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps)

### 2. Add `grader` and grand `sudo` access
`$ sudo adduser grader`
Give 'grader' a user name and set password

Test if 'grader' was succesfully created with:

    finger grader

Give 'grader' sudo access:

    sudo visudo

Under `root (ALL:ALL)` add the following:

    grader (ALL:ALL) ALL


### 3. Configure the key-based authentications for user `grader`

On local machine:

`$ sudo ssh-keygen -f ~/.ssh/udacity_key.rsa` then set passphrase

Copy contents of your new keygen:

`$ sudo cat ~/.ssh/udacity_key.rsa.pub` highlight and copy

Login to your AWS instance as grader:

`$ ssh grader@34.215.68.109 -i yourKeyPair.pem`

Make a new directory '.ssh' make an 'authorized_keys' file:

    $ sudo mkdir /home/grader/.ssh

    $ sudo touch /home/grader/.ssh/authorized_keys

Paste contents from your new keygen:

`$ sudo nano /home/grader/.ssh/authorized_keys` and paste the contents of udacity_key.rsa.pub into the file

Change permissions of the directory and file:

    $ sudo chmod 700 /home/grader/.ssh

    $ sudo chmod 644 /home/grader/.ssh/authorized_keys

    $ sudo chown -R grader:grader /home/grader/.ssh

You can now login to the AWS instance with:

`$ ssh -i ~/.ssh/udacity_key.rsa grader@34.215.68.109` Input passphrase if you set one.

### 4. Enforce Key-Based Authentication

Login as grader with:

`$ ssh -i ~/.ssh/udacity_key.rsa grader@34.215.68.109` Input passphrase if you set one.

Open PasswordAuthentication file

`$ sudo nano /etc/ssh/sshd_config`

Find `PasswordAuthentication` and change `yes` to `no`.

Restart ssh to reset the PasswordAuthentication file:

    $ sudo service ssh restart

### 5. Change the SSH port from 22 to 2200

Open sshd_config file and edit the port line:

`$ sudo nano /etc/ssh/sshd_config`

Look for `# What ports, IPs and protocols we listen for` and change `22` to `2200`.

In the AWS account under the networking tab, add `custom tcp 2200` and delete `ssh tcp 22`.

Restart the ssh service to update the port file:

    $ sudo service ssh restart

You can now login with:

    ssh -i ~/.ssh/udacity_key.rsa -p 2200 grader@34.215.68.109

### 6. Configure UTC TimeZone

`$ sudo dpkg-reconfigure tzdata` select `none` then `utc`

Source: [UbuntuTime](https://help.ubuntu.com/community/UbuntuTime)

### 7. Disable 'root' login
`$ sudo nano /etc/ssh/sshd_config` set `PermitRootLogin` to `no`.

    $ sudo service ssh restart

Source: [AskUbuntu](https://askubuntu.com/questions/27559/how-do-i-disable-remote-ssh-login-as-root-from-a-server)

### 8. Configure the Uncomplicated Fire Wall (UFW)
Enable the UFW:

    $ sudo ufw enable

Check status of UFW:

    $ sudo ufw status verbose

Allow incoming TCP packets on port 2200 (ssh):

    $ sudo ufw allow 2200/tcp

Allow incoming TCP packets on port 80 (http):

    $ sudo ufw allow 80/tcp

Allow incoming UDP packets on port 123 (NTP):

    $ sudo ufw allow 123/udp

Check status of UFW:

    $ sudo ufw status verbose

Source: [Ubuntu](https://help.ubuntu.com/community/UFW)

### 8. Install Apache and mod_wsgi

    $ sudo apt-get install apache 2

    Install mod_wsgi for serving Python apps from Apache and the helper package python-setuptools

    $ sudo apt-get install python-setuptools

    $ sudo apt-get install libapache2-mod-wsgi

    Enable mod_wsgi:

    $ sudo a2enmod wsgi

    $ sudo service apache2 restart

    You can now visit http://34.215.68.109 and the page will display Apache2's 'It Works' page.

### 9. Setup Git

    $ sudo apt-get install git

    $ sudo git config --global user.name "jpingatore"

    $ sudo git cinfig --global user.email "jpingatore@gmail.com"

Source: [Github](https://help.github.com/articles/set-up-git/#platform-linux)

### 10. Clone Item-Catalog App from Github and configure virtual host and wsgi

Change directory into the /var/www file to set up flask application:

    $ sudo cd /var/www

Make a new directory for the flask application:

    $ sudo mkdir catalog

Change owner for catalog folder:

    $ sudo chown -R grader:grader catalog

Change directory into the new 'catalog' folder and clone item-catalog project from github

    $ cd catalog

    $ sudo git clone https://github.com/jp853/item-cat-for-linux.git catalog

Move all the files that focus on the function of the application to the `/var/www/catalog/catalog` folder:

    $ sudo mv /var/www/catalog/catalog/vagrant/item-final/* /var/www/catalog/catalog

Delete the rest of the git file:

    $ cd /var/www/catalog/catalog

    $ sudo rm -rf vagrant

Change name of main python file to __init__.py:

    $ sudo mv projectfinal.py __init__.py

Configure and enable virtual host by creating config file:

    $ sudo nano /etc/apache2/sites-available/catalog.conf

Insert the following lines of code:

```
<VirtualHost *:80>
    ServerName mywebsite.com
    ServerAdmin admin@mywebsite.com
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

Enable virtual host:

    $ sudo a2ensite catalog

Change Directory to /var/www/catalog and make a catalog.wsgi file and paste the following:

    $ cd /var/www/catalog

    $ sudo nano catalog.wsgi

Insert the following into catalog.wsgi

    import sys
    import logging
    activate_this ='/var/www/catalog/venv/bin/activate_this.py'
    execfile(activate_this, dict(__file__=activate_this))
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0, "/var/www/catalog/")

    from catalog import app as application
    application.secret_key='super_secret'

Source: [Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-run-django-with-mod_wsgi-and-apache-with-a-virtualenv-python-environment-on-a-debian-vps)


### 11. Configure Flask Dependencies and the Virtual Env

Install `pip` so python packages can be installed:

    $ sudo apt-get install python-pip

Install the virtual environment:

    $ sudo apt-get install virtualenv

Change directory into catalog folder and make a new virtual environment:

    $ cd /var/www/catalog

    $ sudo virtualenv venv

Activate the virtual environment:

    $ sudo source venv/bin/activate

Adjust permissions to venv:

    $ sudo chmod -R 777 venv

Install Flask and other python dependencies:

    $ sudo pip install Flask

    $ sudo pip install httplib2 requests sqlalchemy

    $ sudo pip install Flask-SQLAlchemy Flask-WTF html5lib

    $ sudo pip install --upgrade oauth2client

Install Python PostgreSQL psycopg:

    $ sudo apt-get install python-psycopg2

Source: [Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)

### 12. Install and Setup PostgreSQL

Install packages for working with postgreSQL:

    $ sudo apt-get install libpq-dev python-dev

Install PostgreSQL:

    $ sudo apt-get install postgresql postgresql-contrib

Check to make sure no remote connections are allowed. This is usually a default.

    $ sudo nano /etc/postgresql/9.3/main/pg_hba.conf

Change users to 'postgres' and connect to the database:

    $ sudo su - postgres

    $ psql

Create user 'catalog' and set password:

    postgres=# CREATE USER catalog WITH PASSWORD 'password';

Give 'catalog' the ability to CREATEDB:

    postgres=# ALTER USER catalog CREATEDB;

Make user 'catalog' the owner of the catalog database:

    postgres=# CREATE DATABASE catalog WITH OWNER catalog;

Connect to the database:

    postgres=# \c catalog

Revoke Rights:

    catalog=# REVOKE ALL ON SCHEMA public FROM public;

Let 'catalog' create tables:

    catalog=# GRANT ALL ON SCHEMA public TO catalog;

Exit the Database and PostgreSQL:

    catalog= # \q

    $ exit

Edit database_setup.py and __init__.py to use PostgreSQL:

Change to `engine` to `engine = create_engine('postgresql://catalog:password@localhost/catalog')` in both database_setup.py and __init__.py

Setup the database:

    $ python /var/www/catalog/catalog/database_setup.py

Source: [Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)

### 13. Update OAuth authorized JavaScript origins

Change the authorized URI to <http://ec2-34-215-68-109.us-west-2.compute.amazonaws.com> on Google developer dashboard.

Source: Udacity and [Apache](http://httpd.apache.org/docs/2.2/en/vhosts/name-based.html)

### 14. Install System Monitor

    $ sudo apt-get install glances

Source: [Glances](http://glances.readthedocs.io/en/stable/quickstart.html)

### 15. Restart Apache and Launch App

    $ sudo service apache2 restart

Visit <http://34.215.68.109> to view the app

