# **Linux Server Configuration Project - Udacity Full Stack Web Developer Nanodegree**

## Objective

Configure a baseline Linux Ubuntu distro on a virtual machine and configure it to host 
the Item Catalog Project. 

## Server Information
    
__Public IP Address__ 18.237.248.104<br>
__Private IP__ 172.26.9.100<br>
__SSH Port__ 2200<br>
__Application URL__ http://18.237.248.104<br>
__User / Password__: grader / grader1<br>

## SERVER CONFIGURATION INSTRUCTIONS

1. ### Creating a AWS Lightsail instance and setting up SSH
    - Go to [AWSLightSail](http://lightsail.aws.amazon.com) and create an OS Only Ubuntu instance. 
       Select the lowest priced plan option (for this project), identify your instance and create.  
    -  Configure the AWS Instance firewall to allow the following ports:
        - __SSH__  TCP/2200
        - __NTP__  TCP/123
    - Download the Default Private Key and place in in a directory on your local    
    - machine.  To connect via putty, go to Connection>Auth and point to your Default 
      Private Key. Enter your IP address to log in using the ubuntu account.


2. ### Server Lockdown
    - __Run the following to grab all updates:__<br>
        `Sudo apt-get update && sudo apt-get upgrade`

    - __Re-configure sshd_config file__<br>
        In terminal, enter `sudo nano /etc/ssh/sshd_config`.<br> 
        change line 5 from __#22__ to __2200__.
        Save and close the file.<br>
        Restart the ssh service<br>
        `sudo service sshd restart`

    - __Configure ufw firewall__<br>
        Enable ufw firewall<br>
        `sudo ufw enable`<br>
        Allow port 2200 for ssh<br>
        `sudo ufw allow 2200`

    - __Set the timezone__<br>
        `set timedatectl set-timezone utc`

    - __Create grader account__<br>
        Login as root and create a grader account<br>
        `sudo su -`<br>
        `adduser grader`<br>

        Set the password and fill out info.<br> 

        Grant sudo access `sudo visudo`<br>
        Add the following line after root  ALL=(ALL:ALL) ALL:<br>
        `grader ALL=(ALL:ALL) ALL` <br>
        Save and close the file. <br>
        
        Logout of root account<br>
        `exit` <br>

        In the `ubuntu` account, log into grader account<br>
        `su - grader`.
        
        Test sudo rights<br>
        `sudo whoami`

        `root` should be the result.

3. ### CREATE SSH KEYPAIR FOR GRADER USING THE SSH-KEYGEN TOOL

     - __On your local Windows machine__<br>
          Run putty key generator to generate a public key<br>
          Create a passphrase and save it to a directory<br>
          Copy the public key to your clipboard

     - __On a putty terminal session__<br>
          In the grader account, create the following directories:<br>
          `sudo mkdir .ssh`<br>
          `cd .ssh`<br>
          `sudo nano authorized_keys`<br>

      - Paste the public key into authorized_keys file. Save and close.

      - Grant permissions<br>
          `sudo chmod 700 .ssh`<br>
          `sudo chmod 644 .ssh/authorized_keys`
  
      - Close putty sesssion and use `grader` account to test connectiity


4. ### PREPARE TO DEPLOY ITEM CATALOG PROJECT

      __** NOTE - Python 2 is used for this project **__

     While logged in as __`grader`__:
  
      - Install Apache2 & WSGI package<br>
        `sudo apt-get install apache2`<br>
        `sudo apt-get install libapache2-mod-wsgi`

      - Allow apache through ufw<br>
	      `sudo ufw allow apache`

      - Confirm install by checking version using terminal & opening a
        web browser<br>
        `apache2 -version`<br>
		    `http://<server IP address>`

        __Install PostgreSQL__

        - Install software
          `sudo apt-get install postgresql`

          Setting up ubuntu role:<br>
          >  Switch to root user:<br>
          >  `sudo -u postgres psql`<br>
          >  Set password<br>
          >  `alter user postgres with password 'postgres';`<br>
          >  Create role<br>
          >  `create role ubuntu superuser;`<br>
          >  Grant login rights to ubuntu account:<br>
          >  `alter role ubuntu with login`<br>
          >  Set password for ubuntu account:<br>
          >  `alter user ubuntu with pasword 'ubuntu';`<br>
          >  Create ubuntu database:<br>
          >  `create database ubuntu`<br>

          Setup catalog role:<br>
          >  Set role for catalog:<br>
          >  `create role catalog;`<br>
          >  Allow catalog role to create a db:<br>
          >  `alter role catalog CREATEDB;`<br>
          >  Allow catalog role to login to psql:<br>
          >  `alter role catalog with login;`<br>
          >  Set password for catalog account:<br>
          >  `alter user catalog with pasword 'catalog';`<br>
          >  Create catalog database<br>
          >  `create database catalog`<br>
          
          Exit psql:<br>
          `\q`

      - Create a new user catalog<br>
         `sudo adduser catalog`
         
        Grant sudo rights.

        Log into `catalog` account and enter `psql` to confirm connectivity.

      - Exit and log back to the `grader` account.

5. ### INSTALL GIT
    Comes pre-installed with AWS Instance

6. ### CLONE AND SETUP THE ITEM CATALOG PROJECT FROM GITHUB

      - Create ItemCatalog directory  
          `sudo mkdir /var/www/ItemCatalog/ItemCatalog`

      -  Clone GitHub Item-Catalog-7iProject to /home directory<br>
	      `git clone https://github.com/jperry801/Item-Catalog-Project.git`

      - Copy Item-Catalog-Project files from home directory to /var/www/ItemCatalog/    ItemCatalog<br>
         
      - Change ownership of the ItemCatalog directory to grader<br>
         `sudo chown -R grader:grader /var/www/ItemCatalog`

      - Install the follwoing dependencies:<br>
            `sudo apt-get install python-pip`<br>
            `sudo pip install flask`<br>
            `sudo pip install flask sqlalchemy`<br>
            `sudo pip install --upgrade google-api-python-client oauth2client`<br>
            `sudo pip install requests`

      - Rename application.py file to `__init__.py`
         `sudo mv /var/www/ItemCatalog/ItemCatalog/applicaton.py /var/www/ItemCatalog/ ItemCatalog/__init__.py`

      - Make the following changes to __init.py__:<br>
          __LINE 336__<br>
             - if __name__ == __main__:<br>
                  app.run()
                      
          __LINE 15__<br>
              - `engine = create_engine('sqlite:////var/www/ItemCatalog/ItemCatalog/     appwithusers.db?check_same_thread=False')`
          
          __LINE 24__<br>
              - `CLIENT_ID = json.loads(open('/var/www/ItemCatalog/ItemCatalog           client_secrets_json', 'r').read())['web']['client_id]`


      - Make the following change in __database_setup.py__:<br>
          __LINE 59__<br>
              - `engine = create_engine('sqlite:////var/www/ItemCatalog/ItemCatalog/     appwithusers.db')` 

      - Confirm app is set up correctly
        `/var/www/ItemCatalog/ItemCatalog`
        `python __init__.py`
    
      You should see the following:
      > Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)

      - Go back to the home directory

