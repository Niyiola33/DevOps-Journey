# My devOps Journey
 Learning devops
Lamp-stack
## Creating a Linux Server in the Cloud
•	Create an amazon account  
•	Register the region close to you  
•	Launch a new ec2 instance of t2.micro  
•	Save the .pem file  
•	Using the .pem file, ssh into to ec2 instance.  
~~~

Ssh -I Olawale.pem ubuntu@PublicIP

~~~  
## Installing apache and updating the firewall
~~~
sudo apt update 
sudo apt install apache2
sudo systemctl status apache2
curl http://localhost:80 
curl http://127.0.0.1:80
use this url in the browser http://<Public-IP-Address>:80
curl -s http://169.254.169.254/latest/meta-data/public-ipv4
~~~
## installing MySQL
~~~
sudo apt install mysql-server
sudo mysql
ALTERUSER'root'@'localhost' IDENTIFIED WITH mysql_native_password BY'PassWord.1';
mysql> exit
sudo mysql_secure_installation
sudo mysql -p
mysql> exit
~~~
## Installing PHP
~~~
sudo apt install php libapache2-mod-php php-mysql    

php -v
~~~
## Creating a Virtual Host for your Website using Apache
~~~
sudo mkdir /var/www/projectlamp
sudo chown -R $USER:$USER /var/www/projectlamp
sudo vi /etc/apache2/sites-available/projectlamp.conf
<VirtualHost *:80> ServerName projectlamp ServerAlias www.projectlamp ServerAdmin webmaster@localhost DocumentRoot /var/www/projectlamp ErrorLog ${APACHE_LOG_DIR}/error.log CustomLog ${APACHE_LOG_DIR}/access.log combined </VirtualHost>
~~~
Hit the esc button on the key board
•	Type wq. W for write and q for quit
•	Hit enter to save the file
~~~
sudo ls /etc/apache2/sites-available
sudo a2ensite projectlamp
sudo a2dissite 000-default
sudo apache2ctl configtest
sudo systemctl reload apache2
sudo echo 'Hello LAMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectlamp/index.html
~~~
•	open this on the browser http://<Public-IP-Address>:80
## Enable PHP on the website
~~~
sudo vim /etc/apache2/mods-enabled/dir.conf
<IfModule mod_dir.c> #Change this: #DirectoryIndex index.html index.cgi index.pl index.php index.xhtml index.htm #To this: DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm </IfModule>
•	sudo systemctl reload apache2
•	vim /var/www/projectlamp/index.php
•	<?php phpinfo();
•	sudo rm /var/www/projectlamp/index.php
~~~

