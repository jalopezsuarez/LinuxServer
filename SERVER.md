## Linux Debian 9 (minimal server)
Debian 9 Server:
* Minimal Distribution
* SSH Server

### Server Package
```
tar zxvf server.tgz
mv server /server
```

### Dependencies

#### Build Deps
```
apt-get update
apt-get install -y build-essential
apt-get install -y autoconf make automake cmake libtool subversion git mercurial checkinstall bison flex unzip
```
#### MySQL Deps
```
apt-get install -y libncurses-dev libssl-dev
```
#### Apache Deps
```
apt-get install -y libapr1 libapr1-dev libaprutil1 libaprutil1-dev libpcre3-dev
```
#### PHP Deps
```
apt-get install -y libpng-dev libjpeg-dev libbz2-dev libcurl4-gnutls-dev libxml2 libxml2-dev libzip-dev libmcrypt-dev libfreetype6-dev libxpm-dev libwebp-dev
```

### Localization
```
locale-gen en_US.UTF-8
```

### VIM Editor (FIX)
```
echo "set nocompatible" > ~/.vimrc
echo "set backspace=indent,eol,start" >> ~/.vimrc
```

### CURL Libraries (FIX)
```
cd /usr/include
ln -s x86_64-linux-gnu/curl
```

### Environment
`vi /etc/environment`
```
LANGUAGE=en_US.UTF-8
LANG=en_US.UTF-8
LC_ALL=en_US.UTF-8
JAVA_HOME=/server/java/jdk8
MYSQL_HOME=/server/mysql
PHP_HOME=/server/php
```

`vi /etc/bash.bashrc`
```
export PATH=$PATH:$JAVA_HOME/bin:$PHP_HOME/bin
```

### Services
```
update-rc.d mysql defaults 70 30
update-rc.d apache defaults 80 20
```
```
ls -l /etc/rc*.d/ | grep mysql*
update-rc.d mysql remove
ls -l /etc/rc*.d/ | grep apache*
update-rc.d apache remove
```

### Security

#### Linux (Version / Release)
```
hostnamectl
```

#### SSH Certificate
```
ssh-keygen -t rsa -b 4096 -C "jalopezsuarez@gmail.com"
secure_rsa_jalopezsuarez_private.key
secure_rsa_jalopezsuarez_server.pub
```