7. ## SETUP APACHE VIRTUALHOST

      - Create ItemCatalog.conf file:<br>
          `sudo nano /etc/apache2/sites-available/ItemCatalog.conf`

      - Copy and paste the following into this file:
        ><VirtualHost *:80><br>
                   >ServerName Your server IP address<br>
                   >ServerAdmin admin@website.com<br>
                   >WSGIScriptAlias / /var/www/ItemCatalog/itemcatalog.wsgi<br>
                   ><Directory /var/www/ItemCatalog/ItemCatalog><br>
                        >Order allow,deny<br>
                        >Allow from all
                    ></Directory><br>
                    >Alias /static /var/www/ItemCatalog/ItemCatalog/static<br>
                    ><Directory /var/www/ItemCatalog/ItemCatalog/static/><br>
                        >Order allow,deny<br>
                        >Allow from all
                    ></Directory><br>
                    >ErrorLog ${APACHE_LOG_DIR}/error.log<br>
                    >LogLevel warn<br>
                    >CustomLog ${APACHE_LOG_DIR}/access.log combined<br>
         ></VirtualHost>

      - Save and close the file.

      - Enable ItemCatalog.conf in Apache:<br>
          `sudo a2ensite ItemCatalog.conf`

      - Disable default apache web page:<br>
          `sudo a2dissite 000-default.conf`

8. ## CREATE THE .wsgi FILE
  
      - Create a .wsgi file to server the Flask App:<br>
         `sudo nano /var/www/ItemCatalog/itemcatalog.wsgi` 

      - Copy and paste the following into this file:
        >#!/usr/bin/python<br>
                >import sys<br>
                >import logging<br>
                >logging.basicConfig(stream=sys.stderr)<br>
                >sys.path.insert(0,"/var/www/ItemCatalog/")<br>
                >from ItemCatalog import app as application<br>
                >application.secret_key = 'Add your secret key'<br>

      - Save and close the file.

      - Change ownership of itemcatalog.wsgi from root to grader:<br>
         `sudo chown grader:grader itemcatalog.wsgi`

      - Restart apache service<br>
          `sudo service apache2 restart`

          

9.  ### SETUP GOOGLE OAUTH

      - On your local machine, log into [Google Developers Console](https://console.developers.google.com) and click on Credentials

      - Create a OAuth Client ID and copy the Client ID

      - In your terminal, open the login.html file<br>
        `sudo nano /var/www/ItemCatalog/ItemCatalog/templates/login.html`
    
      - Paste the Client ID on line 13
      
      - Save and close the file.
    
      - You will also need to download the JSON file from Google Developers Console     to your local machine.

      - When downloaded, go to this directory in your terminal:<br>
        `sudo nano /var/www/ItemCatalog/ItemCatalog/client_secrets.json`

      - On your local machine, open your JSON file (using a text editor), copy 
        and paste the contents into this directory.

      - Save and close the file.

      - Restart apache service<br>
          `sudo service apache2 restart`

      - Visit website by opening up a web browser<br>
          `http://18.237.248.104`  








