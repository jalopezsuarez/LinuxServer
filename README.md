## Linux Debian 9 (minimal server)

### COMMONS

##### Build Packets
```
apt-get update
apt-get install -y build-essential
apt-get install -y autoconf make automake cmake libtool subversion git mercurial checkinstall bison flex unzip
```
##### MySQL Packets
```
apt-get install -y libncurses-dev libssl-dev
```
##### Apache Packets
```
apt-get install -y libapr1 libapr1-dev libaprutil1 libaprutil1-dev libpcre3-dev
```
##### PHP Packes
```
apt-get install -y libpng-dev libjpeg-dev libbz2-dev libcurl4-gnutls-dev libxml2 libxml2-dev libzip-dev libmcrypt-dev libfreetype6-dev libxpm-dev libwebp-dev
```

#### Server Folders

```
mkdir /server
mkdir /server/repos
mkdir /server/sources
```

#### Localization Configuration

```
locale-gen en_US.UTF-8
```

#### Vim Editor (Fix)

```
echo "set nocompatible" > ~/.vimrc
echo "set backspace=indent,eol,start" >> ~/.vimrc
```

#### CURL Libraries (Fix)

```
cd /usr/include
ln -s x86_64-linux-gnu/curl
```

#### Environment Parameters

```
set
```

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

#### Services Installation

```
update-rc.d mysql defaults 70 30
update-rc.d apache defaults 80 20
```

```
ls -l /etc/rc*.d/ | grep mysql*
update-rc.d mysql remove
```
```
ls -l /etc/rc*.d/ | grep apache*
update-rc.d apache remove
```

### SECURITY

#### Debian Version CommandLine

```
hostnamectl
```
#### Generate KeyPair SSH Access

```
ssh-keygen -t rsa -b 4096 -C "jalopezsuarez@gmail.com"
secure_rsa_jalopezsuarez_private.key
secure_rsa_jalopezsuarez_server.pub
```

#### Disable User/Password access method

```
cat /etc/ssh/sshd_config | grep PasswordAuthentication
vi /etc/ssh/sshd_config
PasswordAuthentication no
```

#### Generate/change random 32/64 password 
```
https://passwordsgenerator.net
```
```
passwd
```

### JAVA

```
cd /server/repos
tar zxvf jdk-8u172-linux-x64.tar.gz
mkdir /server/java
mv jdk1.8.0_172 /server/java/jdk8
```

### MYSQL

#### MySQL Community Server (Referencias)

```
https://downloads.mysql.com/archives/community/
https://dev.mysql.com/doc/mysql-sourcebuild-excerpt/5.7/en/installing-source-distribution.html
```

```
cd /server/repos
tar zxvf mysql-boost-5.7.21.tar.gz
mv mysql-5.7.21 /server/sources

mkdir /server/mysql
mkdir /server/mysql/data
chmod 750 /server/mysql/data

groupadd mysql
useradd -r -g mysql -s /bin/false mysql

cmake -DWITH_BOOST=./boost/ -DCMAKE_INSTALL_PREFIX=/server/mysql .
make install
```

#### MySQL Initializate BBDD (no-root)

`vi /server/mysql/my.cnf`
```
# The MySQL database server configuration file.
# For advice on how to change settings please see
# http://dev.mysql.com/doc/refman/5.7/en/server-configuration-defaults.html

[mysqld]

# Remove leading # and set to the amount of RAM for the most important data
# cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
# innodb_buffer_pool_size = 128M

# Remove leading # to turn on a very important data integrity option: logging
# changes to the binary log between backups.
# log_bin

# These are commonly set, remove the # and set as required.
# port = .....
# server_id = .....
# socket = .....

basedir = /server/mysql
datadir = /server/mysql/data

# Remove leading # to set options mainly useful for reporting servers.
# The server defaults are faster for transactions and fast SELECTs.
# Adjust sizes as needed, experiment to find the optimal values.
# join_buffer_size = 128M
# sort_buffer_size = 2M
# read_rnd_buffer_size = 2M

max_connections = 1024
sql_mode = NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
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
/etc/init.d/mysql start
systemctl start mysql.service
systemctl status mysql.service
```

#### MySQL Configuration
```
cd /server/mysql
bin/mysqladmin variables -p
```

