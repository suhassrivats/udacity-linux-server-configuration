# Linux Server Configuration

This is the final project for Udacity's [Full Stack Web Developer Nanodegree](https://www.udacity.com/course/full-stack-web-developer-nanodegree--nd004).

This page explains how to secure and set up a Linux distribution on a virtual machine, install and configure a web and database server to host a web application.
- The Linux distribution is [Ubuntu](https://www.ubuntu.com/download/server) 16.04 LTS.
- The virtual private server is [Amazon Lighsail](https://lightsail.aws.amazon.com/).
- The web application is my [Item Catalog project](https://github.com/boisalai/udacity-catalog-app) created earlier in this Nanodegree program.
- The database server is [PostgreSQL](https://www.postgresql.org/).
- My local machine is a MacBook Pro (Mac OS X 10_12_6).

You can visit http://52.39.68.54/ or http://ec2-52-39-68-54.us-west-2.compute.amazonaws.com/ for the website deployed.

## Get a server

### Step 1: Start a new Ubuntu Linux server instance on Amazon Lightsail

- Login into [Amazon Lightsail](https://lightsail.aws.amazon.com/ls/webapp/home/resources) using an Amazon Web Services account.
- Once you are login into the site, click `Create instance`.
- Choose `Linux/Unix` platform, `OS Only` and  `Ubuntu 16.04 LTS`.
- Choose a instance plan (I took the cheapest, $5/month).
- Keep the default name provided by AWS or rename your instance.
- Click the `Create` button to create the instance.
- Wait for the instance to start up.

**Reference**
- ServerPilot, [How to Create a Server on Amazon Lightsail](https://serverpilot.io/community/articles/how-to-create-a-server-on-amazon-lightsail.html).


### Step 2: SSH into the server

- From the `Account` menu on Amazon Lightsail, click on `SSH keys` tab and download the Default Private Key.
- Move this private key file named `LightsailDefaultPrivateKey-*.pem` into the local folder `~/.ssh` and rename it `lightsail_key.rsa`.
- In your terminal, type: `chmod 600 ~/.ssh/lightsail_key.rsa`.
- To connect to the instance via the terminal: `ssh -i ~/.ssh/lightsail_key.rsa ubuntu@52.39.68.54`,
  where `52.39.68.54` is the public IP address of the instance.


## Secure the server

### Step 3: Update and upgrade installed packages

```
sudo apt-get update
sudo apt-get upgrade
```


### Step 4: Change the SSH port from 22 to 2200

- Edit the `/etc/ssh/sshd_config` file: `sudo nano /etc/ssh/sshd_config`.
- Change the port number on line 5 from `22` to `2200`.
- Save and exit using CTRL+X and confirm with Y.
- Restart SSH: `sudo service ssh restart`.


### Step 5: Configure the Uncomplicated Firewall (UFW)

- Configure the default firewall for Ubuntu to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
  ```
  sudo ufw status                  # The UFW should be inactive.
  sudo ufw default deny incoming   # Deny any incoming traffic.
  sudo ufw default allow outgoing  # Enable outgoing traffic.
  sudo ufw allow 2200/tcp          # Allow incoming tcp packets on port 2200.
  sudo ufw allow www               # Allow HTTP traffic in.
  sudo ufw allow 123/udp           # Allow incoming udp packets on port 123.
  sudo ufw deny 22                 # Deny tcp and udp packets on port 53.
  ```

- Turn UFW on: `sudo ufw enable`. The output should be like this:
  ```
  Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
  Firewall is active and enabled on system startup
  ```

- Check the status of UFW to list current roles: `sudo ufw status`. The output should be like this:

  ```
  Status: active

  To                         Action      From
  --                         ------      ----
  2200/tcp                   ALLOW       Anywhere
  80/tcp                     ALLOW       Anywhere
  123/udp                    ALLOW       Anywhere
  22                         DENY        Anywhere
  2200/tcp (v6)              ALLOW       Anywhere (v6)
  80/tcp (v6)                ALLOW       Anywhere (v6)
  123/udp (v6)               ALLOW       Anywhere (v6)
  22 (v6)                    DENY        Anywhere (v6)
  ```

- Exit the SSH connection: `exit`.

- Click on the `Manage` option of the Amazon Lightsail Instance,
then the `Networking` tab, and then change the firewall configuration to match the internal firewall settings above.

- Allow ports 80(TCP), 123(UDP), and 2200(TCP), and deny the default port 22.

- From your local terminal, run: `ssh -i ~/.ssh/lightsail_key.rsa -p 2200 ubuntu@52.39.68.54`, where `52.39.68.54` is the public IP address of the instance.

**References**
- Official Ubuntu Documentation, [UFW - Uncomplicated Firewall](https://help.ubuntu.com/community/UFW).
- TechRepublic, [How to install and use Uncomplicated Firewall in Ubuntu](https://www.techrepublic.com/article/how-to-install-and-use-uncomplicated-firewall-in-ubuntu/).


## Give `grader` access

### Step 6: Create a new user account named `grader`

- While logged in as `ubuntu`, add user: `sudo adduser grader`.
- Enter a password (twice) and fill out information for this new user.


### Step 7: Give `grader` the permission to sudo

- Edits the sudoers file: `sudo visudo`.
- Search for the line that looks like this:
  ```
  root    ALL=(ALL:ALL) ALL
  ```

- Below this line, add a new line to give sudo privileges to `grader` user.
  ```
  root    ALL=(ALL:ALL) ALL
  grader  ALL=(ALL:ALL) ALL
  ```

- Save and exit using CTRL+X and confirm with Y.
- Verify that `grader` has sudo permissions. Run `su - grader`, enter the password,
run `sudo -l` and enter the password again. The output should be like this:

  ```
  Matching Defaults entries for grader on ip-172-26-3-199.us-west-2.compute.internal:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

  User grader may run the following commands on ip-172-26-3-199.us-west-2.compute.internal:
    (ALL : ALL) ALL
  ```

**Resources**
- DigitalOcean, [How To Add and Delete Users on an Ubuntu 14.04 VPS](https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps)


### Step 8: Create an SSH key pair for `grader` using the `ssh-keygen` tool

- On the local machine:
  - Run `ssh-keygen`
  - Enter file in which to save the key (I gave the name `udacity_key`) in the local directory `~/.ssh`
  - Enter in a passphrase twice. Two files will be generated (  `~/.ssh/udacity_key` and `~/.ssh/udacity_key.pub`)
  - Run `cat ~/.ssh/udacity_key.pub` and copy the contents of the file
  - Log in to the grader's virtual machine
- On the grader's virtual machine:
  - Create a new directory called `~/.ssh` (`mkdir .ssh`)
  - Run `sudo nano ~/.ssh/authorized_keys` and paste the content into this file, save and exit
  - Give the permissions: `chmod 700 .ssh` and `chmod 644 .ssh/authorized_keys`
  - Check in `/etc/ssh/sshd_config` file if `PasswordAuthentication` is set to `no`
  - Restart SSH: `sudo service ssh restart`
- On the local machine, run: `ssh -i ~/.ssh/udacity_key grader@52.39.68.54 -p 2200`.

**References**
- DigitalOcean, [How To Set Up SSH Keys](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2).
- Ubuntu Wiki, [SSH/OpenSSH/Keys](https://help.ubuntu.com/community/SSH/OpenSSH/Keys).


### 9 - Configure Apache to serve a Python mod_wsgi application

- Clone the item-catalog (NHL Teams) app from Github 

`$ cd /var/www $ sudo mkdir catalog $ sudo chown -R grader:grader catalog $ cd catalog $ git clone https://github.com/golgtwins/Udacity-P5-Item-Catalog catalog`

- To make .git directory is not publicly accessible via a browser, create a .htaccess file in the .git folder and put the following in this file: 

`RedirectMatch 404 /\.git`

- Install pip , virtualenv (in /var/www/catalog) 

`$ sudo apt-get install python-pip $ sudo pip install virtualenv $ sudo virtualenv venv $ source venv/bin/activate $ sudo chmod -R 777 venv`

- Install Flask and other dependencies: 

`$ sudo pip install -r catalog/requirements.txt`

- Install Python's PostgreSQL adapter _psycopg2_: 

`$ sudo apt-get install python-psycopg2`
- Configure and Enable a New Virtual Host

Add the following content:

```
<VirtualHost *:80>
   ServerName 52.39.68.54
   ServerAlias ec2-52-39-68-54.us-west-2.compute.amazonaws.com
   ServerAdmin grader@52.39.68.54
   WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
   WSGIProcessGroup catalog
   WSGIScriptAlias / /var/www/catalog/catalog.wsgi
   <Directory /var/www/catalog/catalog/>
       Order allow,deny
       Allow from all
   </Directory>
   Alias /static /var/www/catalog/catalog/static
   <Directory /var/www/catalog/catalog/static/>
       Order allow,deny
       Allow from all
   </Directory>
   ErrorLog ${APACHE_LOG_DIR}/error.log
   LogLevel warn
   CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Enable the new virtual host:
`$ sudo a2ensite catalog`

Create and configure the .wsgi File
`$ cd /var/www/catalog/
$ sudo nano catalog.wsgi`

Add the following content:
```
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
application.secret_key = 'secret'
```

The original NHL Teams app that were developed on local machine needs some tweaks in order to be deployed on AWS. The major modifications include:
Rename `app.py` to `__init__.py`
Update the absolute path of `client_secrets.json` in `__init__.py`
Add app.secret_key for the Flask app in __init__.py
Add the code to create a dummy user in db_seed.py


### 10 - Install and configure PostgreSQL
- Install some necessary Python packages for working with PostgreSQL: `$ sudo apt-get install libpq-dev python-dev`.
- Install PostgreSQL: `$ sudo apt-get install postgresql postgresql-contrib`
- PostgreSQL automatically creates a new user 'postgres' during its installation. So we can connect to the database by using postgres username with: `$ sudo -u postgres psql`
- Create a new user called 'catalog' with his password: `# CREATE USER catalog WITH PASSWORD 'catalog';`
- Give catalog user the CREATEDB permission: `# ALTER USER catalog CREATEDB;`
- Create the 'catalog' database owned by catalog user: `# CREATE DATABASE catalog WITH OWNER catalog;`
- Connect to the database: `# \c catalog`
- Revoke all the rights: `# REVOKE ALL ON SCHEMA public FROM public;`
- Lock down the permissions to only let catalog role create tables: `# GRANT ALL ON SCHEMA public TO catalog;`
- Log out from PostgreSQL: `# \q`. Then return to the grader user: $ exit.
- Edit the `db_seed.py` and `database_setup.py` file:
```
Change engine = create_engine('sqlite:///category.db') to engine = create_engine('postgresql://catalog:catalog@localhost/catalog')
```
- Remote connections to PostgreSQL should already be blocked. Double check by opening the config file: `$ sudo nano /etc/postgresql/9.5/main/pg_hba.conf`


### 11 - Final updates on OAuth to make the app run on AWS
Go to the project on the Developer Console, and navigate to `APIs & Auth > Credentials > Edit Settings`
- Add the hostname and piblic IP address to the Authorized JavaScript origins and (host name + 'oauth2callback'), (host name + 'gconnect') to Authorized redirect URIs.
- Populate the PostgreSQL database with $ python db_seed.py
- Restart Apache to launch the app: `$ sudo service apache2 restart`
- If an internal error shows up when you try to access the app, open Apache error log as a reference for debugging: `$ sudo tail -20 /var/log/apache2/error.log`