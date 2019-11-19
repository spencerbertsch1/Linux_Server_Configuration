# Linux Server Configuration

This is the last project in the Udacity Full Stack Nanodegree. 

This project is designed to teach students how to prepare a baseline linux server to host a flask application. This process includes things such as installing necessary software, configuring firewalls, and installing/ configuring a PostgreSQL database on the server to run as the backend for the application. 

* Public IP Address: 3.218.244.61
* Port Number: 2200

The website is currently deployed at [http://3.218.244.61/](http://3.218.244.61/)

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
6. SSH into the server: `$ ssh -i ~/.ssh/Lightsail-key.rsa ubuntu@3.218.244.61` where `3.218.244.61` is the public IP address of the server

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

`$ ssh -i ~/.ssh/Lightsail-key.rsa ubuntu@3.218.244.61 -p 2200`

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

`ssh -i ~/.ssh/linuxCourseKey -p 2200 grader@3.218.244.61`

## Step 8: Configure the local timezone to UTC

While still logged in as grader...

1. `sudo dpkg-reconfigure tzdata`

2. Set time zone to UTC - in order to make the local timezone match UTC, choose `Europe` > `Isle of Man`

## Step 9: Install and configure Apache to serve a Python mod_wsgi application

While still logged in as grader...

`sudo apt-get install apache2` Install Apache2

Visit `http://3.218.244.61/` to see a live message from Apache

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



## Step 12: Limit permissions to the catalog database to the user 'catalog'

## Step 13: Install git

## Step 14: Clone the Item Catalog repo

## Step 15: Create and populate the catalog database in PostgreSQL 

## Step 16: Run the flask application so it's live at the listed URI

## Resources

During this project I relied on several tutorials and outside resources. See some of the more helpful ones below:

* http://gavinmarsh.io/blog/amazon_lightsail/
* https://medium.com/@zionoyemade/deploying-flask-application-to-amazon-lightsail-199e79bb256a
* https://medium.com/@rodkey/deploying-a-flask-application-on-aws-a72daba6bb80

I also saw many other Udacity student's github accounts showing steps to complete this project. These often yeilded conflicting information, but several were very helpful. Two of the most helpful ones are listed below: 

* https://github.com/boisalai/udacity-linux-server-configuration
* https://github.com/yiyupan/Linux-Server-Configuration-Udacity-Full-Stack-Nanodegree-Project
