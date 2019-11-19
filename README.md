# Linux Server Configuration

This is the last project in the Udacity Full Stack Nanodegree. 

This project is designed to teach students how to prepare a baseline linux server to host a flask application. This process includes things such as installing necessary software, configuring firewalls, and installing/ configuring a PostgreSQL database on the server to run as the backend for the application. 

* Public IP Address: 34.200.251.79
* Port Number: 2200

The website is currently deployed at [http://34.200.251.79/](http://34.200.251.79/)

Steps below are adapted in part from the *project Details* section of Udacity's Linux Server Configuration project page. 

## Step 1: Obtain an AWS instance using Amazon Lightsail 

1. Visit [Amazon Lightsail](lightsail.aws.amazon.com) and choose **create instance.**
2. Choose Linux/Unix and then choose OS Only.
3. Select your configuration - the cheapest option is $3.50/month.
4. Name your instance - or accept the default name
4. Click **Create Instance**

## Step 2: SSH into the new server

1. Navigate to *Accounts* on AWS Lightsail
2. Select SSH Keys and download the private key 
3. Go to your downloads folder and rename the file to Lightsail-key.rsa
4. Move that file to the ~/.ssh directly on the local machine 
5. Set the permissions as owner: `$ chmod 600 ~/.ssh/lightsail_key.rsa`
6. SSH into the server: `$ ssh -i ~/.ssh/Lightsail-key.rsa ubuntu@34.200.251.79` where `34.200.251.79` is the public IP address of the server

## Step 3: Update all of the currently installed packages on the server

Run the following in the command line:

`$ sudo apt-get update`

`$ sudo apt-get upgrade`

## Step 4: Change the SSH port from 22 to 2200

1. Run `$ sudo nano /etc/ssh/sshd_config` to open the sshd_config file

2. Change the port from `22` to `2200`

3. `Ctrl-x` then `Shift-S` then `return` to save and exit

4. Restart the SSH connection: `$ sudo service ssh restart`

## Step 5: Configure the firewall (ufw - uncomplicated firewall)

This step involves closing all of the connections to all ports, then opening only the ones which will be used by the application. Run the following commands in the server's command line:

1. `sudo ufw status` - This lets us see the current firewall status

2. `sudo ufw default deny incoming` - This blocks all traffic into the server

3. `sudo ufw default allow outgoing` - This allows all traffic out of the server

4. `sudo ufw allow 2200/tcp` - This opens port 2200 for all tcp connections

5. `sudo ufw allow www` - This opens port 80 for www connections

6. `sudo ufw allow 123/udp` - This opens port 123 for all udp connections

7. `sudo ufw deny 22` - This blocks all traffic on port 22

8. `sudo ufw enable` - This activates the UFW

9. `sudo ufw status` - This lets us see the current firewall status

A way to check and make sure the firewall is configured correctly is to go to the Manage > Networking section of Amazon Lightsail and be sure that only ports 2200, 80, and 123 are open. If that is not the case, you can always update the firewall manually in Lightsail. 

Exit the current SSH connection by running `exit`, then run the following to SSH into the server through port 2200: 

`$ ssh -i ~/.ssh/Lightsail-key.rsa ubuntu@34.200.251.79 -p 2200`

## Step 5: Create new user account named grader 

Run the following commands in the Ubuntu command prompt to set up the grader user

1. `sudo adduser grader`

2. Come up with a good password and enter it twice

## Step 6: Give grader the permission to sudo

Run the following commands in the Ubuntu command prompt to give grader permission to sudo

1. `sudo visudo` - open the sudoers file

2. Add the line `grader  ALL=(ALL:ALL) ALL` directly under the line `root    ALL=(ALL:ALL) ALL`

3. `Ctrl-x` then `Shift-S` then `return` to save and exit

## Step 7: Create an SSH key pair for grader using the ssh-keygen tool

**Step 1: On the local machine -**

1. `$ cd /Users/YOUR-USER-NAME-HERE/.ssh` cd into the `ssh` directory

2. `ssh-keygen` Create the local keys

3. Enter a name for the file in which to save the key - something like `linuxCourseKey`

4. Enter a passphrase twice

5. `cat ~/.ssh/linuxCourseKey.pub` Copy the output to the clipboard

**Step 2: On the virtual machine, logged in as grader -**

1. `su - grader` Log in as grader

2. `mkdir .ssh` Create the SSH directory

3. `sudo nano ~/.ssh/authorized_keys` paste the contents of the clipboard from `linuxCourseKey.pub`

4. `Ctrl-x` then `Shift-S` then `return` to save and exit

5. `chmod 700 .ssh` and `chmod 644 .ssh/authorized_keys` Update permissions

6. `sudo service ssh restart` Restart SSH

7. `sudo nano /etc/ssh/sshd_config` and set `PasswordAuthentication` to `no`. If it is already `no`, then exit by entering `Ctrl-x`.

8. `Ctrl-x` then `Shift-S` then `return` to save and exit

9. `sudo service ssh restart` Restart SSH

10. It's now possible to SSH into the server as the grader without the password. Run: 

`ssh -i ~/.ssh/linuxCourseKey -p 2200 grader@34.200.251.79`

## Step 8: Configure the local timezone to UTC

While still logged in as grader...

1. `sudo dpkg-reconfigure tzdata`

2. Set time zone to UTC - in order to make the local timezone match UTC, choose `Europe` > `Isle of Man`

## Step 9: Install and configure Apache to serve a Python mod_wsgi application

While still logged in as grader...

`sudo apt-get install apache2` Install Apache2

Visit `http://34.200.251.79/` to see a live message from Apache

## Step 10: Install Python mod_wsgi

`sudo apt-get install libapache2-mod-wsgi python-dev` Install mod-wsgi

`sudo a2enmod wsgi` Enable mod-wsgi

`sudo service apache2 restart` Restart Apache

Here we can check the current version of python by simply running `python` in the termianl, then running `exit()` to exit the python console. 

## Step 10: Install and configure PostgreSQL

While logged in as grader...

1. `sudo apt-get install postgresql` Install PostgreSQL

We now need to ensure that any remote connections are disabled

`sudo nano /etc/postgresql/9.5/main/pg_hba.conf` Open the PostgreSQL .conf file - it should look like the output below

```
"local" is for Unix domain socket connections only
local   all             all                                     peer
# IPv4 local connections:
host    all             all             127.0.0.1/32            md5
# IPv6 local connections:
host    all             all             ::1/128                 md5
```

`sudo su - postgres` Switch to the Postgres user

`psql` Open the PSQL terminal

`postgres=# CREATE ROLE catalog WITH LOGIN PASSWORD 'catalog';` Create the catalog user

`postgres=# ALTER ROLE catalog CREATEDB;` Allow the catalog user to create databases

`postgres=# \du` Show all existing roles in PostgreSQL - the output should resemble the below output

```
                                   List of roles
 Role name |                         Attributes                         | Member of 
-----------+------------------------------------------------------------+-----------
 catalog   | Create DB                                                  | {}
 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}

```

`\q` Exit the PSQL console

`$ exit` log out as `postgres` and switch back to the `grader` user


## Step 11: Create a new database user named catalog

`sudo adduser catalog` Create a new user called `catalog`, provide a password

`sudo visudo` Open the visudo file to give sudo access to the catalog user

Add the line `catalog  ALL=(ALL:ALL) ALL` under the line `grader  ALL=(ALL:ALL) ALL` The result should resemble the following:

```
root    ALL=(ALL:ALL) ALL
grader  ALL=(ALL:ALL) ALL
catalog  ALL=(ALL:ALL) ALL
```

`Ctrl-x` then `Shift-S` then `return` to save and exit

To check that the catalog user does indeed have sudo access...

`su - catalog` And enter the password for the catalog user

`sudo -l` And enter the password for the catalog user, the output should resemble the below output. 

```
Matching Defaults entries for catalog on ip-172-26-7-191.ec2.internal:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User catalog may run the following commands on ip-172-26-7-191.ec2.internal:
    (ALL : ALL) ALL
```

## Step 12: Create a new database called catalog

While still logged in as the catalog user...

`createdb catalog` Create a new database called `catalog`

`$ psql` Open the PSQL terminal 

`catalog=> \l` Look at all the PSQL databases and their information. The results of this command should resemble the below output

```
                                  List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
-----------+----------+----------+-------------+-------------+-----------------------
 catalog   | catalog  | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
(4 rows)
```

`\q` Exit the PSQL console

`exit` Log out of the postgres user and switch back to the `grader` user

## Step 13: Install git

While logged in as grader...

`sudo apt-get install git` Install git

# -------- LEFT OFF HERE ---------

## Step 14: Clone the Item Catalog repo

While logged in as grader...

**1.** `mkdir /var/www/catalog` Create the catalog directory

`cd /var/www/catalog` cd into the new catalog directory

**2.** `sudo git clone URL-OF-REPO-TO-CLONE catalog` Clone the repo

`sudo chown -R grader:grader catalog/` Change permissions to the grader user

`cd /var/www/catalog/catalog` cd into the inner catalog directory

**3.** `mv project.py __init__.py` Rename the `project.py` file to `__init.py__`

**4.** Copy the client secrets back into the `client_secrets.json` file

**5.** `sudo nano __init.py__` open `__init__.py`

On line line 356 of `__init.py__` replace `app.run(host='0.0.0.0', port=5000)` with `app.run()`


**6.** In `__init__.py`, `database_setup.py`, and `database_filler.py`, replace the line:

`engine = create_engine('sqlite:///WinterSports.db')`

with: 

`engine = create_engine('postgresql://catalog:DATABASE-PASSWORD-HERE@localhost/catalog')`

## 15: Prepare Ubuntu for deploying the Flask app

`sudo apt-get install python-pip` Install pip

Install all necessary packages...

```
sudo pip install httplib2
sudo pip install requests
sudo pip install --upgrade oauth2client
sudo pip install sqlalchemy
sudo pip install flask
sudo apt-get install libpq-dev
sudo pip install psycopg2
```

## 16: Set up and enable the virtual host

`sudo touch /etc/apache2/sites-available/catalog.conf` Create the .conf file

```
<VirtualHost *:80>
    ServerName 34.200.251.79
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

`sudo touch /etc/apache2/sites-available/catalog.conf` Enable the virtual host

`sudo service apache2 reload` Restart Apache2

## 17: Create the .wsgi file

`sudo touch /var/www/catalog/catalog.wsgi` Create the `catalog.wsgi` file

Enter the following into the file and save and exit.

```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/catalog/")
sys.path.insert(1, "/var/www/catalog/")

from catalog import app as application
application.secret_key = "super-secret-key"
```

`sudo service apache2 restart` Restart Apache2

## Step 18: Disable the default Apache2 page

`sudo a2dissite 000-defualt.conf` Disable the default Apache2 page

`sudo service apache2 reload` Reload Apache2

## Step 19: Set up the database for the catalog application

`cd /var/www/catalog/catalog` cd into the correct directory

`sudo python database_setup.py` Initialize the catalog database

`sudo python database_filler.py` Populate the new database

`sudo service apache2 restart` Restart Apache2

## Step 20: Run the flask application so it's live at the listed URI

`sudo python __init__.py` Run the application

Now open the public URI in a browser! [http://34.200.251.79/](http://34.200.251.79/)

## Resources

During this project I relied on several tutorials and outside resources. See some of the more helpful ones below:

* http://gavinmarsh.io/blog/amazon_lightsail/
* https://medium.com/@zionoyemade/deploying-flask-application-to-amazon-lightsail-199e79bb256a
* https://medium.com/@rodkey/deploying-a-flask-application-on-aws-a72daba6bb80

I also saw many other Udacity student's github accounts showing steps to complete this project. These often yeilded conflicting information, but several were very helpful. Two of the most helpful ones are listed below: 

* https://github.com/boisalai/udacity-linux-server-configuration
* https://github.com/yiyupan/Linux-Server-Configuration-Udacity-Full-Stack-Nanodegree-Project
