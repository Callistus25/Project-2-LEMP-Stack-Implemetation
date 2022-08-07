## WEB STACK IMPLEMENTATION (LEMP STACK) ##

1. ### STEP 1 – INSTALLING THE NGINX WEB SERVER ###
- To employ Nginx, a high-performance web server, we "ll use the apt package manager to install this package. Update package and install Ngix
```
sudo apt update
sudo apt install nginx
```
![Sudo Apt Update, sudo apt install nginx](https://user-images.githubusercontent.com/107906178/183231623-59c00587-f15c-498f-b15a-7ab5b44e7b63.JPG)

![sudo apt install nginx](https://user-images.githubusercontent.com/107906178/183231624-7177e09e-74d5-4018-879d-05725d57a31c.JPG)
- Verify that nginx was successfully installed and is running as a service in Ubuntu. Output should be green coloured "running".
```
sudo systemctl status nginx
```
![sudo systemctl status nginx](https://user-images.githubusercontent.com/107906178/183231626-c8bc707c-d164-454f-91ea-d3bc44664663.jpg)

- Open TCP port 80 on the EC2 instance, which is default port that web browsers use to access web pages in the Internet.

- Try to check that if can be accessed locally in the Ubuntu shell, run:
```
curl http://localhost:80
or
curl http://127.0.0.1:80
```
![Curl local host](https://user-images.githubusercontent.com/107906178/183231627-71052e48-7e90-459b-8a34-c9c30786024d.JPG)

- Test that the Nginx server can respond to requests from the Internet by opening a web browser accessing the url using the IP address
```
http://<Public-IP-Address>:80
```
![Public-IP-Address, NGinx launch](https://user-images.githubusercontent.com/107906178/183231628-617db554-8902-4eb0-a649-a7fa3a07b9a8.JPG)
Above image shows that the web server is correctly installed and accessible through the firewall.

2. ### STEP 2 — INSTALLING MYSQL ###
- Install a Database Management System (DBMS) to be able to store and manage data for your site in a relational database.
```
sudo apt install mysql-server
```
![sudo apt update my sql](https://user-images.githubusercontent.com/107906178/183231629-bb01acb9-1eb9-431a-b9ed-49c13df79d27.JPG)

- When the installation is finished, log in to the MySQL console by typing:
```
sudo mysql
```
This will connect to the MySQL server as the administrative database user root, which is inferred by the use of sudo when running this command. Output shown below:
![sudo mysql](https://user-images.githubusercontent.com/107906178/183231630-b4ad1f52-c5ae-487f-95a0-6f87af0b23a7.JPG)

It’s recommended that you run a security script that comes pre-installed with MySQL. This script will remove some insecure default settings and lock down access to your database system. Before running the script you will set a password for the root user, using mysql_native_password as default authentication method. Defining this user’s password as ** PassWord.1 **
```
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';
```
- Exit the MySQL:
```
mysql> exit
```
- Start the interactive script by running:
```
sudo mysql_secure_installation
```
This will ask if you want to configure the VALIDATE PASSWORD PLUGIN. Enabling this feature is something of a judgment call. If enabled, passwords which don’t match the specified criteria will be rejected by MySQL with an error. It is safe to leave validation disabled, but you should always use strong, unique passwords for database credentials.
Answer Y for yes, or anything else to continue without enabling.
- Test if you’re able to log in to the MySQL console by typing:
```
sudo mysql -p
```
You need to provide a password to connect as the root user. For increased security, it’s best to have dedicated user accounts with less expansive privileges set up for every database, especially if you plan on having multiple databases hosted on your server.

MySQL server is now installed and secured.

3. ### STEP 3 – INSTALLING PHP ###
- Install PHP to process code and generate dynamic content for the web server. Note that while Apache embeds the PHP interpreter in each request, Nginx requires an external program to handle PHP processing and act as a bridge between the PHP interpreter itself and the web server. This allows for a better overall performance in most PHP-based websites, but it requires additional configuration. Need to install php-fpm, which stands for “PHP fastCGI process manager”, and tell Nginx to pass PHP requests to this software for processing. Additionally, need php-mysql, a PHP module that allows PHP to communicate with MySQL-based databases. Core PHP packages will automatically be installed as dependencies.
To install these 2 packages at once, run:
```
sudo apt install php-fpm php-mysql
```
![sudo apt install php-fpm php-mysql](https://user-images.githubusercontent.com/107906178/183231631-3a3a778c-fd9f-4236-914b-72ad9224a073.JPG)
Next, you will configure Nginx to use them

4. ### STEP 4 — CONFIGURING NGINX TO USE PHP PROCESSOR ###
Nginx web server, we can create server blocks (similar to virtual hosts in Apache) to encapsulate configuration details and host more than one domain on a single server.
- Create the root web directory for ** your_domain ** as follows:
```
sudo mkdir /var/www/projectLEMP
```
-Assign ownership of the directory with the $USER environment variable, which will reference your current system user:
```
sudo chown -R $USER:$USER /var/www/projectLEMP
```
- Open a new configuration file in Nginx’s sites-available directory using any command-line editor. using Nano;
```
sudo nano /etc/nginx/sites-available/projectLEMP
```
This will create a new blank file. Paste in the following bare-bones configuration:
```
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
Here’s what each of these directives and location blocks do:

**listen** — Defines what port Nginx will listen on. In this case, it will listen on port 80, the default port for HTTP.

**Root** — Defines the document root where the files served by this website are stored.

**Index** — Defines in which order Nginx will prioritize index files for this website. It is a common practice to list index.html files with a higher precedence than index.php files to allow for quickly setting up a maintenance landing page in PHP applications. You can adjust these settings to better suit your application needs.

**Server_name** — Defines which domain names and/or IP addresses this server block should respond for. Point this directive to your server’s domain name or public IP address.

**Location /** — The first location block includes a try_files directive, which checks for the existence of files or directories matching a URI request. If Nginx cannot find the appropriate resource, it will return a 404 error.

**Location ~ \.php$** — This location block handles the actual PHP processing by pointing Nginx to the fastcgi-php.conf configuration file and the php7.4-fpm.sock file, which declares what socket is associated with php-fpm.

**Location ~ /\.ht** — The last location block deals with .htaccess files, which Nginx does not process. By adding the deny all directive, if any .htaccess files happen to find their way into the document root ,they will not be served to visitors.

After editing, save and close the file. For nano, you can do so by typing CTRL+X and then y and ENTER to confirm.
- Activate your configuration by linking to the config file from Nginx’s sites-enabled directory:
```
sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/
```
This will tell Nginx to use the configuration next time it is reloaded. You can test your configuration for syntax errors by typing:
```
sudo nginx -t
```
![sudo nginx -t](https://user-images.githubusercontent.com/107906178/183231632-77b2e097-eb44-4fb1-9b74-bee6ce7f26aa.JPG)

- Disable default Nginx host that is currently configured to listen on port 80, for this run:
```
sudo unlink /etc/nginx/sites-enabled/default
```
- Reload Nginx to apply the changes
```
sudo systemctl reload nginx
```
The new website is now active, but the web root /var/www/projectLEMP is still empty.
- Create an index.html file in that location so that we can test that your new server block works as expected:
```
sudo echo 'Hello LEMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectLEMP/index.html
```
- Go to your browser and try to open the website URL using IP address:
```
http://<Public-IP-Address>:80
```
![NGinx index html Launch through IP address](https://user-images.githubusercontent.com/107906178/183231633-f676d8cd-ef5e-45d9-967b-aaa2ed76d010.JPG)

You can also access your website in your browser by public DNS name:

![NGinx index html Launch through DNS](https://user-images.githubusercontent.com/107906178/183231634-920a4ebd-5179-47f3-bf88-8eaac4c8fc16.JPG)

Your LEMP stack is now fully configured.

You can leave this file in place as a temporary landing page for your application until you set up an index.php file to replace it. Once you do that, remember to remove or rename the index.html file from your document root, as it would take precedence over an index.php file by default.

5. ### STEP 5 – TESTING PHP WITH NGINX ###
- Test to validate that Nginx can correctly hand .php files off to your PHP processor by creating a test PHP file in your document root. Open a new file called **info.php** within your document root in your text editor:
```
sudo nano /var/www/projectLEMP/info.php
```
- Type or paste the following lines into the new file. This is valid PHP code that will return information about your server:
```
<?php
phpinfo();
```
- Access this page in your web browser by visiting the domain name or public IP address you’ve set up in your Nginx configuration file, followed by /info.php:
```
http://`server_domain_or_IP`/info.php
```
![LEMP PHP installation complete](https://user-images.githubusercontent.com/107906178/183231636-21daeaf1-6c41-46e6-844d-171d51031f7e.JPG)

After checking the relevant information about your PHP server through that page, it’s best to remove the file you created as it contains sensitive information about your PHP environment and your Ubuntu server. You can use rm to remove that file:
```
sudo rm /var/www/your_domain/info.php
```
6. ### STEP 6 – RETRIEVING DATA FROM MYSQL DATABASE WITH PHP ###
- Create a test database (DB) with simple "To do list" and configure access to it, so the Nginx website would be able to query data from the DB and display it.
- Create a database named **database** and a user named **my_user**, but you can replace these names with different values.

First, connect to the MySQL console using the root account:
```
sudo mysql
```
Create a new database, run the following command from your MySQL console:
```
mysql> CREATE DATABASE `my_database`;
```
Create a new user and grant the user full privileges on the database you have just created, using mysql_native_password as default authentication method and also defining this user’s password as **password**;

```
mysql>  CREATE USER 'my_user'@'%' IDENTIFIED WITH mysql_native_password BY 'password';
```
To grant the user permission over the my_database database while preventing this user from creating or modifying other databases on your server:
```
mysql> GRANT ALL ON my_database.* TO 'my_user'@'%';
```

Now exit the MySQL shell with:
```
mysql> exit
```
- Test if the new user has the proper permissions by logging in to the MySQL console again, this time using the custom user credentials and password:
```
mysql -u my_user -p 
```
- After logging in to the MySQL console, confirm that you have access to the my_database database:
```
mysql> SHOW DATABASES;
```
![Mysql -u user -p](https://user-images.githubusercontent.com/107906178/183231637-f50ffc97-7a96-4fd6-ad57-f450c1c6549c.JPG)

- Create a test table named todo_list. From the MySQL console, run the following statement:
```
CREATE TABLE example_database.todo_list (
mysql>     item_id INT AUTO_INCREMENT,
mysql>     content VARCHAR(255),
mysql>     PRIMARY KEY(item_id)
mysql> );
```
Insert a few rows of content in the test table and repeat the below command with different values:
```
mysql> INSERT INTO my_database.todo_list (content) VALUES ("My first important item");
```
- Confirm that the data was successfully saved to your table, run:
```
mysql>  SELECT * FROM my_database.todo_list;
```
![SELECT FROM example_database todo_list; todo_list](https://user-images.githubusercontent.com/107906178/183231638-253fb654-a8a0-4bd5-aab2-aef56c334812.jpg)

After confirming that you have valid data in your test table, you can exit the MySQL console:
```
mysql> exit
```
- Create a PHP script that will connect to MySQL and query for your content. Create a new PHP file in your custom web root directory using your preferred editor. We’ll use vi for that:
```
nano /var/www/projectLEMP/todo_list.php
```
The following PHP script connects to the MySQL database and queries for the content of the todo_list table, displays the results in a list. If there is a problem with the database connection, it will throw an exception.

Copy the below content into your todo_list.php script:
```
<?php
$user = "my_user";
$password = "password";
$database = "my_database";
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
```
Save and close the file. 

- Access this page in your web browser by visiting the domain name or public IP address configured for your website, followed by /todo_list.php:
```
http://<Public_domain_or_IP>/todo_list.php
```
![PHP successful connection to mysql](https://user-images.githubusercontent.com/107906178/183231641-9127ac22-84f5-40e3-a4cf-a2f6924c1b62.JPG)
