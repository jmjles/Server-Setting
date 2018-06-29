Access my site at : [Here](http://52.13.110.91) 
 
Linux Server Setup 
================== 
1. Create an AWS Lightsail Instance [Here](https://lightsail.aws.amazon.com/ls/webapp/home/instances)  
	- Select: Create Instance
	- Select: Pick your instance image > OS Only > Ubuntu
	- Select: Create
	- Wait a minute and refresh the page so that you can see your instance.
	- Select: Your instance
	- Select: Connect using SSH

2. Update Instance and install all dependencies
	- Type: sudo apt-get update
	- Type: sudo apt-get upgrade
	- Select: First option in Grub list if asked
	- Type: sudo apt update (If packages need to be updated)
	- Type: sudo apt-get install apache2
	- Type: sudo apt-get install libapache2-mod-wsgi python-dev
	- Type: sudo apt-get install git
	- Type: sudo apt-get install libpq-dev python-dev
	- Type: sudo apt-get install postgresql postgresql-contrib
	- Type: sudo apt-get install python-pip
	- Type: sudo pip install virtualenv
	- Type: sudo pip install Flask
	- Type: sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils

3. Configure Ports
	- Back to [AWS Home](https://lightsail.aws.amazon.com/ls/webapp/home/instances)
	- Select: Your instance
	- Select: Networking > Firewall > + Add another
	- Input: Custom > TCP > 2200
	- Select: Save
	- Go back to your instance console
	- Type: sudo nano /etc/ssh/sshd_config
	- Change: "Port 22" to "Port 2200"
	- Type: (Ctrl + o)Enter(Ctrl + x) 
	- Type: sudo service sshd restart
	- Type: sudo ufw default deny incoming
	- Type: sudo ufw default allow outgoing
	- Type: sudo ufw allow 2200/tcp
	- Type: sudo ufw allow www
	- Type: sudo ufw allow ssh
	- Type: sudo ufw allow ntp
	- Type: sudo ufw enable
	- Type: sudo dpkg-reconfigure tzdata
	- Select: Your data accordingly

4. Logging in Remotely
	- Go to [AWS Keys](https://lightsail.aws.amazon.com/ls/webapp/account/keys)
	- Select: Download(Rename the key something simple and the extension as ".rsa") Save it to: C:/Users/[YOURUSERNAME]/.ssh
	- Open: Git
	- Type chmod 400 ~/.ssh/[KEYNAME.rsa]
	- Type: ssh -i ~/.ssh/[KEYNAME.rsa] -p 2200 ubuntu@[YOUR INSTANCE'S PUBLIC IP]

5. Creating Grader User
	- Type: sudo adduser grader
	- Type: your choice of password
	- Type: Hit enter for the rest of the questions and enter y for confirmation
	- Type: sudo -i
	- Type: touch /etc/sudoers.d/grader
	- Type: logout
	- Type: sudo nano /etc/sudoers.d/grader
	- Type: "grader ALL=(ALL:ALL) ALL"(without"") Then: (Ctrl + o)(ENTER)(Ctrl + x)
	- Type: sudo mkdir /home/grader/.ssh 
	- Type: sudo cp /root/.ssh/authorized_keys /home/grader/.ssh/authorized_keys
	- Type: sudo nano /etc/ssh/sshd_config
	- Change: PermitRootLogin prohibit-password TO PermitRootLogin no (Ctrl + o)(ENTER)(Ctrl + x)
	- Type: sudo service ssh restart
	- Type: sudo chown -R grader:grader /home/grader/.ssh
	- Open: New Git
	- Type: ssh -i ~/.ssh/[KEYNAME.rsa] -p 2200 ubuntu@[YOUR INSTANCE'S PUBLIC IP]
	- Type: sudo login grader

6. Running Services
	- Type: sudo a2enmod wsgi
	- Type: sudo service apache2 start
	- Type: virtualenv venv
	- Type: source venv/bin/activate
	- Type: sudo chmod -R 777 venv
	- Type: logout
	- Type: sudo login grader

7. Clone App
	- Type: cd /var/www
	- Type: sudo mkdir catalog
	- Type: sudo chown -R grader:grader catalog
	- Type: cd catalog
	- Type: git clone https://github.com/jmjles/Item-Catalog
	- Type: touch catalog.wsgi
	- Type: sudo nano catalog.wsgi
	- Type: import sys 
	  import logging 
	logging.basicConfig(stream=sys.stderr) 
sys.path.insert(0, "/var/www/catalog/") 
from catalog import app as application 
application.secret_key = 'super_secret_key'(Ctrl + o)(ENTER)(Ctrl + x) 
	- Type: cd Item-Catalog
	- Type: mv b6sg.py __init__.py
	- Type: nano database_setup.py
	- Change: 'sqlite:///sportmenu.db' TO 'postgresql://catalog:catalog@localhost/catalog'(Ctrl + o)(ENTER)(Ctrl + x)
	-Type: nano database_setup.py
	- Change: 'sqlite:///sportmenu.db' TO 'postgresql://catalog:catalog@localhost/catalog'(Ctrl + o)(ENTER)(Ctrl + x)
	-Type: nano lotsofmenuitems.py
	- Change: 'sqlite:///sportmenu.db' TO 'postgresql://catalog:catalog@localhost/catalog'(Ctrl + o)(ENTER)(Ctrl + x)
8. Config and Enable Virtual Host
	- Type: sudo nano /etc/apache2/sites-available/catalog.conf
	- Type:
	ServerName [YOUR INSTANCE PUBLIC IP ADDRESS]  
ServerAdmin [YOUR EMAIL ADDRESS]  
WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/ python2.7/site-packages  
WSGIProcessGroup catalog  
WSGIScriptAlias / /var/www/catalog/Item-Catalog/catalog.wsgi  
Order allow,deny Allow from all  
Alias /static /var/www/catalog/catalog/static  
Order allow,deny Allow from all  
ErrorLog ${APACHE_LOG_DIR}/error.log  
LogLevel warn  
CustomLog ${APACHE_LOG_DIR}/access.log combined (Ctrl + o)(ENTER)(Ctrl + x)
	- Type sudo a2ensite catalog

9. Configuring Postgresql and Apache 
	- Type sudo -u postgres -i 
	- Type psql 
	- Type CREATE USER catalog WITH PASSWORD 'catalog'; 
	- Type ALTER USER catalog CREATEDB; 
	- Type CREATE DATABASE catalog WITH OWNER catalog; 
	- Type \c catalog 
	- Type REVOKE ALL ON SCHEMA public FROM public; 
	- Type GRANT ALL ON SCHEMA public TO catalog; 
	- Type \q 
	- Type exit 
	- Type python /var/www/catalog/Item-Catalog/database_setup.py 
	- Type sudo service apache2 restart 