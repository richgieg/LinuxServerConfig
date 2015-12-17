#Linux Server Configuration

This is the fifth and final project in my pursuit of the Full Stack Web Developer Nanodegree from Udacity. Following is Udacity's description for this project:

You will take a baseline installation of a Linux distribution on a virtual machine and prepare it to host your web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers. A deep understanding of exactly what your web applications are doing, how they are hosted, and the interactions between multiple systems are what define you as a Full Stack Web Developer. In this project, you’ll be responsible for turning a brand-new, bare bones, Linux server into the secure and efficient web application host your applications need.
```

```

##WEB ACCESS INSTRUCTIONS
To access the hosted web application, visit one of the two web URLs listed below. The links are only temporary, however, since the virtual machine only needs to be provisioned long enough for this project to be graded by Udacity.

- http://52.34.202.209
- http://ec2-52-34-202-209.us-west-2.compute.amazonaws.com


If you have a Google account, you can log into the application using the Google Sign-In button in the upper-right of the page. However, you will not have write-access to the site until I approve your account in my custom user-management console. Once you've been approved, you can create new items in the various categories of the catalog.
```

```

##SSH ACCESS INSTRUCTIONS
To access the server via SSH, use the information below. This information is only temporary, however, since the virtual machine only needs to be provisioned long enough for this project to be graded by Udacity.
```
Server IP: 52.34.202.209
SSH Port: 2200

Example: ssh -i ~/.ssh/udacity_key.rsa -p 2200 grader@52.34.202.209
```

*A private key (`udacity_key.rsa`) is required to access SSH and this information is only being shared with
the Udacity project reviewer.*
```

```

