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
*For this I use a free service called [Lets Encrypt](https://letsencrypt.org)*

Run these commands to install Let's Encrypt

    $ cd /usr/local
    $ sudo git clone https://github.com/letsencrypt/letsencrypt

Now this is used to assign a SSL per domain you have setup.

    $ cd /usr/local/letsencrypt
    $ sudo ./letsencrypt-auto --apache -d domain-name.com
    
The last line must be run for every domain and subdomain so for the `www.` version of that site you also have to run this:

    $ sudo ./letsencrypt-auto --apache -d www.domain-name.com

Now when it prompts you, enter an email so it can notify you for renewal and other important info and then the next prompt you can select whether you want anyone entering the site via `HTTP` to be redirected to the `HTTPS` version.

Your SSL Certificate is now valid for 90 days after the 90 days you just have to run the `-d` command again to get a new Cert. If you want to automate this you can follow the link in the **Reasources** tab at the bottom of this documentation where the site will show you how to do this.

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

## How to setup a Free AWS Email Server for a Website and Hook it up to a Form
*This tutorial is using an Ec2 instance as described before.*

### Setting Up the Service on AWS
First you want to log into your console and search for "Simple Email Service" and navigate to that page. By Default you will be in sandbox mode and will have to verify an email address to forward emails to it.

Navigate to "Email Addresses" on the left hand side and click on the blue button that says `Verify a New Email Address`. Type the email you want the form data to be sent to.

Go to that email's inbox and click on the link to verify your email with aws.

Navigate to "SMTP Settings" on the left hand side and create a new set of SMTP Credentials and save them we will need them for later.

### Setting up PHP Mailer
Php should already be installed if not make sure you install it.

Get Composer:
Now we want to install [Composer](https://getcomposer.org/download).

After composer is installed you can now install the PHP Mailer library

    $ composer require phpmailer/phpmailer

After that is installed we now need to hook this up to a DOM so users can interact with it.

### Hooking a DOM Form via PHP into the email
First we want to create a php file in our `public_html` directory to handle the emailing. We will name this file `form-to-email.php`.

In our `index.php` file or the `html` file with the form this is something that the form should look like.

    <form action="form-to-email.php" name="myemailform" method="POST">
        <div class="form-group">
            <label for="">Name: </label>
            <input class="form-control" name="name" type="text" placeholder="Your Name">
        </div>
        <div class="form-group">
            <label for="">Email:</label>
            <input class="form-control" name="email" type="email" placeholder="example@example.com" required>
        </div>
        <div class="form-group">
            <textarea name="message" id="" cols="30" rows="9" class="form-control" placeholder="Message..." required> . </textarea>
        </div>
        <button type="submit" style="margin-top: 20px; display: block;" class="mx-auto btn-success btn">Connect with Steve</button>
    </form>

Now the contents of the form and what it entails is totally customizable and optional. However the most important things here that you want to keep in mind is that:

* The `action` attribute on the `<form>` is the name/path of the php email file we just created, because this is where it is going to send the data.
* The `method` attribute on the `<form>` is `POST` you could also use `GET` but I choose to use `POST`.
* Make sure each `<input>` has a `name` attribute because we will use that later.
* Make sure your form has a `button` of type `submit` because this is what will trigger the sending of the form data to the php file.

Now go into your `form-to-email.php` and put this inside the file:

    <?php
    
    $name = $_POST['name'];
    $visitor_email = $_POST['email'];
    $message = $_POST['message'];
 

What we are doing here is creating variables ex: `$name` and setting their values to the `$_POST['name']` which is referencing the `name` attribute placed on each of the html `input` fields.

Paste this into the file underneath the declared variables the comments tell you what you need to replace:
*Anything Amazon SES SMTP Related is your Credentials we created earlier*

    // If necessary, modify the path in the require statement below to refer to the 
    // location of your Composer autoload.php file.
    require 'vendor/autoload.php';

    use PHPMailer\PHPMailer\PHPMailer;

    // Instantiate a new PHPMailer 
    $mail = new PHPMailer;

    // Tell PHPMailer to use SMTP
    $mail->isSMTP();

    // Replace sender@example.com with your "From" address. 
    // This address must be verified with Amazon SES.
    $mail->setFrom('sender@example.com', 'Sender Name');

    // Replace recipient@example.com with a "To" address. If your account 
    // is still in the sandbox, this address must be verified.
    // Also note that you can include several addAddress() lines to send
    // email to multiple recipients.
    $mail->addAddress('recipient@example.com', 'Recipient Name');

    // Replace smtp_username with your Amazon SES SMTP user name.
    $mail->Username = 'smtp_username';

    // Replace smtp_password with your Amazon SES SMTP password.
    $mail->Password = 'smtp_password';
    
    // Specify a configuration set. If you do not want to use a configuration
    // set, comment or remove the next line.
    $mail->addCustomHeader('X-SES-CONFIGURATION-SET', 'ConfigSet');
 
    // If you're using Amazon SES in a region other than US West (Oregon), 
    // replace email-smtp.us-west-2.amazonaws.com with the Amazon SES SMTP  
    // endpoint in the appropriate region.
    $mail->Host = 'email-smtp.us-west-2.amazonaws.com';

    // The subject line of the email
    $mail->Subject = 'Amazon SES test (SMTP interface accessed using PHP)';

    // The HTML-formatted body of the email
    $mail->Body = '<h1>Email Test</h1>
    <p>This email was sent through the 
    <a href="https://aws.amazon.com/ses">Amazon SES</a> SMTP
    interface using the <a href="https://github.com/PHPMailer/PHPMailer">
    PHPMailer</a> class.</p>';

    // Tells PHPMailer to use SMTP authentication
    $mail->SMTPAuth = true;

    // Enable TLS encryption over port 587
    $mail->SMTPSecure = 'tls';
    $mail->Port = 587;

    // Tells PHPMailer to send HTML-formatted email
    $mail->isHTML(true);

    // The alternative email body; this is only displayed when a recipient
    // opens the email in a non-HTML email client. The \r\n represents a 
    // line break.
    $mail->AltBody = "Email Test\r\nThis email was sent through the 
    Amazon SES SMTP interface using the PHPMailer class.";
   
    if(!$mail->send()) {
    echo "Email not sent. " , $mail->ErrorInfo , PHP_EOL;
    } else {
    echo "Email sent!" , PHP_EOL;
    }  
    
    ?>

The best way to do this in my opinion is to copy the entire `html` file that contains the form and replace the form with the confirmation/error message in the DOM. This looks a lot more professional, than just a white screen with text. Also you can put an `onload="document.getElementsByClassName('contact-container')[0].scrollIntoView({ behavior: 'smooth', block: 'center' });"` attribute on the `<body>` tag and the document will scroll to the error or success message on loading.


    if(!$mail->send()) {
    echo "Email not sent. " , $mail->ErrorInfo , PHP_EOL;
    } else {
    echo "Email sent!" , PHP_EOL;
    }  

This code here is where you can customize what is displayed after the form is submitted just include it within `echo`.

*An alternate way to do this might be changing a `$_SESSION['Variable']` on the main html page to conditionally show the form or message. I have not figured out how to do this yet it is just an idea.*

## How to Setup a Git Deploy Webhook
This allows you to be able to remotely pull the changes into your ec2 instance without having to log in. You can do it from any cloned git repository.
I'm not sure how to do this yet.

## Resources

**Setting Up a Lamp Stack on Ubuntu**

https://www.linode.com/docs/web-servers/lamp/install-lamp-stack-on-ubuntu-18-04/

**Setting Up and Amazon Simple Email Service**

https://docs.aws.amazon.com/ses/latest/DeveloperGuide/send-using-smtp-php.html

**Setting Up SSL Verification for your Site**

https://www.tecmint.com/install-free-lets-encrypt-ssl-certificate-for-apache-on-debian-and-ubuntu/


