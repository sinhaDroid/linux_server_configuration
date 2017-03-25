# Linux Server Configuration

This is the fifth project for "Full Stack Web Developer Nanodegree" on Udacity. 

In this project, a Linux virtual machine needs to be configurated to support the Item Catalog website.

You can visit [http://35.154.191.184][1] for the website deployed.

## Tasks
1. Launch your Virtual Machine with your Udacity account
2. Install & upgrade packages, git
3. Install & configure apache
4. Clone & configure item-catalog
5. Install python packages
6. More apache config
7. NTP config
8. Monitor & ban abuse
9. Add User
10. SSH
11. Firewall config

### Steps to setup ItemCatalog on a Ubuntu server
#### 1. Launch your Virtual Machine with your Udacity account:
1. Download Private Key below
2. Move the private key file into the folder `~/.ssh` (where ~ is your environment's home directory). So if you downloaded the file to the Downloads folder, just execute the following command in your terminal.
 -  `mv ~/Downloads/udacity_key.rsa ~/.ssh/`
 3. Open your terminal, type in
 -  `chmod 600 ~/.ssh/udacity_key.rsa`
 4. In your terminal, type in
 -  `ssh -i ~/.ssh/udacity_key.rsa ubuntu@35.154.191.184`

#### 2. Install & upgrade packages, git:
Ubuntu Docs: [1][3] & [2][4]
 - `sudo apt-get update`
 - `sudo apt-get upgrade`
 - `apt-get install unattended-upgrades`
 - `dpkg-reconfigure -plow unattended-upgrades`
 - `apt-get install git`
 - `git config --global user.name "YOUR_NAME"`
 - `git config --global user.email "YOUR_EMAIL_ADDRESS"`

#### 3. Install & configure apache
Udacity & Ubuntu: [1][5] & [2][6]
 1. Install Apache and Allow in Firewall
 - `apt-get install apache2`
 2. Set Global ServerName
 - `apt-get install python-setuptools libapache2-mod-wsgi`
 - `echo "ServerName HOSTNAME" | sudo tee /etc/apache2/conf-available/app.conf`
 - `a2enconf app`
 - `apt-get install libapache2-mod-wsgi python-dev`
 - `a2enmod wsgi`
 - `service apache2 restart`

#### 4. Clone & configure item-catalog
 - `cd /var/www/`
 - `touch .htaccess | echo "RedirectMatch 404 /\.git" >> .htaccess`
 - `git clone https://github.com/sinhaDroid/item-catalog.git`
 - `mv item-catalog ItemCatalog`
 - `cd ItemCatalog`
 
#### 5. Install python packages
 - `sudo apt-get install python-pip`
 - `sudo apt-get install python-psycopg2`
 - `sudo pip install virtualenv`
 - `sudo virtualenv venv` 
 - `sudo chmod -R 777 venv`
 - `sudo pip install -r requirements.txt`
 - `sudo pip install requests`
 - `sudo pip install --upgrade oauth2client`
 - `sudo pip install sqlalchemy`

#### 6. More apache config
Apache Docs & Digital Ocean: [1][7] & [2][8]
1. Create app.conf to edit: 
- `sudo nano /etc/apache2/sites-available/app.conf`
```
"<VirtualHost *:80>
	      ServerName 35.154.191.184
	      ServerAdmin ubuntu@35.154.191.184
	      WSGIScriptAlias / /var/www/app.wsgi
	      <Directory /var/www/ItemCatalog/>
	          Order allow,deny
	          Allow from all
	      </Directory>
	      Alias /static /var/www/ItemCatalog/static
	      <Directory /var/www/ItemCatalog/static/>
	          Order allow,deny
	          Allow from all
	      </Directory>
	      ErrorLog ${APACHE_LOG_DIR}/error.log
	      LogLevel warn
      CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>" >> 
```
2. Enable the virtual host with the following command:
- `sudo a2ensite app`
3. Create the .wsgi file under /var/www/:
- `cd /var/www/`
- `sudo nano app.wsgi`
4. Add the following lines of code to the app.wsgi file:
 ```
 #!/usr/bin/python
 import sys
 import logging
 logging.basicConfig(stream=sys.stderr)
 sys.path.insert(0,"/var/www/")
 
 from ItemCatalog import app as application
 application.secret_key = "Add your secret key"
```
3. Restart Apache:
- `service apache2 restart`

#### 7. NTP config
Ubuntu Docs: [1][11] & [2][12]
 - `dpkg-reconfigure tzdata`
 - `apt-get install ntp`
 - `vim /etc/ntp.conf` (edited to proper ntp server pool)

#### 8. Monitor & ban abuse
Glances & Fail2ban: [1][13] & [2][14]
 - `apt-get install python-pip build-essential python-dev`
 - `pip install Glances`
 - `apt-get install lm-sensors`
 - `pip install PySensors`
 - `apt-get install fail2ban`
 - `cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local`
 - `vim /etc/fail2ban/jail.local`
  - set bantime  = 900
  - destemail = grader@localhost
 - `apt-get install sendmail iptables-persistent`
 - `service fail2ban restart`

#### 9. Add User
 -  `adduser grader`
 -  `visudo` ( add "grader ALL=(ALL:ALL) ALL" under line "root ALL ..." )
 
#### 10. SSH
Arch Linux: [1][15]
 - `vim /etc/ssh/sshd_config` (Enable password login)
 - On local machine: `ssh-keygen`
 - `scp ~/.ssh/id_rsa.pub root@35.154.191.184:`
 - Back on root session: `su - `
 - `mkdir ~/.ssh`
 - `chmod 700 ~/.ssh`
 - `cat ~/id_rsa.pub >> ~/.ssh/authorized_keys`
 - `rm ~/id_rsa.pub`
 - `chmod 600 ~/.ssh/authorized_keys`
 - `exit`
 - `vim /etc/ssh/sshd_config` (Change ssh to 2200, Enforce ssh key login, Don't permit root login)
 - `service ssh restart`

#### 11. Firewall config
Ufw Docs & Digital Ocean: [1][16] & [2][17]
 - `ufw enable`
 - `ufw allow 2200/tcp`
 - `ufw allow 80/tcp`
 - `ufw allow 123/udp`
 - `service ufw restart`
 - `exit` (Log out of root)


[1]: http://35.154.191.184/
[3]: https://wiki.ubuntu.com/Security/Upgrades
[4]: https://help.ubuntu.com/lts/serverguide/automatic-updates.html
[5]: http://blog.udacity.com/2015/03/step-by-step-guide-install-lamp-linux-apache-mysql-python-ubuntu.html
[6]: https://help.ubuntu.com/lts/serverguide/httpd.html
[7]: http://httpd.apache.org/docs/2.2/en/mod/core.html#virtualhost
[8]: https://www.digitalocean.com/community/tutorials/how-to-configure-the-apache-web-server-on-an-ubuntu-or-debian-vps
[9]: https://help.ubuntu.com/community/PostgreSQL
[10]: https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-14-04
[11]: https://help.ubuntu.com/community/UbuntuTime
[12]: https://help.ubuntu.com/lts/serverguide/NTP.html
[13]: https://pypi.python.org/pypi/Glances
[14]: https://www.digitalocean.com/community/tutorials/how-to-install-and-use-fail2ban-on-ubuntu-14-04
[15]: https://wiki.archlinux.org/index.php/SSH_keys
[16]: https://help.ubuntu.com/community/UFW
[17]: https://www.digitalocean.com/community/tutorials/how-to-setup-a-firewall-with-ufw-on-an-ubuntu-and-debian-cloud-server