```
bin/mysqladmin --help
bin/mysqld --help --verbose | grep my.cnf
```

#### MySQL Reference Configuration Statements
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

### APACHE
```
cd /server/repos
tar zxvf httpd-2.4.33.tar.gz 
mv httpd-2.4.33 /server/sources
cd /server/sources/httpd-2.4.33
```

```
./configure --prefix=/server/apache --enable-module=so
make install
```

`vi /server/apache/conf/httpd.conf`
```
Listen 0.0.0.0:80
Listen 0.0.0.0:443

LoadModule ssl_module modules/mod_ssl.so
LoadModule rewrite_module modules/mod_rewrite.so

ServerName 0.0.0.0:80

# ========================================================
# MultiHost
# ========================================================
<VirtualHost *:80>
    DocumentRoot /server/apache/htdocs
    ServerAdmin webmaster@localhost
    DirectoryIndex index.php index.phtml index.phps index.html index.htm
</VirtualHost>
<VirtualHost *:80>
    DocumentRoot /server/apache/htdocs/agronote_com
    ServerName agronote.com
    ServerAlias www.agronote.com
    ServerAlias agronote.es
    ServerAlias www.agronote.es
    ServerAdmin webmaster@localhost
    DirectoryIndex index.php index.phtml index.phps index.html index.htm
</VirtualHost>
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
<VirtualHost *:80>
    DocumentRoot /server/apache/htdocs/vemovi_com
    ServerName vemovi.com
    ServerAlias www.vemovi.com
    ServerAdmin webmaster@localhost
    DirectoryIndex index.php index.phtml index.phps index.html index.htm
</VirtualHost>
# ========================================================
# /MultiHost
# ========================================================
```

#### Apache Server HTML
`vi /server/apache/htdocs/index.html`
```
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8" />
<title>...</title>
</head>
<body>...</body>
</html>
```

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
/etc/init.d/apache start
systemctl start apache.service
systemctl status apache.service
```

#### Configurar SSL https (443)
```
mkdir /server/apache/conf/ssl
apidox_net_private.key
apidox_net_certificate.cer
apidox_net_intermediate.cer
```

### PHP
```
tar zxvf /server/repos/php-7.2.7.tar.gz
mv /server/repos/php-7.2.7 /server/sources
cd /server/sources/php-7.2.7
```

```
./configure \
--prefix=/server/php \
--with-config-file-path=/server/php/etc \
--with-libdir=lib/x86_64-linux-gnu \
--with-apxs2=/server/apache/bin/apxs \
\
--enable-pdo=shared \
--with-pdo_mysql=shared \
--without-pdo-sqlite \
--without-sqlite3 \
\
--with-gd=shared \
--with-freetype-dir \
--with-jpeg-dir \
--with-png-dir \
--with-xpm-dir \
--with-webp-dir
\
--with-bz2 \
--with-curl \
--enable-fileinfo \
--with-gettext \
--enable-mbstring \
--with-openssl \
--enable-soap \
--enable-sockets \
--with-xmlrpc \
\
--enable-bcmath \
--enable-zip \
--with-zlib \
--with-libzip \
--with-libxml-dir 
```

```
make install
cp php.ini-production /server/php/etc/php.ini
```

`vi /server/php/etc/php.ini`
```
memory_limit = 512M
post_max_size = 512M
upload_max_filesize = 512M
max_file_uploads = 512
date.timezone = "Europe/Madrid"
```

#### PHP Extensions Shared
```
cd /server/php/bin
```

```
ref: https://pecl.php.net/index.php
pecl channel-update pecl.php.net
```

```
pecl install mcrypt-1.0.1
extension=mcrypt.so
```

### APACHE / PHP
`vi /server/apache/conf/httpd.conf`
```
LoadModule php7_module modules/libphp7.so

# ========================================================
# PHP
# ========================================================
AddType application/x-httpd-php .php
AddType application/x-httpd-php .phtml
AddType application/x-httpd-php-source .phps
PHPIniDir "/server/php/etc/php.ini"
# ========================================================
# /PHP
# ========================================================
```

#### Apache Server PHP Testing
`vi /server/apache/htdocs/phpinfo.php`
```
<?php phpinfo(); ?>
```

