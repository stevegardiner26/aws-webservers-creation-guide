# aws-webservers-creation-guide
A creation guide to running a web server on AWS ```In this tutorial we use Ubuntu EC2 and an apache web server``` 
Here's a quick look of what I explain
* Creation of Ec2 Instance
* Creation of LAMP Stack WebServer in Instance
* Linking Domains to Websites on WebServer (Route 53 & Google Domains)
* Sending Emails using Php via Amazon SES
* SSL Verification for Domains
* Setting up a Git Deploy Webhook

## Creation of AWS Account
The first step on our journey here is creating an aws account. So head over to https://aws.amazon.com/ and do that.
Which in itself is very self explanatory, however you must make sure you:
  * *Veryfiy your Email*
  * *Verify your Address via Fax of a (Credit Card, Utilities, Phone Bill etc. anything with your address and name on it)*
  
If you do not do these two things within three days your account will be locked. And if you wait 90 days there will be no option to reopen you aws account to use the services/console.

## Creating an Ec2 Instance
This is how we are going to store our webserver on, and in this tutorial we are going to create an Ubuntu version of Ec2.

### Creating the Instance
1. Log into the Amazon Console and Navigate to the Ec2 tab under the Compute Category.
1. Click on the `Launch Instance` button to create a new Ec2 Instance.
1. Select `Ubuntu Server 18.04 LTS (HVM), SSD Volume Type` and make sure it says "Free Tier Eligible" on the side image, so you do not have to pay for it.
1. Keep Clicking Through and Create the Instance using all of the default options.

When you created the instance you should have gotten a `instance-name.pem` file. Keep this in a safe place because this is the only copy of the file you will get and it is important. 

### Creating a Security Group
Navigate to the Security Groups Tab on the lefthand side of the Instance Dashboard.

If you do not have a security group in the list shown. Create a new security group with the following rules (The name and description can be named whatever you want). 
If you already have a default security group select the group and click the `Actions` button and edit inbound and outbound rules.
`sg-id` is your security group ID which can be found on the security groups page as Security Group: sg-#######

   **Inbound Rules**
   
    HTTP        | TCP | 80      | Custom | 0.0.0.0/0
    All Traffic | All | 0-65535 | Custom | sg-id
    SSH         | TCP | 22      | My Ip  | [amazon will fill this in]

   **Outbound Rules**
   
    All Traffic | All | 0 - 65535 | Custom | 0.0.0.0/0 

The inbound `http` is needed so that we can access the webserver's pages from an inbound `http` request and by default `http` runs on Port `80`.
The `ssh` is so we can remote into the instance and `ssh` operates on Port `22`.

### Connecting via SSH
Now when we start to connect to our instance via ssh there are several ways we can do this and you can find this from going to your instance dashboard and selecting the "Running Instance" Tab and then clicking on the Connect button.

