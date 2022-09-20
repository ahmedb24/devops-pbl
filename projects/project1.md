# **WEB STACK IMPLEMENTATION (LAMP STACK) IN AWS**
LAMP web stack is a technology stack that combines a set of frameworks and tools specifically chosen to work 
<br>together and used to develop a software product.  
<br>

# **Step 1 - Preparing prerequisites** 
In order to complete this project, an AWS account and a virtual server with Ubuntu Server OS is required.  
<br>

Creation of a new AWS account gives access to the free tier plan which allows to spin up a new EC2 instance
<br>(an instance of a virtual server) for free in only a matter of a few clicks.  
<br>

You can watch the videos below to learn how to Provision a server and connect to it.
- [AWS account setup and Provisioning an Ubuntu Server](https://www.youtube.com/watch?v=xxKuB9kJoYM&list=PLtPuNR8I4TvkwU7Zu0l0G_uwtSUXLckvh&index=6) 
- [Connecting to your EC2 Instance](https://www.youtube.com/watch?v=TxT6PNJts-s&list=PLtPuNR8I4TvkwU7Zu0l0G_uwtSUXLckvh&index=7)    
<br>

![instance running](../screenshots/project1/instance_running.jpg)
*EC2 machine in running state*  
<br>

Let us move on and configure our EC2 machine to serve a Web server!  
<br>

# **STEP 2 — INSTALLING APACHE AND UPDATING THE FIREWALL** 
What exactly is Apache?  
<br>

Apache HTTP Server is the most widely used web server software. Developed and maintained by Apache Software Foundation, 
<br>Apache is an open source software available for free. It runs on 67% of all webservers in the world. It is fast, reliable, and secure.  
<br>

The Apache web server is among the most popular web servers in the world. It’s well documented, has an active community
<br>of users, and has been in wide use for much of the history of the web, which makes it a great default choice for hosting 
<br>a website.   
<br>   

Install Apache using Ubuntu’s package manager ‘*apt*’:
<br>   

> #update a list of packages in package manager
<br>`sudo apt update`
<br>
<br>#run apache2 package installation
<br>`sudo apt install apache2`   

<br>To verify that apache2 is running as a Service in our OS, use following command:
>`sudo systemctl status apache2`  
<br>          

<br>![apache2 running](../screenshots/project1/apache2_running.jpg)
*Verifying apache2 is running*  
<br>

Before we can receive any traffic by our Web Server, we need to open TCP port 80 which is the default port that web browsers use 
<br>to access web pages on the Internet   

<br>![Opened port 80](../screenshots/project1/open_port_80.jpg)
*Opened port 80*  

<br>Now it is time for us to test how our Apache HTTP server can respond to requests from the Internet.
<br>Open a web browser of your choice and try to access following url
>`http://<Public-IP-Address>:80`

<br>If you see following page, then your web server is now correctly installed and accessible through your firewall.
<br>![Apache2 default page](../screenshots/project1/apache2_default_page.jpg)
<br>*Apache2 default page*   
<br>

# **STEP 3 — INSTALLING MYSQL** 
Now that you have a web server up and running, you need to install a Database Management System (DBMS) to be able to store and manage 
<br>data for your site in a relational database. MySQL is a popular relational database management system used within PHP environments, 
<br>so we will use it in our project.  
<br>
  
Again, use ‘apt’ to acquire and install this software:   
>`sudo apt install mysql-server`  
<br>       

<br>![mysql running](../screenshots/project1/mysql_running.jpg)
<br>*Verifying MYSQL is running*   
<br>

<br>It’s recommended that you run a security script that comes pre-installed with MySQL. This script will remove some insecure default settings 
<br>and lock down access to your database system. Before running the script you will set a password for the root user, using mysql_native_password 
<br>as default authentication method. We’re defining this user’s password as PassWord.1.   
<br>

>#Login to mysql as root
<br>`sudo mysql`  
<br>
#Change password
<br>`ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1'`;
<br>   
#exit the console
<br>`exit`
<br>  

<br>Start the interactive script by running:
>`sudo mysql_secure_installation`
<br>     

<br>When you’re finished, test if you’re able to log in to the MySQL console by typing:
>`sudo mysql -p`  
<br>#exit the console
<br>`exit`   
<br>

<br>For increased security, it’s best to have dedicated user accounts with less expansive privileges set up for every database, especially if you plan on 
<br>having multiple databases hosted on your server.   
<br>

Your MySQL server is now installed and secured. Next, we will install PHP, the final component in the LAMP stack.   
<br>

# **STEP 4 — INSTALLING PHP** 
You have Apache installed to serve your content and MySQL installed to store and manage your data. PHP is the component of our setup that will process
<br>code to display dynamic content to the end user. In addition to the php package, you’ll need php-mysql, a PHP module that allows PHP to communicate 
<br>with MySQL-based databases. You’ll also need libapache2-mod-php to enable Apache to handle PHP files. Core PHP packages will automatically be installed 
<br>as dependencies.  
<br>

To install these 3 packages at once, run:
>`sudo apt install php libapache2-mod-php php-mysql`   
<br>
#Confirm php is installed
<br>`php -v`   

<br>![php running](../screenshots/project1/php_running.jpg)
<br>*Verifying PHP is installed*   
<br>   

To test your setup with a PHP script, it’s best to set up a proper Apache Virtual Host to hold your website’s files and folders. Virtual host allows you to have <br>multiple websites located on a single machine and users of the websites will not even notice it.   
<br>

# **STEP 5 — CREATING A VIRTUAL HOST FOR YOUR WEBSITE USING APACHE** 
In this project, you will set up a domain called projectlamp, but you can replace this with any domain of your choice.   
<br> 

Apache on Ubuntu 20.04 has one server block enabled by default that is configured to serve documents from the /var/www/html directory.
<br>
We will leave this configuration as is and will add our own directory next next to the default one.   
<br>

Create directory and assign ownership of the directory with your current system user:
>#Create directory for projectlamp
<br>`sudo mkdir /var/www/projectlamp`   
<br>#Assign ownership
<br> `sudo chown -R $USER:$USER /var/www/projectlamp`   
<br>

Then, create and add configuration settings to a new configuration file in Apache’s sites-available directory using your preferred command-line editor:   
>`sudo vi /etc/apache2/sites-available/projectlamp.conf`

<br>![config created](../screenshots/project1/config_created.jpg)
<br>*Verifying porjectlamp config creation*   
<br>   

You might want to disable the default website that comes installed with Apache. This is required if you’re not using a custom domain name, because in this 
<br>case Apache’s default configuration would overwrite your virtual host. To disable Apache’s default website use a2dissite command and enable the newly 
<br>created virtual host, type:
>#enable virtual host
<br>`sudo a2ensite projectlamp`   
<br>#disable default virtual host
<br>`sudo a2dissite 000-default`   
<br>#Validate config file for syntax errors 
<br>`sudo apache2ctl configtest`   
<br>#Reload apache2 to effect changes
<br>`sudo systemctl reload apache2`   


<br>Your new website is now active, but the web root /var/www/projectlamp is still empty. Create an index.html file in that location so that we can test that the 
<br>virtual host works as expected:   
<br>

>`sudo echo 'Hello LAMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectlamp/index.html`   
<br>

<br>![virtualhost running](../screenshots/project1/virtualhost_running.jpg)
<br>*Verifying virtual host is serving projectlamp*   
<br>   

# **STEP 6 — ENABLE PHP ON THE WEBSITE** 
With the default DirectoryIndex settings on Apache, a file named index.html will always take precedence over an index.php file. This is useful for setting up <br>maintenance pages in PHP applications, by creating a temporary index.html file containing an informative message to visitors. Because this page will take <br>precedence over the index.php page, it will then become the landing page for the application. Once maintenance is over, the index.html is renamed or removed 
<br>from the document root, bringing back the regular application page.   
<br>  

Finally, we will create a PHP script to test that PHP is correctly installed and configured on your server.   
<br>

Now that you have a custom location to host your website’s files and folders, we’ll create a PHP test script to confirm that Apache is able to handle and process <br>requests for PHP files.   
<br>

Create a new file named index.php inside your custom web root folder and add the following file content:   
>`vim /var/www/projectlamp/index.php`
<br>
<br>#File content
<br>`<?php`
<br>`phpinfo();`   
<br>

<br>![virtualhost with php running](../screenshots/project1/virtualhost_with_php_running.jpg)
<br>*Verifying virtualhost with php*   
<br>   

After checking the relevant information about your PHP server through that page, it’s best to remove the file you created as it contains sensitive information about <br>your PHP environment -and your Ubuntu server. You can use rm to do so:   
>sudo rm /var/www/projectlamp/index.php   
<br>

<br>Congratulations! You have finished your very first REAL LIFE PROJECT by deploying a LAMP stack website in AWS Cloud!