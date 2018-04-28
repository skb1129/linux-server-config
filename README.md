# Linux Server Configuration

### Description

This repository shows how to deploy your Item-Catalog Flask application to a remote server using ssh service from your computer.

### Prerequisites

* I have used AWS(Amazon Web Services) to create a remote server.
* I have used my AWS account to create an instance of EC2 and used the private key provided by AWS to login to the server.
* The OS used on the server is Ubuntu 16.04 .
* The Public IPv4 is used to login using ssh.
* My Public IPv4 is <b>35.154.26.246</b>.

## Instructions

### Login using AWS private key

The following command is used to login to remote server as Administrator:
- `$ ssh -i aws_secret_key.pem ubuntu@35.154.26.246`

### Upgrades

Use the following commands to upgrade older packages:
- `$ sudo apt update`
- `$ sudo apt upgrade`

### Add new user

* Add new user <b>grader</b> `$ sudo adduser grader`. Set its password the then some attributes that are asked.
* Give <b>grader</b> sudo permission, run command:`$ sudo nano /etc/sudoers.d/grader` then add this to the file:`grader ALL=(ALL) NOPASSWD:ALL`.
* Run `$ sudo service ssh restart` to refresh ssh.

### Create new key-pair

We need to create a new RSA key pair for user <b>grader</b>. For this we will use the ssh-keygen tool.
* On you local machine run `$ ssh-keygen`.
* Give the key whatever name you like(eg. `grader`) but it is recommended to store the key in `~/.ssh/` directory.
* There will be two files generated, `~/.ssh/grader` and `~/.ssh/grader.pub`.

### Setting up key for grader account

* Switch to user <b>grader</b> using: `$ su - grader`.
* Create directory to store keys: `$ mkdir .ssh`.
* Create and edit file: `$ nano .ssh/authorized_keys` and add the contents of `~/.ssh/grader.pub` from your local machine to this file.
* Alter permissions for security reasons: `$ sudo chmod 700 .ssh` and `$ sudo chmod 644 .ssh/authorized_keys`.
* Run `$ sudo service ssh restart` to refresh ssh.

Now, you can login in to <b>grader</b> using:
- `$ ssh -i ~/.ssh/grader grader@35.154.26.246`

### Configure SSH

* Open ssh configuration file using: `$ sudo nano /etc/ssh/sshd_config`.
* Change default ssh port by changing line:
`Port 22` ---> `Port 2200`
* Disable root login by changing line:
`PermitRootLogin` ---> `PermitRootLogin no`
* Disable password login by changing line:
`PasswordAuthentication yes` ---> `PasswordAuthentication no`
* Save the file, then run `$ sudo service ssh restart` to refresh ssh.

Now, you can login in to <b>grader</b> using:
- `$ ssh -i ~/.ssh/grader grader@35.154.26.246 -p 2200`

### Configure UFW

* Install NTP package using: `$ sudo apt install ntp`.
* Run commands:
	- `$ sudo ufw disable`
	- `$ sudo ufw default deny incoming`
	- `$ sudo ufw default allow outgoing`
	- `$ sudo ufw allow 2200/tcp`
	- `$ sudo ufw allow www`
	- `$ sudo ufw allow ntp`
	- `$ sudo ufw enable`
* The firewall is now configured.

### Install Project Dependencies

* Install deployment dependencies using apt:
	- `$ sudo apt install apache2 libapache2-mod-wsgi postgresql git python-dev python-pip`
* Install project dependencies using pip:
	- `$ sudo pip install flask`
	- `$ sudo pip install httplib2`
	- `$ sudo pip install sqlalchemy`
	- `$ sudo pip install requests`
	- `$ sudo pip install psycopg2`
	- `$ sudo pip install oauth2client`

### Setup Postgresql

* Login to Postgres using: `$ sudo su - postgres`.
* Run command: `$ psql`.
* Create new database using: `postgres=# CREATE DATABASE catalog;`.
* Create new user using: `postgres=# CREATE USER catalog WITH PASSWORD 'password';`.
* Grant database permissions: `postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;`.
* Quit Postgres using: `postgres=# \q` then `$ exit`.

### Setting up Item-Catalog

* Change director: `$ cd /var/www/`.
* Create directory: `$ sudo mkdir moviecafe`.
* Goto new directory: `$ cd moviecafe`.
* Clone the project: `$ sudo git clone -b deployment https://github.com/skb1129/item-catalog.git`.
* Move the `app.wsgi` to parent directory: `$ sudo mv app.wsgi ../app.wsgi`.
* Move the `moviecafe.conf` file: `$ sudo mv moviecafe.conf /etc/apache2/sites-available/moviecafe.conf`.
* Enable the website using: `$ sudo a2ensite moviecafe`.
* Restart apache2 service: `$ sudo service apache2 reload`.

### FINISH

And, thats all we need to do.
Go to <b>35.154.26.246</b> or <b>ec2-35-154-26-246.ap-south-1.compute.amazonaws.com</b> on your browser and use the application.

## Contents of some files

### app.wsgi
```
#!/usr/bin/python
import os, sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/moviecafe/")

from moviecafe.app import app as application
application.secret_key = os.urandom(12)
```

### moviecafe.conf
```
<VirtualHost *:80>
	ServerName 35.154.26.246
	ServerAlias ec2-35-154-26-246.ap-south-1.compute.amazonaws.com
	ServerAdmin skbansal.cse15@chitkara.edu.in
	WSGIScriptAlias / /var/www/moviecafe/app.wsgi
	<Directory /var/www/moviecafe/moviecafe/>
		Order allow,deny
		Allow from all
	</Directory>
	Alias /static /var/www/moviecafe/moviecafe/static
	<Directory /var/www/moviecafe/moviecafe/static/>
		Order allow,deny
		Allow from all
	</Directory>
	ErrorLog ${APACHE_LOG_DIR}/error.log
	LogLevel warn
	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

## References

* Udacity
* StackOverflow
* [Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-set-up-apache-virtual-hosts-on-ubuntu-14-04-lts)


Contact:<br>
Surya Kant Bansal<br>
e-mail: skbansal.cse15@chitkara.edu.in
