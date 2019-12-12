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
-----------------
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

### Give grader access
-----------------
#### Create a new user grader and Give him `sudo` access
```
sudo adduser grader
sudo sudo visudo
```
Then add the following text `grader ALL=(ALL:ALL) ALL`

- verify grader sudo permissions.
	- switch to the grader user: `su - grader` then type the password
	- list any sudo privileges you have `sudo -l` then type the password

#### Setup SSH keys for grader
* On local machine 
	`ssh-keygen`
	- Enter the default path and name the file grader_key `~/.ssh/grader_key`.
	- Copy the public key from the grader_key.pub file.
	```
	cat ~/.ssh/grader_key.pub
	```
* On remote machine home as user grader
	- Create .ssh directory in the home directory, and authorized_keys file in that directory
	```
	mkdir .ssh
	touch .ssh/authorized_keys
	```
	Change permission of .ssh and .ssh/authorized_keys
	```
	sudo chmod 700 .ssh
	sudo chmod 600 .ssh/authorized_keys
	```
	add the public key to the authorized_keys
	```
	nano .ssh/authorized_keys
	```
	Then paste the contents of the public key copied from the grader_key.pub on the local machine

### Secure your server
-----------------
#### Update all currently installed packages.
```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get dist-upgrade
```

#### Change the SSH port from 22 to 2200 | Enforce key-based authentication | Disable login for root user
```
sudo nano /etc/ssh/sshd_config
```
- Then change the following:
   2. Change `PasswordAuthentication` to `no`.
   3. Change `Port` from `22` to `2200`.
   4. Change `PermitRootLogin` to `no`
- Save the file and run `sudo service ssh restart`

#### Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
- In AWS Lightsail Security Group,  add these rules ports 80(TCP), 123(UDP), and 2200(TCP), and delete port 22.
- in the terminal
```bash
$ sudo ufw default deny incoming
$ sudo ufw default allow outgoing
$ sudo ufw allow www
$ sudo ufw allow 123/udp
$ sudo ufw allow 2200/tcp
$ sudo ufw enable
```

### Prepare to deploy the project
-----------------
- Connect to yor server instance as grader user:
```
ssh -i /path/to/grader_key grader@ec2-YOUR_INSTANCE_PUBLIC_IP.compute.amazonaws.com
```
  In my case:
```
ssh -i grader_key -p 2200 grader@ec2-3-8-150-202.eu-west-2.compute.amazonaws.com
```

#### Change timezone to UTC and Fix language issues
```
sudo timedatectl set-timezone UTC
sudo update-locale LANG=en_US.utf8 LANGUAGE=en_US.utf8 LC_ALL=en_US.utf8
```

#### Install Apache, mod_wsgi and Git

1. `$ sudo apt-get install apache2`.
2. `$ sudo apt-get install libapache2-mod-wsgi python-dev` to install *mod_wsgi*.
	Note: For Python3 replace `libapache2-mod-wsgi` with `libapache2-mod-wsgi-py3`
3. `$ sudo a2enmod wsgi` to enable *mod_wsgi*.
4. `$ sudo service apache2 start`.
5. `$ sudo apt-get install git`.
