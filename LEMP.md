#LEMP
## Installing the Nginx Web Server
1. install Nginx by running
    ~~~
    sudo apt update
    sudo apt install nginx
    ~~~
2. verify it is successfully installed
   ~~~
   sudo systemctl status nginx
   ~~~
   ![alt text](<Nginx successfully running.png>)
3. access the server in the ubuntu shell
   ~~~
   curl http://localhost:80
   ~~~
    or
   ~~~
   curl http://127.0.0.1:80
   ~~~
4. Run this on any browser [Open in the browser] (http://<Public-IP-Address>:80)
![alt text](<Installing the Nginx Web Server.png>)
## Installing MySQL
* use 'apt' to acquire and install this software:
  ~~~
   sudo apt install mysql-server
  ~~~
* Confirm by typing y and enter
* Log in to the console to connect to the user root
  ~~~
  sudo mysql
  ~~~
* set the password with:
  ~~~
  ALTERUSER'root'@'localhost' IDENTIFIED WITH mysql_native_password BY'PassWord.1';
  ~~~
  ![alt text](<Connecting to MySQL.png>)
* Exit the shell with
  ~~~
  exit
  ~~~
* start script
  ~~~
  sudo mysql_secure_installation
  ~~~
* Answer y to continue
* set the password
* Test MySQL with:
  ~~~
  sudo mysql -p
  ~~~
* Exit console
  ~~~
  exit
  ~~~
## Installing PHP
*  install php-fpm and php-mysql
  ~~~
  sudo apt install php-fpm php-mysql
  ~~~
* type 'y' and enter
## Configuring Nginx to Use PHP Processor
 * Create the root web directory for your_domain as follows:
   ~~~
   sudo mkdir /var/www/projectLEMP
   ~~~
   ![alt text](<Web directory.png>)
 * assign ownership of the directory with the $USER environment variable
  ~~~
  sudo chown -R $USER:$USER /var/www/projectLEMP
  ~~~
 * open a new configuration file in Nginx’s sites-available directory
  ~~~
  sudo nano /etc/nginx/sites-available/projectLEMP
  ~~~
  * This will create a new blank file. Paste in the following bare-bones configuration:
 ~~~
server {
    listen 80;
    server_name projectLEMP www.projectLEMP;
    root /var/www/projectLEMP;

    index index.html index.htm index.php;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
    }

    location ~ /\.ht {
        deny all;
    }
}
~~~
  * Activate your configuration by linking to the config file from Nginx’s sites-enabled directory:
    ~~~
    sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/ 
    ~~~
  * Test config
    ~~~
    sudo nginx -t
    ~~~
    ![alt text](<sudo nginx -t.png>)
  * We also need to disable default Nginx host that is currently configured to listen on port 80
    ~~~
    sudo unlink /etc/nginx/sites-enabled/default
    ~~~
  * reload Nginx
    ~~~
    sudo systemctl reload nginx
    ~~~
  * Run in browser http://<Public-IP-Address>:80
  ![alt text](<Nginx site.png>)
## Testing PHP with Nginx
  * create a test file
  ~~~
  nano /var/www/projectLEMP/info.php
  ~~~
  ![alt text](<Open new file.png>)
  * Run the information using:
    ~~~
    <?php
    phpinfo();
    ~~~
 * [Run in browser] (http://`server_domain_or_IP`/info.php)
 ![alt text](<web page containing detailed information about your server.png>)
 * remove the file
   ~~~
   sudo rm /var/www/your_domain/info.php
   ~~~
 ## Retrieving data from MySQL database with PHP
 1. Connect MySQL console
    ~~~
    sudo mysql
    ~~~
 2. create new database
    ~~~
    mysql> CREATE DATABASE `example_database`;
    ~~~
3. Create a new user
  ~~~
  mysql>  CREATE USER 'example_user'@'%' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';
  ~~~
4. Grant permission
~~~~
mysql> GRANT ALL ON example_database.* TO 'example_user'@'%';
~~~~
5. Exit MySQL shell
~~~
exit
~~~
6. Log into MySQL console
~~~
mysql -u example_user -p
~~~
7. Confirm access   
~~~
SHOW DATABASES
~~~
 ![alt text](<SHOW DATABASES.png>)
8. Create a test table
~~~
CREATE TABLE example_database.todo_list (
    item_id INT AUTO_INCREMENT,
    content VARCHAR(255),
    PRIMARY KEY(item_id)
);
~~~
~~~
INSERT INTO example_database.todo_list (content) VALUES ("My first important item");
~~~
9. Run the code
~~~
SELECT * FROM example_database.todo_list;
~~~
10. Exit
~~~
exit
~~~
11. Create a PHP script that will connect to MySQL and query for your content.
~~~
nano /var/www/projectLEMP/todo_list.php
~~~
~~~
<?php
$user = "example_user";
$password = "PassWord.1";
$database = "example_database";
$table = "todo_list";

try {
  $db = new PDO("mysql:host=localhost;dbname=$database", $user, $password);
  echo "<h2>TODO</h2><ol>";
  foreach($db->query("SELECT content FROM $table") as $row) {
    echo "<li>" . $row['content'] . "</li>";
  }
  echo "</ol>";
} catch (PDOException $e) {
    print "Error!: " . $e->getMessage() . "<br/>";
    die();
}
~~~
12. Run on the browser;

    http://<Public_domain_or_IP>/todo_list.php

    ![alt text](<Retrieving data from MySQL database with PHP.png>)


