# Linux Server Configuration

> By: Mohamed Fawzy

## About

This is a project for the Udacity Full Stack Nanodegree Program.
A Linux Server Configuration for hosting a Flask application like [Item Catalog](https://github.com/Mohamed-fawzi/Item-Catalog).


## Server info

Address:`http://35.177.162.124/`
ssh: 2200

## Prerequisites

- Ubuntu18.04 LTS instance on [AWS LightSail](https://lightsail.aws.amazon.com/).
- Logged in using LightSail.

## Getting Started

#### Update
- Update ubuntu packages using apt.

```bash
sudo apt update && sudo apt upgrade && sudo apt full-upgrade
```

#### Create a new user

Create new user `grader`

```bash
sudo adduser gader
```

Add `grader` to the `sudo` group

```bash
sudo usermod -aG sudo grader
```

Switch to `grader` acount

```bash
su grader
```

#### Create ssh key

##### On the local machine

- Create ssh pair of keys using `ssh-keygen`, enter the name of the key e.g. `grader`

##### On the server

- Create a new directory `/.ssh ` in the `grader` home directory

```bash
mkdir .ssh
```

- Change `.ssh` permissions

```bash
chmod 700 .ssh
```

- Change the directory to get in `.ssh` directory

```bash
cd .ssh
```

- Create `authorized_keys` file and copy the content of the `grader.pub` into `authorized_keys`

```bash
nano authorized_keys
```

- Change `authorized_keys` permissions

```bash
chmod 644 authorized_keys
```

#### Edit the configuration file

- First, Edit the  [LightSail](https://lightsail.aws.amazon.com/) instance.

```bash
Head to your instance - >
                     Networking ->
                            Firewall ->
                             Add another->
                              2200/tcp as custom port.
                              save
```

- Second, Edit the `sshd_config` file.

```bash
sudo nano /etc/ssh/sshd_config
```

###### Disable login using the password

- Update `PasswordAuthentication` to be `no`.

###### Disable the root login

- Update `PermitRootLogin` to be `no`.

###### Force login using Public key

- Update `PubkeyAuthentication` to be `yes`.

###### Change the default port to 2200

- Update `port` to be `2200`.

###### Save the changes
- `ctl + o`
- `ctl + x`

###### Restart the ssh
- ```bash sudo service ssh restart ```

After the termination of the connection, Connect again using the `Pulic Key`


```bash
ssh grader@server-ip-address -p 2200 -i ~/.ssh/grader
```

#### Configure  the Firewall

- Using Ubuntu 18.04 UFW firewall

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2200/tcp
sudo ufw allow www
sudo ufw allow ntp
sudo ufw enable
```

#### Configure the timezone
- Using `dpkg-reconfigure`

```bash
sudo dpkg-reconfigure tzdata
```

- Choose `none of the above` then `UTC`.

#### Install Apache

```bash
sudo apt install apache2
```

#### Install Python mod_wsgi package

```bash
sudo apt install libapache2-mod-wsgi
```

- Enale mod_wsgi

```bash
sudo a2enmod wsgi
```

#### Install Apache

```bash
sudo apt install libpq-dev python-dev postgresql postgresql-contrib
```

#### Clone the flask app

Using  [item catalog](https://github.com/Mohamed-fawzi/Item-Catalog) application for this example.

- Create a new directory for the project in `/var/www/`

```bash
sudo mkdir /var/www/catalog
```

- Change the owner of the new directory to `grader`

```bash
sudo chown -R grader:grader /var/www/catalog
```

- Change the working `cd /var/www/catalog`.

- Clone the app from github

```bash
git clone https://github.com/Mohamed-fawzi/Item-Catalog.git && mv item-catalog catalog
```

- Create a new branch

```bash
cd catalog && checkout -b deployment
```

- Change `project.py` file to `__init__.py`

```bash
mv project.py __init__.py
```

- Make `.git` inaccessible

```bash
cd /var/www/catalog/
sudo nano .htaccess
```

- Add `RedirectMatch 404 /\.git` then save.

### Install requirement

```bash
sudo apt install python-pip
sudo pip install Flask httplib2
sudo pip install oauth2client sqlalchemy sqlalchemy_utils
sudo pip install requests
sudo pip install psycopg2
```

### Create Database

- Login as postgres user

```bash
su postgres
```

- Cahnge the working directory to the app file

```bash
cd /var/www/catalog/catalog
```

- Open PostgreSQL shell

```bash
psql
```

- Create a new user `catalog`

```shell
CREATE USER catalog WITH PASSWORD 'password';
```

- Create a new Database `catalog`

```shell
CREATE DATABASE catalog WITH OWNER catalog;
```

- Connect to the `catalog` database

```shell
\c catalog
```

- Revoke all rights

```shell
 REVOKE ALL ON SCHEMA public FROM public;
```

- Lock down the permissions only to user catalog

```shell
GRANT ALL ON SCHEMA public TO catalog;
```

- Logout form the PostgreSQL

```shell
\q
```

- Return to the `grader` user

```bash
exit
```

- Edit the project and database_setup files and update the parameter of `create_engine` function to be

```python
create_engine('postgresql://catalog:12345678@localhost/catalog')
```

- Run `database_setup` file to initialize the database

```bash
python database_setup.py
```

### Create wsgi file

- Cahnge the working directory `/var/www/catalog`

```bash
cd /var/www/catalog
```

- Create `itemsCatalog.wsgi` file

```bash
nano itemsCatalog.wsgi
```

- Add this content

```python
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/")

from catalog import app as application
application.secret_key = 'secret_key'
```

### Configure Apache2 file
- Edit apache's default virtual file
```bash
sudo nano /etc/apache2/sites-available/000-default.conf
```

- Add this content:

```shell
<VirtualHost *:80>
                ServerName servername
                ServerAdmin email
                WSGIScriptAlias / /var/www/catalog/itemsCatalog.wsgi
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

- Save then restart Apache2

```bash
sudo services apache2 restart
```
- From your broweser visit: [Restaurants](http://35.177.162.124/)

## Resources

- [DigitalOcean](https://www.digitalocean.com/)


