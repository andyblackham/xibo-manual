The commands below are a guide to installing and configuring Xibo on a Centos 6 platform, specifically a Centos 6 Minumal installation.  A typical use case is where your server only has a CD-ROM drive, rather than a DVD-ROM and cannot boot from USB or other media.  Centos 6 Mininal install is available for CD-ROM installtion.

These instructions assume that your hardware is functioning correctly, any RAID etc is externally managed (maybe in a RAID card) and that a vanilla Centos 6 Mininal Installation has completed suceesfully without errors.

Centos 6 is available for x86 and x64 archtitecture here: http://wiki.centos.org/Download

Some of the steps below require additional inputs from you the administrator, these are marked; please also refer to the Security Page for further security advice and considerations.

Built-in ethernet is not setup on a fresh install, lets change to DHCP;
Further info on the structure of this file is here: https://www.centos.org/docs/5/html/Deployment_Guide-en-US/s1-networkscripts-interfaces.html
```
vi /etc/sysconfig/network-scripts/ifcfg-eth0
```
Set the machines hostname
```
hostname XiboCMS
```

Open port 80 on the built-in firewall
```
iptables -A INPUT -p tcp -m tcp --sport 80 -j ACCEPT iptables -A OUTPUT -p tcp -m tcp --dport 80 -j ACCEPT 
```

Restart network services to acquire / use an IP address
```
service network restart
```

update any base components
```
yum update -y
```
install some basic tools for a happy system admin
```
yum install nano wget bash-completion ntp ntpdate -y
```
set the date and time, then start nptd to keep it in sync
```
ntpdate
service ntpd start
chkconfig ntpd on
```
We need additional repositories for Mcrypt, lets add them in
```
wget http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
wget http://rpms.famillecollet.com/enterprise/remi-release-6.rpm
sudo rpm -Uvh remi-release-6*.rpm epel-release-6*.rpm
```
install apache (httpd) PHP and mySql server
```
yum install httpd php-devel mysql mysql-server json json-c
yum install php php-cli php-common php-devel php-pdo php-soap php-mysql php-mcrypt* php-common php-cli php-devel php-fpm php-gd php-imap php-intl php-mysql php-process php-xml php-xmlrpc php-zts
```
Configure apache, start it and the set it to start on boot
```
nano /etc/httpd/conf/httpd.conf
service httpd start
chkconfig httpd on
```

Start mysql, configure and set to autostart on boot
```
service mysqld start
mysql_secure_installation
chkconfig mysqld on
```
edit this
```
nano /etc/php.d/json.ini
```
It needs to be like this:
```
# Json Extension
extension=json.so
```

edit config php to suit
```
nano /etc/php.ini
```

At this point
+ your server is on your network  
+ the hostname, date & time are set
+ you have a basic webserver installed with PHP and mySQL
+ you know a mySQL username and password
+ the Xibo required PHP modules are all installed
+ PHP has been set to allow larger than normal uploads (necessary for large video files)  

Now download and install xibo into the standard webroot
```
cd /var/www/html
wget https://github.com/xibosignage/xibo-cms/archive/1.7.1.tar.gz
sudo tar zxvf 1.7.1.tar.gz
sudo mv xibo-cms-1.7.1 xibo
chmod 777 -R xibo
cd ../..
mkdir xibo-library
chmod 777 xibo-library
```

now check your servers IP address with:
```
ifconfig | grep inet
```
it will show you, maybe the last line.

then visit `http://<youripaddress>/xibo`
and begin the setup

Once it's running, I generally change the webroot directory in httpd.conf to /var/www/html/xibo so the access address is simply `http://<youripaddress>`
