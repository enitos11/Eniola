## INSTALLING THE NGINX WEB SERVERIn order to display web pages to our site visitors, we are going to employ Nginx, a high-performance web server. We’ll use the apt package manager to install this package.

Since this is our first time using apt for this session, start off by updating your server’s package index. Following that, you can use apt install to get Nginx installed:

sudo apt update
sudo apt install nginx
When prompted, enter Y to confirm that you want to install Nginx. Once the installation is finished, the Nginx web server will be active and running on your Ubuntu 20.04 server.

To verify that nginx was successfully installed and is running as a service in Ubuntu, run:

sudo systemctl status nginx

![project 2 step 2](https://user-images.githubusercontent.com/105660679/172173490-c468fb65-4ae4-4c51-95bf-7f0690bfcf16.JPG)
Our server is running and we can access it locally and from the Internet (Source 0.0.0.0/0 means ‘from any IP address’).

First, let us try to check how we can access it locally in our Ubuntu shell, run:

##curl http://localhost:80
or
##curl http://127.0.0.1:80
curl -s http://169.254.169.254/latest/meta-data/public-ipv4
![Project 2 step 4](https://user-images.githubusercontent.com/105660679/172174306-344c1269-df4a-45b6-b03c-c6aa747cc394.JPG)
![project 2 step 3](https://user-images.githubusercontent.com/105660679/172173772-cb6605e1-c6d8-4d05-adc3-5af3c394a6dd.JPG)

 ##INSTALLING MYSQL
 Now that you have a web server up and running, you need to install a Database Management System (DBMS) to be able to store and manage data for your site in a relational database. MySQL is a popular relational database management system used within PHP environments, so we will use it in our project.

Again, use ‘apt’ to acquire and install this software:

$ sudo apt install mysql-server
When prompted, confirm installation by typing Y, and then ENTER.

When the installation is finished, log in to the MySQL console by typing:

$ sudo mysql
mysql> exit
Notice that you need to provide a password to connect as the root user.

For increased security, it’s best to have dedicated user accounts with less expansive privileges set up for every database, especially if you plan on having multiple databases hosted on your server.

Note: At the time of this writing, the native MySQL PHP library mysqlnd doesn’t support caching_sha2_authentication, the default authentication method for MySQL 8. For that reason, when creating database users for PHP applications on MySQL 8, you’ll need to make sure they’re configured to use mysql_native_password instead. We’ll demonstrate how to do that in Step 6.
Your MySQL server is now installed and secured. Next, we will install PHP, the final component in the LEMP stack.

PREV
step 1 – installing the nginx web server
NEXT
step 3 – installing php
Layer-2
Career Path
AWS
Google
Microsoft
Digital Ocean
Links
Contact Us
DAREY.IO - A Registered Trademark of Zooto Technologies LTD UK



![Project 2 step 4](https://user-images.githubusercontent.com/105660679/172174306-344c1269-df4a-45b6-b03c-c6aa747cc394.JPG)
![Project 2 step 5](https://user-images.githubusercontent.com/105660679/172174783-d96e6524-9465-4a29-8143-52840acc5cfa.JPG)

#INSTALLING PHP
Installing PHP
You have Nginx installed to serve your content and MySQL installed to store and manage your data. Now you can install PHP to process code and generate dynamic content for the web server.

While Apache embeds the PHP interpreter in each request, Nginx requires an external program to handle PHP processing and act as a bridge between the PHP interpreter itself and the web server. This allows for a better overall performance in most PHP-based websites, but it requires additional configuration. You’ll need to install php-fpm, which stands for “PHP fastCGI process manager”, and tell Nginx to pass PHP requests to this software for processing. Additionally, you’ll need php-mysql, a PHP module that allows PHP to communicate with MySQL-based databases. Core PHP packages will automatically be installed as dependencies.

To install these 2 packages at once, run:

sudo apt install php-fpm php-mysql
![Project 2 step 5](https://user-images.githubusercontent.com/105660679/172175645-405d76f1-577b-459a-b1bc-ada18373d3b9.JPG)
![Project 2 step 6](https://user-images.githubusercontent.com/105660679/172182918-8b4894f3-d2a3-4814-9ebb-50577dc0dcbe.JPG)

##CONFIGURING NGINX TO USE PHP PROCESSOR
 Configuring Nginx to Use PHP Processor
When using the Nginx web server, we can create server blocks (similar to virtual hosts in Apache) to encapsulate configuration details and host more than one domain on a single server. In this guide, we will use projectLEMP as an example domain name.

On Ubuntu 20.04, Nginx has one server block enabled by default and is configured to serve documents out of a directory at /var/www/html. While this works well for a single site, it can become difficult to manage if you are hosting multiple sites. Instead of modifying /var/www/html, we’ll create a directory structure within /var/www for the your_domain website, leaving /var/www/html in place as the default directory to be served if a client request does not match any other sites.

Create the root web directory for your_domain as follows:
sudo mkdir /var/www/projectLEMP
![Project 2 step 6](https://user-images.githubusercontent.com/105660679/172182918-8b4894f3-d2a3-4814-9ebb-50577dc0dcbe.JPG)

sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/
This will tell Nginx to use the configuration next time it is reloaded. You can test your configuration for syntax errors by typing:

sudo nginx -t
You shall see following message:

nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
If any errors are reported, go back to your configuration file to review its contents before continuing.

We also need to disable default Nginx host that is currently configured to listen on port 80, for this run:

sudo unlink /etc/nginx/sites-enabled/default
When you are ready, reload Nginx to apply the changes:

sudo systemctl reload nginx
Your new website is now active, but the web root /var/www/projectLEMP is still empty. Create an index.html file in that location so that we can test that your new server block works as expected:

sudo echo 'Hello LEMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectLEMP/index.html



sudo mkdir /var/www/projectLEMP
Next, assign ownership of the directory with the $USER environment variable, which will reference your current system user:

sudo chown -R $USER:$USER /var/www/projectLEMP
Then, open a new configuration file in Nginx’s sites-available directory using your preferred command-line editor. Here, we’ll use nano:

sudo nano /etc/nginx/sites-available/projectLEMP
This will create a new blank file. Paste in the following bare-bones configuration:

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
![project 2](https://user-images.githubusercontent.com/105660679/172188090-e3fe563c-c0c3-423d-81bd-ffeafff7a0b9.JPG)
Now go to your browser and try to open your website URL using IP address:

http://<Public-IP-Address>:80
If you see the text from ‘echo’ command you wrote to index.html file, then it means your Nginx site is working as expected.
In the output you will see your server’s public hostname (DNS name) and public IP address. You can also access your website in your browser by public DNS name, not only by IP – try it out, the result must be the same (port is optional)

http://<Public-DNS-Name>:80
  ![Project 2 step 7,1](https://user-images.githubusercontent.com/105660679/172184269-32bbf6a7-eb94-4a44-ad40-f62390d995dd.JPG)

  
  ##TESTING PHP WITH NGINX
  Testing PHP with Nginx
Your LEMP stack should now be completely set up.

At this point, your LAMP stack is completely installed and fully operational.

You can test it to validate that Nginx can correctly hand .php files off to your PHP processor.

You can do this by creating a test PHP file in your document root. Open a new file called info.php within your document root in your text editor:

sudo nano /var/www/projectLEMP/info.php
Type or paste the following lines into the new file. This is valid PHP code that will return information about your server:

<?php
phpinfo();
You can now access this page in your web browser by visiting the domain name or public IP address you’ve set up in your Nginx configuration file, followed by /info.php:

http://`server_domain_or_IP`/info.php

![Project 2 step 7,2](https://user-images.githubusercontent.com/105660679/172185121-18d3e694-0fab-4340-b939-c47b9e2870b5.JPG)
![Project 2 step 7,1](https://user-images.githubusercontent.com/105660679/172183778-33c613bb-7642-4cb4-bbf8-1c0658c32b28.JPG)

 ##RETRIEVING DATA FROM MYSQL DATABASE WITH PHP (CONTINUED)
Retrieving data from MySQL database with PHP
In this step you will create a test database (DB) with simple "To do list" and configure access to it, so the Nginx website would be able to query data from the DB and display it.

At the time of this writing, the native MySQL PHP library mysqlnd doesn’t support caching_sha2_authentication, the default authentication method for MySQL 8. We’ll need to create a new user with the mysql_native_password authentication method in order to be able to connect to the MySQL database from PHP.

We will create a database named example_database and a user named example_user, but you can replace these names with different values.

First, connect to the MySQL console using the root account:

sudo mysql
To create a new database, run the following command from your MySQL console:

mysql> CREATE DATABASE `example_database`;
Now you can create a new user and grant him full privileges on the database you have just created.

The following command creates a new user named example_user, using mysql_native_password as default authentication method. We’re defining this user’s password as password, but you should replace this value with a secure password of your own choosing.

mysql>  CREATE USER 'example_user'@'%' IDENTIFIED WITH mysql_native_password BY 'password';
Now we need to give this user permission over the example_database database:

mysql> GRANT ALL ON example_database.* TO 'example_user'@'%';
![Project 2 step 7,2](https://user-images.githubusercontent.com/105660679/172185121-18d3e694-0fab-4340-b939-c47b9e2870b5.JPG)
![Project 2 step 7](https://user-images.githubusercontent.com/105660679/172186129-48bb4a99-91af-4e11-944d-107f6b52d7df.JPG)
mysql> exit
You can test if the new user has the proper permissions by logging in to the MySQL console again, this time using the custom user credentials:

mysql -u example_user -p
Notice the -p flag in this command, which will prompt you for the password used when creating the example_user user. After logging in to the MySQL console, confirm that you have access to the example_database database:

mysql> SHOW DATABASES;
This will give you the following output:

Output
+--------------------+
| Database           |
+--------------------+
| example_database   |
| information_schema |
+--------------------+
2 rows in set (0.000 sec)
Next, we’ll create a test table named todo_list. From the MySQL console, run the following statement:

CREATE TABLE example_database.todo_list (
mysql>     item_id INT AUTO_INCREMENT,
mysql>     content VARCHAR(255),
mysql>     PRIMARY KEY(item_id)
mysql> );
Insert a few rows of content in the test table. You might want to repeat the next command a few times, using different VALUES:

mysql> INSERT INTO example_database.todo_list (content) VALUES ("My first important item");
To confirm that the data was successfully saved to your table, run:

mysql>  SELECT * FROM example_database.todo_list;
You’ll see the following output:

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
After confirming that you have valid data in your test table, you can exit the MySQL console:

mysql> exit
Now you can create a PHP script that will connect to MySQL and query for your content. Create a new PHP file in your custom web root directory using your preferred editor. We’ll use vi for that:

nano /var/www/projectLEMP/todo_list.php
The following PHP script connects to the MySQL database and queries for the content of the todo_list table, displays the results in a list. If there is a problem with the database connection, it will throw an exception.

Copy this content into your todo_list.php script:

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
![Project 2 step 7](https://user-images.githubusercontent.com/105660679/172186644-e1fe712f-bf17-4037-aea3-19f7403cc874.JPG)
![Project 2 step 8](https://user-images.githubusercontent.com/105660679/172186857-d8cd7dca-338b-48ca-8227-742a206d6c18.JPG)