##REQUIRED FUNCTIONALITY
- Configured and secured Amazon EC2 virtual server running Ubuntu
- Performed necessary config to host my [ItemCatalog application](http://github.com/richgieg/ItemCatalog) (Udacity Full Stack Project #3) using Apache and PostgreSQL
```

```

##ADDITIONAL FUNCTIONALITY

**Enabled UFW limit rule so repeated SSH connection attempts will be blocked**

Testing & Verification Instructions:

1. Initiate SSH connection
2. Logoff
3. Repeat steps 1 and 2 as rapidly as possible, until the server starts to refuse connections. After roughly 30 seconds, you should be able to reconnect.

**Enabled automatic weekly updates, with logging, using Cron and Apt-Get**

Testing & Verification Instructions:

1. `sudo run-parts -v /etc/cron.weekly`
2. `ls /var/log/autoupdate`
3. `cat /var/log/autoupdate/########-####.log` (#'s are placeholders)

**Enabled application monitoring via Monit**

Testing & Verification Instructions:

1. `sudo monit status` to verify services are running
2. `sudo service postgresql stop`
3. `sudo service apache2 stop`
4. `sudo service ssh stop`
5. Log out of SSH session
6. Try logging in, but observe that it fails because SSH service stopped
7. Wait two minutes
8. Log back in via SSH, since SSH service should be restarted by now
9. `sudo monit status` to verify all services are running again
```

```

##THIRD-PARTY RESOURCES THAT WERE HELPFUL

- http://flask.pocoo.org/docs/0.10/deploying/mod_wsgi/
- http://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps
- http://www.digitalocean.com/community/tutorials/additional-recommended-steps-for-new-ubuntu-14-04-servers
- http://stackoverflow.com/questions/12081789/pythons-working-directory-when-running-with-wsgi-and-apache
- http://help.ubuntu.com/community/AutoWeeklyUpdateHowTo
- http://www.cyberciti.biz/faq/linux-unix-formatting-dates-for-display/
- http://manpages.ubuntu.com/manpages/wily/en/man8/ufw.8.html
- https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-monit
- https://mmonit.com/wiki/Monit/ConfigurationExamples
```

```

##MY CONFIGURATION STEPS
```
# Connect to Amazon VM as root (x.x.x.x is the server IP).
ssh -i ~/.ssh/udacity_key.rsa root@x.x.x.x

#Create the grader user account.
adduser grader

# Add grader user account to the sudo group.
adduser grader sudo

# Move authorized_keys file from root to grader, effectively disabling
# public key authentication for root and enabling it for the grader account.
mv .ssh /home/grader
chown -R grader:grader /home/grader/.ssh

# Edit /etc/hosts in order to map the current hostname to 127.0.0.1 to
# prevent the "unable to resolve host" error message when using sudo.
nano /etc/hosts

# Edit /etc/ssh/sshd_config.
nano /etc/ssh/sshd_config

# Ensure sshd_config has the following settings:
Port 2200
PermitRootLogin no
PasswordAuthentication no

# Restart SSH service.
service ssh restart

# Disconnect the root session.
exit

# Reconnect to Amazon VM as grader (x.x.x.x is the server IP).
ssh -i ~/.ssh/udacity_key.rsa -p 2200 grader@x.x.x.x

# Configure firewall to allow only SSH (TCP), HTTP (TCP) and NTP (UDP).
# Typically I would only allow SSH connections from the IP of the machine
# I would be using to initiate SSH connections. However, in this case
# ,since the project reviewer will need SSH access, I will be allowing
# SSH from any IP.
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw limit 2200/tcp
sudo ufw allow 80/tcp
sudo ufw allow 123/udp
sudo ufw enable

# Ensure local timezone is set to UTC
sudo dpkg-reconfigure tzdata

# Update packages.
sudo apt-get update
sudo apt-get upgrade
sudo reboot

# Reconnect to Amazon VM as grader (x.x.x.x is the server IP).
ssh -i ~/.ssh/udacity_key.rsa -p 2200 grader@x.x.x.x

# Enable automatic weekly updates, with logging enabled, by placing a
# script in /etc/cron.weekly.
sudo nano /etc/cron.weekly/autoupdate

# The autoupdate script should contain the following:
--------------------------------------------------------------
#!/bin/bash

# Create log directory if it doesn't exist.
logdir="/var/log/autoupdate"
mkdir -p $logdir

# Create name for logfile using the current date and time.
now=$(date +"%Y%m%d-%H%M")
logfile="$logdir/$now.log"

# Update system, appending STDOUT and STDERR to log file.
apt-get update >>$logfile 2>&1
apt-get upgrade -y >>$logfile 2>&1
apt-get autoclean >>$logfile 2>&1
--------------------------------------------------------------

# Make the autoupdate script executable.
sudo chmod 755 /etc/cron.weekly/autoupdate

# Test the autoupdate script by executing run-parts, then examine the log.
sudo run-parts -v /etc/cron.weekly
ls /var/log/autoupdate
cat /var/log/autoupdate/########-####.log

# Install Apache web server, mod_wsgi. and PostgreSQL
sudo apt-get install apache2 libapache2-mod-wsgi postgresql

# Install Flask, SQLAlchemy and Pip.
sudo apt-get install python-flask python-sqlalchemy python-pip

# Install psycopg2 and its prerequisites.
sudo apt-get install python-dev libpq-dev
sudo pip install psycopg2

# Install oauth2client, requests and httplib2.
sudo pip install oauth2client requests httplib2

# Install Git and clone ItemCatalog application repository to /var/www/app.
sudo apt-get install git
sudo git clone https://github.com/richgieg/ItemCatalog.git /var/www/app

# Ensure remote connections to PostgreSQL are not allowed by
# verifying rules in the host-based authentication file.
sudo nano /etc/postgresql/9.3/main/pg_hba.conf

# Create the application's PostgreSQL database.
sudo -u postgres psql
create user catalog with password 'catalog';
create database catalog owner catalog;
revoke all on database catalog from public;
\q

# Ensure seed.py and application.py are using the PostgreSQL connect
# string rather than the SQLite connect string.
cd /var/www/app/vagrant
sudo nano seed.py
sudo nano application.py

# Seed the database.
sudo python seed.py

# Ensure that client_secrets.json contains the correct settings.
sudo nano client_secrets.json

# Create WSGI file that imports the 'app' object as 'application'.
sudo nano application.wsgi

# Give www-data write-access to /var/ww/app/vagrant/img directory.
sudo chown -R www-data:www-data img

# Remove the default Apache site from sites-enabled.
sudo rm /etc/apache2/sites-enabled/000-default.conf

# Create new Apache site, in sites-available, for hosting the application.
sudo nano /etc/apache2/sites-available/catalog.conf

# The catalog.conf file should contain the following lines (x-x-x-x is the IP):
--------------------------------------------------------------
# Set server name.
ServerName ec2-x-x-x-x.us-west-2.compute.amazonaws.com

# Ensure the app has access to necessary modules.
WSGIPythonPath /var/www/app/vagrant

<VirtualHost *:80>
    # Point to the application's main Python script.
    WSGIScriptAlias / /var/www/app/vagrant/application.wsgi

    # Host the images.
    Alias /img/ /var/www/app/vagrant/img/

    # Host Flask's static directory.
    Alias /static/ /var/www/app/vagrant/static/
</VirtualHost>
--------------------------------------------------------------

# Add the new site to sites-enabled.
sudo ln -s /etc/apache2/sites-available/catalog.conf /etc/apache2/sites-enabled

# Restart Apache service.
sudo apache2ctl restart

# Get the Google Sign-In button working by updating "Authorized JavaScript
# Origins" for the application in the Google Developers Console so it
# includes the following origins (x.x.x.x/x-x-x-x are the IP):
http://x.x.x.x
http://ec2-x-x-x-x.us-west-2.compute.amazonaws.com

# Test site by visiting the following URL in a browser (x-x-x-x is the IP):
http://ec2-x-x-x-x.us-west-2.compute.amazonaws.com

# Enable application monitoring via Monit.
sudo apt-get install monit

# Ensure the Monit web service is enabled in so that the status can
# be attained via the command line tool. The following lines should
# be in /etc/monit/monitrc:
--------------------------------------------------------------
set httpd port 2812 and
    use address localhost
    allow localhost
--------------------------------------------------------------

# Configure Monit to watch critical services (Apache, PostgreSQL, SSH)
# and to start them if they stop running. The following lines should
# be in etc/monit/monitrc:
--------------------------------------------------------------
# Check the Apache web server status and restart if needed.
check process apache with pidfile /run/apache2/apache2.pid
    start program = "/usr/bin/service apache2 start" with timeout 60 seconds
    stop program = "/usr/bin/service apache2 stop"

# Check the PostgreSQL database server status and restart if needed.
check process postgresql with pidfile /run/postgresql/9.3-main.pid
    start program = "/usr/bin/service postgresql start" with timeout 60 seconds
    stop program = "/usr/bin/service postgresql stop"

# Check the SSH daemon status and restart if needed.
check process ssh with pidfile /run/sshd.pid
    start program = "/usr/bin/service ssh start" with timeout 60 seconds
    stop program = "/usr/bin/service ssh stop"
    if failed port 2200 protocol ssh then restart
--------------------------------------------------------------

# Check Monit's control file syntax and reload Monit to pick up config changes.
sudo monit -t
sudo monit reload

# View Monit's status output to verify that it's watching the services.
sudo monit status
```
