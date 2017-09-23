# Linux Server Configuration
Public IP: 35.176.110.99
SSH Port: 2200



A baseline installation of a Linux distribution on a virtual machine and preparation of it to host a web application, includes installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

  1. Create new Ubuntu instance, download the key and launch it [https://lightsail.aws.amazon.com/ls/webapp/home/resources]
  2. Move the key into .ssh directory on your local machine
  *mv ~/Downloads/thekey.pem /c/Users/[user]/.ssh/thekey.pem*
  3. Set file rights
  *chmod 600 /c/Users/[user]/.ssh/thekey.pem*
  4. In your terminal type
  *ssh -i /c/Users/[user]/.ssh/thekey.pem ubuntu@PUBLIC-IP*
 # Create new user grader
  1. *sudo adduser grader*
  2. *sudo nano /etc/sudoers.d/grader* 
  Add the following line of code to the file, after the root user in the *User privilege specification* section:
    *grader ALL=(ALL:ALL) ALL*   save and quit.
# Login using the key
  1. *su - grader* 
  2. *mkdir .ssh*
  Makes a new directory
  3. *touch .ssh/authorized_keys* 
  Creates a new *authorized_keys* file
  4. *ssh-keygen* Generates a rsa key. Save it to your *.ssh* directory (/c/Users/[user]/.ssh/id_rsa)
  5. *sudo nano .ssh/authorized_keys* 
  Copy the content from your *id_rsa* file and paste it to authorized_keys. Save and quit.
  6. *chmod 700 .ssh*
  7. *chmod 644 .ssh/authorized_keys*
  8. *sudo nano /etc/ssh/sshd_config* . Change *PASSWORDAUTHENTICATION* from *no* to *yes*
    *sudo service ssh restart*
  9. Login with the new user
    *ssh -i [path to id_rsa] grader@PUBLIC-IP*
  10. *sudo nano /etc/ssh/sshd_config* . Change *PASSWORDAUTHENTICATION* from *yes* to *no*
   *sudo service ssh restart*
# Update the packages
  1. *sudo apt-get update*
  2. *sudo apt-get upgrade*
  3. *sudo apt-get install unattended-upgrades*
  4. *sudo dpkg-reconfigure -plow unattended-upgrades*
# Change the port from 22 to 2200
  1. Change PermitRootLogin to no
  2. *sudo nano /etc/ssh/sshd_config*. Modify the port number, save and quit.
  3. *sudo service ssh restart*
# Configure the UFW
  1. *sudo ufw allow 2200/tcp*
  2. *sudo ufw allow 80/tcp*
  3. *sudo ufw allow 123/udp*
  4. *sudo ufw enable*
# Configure the local timezone to UTC
  1. *sudo dpkg-reconfigure tzdata*
# Install apache, postgresql and git
  1. *sudo apt-get install apache2*
  2. *sudo apt-get install libapache2-mod-wsgi python-dev*
  3. *sudo apt-get install git*
  4. Go to www directory *cd /var/www*
  5. Make a new directory *sudo mkdir catalog*
  6. *cd catalog*
  7. *sudo mkdir catalog* and *cd catalog*
  8. *sudo mkdir static templates*
  9. *sudo nano __init__.py*
  10. Paste 
*from flask import Flask
app = Flask(__name__)
@app.route("/")
def hello():
    return "Hello, User!"
if __name__ == "__main__":
    app.run()*
     Save and quit.
  11. *sudo apt-get install python-pip*
  12. *sudo -H pip install virtualenv*
  13. *sudo virtualenv venv*
  14. *sudo chmod -R 777 venv*
  15. *source venv/bin/activate*
  16. *pip install Flask*
  17. *python __init__.py*
  18. *deactivate*
  19. *sudo nano /etc/apache2/sites-available/catalog.conf*
  20. <VirtualHost *:80>
      ServerName PUBLIC-IP-ADDRESS
      ServerAdmin admin@PUBLIC-IP-ADDRESS
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
  21. *sudo a2ensite catalog*
  22. *cd /var/www/catalog*
  23. *sudo nano catalog.wsgi*
  24.   #!/usr/bin/python
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0,"/var/www/catalog/")
  
  from catalog import app as application
  application.secret_key = 'Add your secret key'
 25. *sudo service apache2 restart*
 26. Go to your browser and type *http://35.176.110.99/*. Hello, User!!!
# Setup the Item Catalog project
  1. *sudo chown -R grader /var/www*
  2. *git clone https://github.com/mariastratulat/Item-Catalog.git*. Move the content to /var/www/catalog/catalog and delete the Item-Catalog directory.
  3. *cd /var/www/catalog*, *sudo nano .htaccess* and paste *RedirectMatch 404 /\.git* .Save and quit.
  4. *cd /var/www/catalog/catalog*
  5. *source venv/bin/activate*
  6. Install httplib2, requests, flask, aouth2client, sqlalchemy, postgresql
  7. *sudo apt-get install postgresql*
  8. *sudo nano database_setup.py*. Change the engine to *engine = create_engine('postgresql://catalog:catalog@localhost/catalog')*
    Do the same in project.py. Change the path of client_secrets.json to the full path of the file. Change the engine in modelsinfo.py too.
  9. Rename project.py *mv project.py __init__.py*
  10. *sudo adduser catalog*
  11. *sudo su - postgres*
  12. *psql*
  13. *CREATE USER catalog WITH PASSWORD 'catalog';*
  14. *ALTER USER catalog CREATEDB;*
  15. *CREATE DATABASE catalog WITH OWNER catalog;*
  16. *\c catalog*
  17. *REVOKE ALL ON SCHEMA public FROM public;*
  18. *GRANT ALL ON SCHEMA public TO catalog;*
  19. *\q* and *exit*
  20. Run *python database_setup.py* from catalog directory
  21. Restart apache2 *sudo service apache2 restart*
  22. Open in your browser [http://35.176.110.99/]
  23. *sudo nano /etc/apache2/sites-available/catalo.conf* and add *ServerAlias HOSTNAME ec2-35-176-110-99.eu-west-2.compute.amazonaws.com* below ServerAdmin.
  24. Go to [https://console.developers.google.com/cloud-resource-manager] and add *htts://35.176.110.99* to Authorized JavaScript origins and *http://ec2-35-176-110-99.eu-west-2.compute.amazonaws.com* to Authorized redirect URIs (APIs& auth>Credentials>Edit)
  25. Go to [https://developers.facebook.com/apps]. In Facebook Login > Settings > Valid OAuth redirect URIs add *http://35.176.110.99/* 
  26. To see the errors details *sudo tail /var/log/apache2/error.log*
  27. Run *python modelsinfo.py* to populate the database. It works! Ok, almost. Open __init__.py and modify in fbconnect and gconnect functions with the full path oc secrets files(fb_client_secrets.json and client_secrets.json)
  28. *sudo service apache2 restart*
# References
  1. Digital Ocean [https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps]
  2. Udacity Forums [https://discussions.udacity.com/]
 


 


