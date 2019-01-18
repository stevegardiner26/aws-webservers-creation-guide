# aws-webservers-creation-guide
A creation guide to running a web server on AWS [In this tutorial we use Ubuntu EC2 and an apache web server]

## Creation of AWS Account
The first step on our journey here is creating an aws account. So head over to https://aws.amazon.com/ and do that.
Which in itself is very self explanatory, however you must make sure you:
  * *Veryfiy your Email*
  * *Verify your Address via Fax of a (Credit Card Bill, Utilities, Phone Bill etc [Anything with your address and name on it])*
  
If you do not do these two things within three days your account will be locked. And if you wait 90 days there will be no option to reopen you aws account to use the services/console.

## Ec2 Instance
This is how we are going to store our webserver on, and in this tutorial we are going to create an Ubuntu version of Ec2.

### Creating the Instance
First Log into the Amazon Console and Navigate to the Ec2 tab

### Creating a Security Group

### Connecting via SSH

### Setting Up Web Servers
https://www.linode.com/docs/web-servers/lamp/install-lamp-stack-on-ubuntu-18-04/

### Assigning Elastic IP

## Pushing a Domain to the Ec2 Instance

### Using Route 53

### Using Google Domains

## Verifying Domain over HTTPS


## Other Useful Tips

### How to add a new website to the apache server
https://www.linode.com/docs/web-servers/lamp/install-lamp-stack-on-ubuntu-18-04/

### How to setup an email server for a website
https://docs.aws.amazon.com/ses/latest/DeveloperGuide/send-using-smtp-php.html
-Talk about SESSION?? and linking a form
-Talk about Scrolling to the new page

### How to Setup a Git Deploy Webhook
This allows you to be able to remotely pull the changes into your ec2 instance without having to log in. You can do it from any cloned git repository.
