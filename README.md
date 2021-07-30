# AWS-EC2-Setup


## UPDATE:
I used 'sudo tasksel install lamp-server' instead of 'sudo tasksel' (then selecting) to stop the SSH problems I've been having with ubuntu 12.04


Create a amazon aws account (http://aws.amazon.com) and enter the AWS management dashboard (https://console.aws.amazon.com/ec2/home). Select the EC2 tab at the top of the dashboard.

#### STEP#1 
Create a Security Group that allows HTTP connections and limited SSH to configure the instance. Click on Security Groups > Create Security Group

 - Name: web_access
 - Click on the newly created Security Group, the select the Inbound tab
 - Select HTTP from the Create a new rule drop-down and leave the Source as 0.0.0.0/0 to allow HTTP connections from anywhere
 - Select SSH from the Create a new rule drop-down and change the Source to your IP address(/32) to restrict SSH connections to your computer or network. Again click on Add Rule.
 - Now click on Apply Rule Changes

#### STEP#2 
Create an EC2 instance that provides a very clean and quick LAMP installation. Click on Instances > Launch Instance to initiate the Request Instance Wizard.

 - Click on the Community AMIs tab
 - We will use a the 32-bit Ubuntu 10.10 Maverick EBS boot provided by alestic. Identifier: ami-ccf405a5.
 - From now on we will mostly use default settings. Select Micro (Or whatever you think you need, prices will vary. Micro is free for a year as of 3/22/2010) from the Instance Type drop-down.
 - Leave all the default settings in Advanced Instance Options and click Continue.
 - Add any tags you want...They are simply used to identify one instance from another. A name is suggested but optional.
 - Generate a Key Pair. The Key Pair will allow you to log in to the instance via SSH. Enter a suitable name and click on Create & Download your Key Pair.
 - A mykeypair.pem file will be downloaded. Save it somewhere. If you plan on using this instance for production, make sure the file is properly secured. Click on Continue.
 - In Configure Firewall select Choose one or more of your existing Security Groups. Choose the Security Group that we created on step 1 and click on Continue.
 - Review all the settings and click on Launch. The instance has to boot and thus will take a few seconds to be ready.

#### STEP#3  
Create an Elastic (static) IP for the server

 - Click on Elastic IPs > Allocate New Address > Yes, Allocate. This will create a new Elastic IP but will not associate it to the instance yet.
 - Select the newly created IP address and click on Associate Address. Choose the instance you created from the drop-down and click on Yes, Associate.

#### STEP#4 
Install LAMP via SSH

 - Open the console and navigate to the Key Pair file (replace Desktop with the save path). In order to be able to use it for SSH we need to restrict its permissions:

   ****Terminal*****
   cd ~/Desktop
   sudo chmod 600 mykeypair.pem
   ****Terminal*****

 - Connect via ssh (replace 192.168.0.1 with the elastic ip you created)

   ****Terminal*****
   ssh -i mykeypair.pem ubuntu@192.168.0.1
   ****Terminal*****

   - Type 'yes' and press enter when prompted to continue

 - Update the package database of the Ubuntu installation.

   ****Terminal*****
   sudo apt-get update
   ...
   sudo apt-get upgrade
   ****Terminal*****

 - Install LAMP using tasksel

   ****Terminal*****
   sudo tasksel
   ****Terminal*****

   - Scroll down to LAMP server and press space, then Enter.
   - The LAMP installation should run smoothly. You will be only asked to provide and confirm the password of the root user of the MySQL database.

 - Install phpMyAdmin in order to test the PHP and MySQL installation and also to conveniently configure the MySQL database.

   ****Terminal*****
   sudo apt-get install phpmyadmin
   ****Terminal*****

   - Select apache2 as the web server
   - Leave the database configuration for later (choose No on the second prompt).

#### STEP#5
Prepare apache for multiple websites (This is unnecessary if you are only using server for a single website, you would simply put all files in the web root [/var/www])

 - Browse to web root

   ****Terminal*****
   cd /var/www
   ****Terminal*****

 - Make directories for every sites

   ****Terminal*****
   sudo mkdir domain1.com
   sudo mkdir domain2.com
   ****Terminal*****

 - Browse to apache config

   ****Terminal*****
   cd /etc/apache2/sites-available
   ****Terminal*****

 - Create domain config files

   ****Terminal*****
   sudo touch domain1.com
   sudo vi domain1.com
   ****Terminal*****

   - Press i to start editing file and type this into the document

   <VirtualHost *:80>
   ServerName domain1.com
   ServerAlias www.domain1.com
   ServerAdmin YourEmail@email.com
   DocumentRoot /var/www/domain1.com
   </VirtualHost>

  - Press esc to stop editing and shift + zz to save

 - enable site

   ****Terminal*****
   sudo a2ensite domain1.com
   ****Terminal*****

 - Repeat this for each domain and restart apache for changes to take effect

   ****Terminal*****
   sudo /etc/init.d/apache2 reload
   ****Terminal*****


#### STEP#6 
Add user (ubuntu) to www-data group and set editing permissions for web files

 - Edit groups file

   ****Terminal*****
   sudo vigr
   ****Terminal*****

   - Press i to edit scroll down to the www-data listing and change this line

   www-data:x:33:

   - To this

   www-data:x:33:ubuntu

   - Press esc to stop editing and shift + zz to save

 - Change permissions for web root folder

   ****Terminal*****
   sudo chown -R www-data /var/www
   sudo chgrp -R www-data /var/www
   sudo chmod -R 775 /var/www
   
   //for permission errors i've been getting on 12.04 do the following
   sudo find /var/www -type d -exec chmod 2775 {} \;
   ****Terminal*****

   extras:
   sudo apt-get install php5-curl
   sudo a2enmod rewrite
   sudo apt-get install unzip

Finished! Now you can point your domains A record at the elastic IP.  They will be redirected to their corresponding folder in the web root.

Config your sFTP software:

host: [elasticIP]
un: ubuntu
ps: *blank*

And attach the mykeypair.pem file

source:  http://www.robotmedia.net/2011/04/how-to-create-an-amazon-ec2-instance-with-apache-php-and-mysql-lamp/comment-page-1/
NetTuts+ "The Ins and Outs of Amazon-EC2"

## License

This source code is licensed under the MIT license.

## Author

<table>
  <tr>
    <td>
      <img src="https://fingersfun.com/img/hand_logo.png?s=100" width="100">
    </td>
    <td>
      Fingers Fun<br />
      <a href="mailto:kirankumar.mentam@gmail.com">kirankumar.mentamq@gmail.com</a><br />
      <a href="https://fingersfun.com/">https://fingersfun.com/</a>
    </td>
  </tr>
</table>
