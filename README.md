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

#### Install and configure PostgreSQL

1. Install some necessary Python packages for working with PostgreSQL: `$ sudo apt-get install libpq-dev python-dev`.
2. Install PostgreSQL: `$ sudo apt-get install postgresql postgresql-contrib`
3. PostgreSQL automatically creates a new user 'postgres' during its installation. So we can connect to the database by using postgres username with: `$ sudo -u postgres psql`
4. Create a new user called 'catalog' with a 'catalog' password: `# CREATE USER catalog WITH PASSWORD 'catalog';`
5. Give *catalog* user the CREATEDB permission: `# ALTER USER catalog CREATEDB;`
6. Create the 'catalog' database owned by *catalog* user: `# CREATE DATABASE catalog WITH OWNER catalog;`
7. Connect to the database: `# \c catalog`
8. Revoke all the rights: `# REVOKE ALL ON SCHEMA public FROM public;`
9. Lock down the permissions to only let *catalog* role create tables: `# GRANT ALL ON SCHEMA public TO catalog;`
10. Log out from PostgreSQL: `# \q`. Then return to the *grader* user: `$ exit`.
11. Edit the `database_setup.py` and `lotsofmenus.py` file:   
      Change `engine = create_engine('sqlite:///category.db')`
      to `engine = create_engine('postgresql://catalog:catalog@localhost/catalog')`
12. Remote connections to PostgreSQL should already be blocked. Double check by opening the config file:
    ```
    $ sudo nano /etc/postgresql/9.5/main/pg_hba.conf
    ```
    your file should look like this code :
    ```
    local   all             postgres                                peer
    local   all             all                                     peer
    host    all             all             127.0.0.1/32            md5
    host    all             all             ::1/128                 md5
    ```
Source: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps).

#### Configure Apache to serve a Python mod_wsgi application

1. Clone the *Item Catalog - Restaurants and Menus* app from Github in the /var/www/catalog/catalog directory.
   ```
   $ cd /var/www
   $ sudo mkdir catalog
   $ sudo chown -R grader:grader catalog
   $ cd catalog
   $ git clone https://github.com/THEPEACEMAKER/itemsCatalog catalog
   ```
   Run `mv application.py __init__.py` to rename `application.py` to `__init__.py`.
       - In `__init__.py` replace :
    ```
      app.debug = True
      app.run(host='0.0.0.0', port=5000)
    ```
    to :
    ```
      app.run()
    ```
    - In `__init__.py`, `database_setup.py` and `lotsofmenus.py` replace :
    ```
      engine = create_engine('sqlite:///restaurantmenuwithusers.db')
    ```
    to :
    ```
      engine = create_engine('postgresql://catalog:catalog@localhost/catalog')
    ```
    - In `__init__.py` replace :
    ```
      CLIENT_ID = json.loads(
        open('client_secrets.json', 'r').read())['web']['client_id']      
    ```
    to :
    ```
      CLIENT_ID = json.loads(
        open(r'/var/www/catalog/catalog/client_secrets.json').read())['web']['client_id']
    ```
    and
    ```
      CLIENT_SECRET_FILE = 'client_secrets.json'
    ```
    to :
    ```
      CLIENT_SECRET_FILE = '/var/www/catalog/catalog/client_secrets.json'
    ```

#### Setting up a virtual environment
Setting up a virtual environment will keep the application and its dependencies isolated from the main system. Changes to it will not affect the cloud server's system configurations.
3. Install pip , virtualenv (in /var/www/catalog)
   ```
   $ sudo apt-get install python-pip
   $ sudo pip install virtualenv
   $ sudo virtualenv venv
   $ source venv/bin/activate
   $ sudo chmod -R 777 venv
   ```
4. Install Flask and other dependencies (in /var/www/catalog/catalog) :
```
  pip install flask
  pip install sqlalchemy
  pip install psycopg2
  pip install httplib2
  pip install requests
  pip install --upgrade oauth2client
  <!-- ADEL: you might need more -->
```
- `python __init__.py` to test that everything works fine.
- `deactivate` to deactivate the virtual environment.
