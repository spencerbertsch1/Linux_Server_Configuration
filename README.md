# Linux Server Configuration

This is the last project in the Udacity Full Stack Nanodegree. 

This project is designed to teach students how to prepare a baseline linux server to host a flask application. This process includes things such as installing necessary software, configuring firewalls, and installing/ configuring a PostgreSQL database on the server to run as the backend for the application. 

* Public IP Address: 3.218.244.61
* Port Number: 2200

The website is currently deployed at [http://3.218.244.61/](http://3.218.244.61/)

Steps below are adapted in part from the *project Details* section of the Linux Server Configuration project page. 

## Step 1: Obtain an AWS instance using Amazon Lightsail 

1. Visit [Amazon Lightsail](lightsail.aws.amazon.com) and choose **create instance.**
2. Choose Linux/Unix and then choose OS Only.
3. Select your configuration - the cheapest option is $3.50/month.
4. Name your instance - or accept the default name
4. Click **Create Instance**

## Step 2: SSH into the new server

## Step 3: Update all of the currently installed packages on the server

Run the following in the command line:

$ sudo apt-get update
$ sudo apt-get upgrade

## Step 4: Change the SSH port from 22 to 2200

## Step 5: Create new user account named grader 

## Step 6: Give grader the permission to sudo

## Step 7: Create an SSH key pair for grader using the ssh-keygen tool

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
