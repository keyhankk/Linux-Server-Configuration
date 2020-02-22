# Linux Server Project

# 1. Log in First Time

Save the private key from your profive and use it to log in into the Linux Machine:
```sh
$ ssh -i LightsailDefaultKey-us-west-2.pem -p 20 ubuntu@34.220.238.242
```
# 2. Update and Upgrade Packages:

```sh
$ sudo apt-get update
$ sudo apt-get upgrade
```

# 3. Configure SSH

Edit the file /etc/ssh/sshd_config using:

```sh
$ sudo nano /etc/ssh/sshd_config
```

Change line
``` sh
# What ports, IPs and protocols we listen for
Port 22
```
to:
``` sh
# What ports, IPs and protocols we listen for
Port 2200
```

# 4. Setup the UFW

```sh
$ sudo ufw status
$ sudo ufw default deny incoming
$ sudo ufw default allow outgoing
$ sudo ufw allow 2200/tcp
$ sudo ufw allow 80/tcp
$ sudo ufw deny 22
```

Enable the UFW
```sh
sudo ufw enable
```
Check the status to be ready:
```sh
sudo ufw status
```

Add the following port in the Lightsail Amazon Website as a TCP Port.

```sh
Custom	TCP	2200
```

Now you can exit and log in back on port 2200:

```sh
$ exit
$ ssh -i LightsailDefaultKey-us-west-2.pem -p 2200 ubuntu@34.220.238.242
```

# 5. Setup Grader Account


Edit the sshd_config


```sh
sudo nano /etc/ssh/sshd_config
```

Set the follwoing:

```sh
Set PasswordAuthentication to no
Set PermitRootLogin to no
```

Now Adding teh garder user:

```sh
sudo adduser grader
sudo nano /etc/sudoers.d/grader
```
Add the following line to the grader file:

```sh
grader ALL=(ALL:ALL) ALL
```
Run the following to generate the public and private keys on your local machine, specify teh name as grader, it will create two files: 'grader' and 'grader.pub'

```sh
ssh-keygen
```
On the remote server run the followings:

```sh
$ su - grader
$ mkdir .ssh
$ sudo touch .ssh/authorized_keys
$ sudo nano .ssh/authorized_keys
```
Copy paste the contents of grader.pub into this file (authorized_keys).

Then log out.

```sh
$ service ssh restart
$ exit
```

Log in using the follwoing:

```sh
ssh -i grader -p 2200 grader@34.220.238.242
```

# 6. Setting UP Apache

Run the following commands to install Python, Apache Server, and git please note that it installs Python 2.

```sh
$ sudo apt-get install python
$ sudo apt-get install apache2 to install apache.
$ sudo apt-get install python-setuptools libapache2-mod-wsgi
$ sudo apt-get install git
```

Add the following file:

```sh
$ sudo nano /etc/apache2/sites-available/catalog.conf
```

Edit it with the following contents:

```sh
<VirtualHost *:80>
    ServerName 34.220.238.242
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi
    <Directory /var/www/catalog/>
        Allow from all
        Require all granted
    </Directory>
    Alias /static /var/www/catalog/static
    <Directory /var/www/catalog/static/>
          Allow from all
          Require all granted
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Change the following lines in Application.py
```sh
if __name__ == '__main__':
  app.secret_ley = 'super_secret_key'
  app.debug = True
  app.run(host='34.220.238.242', port=80)
```

# 7. Installing and configuring the Postresql

```sh
$ sudo apt-get install libpq-dev python-dev
$ sudo apt-get install postgresql postgresql-contrib
$ sudo su - postgres
```

Create the user catalog with password: 'welcome1'

```sh
CREATE USER catalog WITH PASSWORD 'welcome1';
ALTER USER catalog CREATEDB;
CREATE DATABASE catalog WITH OWNER catalog;
\c catalog
REVOKE ALL ON SCHEMA public FROM public;
GRANT ALL ON SCHEMA public TO catalog;
\q
exit
```

Change the follwoing lines in application.py, databse_setup.py, and lotsofmenus.py:
```sh
engine = create_engine('sqlite:///itemcategory.db')
```
with:
```sh
engine = create_engine('postgresql://catalog:welcome1@localhost/catalog')
```

Run the following to make and populate the databse:
```sh
$ sudo python database_setup.py
$ sudo python lotsofmneus.py
```

# 8. Checking the Website

Reload and restart the Apache Server to view the website:

```sh
$ sudo service apache2 reload
$ sudo service apache2 restart
```

The website is located at:
```sh
http://www.34.220.238.242.xip.io/
```
