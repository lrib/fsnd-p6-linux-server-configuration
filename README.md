Linux Server Configuration
=================================


Udacity - Full Stack Web Development Nanodegree
---------------------------------------------
Project 6: Linux Server Configuration

This project's goal is to set up a Linux server to host one of the web applications developed

## Server Basic Information:
- Public IP: 18.195.234.224
- SSH port: 2200
- App URL: http://18.195.234.224.xip.io/
- VM Ubuntu [Ubuntu 16.04];
- Python 2
- Putty;
- Apache 2;
- POSTGRESQL;
- Flask App

Configuration Steps:
------------

## Criar uma VM no Amazon:
- Create a VM on Amazon Lightsail [https://lightsail.aws.amazon.com/ls/webapp/home/instances];
- Create an instance of a Linux VM: [Create Instance] -> [Linux] -> [Ubuntu 16.04] -> [Create Instance]
- Wait for "running".

### Update all currently installed packages:

- Update command
```shell
$ sudo apt-get update
$ sudo apt-get upgrade
$ sudo apt-get dist-upgrade
```
- VM reboot
```shell
$ sudo reboot
```

### Update timezone:

```shell
$ tzselect
```
Open terminal and change the timezone. following the instruction.

### Create a new SSH key for segure connections:

- Using the Windows Gitbash, get a new key:
```sh
$ ssh-keygen
```
Save the key: /c/Users/<YOUR_WINDOWS_USERNAME>/.ssh/main_key
I've paste the contents of the grader user's SSH key into the "Notes to Reviewer" field.
### Add new user (Using AWS console):

- Using the SSH Amamzon Lightsail console:
```sh
$ sudo adduser grader
```
 - Assign and confirm a password for the new user
 - Enter any additional information about the new user. This is completely optional and can be ignored if you do not want to use these fields.
 - After then, you will be prompted to confirm that the information provided is correct. Enter Y to continue.

- Assign access to the newly created user.
```sh
$ sudo touch /etc/sudoers.d/grader
$ sudo nano /etc/sudoers.d/grader
```
- Add in the grader file the command: grader ALL=(ALL:ALL) ALL
- Send CTRL+O (salve), ENTER (confirm), CTRL+X (exit nano). 
- User grader already has sudo privilege.

### Allow secure user access to grader:
- Copy the public key generated on your local machine (main_key.pub) to this file and save
```sh
$ su - grader
$ mkdir .ssh
$ sudo touch .ssh/authorized_keys
$ sudo nano .ssh/authorized_keys
```
- Change permission for /.ssh folder and authorized_keys file.
```sh
$ chmod 700 .ssh
$ chmod 644 .ssh/authorized_keys
```
### Secure SSH access without the Amazon console:
- On your instance dashboard, Networking tab, add a Custom TCP application with Port range = 2200 to your instance's Firewall
- Use the command below to open the file and edit line 4 from Port 22 to Port 2200. Save and quit nano.
```sh
$ sudo nano /etc/ssh/sshd_config
```
- reload SSH using:
```sh
$ service ssh restart
```
- now you can use ssh to login with the new user you created, at your computer:
```sh
ssh -i [privateKeyFilename] grader@52.24.125.52
```
### Change the SSH port from 22 to 2200
- Use
```sh
$ sudo vim /etc/ssh/sshd_config
```
- and then change Port 22 to Port 2200 , save & quit.
- Now, use the command to enter server.
```sh
ssh grader@18.195.234.224 -i [privateKeyFilename] -p 2200
```
### Create Rules and Enable Uncomplicated Firewall (UFW) 
- Check the current UFW status
```sh
sudo ufw status
```
It's expected that the UFW is disabled;
- Block all incoming requests initially
```sh
$ sudo ufw default deny incoming
```
- Allow all outgoing connections:
```sh
$ sudo ufw default allow outgoing
```
- Allow SSH setting port to 2200
```sh
$ sudo ufw allow 2200/tcp
```
- Allow HTTP on its default port 80
```sh
$ sudo ufw allow www
```
- Allow incoming connection for NTP on port 123
```sh
$ sudo ufw allow ntp
```
- Enable firewall 
```sh
$ sudo ufw enable
```
- Check UFW states once again:
```sh
$ sudo ufw status
```
### Remove root user access and password authentication
- Open the right file by using
```sh
$ sudo nano /etc/ssh/sshd_config
```
- Change PermitRootLogin to no
```sh
PermitRootLogin no
PasswordAuthentication no
```
- Save the file before exiting

## Installing Apache, Python with mod_wsgi

- Install Apache 
```sh
$ sudo apt-get install apache2
```
- Install mod_wsgi 
```sh
$ sudo sudo apt-get install python-setuptools libapache2-mod-wsgi
```
- Restart Apache 
```sh
$ sudo service apache2 restart
```
- Install and configure PostgreSQL

- Install PostgreSQL 
```sh
$ sudo apt-get install postgresql
```
- Login as user "postgres" 
```sh
$ sudo su - postgres
```
- Get into postgreSQL shell ```sh psql ```

- Create a new database named project and create a new user named project in postgreSQL shell
```sh
postgres=# CREATE DATABASE project;
postgres=# CREATE USER project;
```
- Set a password for user project
```sh
postgres=# ALTER ROLE project WITH PASSWORD 'password';
```
- Give user "project" permission to "project" application database
```sh
postgres=# GRANT ALL PRIVILEGES ON DATABASE project TO project;
```
- Quit postgreSQL 
```sh
postgres=# \q
```
- Exit from user "postgres"
```sh
$ exit
```
- Install Git using 
```sh
sudo apt-get install git
```
- Go to var directory
```sh
cd /var/www
```
- Create the application directory
```sh
sudo mkdir FlaskApp
cd FlaskApp
```
- Clone the Catalog App (Branch: lrib-ubuntu-server-1)  to the virtual machine
- This new branch has 
```sh
sudo git clone -b lrib-ubuntu-server-1 https://github.com/lrib/Udacity_catalog_app-4
```
- Rename the project's name and Move to the inner FlaskApp directory
```sh
sudo mv ./Udacity_catalog_app-4 ./FlaskApp
cd FlaskApp
```
- Rename project.py to __init__.py using sudo mv project.py __init__.py
```sh
sudo mv project.py __init__.py
```
- Install pip 

```sh
sudo apt-get install python-pip
```
- Use pip to install dependencies
```sh
sudo pip install -r requirements.txt
```
- Install psycopg2
```sh
sudo apt-get -qqy install postgresql python-psycopg2
```
## Configure and Enable a New Virtual Host

- Create FlaskApp.conf to edit
```sh
sudo nano /etc/apache2/sites-available/FlaskApp.conf
```
- Add the following lines of code to the file to configure the virtual host.
```sh
<VirtualHost *:80>
        ServerName 18.195.234.224
        ServerAlias 18.195.234.224.xip.io
        ServerAdmin lribeirojunior@gmail.com
        WSGIScriptAlias / /var/www/FlaskApp/FlaskApp/flaskapp.wsgi
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
CTRL+O (salvar), ENTER (confirmar), CTRL+X (sair do nano).

- Enable the virtual host with the following command
```sh
sudo a2ensite FlaskApp
```
## Create the .wsgi File
- Create the .wsgi File under /var/www/FlaskApp:
```sh
cd /var/www/FlaskApp/FlaskApp
sudo nano flaskapp.wsgi 
```
- Add the following lines of code to the flaskapp.wsgi file:
```sh
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/FlaskApp/FlaskApp")

from __int__ import app as application
application.secret_key = 'your_secret_key'
```
- Restart Apache
```sh
sudo service apache2 restart
```
## - The error and accees log
-  You can access a log for this application with the following command:
- Error
```sh
sudo tail -f /var/log/apache2/error.log
```
- Acess
```sh
sudo tail -f /var/log/apache2/access.log
```

Resultado
---------

Fontes consultadas
---------

https://github.com/jungleBadger/-nanodegree-linux-server
https://askubuntu.com/questions/1019891/connecting-to-amazon-lightsail-ubuntu-server-using-different-ssh-port
https://askubuntu.com/questions/27559/how-do-i-disable-remote-ssh-login-as-root-from-a-server
https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2



