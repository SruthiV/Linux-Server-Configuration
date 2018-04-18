# Linux Server Configuration

> Public IP: 18.221.145.45  
> Host name: ec2-18-221-145-45.us-east-2.compute.amazonaws.com

### How To:  
#### Amazon Lightsail
1. Create Lightsail account
2. Create instance
3. Connect using SSH
4. Download private key
5. In the Networking tab, add two new custom ports - 123 and 2200
#### Server configuration
6. Place private key in .ssh
7. `$ chmod 600 ~/.ssh/LightsailDefaultPrivateKey-us-east-2.pem`
8. `$ ssh -i ~/.ssh/LightsailDefaultPrivateKey-us-east-2.pem ubuntu@18.221.145.45`
#### Create new account grader
9. `$ sudo su -`
10. `$ sudo nano /etc/sudoers.d/grader`
    grader ALL=(ALL:ALL) ALL
11. `$ sudo nano /etc/hosts`
    Under 127.0.1.1:localhost add 127.0.1.1 ip-10-20-37-65
#### Install updates and finger package
12. `$ sudo apt-get update`
    `$ sudo apt-get upgrade`
    `$ sudo apt-get install finger`
#### Keygen
13. In a new terminal, `$ ssh-keygen -f ~/.ssh/udacity_key.rsa`
14. `$ cat ~/.ssh/udacity_key.rsa.pub`
15. In the original terminal, `$ cd /home/grader`
16. `$ mkdir .ssh`
17. `$ touch .ssh/authorized_keys`
18. `$ nano .ssh/authorized_keys`
19. Permissions:
    `$ sudo chmod 700 /home/grader/.ssh`
    `$ sudo chmod 644 /home/grader/.ssh/authorized_keys`
20. `$ sudo chown -R grader:grader /home/grader/.ssh`
21. `$ sudo service ssh restart`
22. To disconnect:
    `$ ~.`
23. `$ ssh -i ~/.ssh/udacity_key.rsa grader@18.221.145.45`
#### Enforce key based authentication
24. `$ sudo nano /etc/ssh/sshd_config`
25. Find the PasswordAuthentication line and change text after to no
26. `$ sudo service ssh restart`
#### Change port
27. `$ sudo nano /etc/ssh/sshd_config`
28. Find the Port line and change 22 to 2200
29. `$ sudo service ssh restart`
30. `$ ~.`
31. `$ ssh -i ~/.ssh/udacity_key.rsa -p 2200 grader@18.221.145.45`
#### Disable root login
32. `$ sudo nano /etc/ssh/sshd_config`
33. Find the PermitRootLogin line and edit to no
34. `$ sudo service ssh restart`
#### Configure UFW
35. `$ sudo ufw allow 2200/tcp`
    `$ sudo ufw allow 80/tcp`
    `$ sudo ufw allow 123/udp`
    `$ sudo ufw enable`
#### Install Apache and GIT
36. `$ sudo apt-get install apache2`
    `$ sudo apt-get install libapache2-mod-wsgi python-dev`
    `$ sudo apt-get install git`
#### Enable mod_wsgi
37. `$ sudo a2enmod wsgi`
    `$ sudo service apache2 start`
#### Setup Folders
38. `$ cd /var/www`
    `$ sudo mkdir catalog`
    `$ sudo chown -R grader:grader catalog`
    `$ cd catalog`
#### Clone Catalog Project
39. `$ git clone https://github.com/SruthiV/Item-Catalog.git catalog`
#### Create .wsgi file
40. `$sudo nano catalog.wsgi`
```
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0, "/var/www/catalog/")

    from catalog import app as application
    application.secret_key = 'super_secret_key'
```
41. Rename the application.py to __init__.py
#### Virtual Machine
42. `$ sudo pip install virtualenv`
    `$ sudo virtualenv venv`
    `$ source venv/bin/activate`
    `$ sudo chmod -R 777 venv`
#### Install flask and other packages
43. `$ sudo apt-get install python-pip`
    `$ sudo pip install Flask`
    
    

44. nano __init__.py
    change the client_secrets.json line to /var/www/catalog/catalog/client_secrets.json
45. change the host to your Amazon Lightsail public IP address and port to 80
#### Configure virtual host

46. `$ sudo nano /etc/apache2/sites-available/catalog.conf`
```
    <VirtualHost *:80>
    ServerName [18.221.145.45]
    ServerAlias [ec2-18-221-145-45.us-east-2.compute.amazonaws.com]
    ServerAdmin admin@18.221.145.45
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
 
#### Database
47. `$ sudo apt-get install libpq-dev python-dev`
    `$ sudo apt-get install postgresql postgresql-contrib`
    `$ sudo su - postgres`
    `$ psql`
48. `$ CREATE USER catalog WITH PASSWORD 'password';`
    `$ ALTER USER catalog CREATEDB;`
    `$ CREATE DATABASE catalog WITH OWNER catalog;`
    Connect to database $ \c catalog
    `$ REVOKE ALL ON SCHEMA public FROM public;`
    `$ GRANT ALL ON SCHEMA public TO catalog;`
    Quit the postgrel command line: $ \q and then $ exit
49. use nano again to edit your__init__.py, database_setup.py, and createitems.py files to change the database engine from sqlite://catalog.db to postgresql://username:password@localhost/catalog
50. `$ sudo service apache2 restart`

### Reference:
https://github.com/callforsky/udacity-linux-configuration
https://github.com/mulligan121/Udacity-Linux-Configuration