We are going to connect over a standalone ssh client. I am currently running this in iterm if you are on windows you are probably better off using the Java client unless you have a linux emulator such as [Ubuntu 18.04 for Windows](https://www.microsoft.com/en-us/p/ubuntu-1804-lts/9n9tngvndl3q?activetab=pivot:overviewtab).

1. Open up iterm or you ubuntu console or whatever you are deciding to use.
1. If you need a new ssh key enter:
    
     `$ ssh-keygen -t rsa -b 4096 -C "your_email@example.com"`
1. Remember that `instance-name.pem` file I said to save for later? Now navigate so that you are in the same directory as this file.
1. Enter this line to make sure that the file has the right permissions:

    `$ chmod 400 Web Servers.pem`
1. Now to start up an ssh connection to your instance type:

    `$ ssh -i "instance-name.pem" ubuntu@ec2-#-##-###-###.compute-1.amazonaws.com`

This is using the file, and your instance's public DNS which is different from mine and should be found easily in the connect button.

Now you are in the ubuntu virtual instance!
If you simply want to exit the instance just type: `$ logout`

##### Optional SSH Verification:
If you want to set up SSH verification so you do not need to reference the `.pem` file everytime you log in this is how you do that.

1. In your iterm type: 

    `$ cat ~/.ssh/id_rsa.pub | pbcopy`
1. Now log back into the instance using the `.pem` file this time.
1. When you are in your Ec2 instance use: 
    
    `$ nano ~/.ssh/authorized_keys`
    
1. Paste in your ssh key, and save.

Now you should be able to log into the instance with

    $ ssh ubuntu@ec2-#-##-###-###.compute-1.amazonaws.com


### Setting Up Web Servers
Now to set up the web server you need to be logged into the instance. We are going to be installing a LAMP Stack Server.

Make sure your ubuntu version is updated by running:
    
    $ sudo apt update && sudo apt upgrade

#### Installing Packages for LAMP Server
Install Apache 2.4 from Ubuntu

    $ sudo apt install apache2

Install the `mysql-server` package:

    $ sudo apt install mysql-server

Install Php with apache and mysql support:

    $ sudo apt install php7.2 libapache2-mod-php7.2 php-mysql
    
Optionally, install additional cURL, JSON, and CGI support:

    $ sudo apt install php-curl php-json php-cgi
    
#### Setting Up Apache
Now we are going to configure apache to run the server how we want it.
First lets configure the first apache config file, make sure you are in the root directory by executing `$ cd ~`. We are going to edit with vim by using:

    $ sudo vim /etc/apache2/apache2.conf
    
Make sure these values are set within the file, then save and exit:

    KeepAlive On
    MaxKeepAliveRequests 50
    KeepAliveTimeout 5
    
Now we want to restart apache via: 

    sudo systemctl restart apache2

#### Setting Up a WebServer/VirtualHost
For all of these steps make sure you replace `example.com` with your domain.
Make sure you are in the root directory still and copy the default `.conf` file and then we are going to edit the `example.com.conf`.

    $ sudo cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/example.com.conf
    $ sudo vim /etc/apache2/sites-available/example.com.conf

Make sure all these values in the file are the same as what I display below. Your file may not have certain lines so you will have to add them in. Then save and exit the file by pressing <kbd>ESC</kbd> and then typing `:wq!` and pressing <kbd>Enter</kbd>.

    <Directory /var/www/html/example.com/public_html>
        Require all granted
    </Directory>
    <VirtualHost *:80>
        ServerName example.com
        ServerAlias www.example.com
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html/example.com/public_html

        ErrorLog /var/www/html/example.com/logs/error.log
        CustomLog /var/www/html/example.com/logs/access.log combined
    </VirtualHost>

Now we must create the directories we just referenced in that file by executing:

    $ sudo mkdir -p /var/www/html/example.com/{public_html,logs}

*Make sure that you do not have a space bewteen `{public_html,` and `logs}`.*

Enable your site via apache, and disable the `default.conf` to limit security risks and the reload apache2:

    $ sudo a2ensite example.com
    $ sudo a2dissite 000-default.conf
    $ sudo systemctl reload apache2
   
Now virtual hosting your site is enabled. 

#### Setting Up MySQL (If you are not planning on using a database with your site then skip this step)
Log into the mysql console and create a database:

    $ sudo mysql -u root
    CREATE DATABASE webdata;
    GRANT ALL ON webdata.* TO 'webuser' IDENTIFIED BY 'password';
    quit
    
Use this command secure installation to install it. You can skip the password stage.

    $ sudo mysql_secure_installation
    
Enter **Y** for all the following prompts:

* Remove anonymous users?
* Disallow root login remotely?
* Remove test database and access to it?
* Reload privilege tables now?

#### Setting Up Php
To configure php properly use this command, and make sure you are in the root directory:

    $ sudo vim /etc/php/7.2/apache2/php.ini
    
Make sure these following lines are entered within the file.    

    error_reporting = E_COMPILE_ERROR | E_RECOVERABLE_ERROR | E_ERROR | E_CORE_ERROR
    max_input_time = 30
    error_log = /var/log/php/error.log
    
Create this log directory for php, give ownership to apache and then restart apache:

    $ sudo mkdir /var/log/php
    $ sudo chown www-data /var/log/php
    $ sudo systemctl restart apache2

#### Setting Up a Test Page
If you want to test if the server and website is up you will need a test page to navigate to after assigning the instance an elastic ip. You can remove the file after testing.

Navigate to `/var/www/html/example.com/public_html` and:

    $ touch servertest.php
    $ sudo vim servertest.php
    
Paste this into the `servertest.php`:

    <html>
      <head>
        <title>PHP Test</title>
      </head>
      <body>
        <?php echo '<p>Hello World</p>';

        // In the variables section below, replace user and password with your own MySQL credentials as created on your server
        $servername = "localhost";
        $username = "webuser";
        $password = "password";

        // Create MySQL connection
        $conn = mysqli_connect($servername, $username, $password);

        // Check connection - if it fails, output will include the error message
        if (!$conn) {
          die('<p>Connection failed: <p>' . mysqli_connect_error());
        }
        echo '<p>Connected successfully</p>';
        ?>
      </body>
    </html>
    
#### Inserting a Git Repo to the WebServer
If you want to put a repository in the webserver then enter this command from the root directory:

    $ sudo git pull [git-repo-clone-link] /var/www/html/example.com/public_html

### Assigning Elastic IP
Now we are going to assign an Elastic Ip to an Instance which is essential to route a domain or several domains to your instance.

*Be aware that amazon charges you for elastic ips that are not connected to any instance. So as long as you have connected ips you shouldn't be charged*

1. Go to the Ec2 Dashboard and go to the left side and scroll down untill you see the link that reads `Elastic IPs`.
1. Click on the "Allocate New Address" button and allocate from the Amazon pool and you should get an ip.
1. Select this Ip in your list, and click on the "Actions" dropdown and click "Associate Address".
1. Select the instance and private ip there should only be one option if this is the only instance you created.

That's it! Your Ec2 instance now has an ip binded to it. If you want to test if it is working go to `http://elastic-ip/servertest.php` and there should be the test webpage displaying that we set up in an earlier step. Read on to find out how to use this ip to link a domain.

## Pushing a Domain to the Ec2 Instance

### Using Google Domains
If you have a domain registered go to the manage tab. If you do not have one registered, register it and then go to the manage tab. 

1. Make sure the domain isn't already forwarding to another site.
1. Navigate to the DNS section of the manager.
1. Scroll down to "Custom Resource Records" and enter two records:

This is so we can support `domain-name.com` and `www.domain-name.com`

      @   | A     | 1h | elastic-ip
      www | CNAME | 1h | domain-name.com

Give your domain some time to update and sync itself, and then it should work on `http`. Many sources say it takes max 48 hrs to update the domain. 

However I have experienced after several minutes it updates. If you do not see any updates you may have to clear your cookies and cache and try again. That may solve the issue.

### Using Route 53
Navigate to Route 53 through the Amazon Web Services Console.

1. Register a domain through Route 53.
1. Navigate to "Hosted Zones" on the left side navigation menu. 
1. If you registered the domain properly there should be a hosted zone already created if not create one.
1. Click on the hosted zone for your domain and create a new "Record Set"

Leave the Name Blank, make sure the type is "A" and put your elastic-ip you connected to your Ec2 instance in the value section. Then create it. It may take a few minutes to update but then you should be able to navigate to your site from the domain name over `HTTP`. 

*You may have to clear you cache/cookies if you were visiting the site before you linked it with the domain.*

## Verifying Domain over HTTPS/SSL Cert
//TODO Im still trying to figure this out=
https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-apache-in-ubuntu-18-04


## How to add a new website to the apache server
For all of these steps make sure you replace `example.com` with your domain.
Make sure you are in the root directory still and copy the default `.conf` file and then we are going to edit the `example.com.conf`.

    $ sudo cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/example.com.conf
    $ sudo vim /etc/apache2/sites-available/example.com.conf

Make sure all these values in the file are the same as what I display below. Your file may not have certain lines so you will have to add them in. Then save and exit the file by pressing <kbd>ESC</kbd> and then typing `:wq!` and pressing <kbd>Enter</kbd>.

    <Directory /var/www/html/example.com/public_html>
        Require all granted
    </Directory>
    <VirtualHost *:80>
        ServerName example.com
        ServerAlias www.example.com
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html/example.com/public_html

        ErrorLog /var/www/html/example.com/logs/error.log
        CustomLog /var/www/html/example.com/logs/access.log combined
    </VirtualHost>
    
Now we must create the directories we just referenced in that file by executing:

    $ sudo mkdir -p /var/www/html/example.com/{public_html,logs}

*Make sure that you do not have a space bewteen `{public_html,` and `logs}`.*

**Enabling Sites**
Use this command to enable your site:

    $ sudo a2ensite example.com

**Disabling Sites**
Only use this command if you want to disable your site:

    $sudo a2dissite example.com
    
Now reload apache:

    $ sudo systemctl reload apache2

Now navigate to `/var/www/html/example.com` and execute this to place your repo within the site

    $ sudo git clone [git-repo-clone-link] public_html
    
Now connect the domain via Pushing a Domain to Ec2 Instance, and Enabling SSL Cert, and then you are all set.

## How to setup an email server for a website
https://docs.aws.amazon.com/ses/latest/DeveloperGuide/send-using-smtp-php.html
-Talk about SESSION?? and linking a form
-Talk about Scrolling to the new page
-Talk about sandbox mode only using verified emails 
-Explain the PHP
//moving the var file

## How to Setup a Git Deploy Webhook
This allows you to be able to remotely pull the changes into your ec2 instance without having to log in. You can do it from any cloned git repository.

## Resources

**Setting Up a Lamp Stack on Ubuntu**
https://www.linode.com/docs/web-servers/lamp/install-lamp-stack-on-ubuntu-18-04/
