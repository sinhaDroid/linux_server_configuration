# Linux Server Configuration

Live at [54.70.214.60][1]

### Steps to setup item-catalog on a Ubuntu server
#### 1. Download RSA Key, restrict key access, and ssh into instance:
 -  `mv ~/Downloads/udacity_key.rsa ~/.ssh/`
 -  `chmod 600 ~/.ssh/udacity_key.rsa`
 -  `ssh -i ~/.ssh/udacity_key.rsa root@54.70.214.60`

#### 2. Install & upgrade packages, git:
Ubuntu Docs: [1][3] & [2][4]
 -  `apt-get update`
 - `sudo apt-get upgrade`
 - `apt-get install unattended-upgrades`
 - `dpkg-reconfigure -plow unattended-upgrades`
 - `apt-get install git`
 - `git config --global user.name "Deepanshu Sinha"`
 - `git config --global user.email "107sinha@gmail.com"`

#### 3. Install & configure apache
Udacity & Ubuntu: [1][5] & [2][6]
 - `apt-get install apache2`
 - `apt-get install python-setuptools libapache2-mod-wsgi`
 - `echo "ServerName HOSTNAME" | sudo tee /etc/apache2/conf-available/fqdn.conf`
 - `a2enconf fqdn`
 - `apt-get install libapache2-mod-wsgi python-dev`
 - `a2enmod wsgi`
 - `service apache2 restart`

#### 4. Clone & configure item-catalog
 - `cd /var/www/`
 - `touch .htaccess | echo "RedirectMatch 404 /\.git" >> .htaccess`
 - `mkdir app`
 - `cd app`
 - `git clone https://github.com/sinhaDroid/item-catalog.git`
 - `cd item-catalog`
 - `rm README.md`
 - `mv project.py __init__.py`
 - `mv * ..`
 - `cd ..`
 - `rm -rf item-catalog`

#### 5. Install python packages
 - `apt-get install python-pip`
 - `pip install virtualenv`
 - `virtualenv venv` 
 - `chmod -R 777 venv`
 - `pip install Flask-Seasurf`
 - `pip install requests`
 - `pip install --upgrade oauth2client`
 - `pip install sqlalchemy`
 - `apt-get install python-psycopg2`
 - `pip install bleach`

#### 6. More apache config
Apache Docs & Digital Ocean: [1][7] & [2][8]

```
touch /etc/apache2/sites-available/app.conf | echo 
  "<VirtualHost *:80>
	      ServerName 54.70.214.60
	      ServerAdmin admin@54.70.214.60
	      WSGIScriptAlias / /var/www/app/app.wsgi
	      <Directory /var/www/app/app/>
	          Order allow,deny
	          Allow from all
	      </Directory>
	      Alias /static /var/www/app/app/static
	      <Directory /var/www/app/app/static/>
	          Order allow,deny
	          Allow from all
	      </Directory>
	      ErrorLog ${APACHE_LOG_DIR}/error.log
	      LogLevel warn
      CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>" >> 
/etc/apache2/sites-available/app.conf
```
 - `a2ensite app`
 - `cd ..` 
 ```
touch app.wsgi | echo '#!/usr/bin/python
 import sys
 import logging
 logging.basicConfig(stream=sys.stderr)
 sys.path.insert(0,"/var/www/app/")
 
 from app import app as application
 application.secret_key = "Add your secret key"' >> app.wsgi
```
- `service apache2 restart`

#### 7. Install & config postgres & instantiate db
Ubuntu Docs & Digital Ocean: [1][9] & [2][10]
 - `apt-get install postgresql postgresql-contrib`
 - `cd /etc/postgresql/9.3/main/`
 - `su - postgres`
 - `echo "CREATE USER catalog WITH PASSWORD 'catalogpw';" | psql`
 - `echo "ALTER USER catalog CREATEDB;" | psql`
 - `echo "CREATE DATABASE appdb WITH OWNER catalog;" | psql`
 - `echo "REVOKE ALL ON SCHEMA public FROM public;" | psql`
 - `echo "GRANT ALL ON SCHEMA public TO catalog;" | psql`
 - `exit`
 - `cd /var/www/app/app/`
 **Note: Refactored __init__.py, database_setup.py & createDB.py for postgres compatibility.**
 - `python database_setup.py`
 - `python createDB.py`

#### 8. NTP config
Ubuntu Docs: [1][11] & [2][12]
 - `dpkg-reconfigure tzdata`
 - `apt-get install ntp`
 - `vim /etc/ntp.conf` (edited to proper ntp server pool)

#### 9. Monitor & ban abuse
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

#### 10. Add User
 -  `adduser grader`
 -  `visudo` ( add "grader ALL=(ALL:ALL) ALL" under line "root ALL ..." )
 
#### 11. SSH
Arch Linux: [1][15]
 - `vim /etc/ssh/sshd_config` (Enable password login)
 - On local machine: `ssh-keygen`
 - `scp ~/.ssh/id_rsa.pub grader@52.24.181.212:`
 - Back on root session: `su - grader`
 - `mkdir ~/.ssh`
 - `chmod 700 ~/.ssh`
 - `cat ~/id_rsa.pub >> ~/.ssh/authorized_keys`
 - `rm ~/id_rsa.pub`
 - `chmod 600 ~/.ssh/authorized_keys`
 - `exit`
 - `vim /etc/ssh/sshd_config` (Change ssh to 2200, Enforce ssh key login, Don't permit root login)
 - `service ssh restart`

#### 12. Firewall config
Ufw Docs & Digital Ocean: [1][16] & [2][17]
 - `ufw enable`
 - `ufw allow 2200/tcp`
 - `ufw allow 80/tcp`
 - `ufw allow 123/udp`
 - `service ufw restart`
 - `exit` (Log out of root)


[1]: http://54.70.214.60/
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
