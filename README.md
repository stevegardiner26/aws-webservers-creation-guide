# aws-webservers-creation-guide
A creation guide to running a web server on AWS ```[In this tutorial we use Ubuntu EC2 and an apache web server]```

## Creation of AWS Account
The first step on our journey here is creating an aws account. So head over to https://aws.amazon.com/ and do that.
Which in itself is very self explanatory, however you must make sure you:
  * *Veryfiy your Email*
  * *Verify your Address via Fax of a (Credit Card Bill, Utilities, Phone Bill etc [Anything with your address and name on it])*
  
If you do not do these two things within three days your account will be locked. And if you wait 90 days there will be no option to reopen you aws account to use the services/console.

## Ec2 Instance
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

1. In your iterm type: `$ cat ~/.ssh/id_rsa.pub | pbcopy`
1. Now log back into the instance using the `.pem` file this time.
1. When you are in your Ec2 instance use: `$ nano ~/.ssh/authorized_keys`
1. Paste in your ssh key, and save.

Now you should be able to log into the instance with

    `$ ssh ubuntu@ec2-#-##-###-###.compute-1.amazonaws.com`


### Setting Up Web Servers


### Assigning Elastic IP

## Pushing a Domain to the Ec2 Instance

### Using Route 53

### Using Google Domains

## Verifying Domain over HTTPS
https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-apache-in-ubuntu-18-04

## Other Useful Tips

### How to add a new website to the apache server
https://www.linode.com/docs/web-servers/lamp/install-lamp-stack-on-ubuntu-18-04/

### How to setup an email server for a website
https://docs.aws.amazon.com/ses/latest/DeveloperGuide/send-using-smtp-php.html
-Talk about SESSION?? and linking a form
-Talk about Scrolling to the new page

### How to Setup a Git Deploy Webhook
This allows you to be able to remotely pull the changes into your ec2 instance without having to log in. You can do it from any cloned git repository.

## Resources
https://www.linode.com/docs/web-servers/lamp/install-lamp-stack-on-ubuntu-18-04/
