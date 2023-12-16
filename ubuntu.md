### INSTALLING THE NGINX WEB SERVER
```
sudo apt update
sudo apt install nginx
```


## Verify that nginx was successfully installed and active 
```
sudo systemctl status nginx

## Before we can receive any traffic by our Web Server, we need to open TCP port 80
## which is default port that web brousers use to access web pages in the Internet.
```
## Access it locally in our Ubuntu shell, run:
```
curl http://localhost:80
or
curl http://127.0.0.1:80
```
## Test how our Nginx server can respond to requests from the Internet
```
http://<Public-IP-Address>:80
```
## Installing MYSQL
```
sudo apt install mysql-server

sudo mysql

## Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 11
Server version: 8.0.22-0ubuntu0.20.04.3 (Ubuntu)

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>

## exit 
```
## Set a password for the root user, using mysql_native_password as default authentication method
```
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';
```
## Run security script
```
sudo mysql_secure_installation

## This will ask if you want to configure the VALIDATE PASSWORD PLUGIN
VALIDATE PASSWORD PLUGIN can be used to test passwords
and improve security. It checks the strength of password
and allows the users to set only those passwords which are
secure enough. Would you like to setup VALIDATE PASSWORD plugin?

Press y|Y for Yes, any other key for No:

## There are three levels of password validation policy:

LOW    Length >= 8
MEDIUM Length >= 8, numeric, mixed case, and special characters
STRONG Length >= 8, numeric, mixed case, special characters and dictionary              file

Please enter 0 = LOW, 1 = MEDIUM and 2 = STRONG: 1

```
## VALIDATE PASSWORD PLUGIN

```
Estimated strength of the password: 100 
Do you wish to continue with the password provided?(Press y|Y for Yes, any other key for No) : y
```

## Test if you’re able to log in to the MySQL console by typing
```
sudo mysql -p

## exit
exit
```
## Installing PHP
```
sudo apt install php-fpm php-mysql
```
## Configuring NGINX
```
## Create the root web directory for your_domain as follows
sudo mkdir /var/www/projectLEMP

## assign ownership of the directory with the $USER environment variable, which will reference your current system user
sudo chown -R $USER:$USER /var/www/projectLEMP

## open a new configuration file in Nginx’s sites-available directory
sudo vi /etc/nginx/sites-available/projectLEMP

## This will create a new blank file. Paste in the following bare-bones configuration
#/etc/nginx/sites-available/projectLEMP

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
```

## Activate your configuration by linking to the config file from Nginx’s sites-enabled directory
```
sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/

## This will tell Nginx to use the configuration next time it is reloaded. You can test your configuration for syntax errors by typing
sudo nginx -t

## We also need to disable default Nginx host that is currently configured to listen on port 80, for this run
sudo unlink /etc/nginx/sites-enabled/default

## reload Nginx to apply the changes
sudo systemctl reload nginx

## Your new website is now active, but the web root /var/www/projectLEMP is still empty. 
## Create an index.html file in that location so that we can test that your new server block works as expected.

sudo echo 'Hello LEMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP'
$(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectLEMP/index.html

## open your website URL using IP address
http://<Public-IP-Address>:80
```
## Testing PHP with NGINX 
```
## Open a new file called info.php within your document root in your text editor
sudo vi /var/www/projectLEMP/info.php

## This is valid PHP code that will return information about your server
<?php
phpinfo();
```
## You can now access this page in your web browser by visiting the domain name or public IP address you’ve set up in your Nginx configuration file, followed by /info.php
```
http://`server_domain_or_IP`/info.php
```
## After checking the relevant information about your PHP server through that page, 
## it’s best to remove the file you created as it contains sensitive information about your PHP environment and your Ubuntu server. 
## You can use rm to remove that file
```
sudo rm /var/www/your_domain/info.php
```
## Retrieving data from MYSQL database with php (continued)
```
## First, connect to the MySQL console using the root account
sudo mysql

## To create a new database, run the following command from your MySQL console
CREATE DATABASE `example_database`;

## creates a new user named example_user, using mysql_native_password as default authentication method.
CREATE USER 'example_user'@'%' IDENTIFIED WITH mysql_native_password BY 'password';

## give this user permission over the example_database database
GRANT ALL ON example_database.* TO 'example_user'@'%';

## exit the MySQL shell with
exit

## You can test if the new user has the proper permissions by logging in to the MySQL console again
mysql -u example_user -p

SHOW DATABASES;
```

## This will give you the following output
```
Output
+--------------------+
| Database           |
+--------------------+
| example_database   |
| information_schema |
+--------------------+
2 rows in set (0.000 sec)

## Next, we’ll create a test table named todo_list. From the MySQL console, run the following statement
CREATE TABLE example_database.todo_list (
mysql>     item_id INT AUTO_INCREMENT,
mysql>     content VARCHAR(255),
mysql>     PRIMARY KEY(item_id)
mysql> );

## Insert a few rows of content in the test table
INSERT INTO example_database.todo_list (content) VALUES ("My first important item");

## To confirm that the data was successfully saved to your table, run
SELECT * FROM example_database.todo_list;mysql>  SELECT * FROM example_database.todo_list;

## You’ll see the following output
Output
+---------+--------------------------+
| item_id | content                  |
+---------+--------------------------+
|       1 | My first important item  |
|       2 | My second important item |
|       3 | My third important item  |
|       4 | and this one more thing  |
+---------+--------------------------+
4 rows in set (0.000 sec)
 ## exit
```
## Create a new PHP file in your custom web root directory using your preferred editor
```
vi /var/www/projectLEMP/todo_list.php

## Copy this content into your todo_list.php script
<?php
$user = "example_user";
$password = "password";
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

## Save and close the file when you are done editing
```

## You can now access this page in your web browser
```
http://<Public_domain_or_IP>/todo_list.php
```








