# Udacity's Linux Server Configuration Project Submission
## What is required?
**You will take a baseline installation of a Linux distribution on a virtual machine and prepare it to host your web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.**

## Summary
deploy the item catalog project that we previously made in this project using apache2 and mod wsgi

## You can visit the website here
[35.243.189.230.xip.io](http://35.243.189.230.xip.io)

## Note:
# This project was done using the google compute engine.

## Additional information
**SSH Port:** 2200
**Ip Address:** 35.243.189.230

### GuideLine to deploying the website into the server
## 1. Visit (Google Compute Engine website)[cloud.google.com/compute] and Sign up or login here is a guide into creating a google compute
## (guide)[https://cloud.google.com/compute/docs/how-to]

## 2. After you have finished go to your vm instances page and click on the ssh button beside your instance to ssh into the server

## 3. Update all current Packages
  sudo apt-get update
  sudo apt-get upgrade

## 4. Change SSH port from 22 to 2200
#  A- First off, from the google right Drawer under networking hover over VPC network then choose Firewall Rules
  Click CREATE FIREWALL RULE name it **default-allow-ssh** then check **Specified protocols and ports** and add 2200 to the tcp field
  then Click Create

#  B- Back to our server type this:
  sudo nano /etc/ssh/sshd_config
  then change the port from 22 to 2200 and save
  restart ssh, sudo service ssh restart
  **Note**: when you log back into the server click on the ssh button drop down and click open on browser window on custom port
  and type in the port in our case 2200, there is also an option where you can go to the gcloud shell (also from the drop down) and add to -gcloud compute --project "linux-configuration-system" ssh --zone "us-east1-b" "instance-1"- this command '--ssh-flag="-p 2200"

## 5. Configure Uncomplicated Firewall to allow incoming and outgoing connections
#  A- Just as mentioned in step 4(a) add each firewall rule you want to the the firewall rules of your engine, in our case we already setup        ssh. now let us add http and ntp, again name them default-allow-http and default-allow-ntp and fill in their tcp with their number (http 80,    ntp 123)

#  B- Back to our terminal, execute the following commands to block any incoming requests and allow outgoing requests
  sudo ufw default deny incoming
  sudo ufw default allow outgoing
  **now add the firewall rules**
  sudo ufw allow 2200
  sudo ufw allow 80
  sudo ufw allow 123
  **now enable the firewall**
  sudo ufw enable
  **check ufw status**
  sudo ufw status

## 6. Create user 'grader'
   sudo adduser grader
   **allow grader to sudo**
   sudo cp /etc/sudoers.d/google_sudoers /etc/sudoers.d/grader
   now, nano into the grader file and you'll find
   **%google-sudoers ALL=(ALL:ALL) NOPASSWD:ALL**
   change google-sudoers to grader

## 7. generate key pair for grader
  now from the ssh drop down choose 'view gcloud command' then click 'start cloud shell' it will lead you into the internal local of the engine
  run **ssh-key** and enter a name for your key pair and don't enter a password for the paraphrase
  copy the public key
  from /home/<yourUserName>/.ssh/<name of ssh key pair>
  **Back to the server, switch to grader**
  su - grader
  enter password for grader (that you choose when you created it)
  **create a directory that has a file in it and paste in it the public key you just copied**
  mkdir .ssh
  cd mkdir
  sudo touch authorized_keys 
  **now give file permissions**
  sudo chmod 700 .ssh
  sudo chmod 644 .ssh/authorized_keys

## Now you can sign in through the cloud shell through this command
ssh -i ~/.ssh/<KeyFileName> -p 2200 <user name>@External ip
#### in my case
ssh -i ~/.ssh/grader -p 2200 grader@35.243.189.230

# Deploying the project
## 8. Configure local timezone
   sudo dpkg-reconfigure tzdata

## 9. Install Apache and mod_wsgi and git
   sudo apt-get install apache2
   sudo apt-get install python-setuptools libapache2-mod-wsgi
   **Now restart apache**
   sudo service apache2 restart
   sudo apt-get install git

## 10. Install Postgresql
   sudo apt-get install postgresql
   **check that there is limited access to the database**
   sudo nano /etc/postgresql/9.3/main/pg_hba.conf
   **Now create database** 
   sudo su - postgres
   psql 
   postgres=# CREATE DATABASE catalog;
   postgres=# ALTER ROLE catalog WITH PASSWORD 'password';
   postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
   \q
   exit
   **Note:** in my case i have created the database with owner catalog but i found that this technique is simpler

## 11. Clone git directory
   cd /var/www
   git clone https://github.com/AbdulkareemBarghouthi/Catalog-website catalog
   **rename the project file to __init__.py**
   **change the engine in the project to postgresql://catalog:password@localhost/catalog**
   **change any client_secret.json path to /var/www/catalog/catalog/client_secret.json**

## 12. install your project packages
    sudo apt-get install python-pip
    **install flask,postgresql,sqlalchemy and all packages that are in the project for your python packages**
    sudo pip install oauth2client sqlalchemy httplib2 psycogp2 sqlalchemy_utils
## 13. Now create a configuration file for your project in the server through virtual host
    sudo nano /etc/apache2/sites-available/catalog-website.conf

<mark>
    <VirtualHost *:80>
        ServerName 35.243.189.230.xip.io
        ServerAdmin admin@35.243.189.230
        WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
        WSGIProcessGroup catalog
        WSGIScriptAlias / /var/www/catalog/catalog-website.wsgi
        <Directory /var/www/catalog/catalog-website>
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
</mark>
   **enable virtual host**
   sudo a2ensite catalog-website

## 14. Configure mod_wsgi for your flask application
   **Create wsgi file and configure it for your flask application**
   sudo touch /var/www/catalog/catalog-website.wsgi
   sudo nano  /var/www/catalog/catalog-website.wsgi
   **add the following**
   <mark>
   import sys
   import logging
   logging.basicConfig(stream=sys.stderr)
   
   sys.path.insert(0, "/var/www/catalog/")
   from catalog import app as application
   application.secret_key = 'some_key'
   </mark>
   
   **Now restart apache2 server**
   sudo service apache2 restart

## 15. Finally, Visit the ip i have mentioned earlier and you'll find your website deployed!

Links:
[https://cloud.google.com/compute/docs/how-to]
   



