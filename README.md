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

3. Type `Ctrl-x` then `Shift-S` then `return` to save and exit

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

3. Type `Ctrl-x` then `Shift-S` then `return` to save and exit

## Step 7: Create an SSH key pair for grader using the ssh-keygen tool

**Step 1: On the local machine - **

1. `$ cd /Users/YOUR-USER-NAME-HERE/.ssh` cd into the `ssh` directory

2. `ssh-keygen` Create the local keys

3. Enter a name for the file in which to save the key - something like `linuxCourseKey`

4. Enter a passphrase twice

5. `cat ~/.ssh/linuxCourseKey.pub` Copy the output to the clipboard

**Step 2: On the virtual machine, logged in as grader - **

1. `su - grader` Log in as grader

2. `mkdir .ssh` Create the SSH directory

3. `sudo nano ~/.ssh/authorized_keys` paste the contents of the clipboard from `linuxCourseKey.pub`

4. Type `Ctrl-x` then `Shift-S` then `return` to save and exit

5. `chmod 700 .ssh` and `chmod 644 .ssh/authorized_keys` Update permissions

6. `sudo service ssh restart` Restart SSH

7. Run `cd /etc/ssh/sshd_config` and set `PasswordAuthentication` to `no`

8. `sudo service ssh restart` Restart SSH

9. It's now possible to SSH into the server as the grader without the password. Run: 

`ssh -i ~/.ssh/linuxCourseKey -p 2200 grader@3.218.244.61`

## Step 8: Configure the local timezone to UTC

## Step 9: Install and configure Apache to serve a Python mod_wsgi application

## Step 10: Install and configure PostgreSQL

## Step 11: Do not allow remote connections

## Step 12: Create a new database user named catalog

## Step 13: Limit permissions to the catalog database to the user 'catalog'

## Step 14: Install git

## Step 15: Clone the Item Catalog repo

## Step 16: Create and populate the catalog database in PostgreSQL 

## Step 17: Run the flask application so it's live at the listed URI

## Resources

During this project I relied on several tutorials and outside resources. See some of the more helpful ones below:

* http://gavinmarsh.io/blog/amazon_lightsail/
* https://medium.com/@zionoyemade/deploying-flask-application-to-amazon-lightsail-199e79bb256a
* https://medium.com/@rodkey/deploying-a-flask-application-on-aws-a72daba6bb80

I also saw many other Udacity student's github accounts showing steps to complete this project. These often yeilded conflicting information and, although sometimes helpful, were'nt an extremely useful resource in configuring the server correctly. 