#### SSH Username / Password (only certificate)
Generate random password (https://passwordsgenerator.net):
```
passwd
```
SSH Server disable password:
```
cat /etc/ssh/sshd_config | grep PasswordAuthentication
```
`vi /etc/ssh/sshd_config`
```
PasswordAuthentication no
```

#### SSH Certificates
Private Key (client side):
```
secure_rsa_jalopezsuarez_private.key
```
Public Key (server side):
```
cat secure_rsa_jalopezsuarez_server.pub >> ~/.ssh/authorized_keys
```

### MySQL

#### MySQL Server
```
groupadd mysql
useradd -r -g mysql -s /bin/false mysql
```

```
rm -rf /server/mysql/data
mkdir /server/mysql/data
chmod 750 /server/mysql/data
```

```
cd /server/mysql
bin/mysqld --initialize --user=mysql --datadir=/server/mysql/data
bin/mysql_ssl_rsa_setup --datadir=/server/mysql/data
bin/mysqld_safe --user=mysql &
```

```
bin/mysql -u root -p
ALTER USER 'root'@'localhost' IDENTIFIED BY 'root';
bin/mysqladmin -u root -p shutdown
```

#### MySQL Service
```
cp /server/mysql/support-files/mysql.server /etc/init.d/mysql
chmod +x /etc/init.d/mysql
```
```
systemctl start mysql.service
systemctl status mysql.service
```

#### MySQL Status
```
cd /server/mysql
bin/mysqladmin variables -p
bin/mysqladmin --help
bin/mysqld --help --verbose | grep my.cnf
```

#### MySQL References
```
Create a user to access externally, so you dont need to use root for security:
CREATE USER 'MYUSER'@'%.%.%.%' IDENTIFIED BY 'MYPASSWORD'; 
GRANT ALL PRIVILEGES ON *.* TO 'MYUSER'@'%.%.%.%' IDENTIFIED BY 'MYPASSWORD';
FLUSH PRIVILEGES;

Change user root administrator password:
UPDATE user SET Password=PASSWORD('MYPASSWORD') WHERE User='root';
FLUSH PRIVILEGES;

Add permissions to root administrator user to access from outside:
GRANT ALL PRIVILEGES ON *.* TO 'root'@'192.168.0.%' IDENTIFIED BY 'MYPASSWORD';
GRANT ALL PRIVILEGES ON *.* TO 'root'@'127.0.0.1' IDENTIFIED BY 'MYPASSWORD';
GRANT ALL PRIVILEGES ON *.* TO 'root'@'localhost' IDENTIFIED BY 'MYPASSWORD';

SELECT CURRENT_USER();

SET PASSWORD FOR 'root'@'localhost' = PASSWORD('my_password');
CREATE USER 'root'@'192.168.%.%' IDENTIFIED BY 'my_password'; 
GRANT ALL PRIVILEGES ON *.* TO 'root'@'192.168.%.%' IDENTIFIED BY 'my_password';

CREATE USER 'general'@'localhost' IDENTIFIED BY 'my_password'; 
CREATE USER 'general'@'192.168.%.%' IDENTIFIED BY 'my_password'; 
GRANT ALL PRIVILEGES ON *.* TO 'general'@'192.168.%.%' IDENTIFIED BY 'my_password';

GRANT SELECT ON *.* TO 'general'@'localhost' IDENTIFIED BY 'my_password';
GRANT SELECT ON *.* TO 'general'@'192.168.%.%' IDENTIFIED BY 'my_password';
```

### Apache

#### Apache Service
```
cp /server/apache/bin/apachectl /etc/init.d/apache
chmod +x /etc/init.d/apache
```

`vi /etc/init.d/apache`
```
# Comments to support LSB init script conventions
### BEGIN INIT INFO
# Provides: apache
# Required-Start: $local_fs $network $remote_fs $syslog
# Required-Stop: $local_fs $network $remote_fs $syslog
# Default-Start: 2 3 4 5
# Default-Stop: 0 1 6
# Short-Description: start and stop Apache
# Description: Apache is a open source HTTP web server.
### END INIT INFO
```

```
systemctl start apache.service
systemctl status apache.service
```

#### Apache SSL (https/443)
```
mkdir /server/apache/conf/ssl
apidox_net_private.key
apidox_net_certificate.cer
apidox_net_intermediate.cer
```

`vi /server/apache/conf/httpd.conf`
```
# ========================================================
# /MultiHost
# ========================================================
<VirtualHost *:80>
    DocumentRoot /server/apache/htdocs/apidox_net
    ServerName apidox.net
    ServerAlias www.apidox.net
    ServerAdmin webmaster@localhost
    DirectoryIndex index.php index.phtml index.phps index.html index.htm
</VirtualHost>
<VirtualHost *:443>
    DocumentRoot /server/apache/htdocs/apidox_net
    ServerName apidox.net
    ServerAlias www.apidox.net
    ServerAdmin webmaster@localhost
    DirectoryIndex index.php index.phtml index.phps index.html index.htm
    SSLEngine on
    SSLCertificateFile /server/apache/conf/ssl/apidox_net_certificate.cer
    SSLCertificateKeyFile /server/apache/conf/ssl/apidox_net_private.key
    SSLCertificateChainFile /server/apache/conf/ssl/apidox_net_intermediate.cer
</VirtualHost>
# ========================================================
# /MultiHost
# ========================================================
```

### Gearman

#### Gearman Service
`/etc/system/systemd/gearman.service`
```
[Unit]
Description=Gearman Service

[Service]
ExecStart=/server/java/jdk8/bin/java -jar /server/gearman/gearman-server-0.8.11-20150731.182506.jar
WorkingDirectory=/server/gearman/
Restart=always

[Install]
WantedBy=multi-user.target
```

```
systemctl enable gearman.service
systemctl daemon-reload
systemctl start gearman.service
```
