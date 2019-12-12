# Linux-Server-Configuration-Project
Set-up information for Udacity Full Stack Nanodegree project on how to configure a Linux Server.

## Project Description

> Taking a baseline installation of a Linux distribution on a virtual machine and prepare it to host a web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

The application deployed here is the **[Item Catalog - Restaurants and Menus](https://github.com/THEPEACEMAKER/itemsCatalog)**.

## Server/App Info

- IP address: 3.8.150.202

- SSH port: 2200.

- Application URL: [http://3.8.150.202.xip.io/](http://3.8.150.202.xip.io/).

- Username and password for Udacity reviewer: `grader`, `grader`

## Configurations Steps
### Get your server
#### Start a new Ubuntu Linux server instance on Amazon Lightsail.
- Login into [Amazon Lightsail](https://lightsail.aws.amazon.com/ls/webapp/home/resources) using your Amazon Web Services account. If you don't already have an Amazon Web Services account, you'll be prompted to create one.
- Once you are login into, `Create instance`. 
- Select `Linux/Unix` platform, Select `OS Only` blueprint and Choose `Ubuntu 16.04 LTS`.
- Choose a instance plan.
- Give your instance a hostname.
- Then click `Create` to create the instance.
- Wait for it to start up.

#### Connect to the Server using ssh
- Open the instance you created on Amazon Lightsail.
- Click on `connect` tab and click on `Account page` at the end of the page then download the Default Private Key.
- In a terminal window run the following command after you specify the requeired information
```
ssh -i /path/my-key-pair.pem THE_USER_NAME_FOR_YOUR_AMI@ec2-YOUR_INSTANCE_PUBLIC_IP.compute.amazonaws.com
```
  In my case:
```
ssh -i LightsailDefaultKey.pem ubuntu@ec2-3-8-150-202.eu-west-2.compute.amazonaws.com
```
Source: [Connecting to Your Linux Instance Using SSH](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html).
