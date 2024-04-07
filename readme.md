
url:: [GitHub - fryctze/lamp-config](https://github.com/fryctze/lamp-config)
up:: [[Linux]]
```sh
sudo pacman -S apache
sudo pacman -S mysql
sudo mysql_install_db --user=mysql --basedir=/usr --datadir=/var/lib/mysql
sudo systemctl start mariadb
sudo mysql_secure_installation
sudo pacman -S php php-fpm php-gd
sudo pacman -S phpmyadmin
```
1. Install apache
2. Install mysql
3. setup mysql
4. install php & php-fpm
5. [[#Enable php extension]] in /etc/php/php.ini 
6. install phpmyadmin
7. [[#Create php-fpm & phpmyadmin conf]] in /etc/httpd/conf/extra
8. [[#Enable module and include config file to httpd.conf]]
9. Extra: [[#Alias]] bash zshrc 
10.  [[#Troubleshoots Warning]]
#### Enable php extension
/etc/php/**php.ini**
```json
bz2
gd
iconv
mysqli
pdo_mysql

date.timezone = Asia/Jakarta
```

#### Create php-fpm and phpmyadmin conf
put these files into /etc/httpd/conf/extra/

php-fpm.conf 
```json
DirectoryIndex index.php index.html
<FilesMatch \.php$>
    SetHandler "proxy:unix:/run/php-fpm/php-fpm.sock|fcgi://localhost/"
</FilesMatch>
```

phpmyadmin.conf
```json
Alias /phpmyadmin "/usr/share/webapps/phpMyAdmin"
<Directory "/usr/share/webapps/phpMyAdmin">
    DirectoryIndex index.php
    AllowOverride All
    Options FollowSymlinks
    Require all granted
</Directory>
```

#### Enable module and include conf
/etc/httpd/conf/**httpd.conf**
```json
LoadModule unique_id_module modules/mod_unique_id.so
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_fcgi_module modules/mod_proxy_fcgi.so

// Add this too
Include conf/extra/php-fpm.conf
Include conf/extra/phpmyadmin.conf
```

#### Alias
.zshrc
```sh
function lamp {

# Check for required arguments (action and service name)
  if [[ -z "$1" ]]; then
    echo "Invalid Usage: service_control (start|stop|restart) <service_name>"
    return 1  # Exit with error code if arguments missing
  fi

  # Validate action argument
  if [[ ! ( "$1" == "start" || "$1" == "stop" || "$1" == "restart" ) ]]; then
    echo "Invalid action: $1. Valid options are (start|stop|restart)."
    return 1
  fi

  # Perform systemctl command based on arguments
  sudo systemctl "$1" php-fpm mysqld httpd && echo "Successfully $1 lamp-services" || echo "Failed to $1 lamp-services"
}
```

#### Troubleshoots Warning
##### Solving Warning - cfg is not accessible
`The $cfg['TempDir'] (/usr/share/webapps/phpMyAdmin/tmp/) is not accessible. phpMyAdmin is not able to cache templates and will be slow because of this.`

**Solved by**:
adding this to `/usr/share/webapps/phpMyAdmin/config.inc.php`
```json
$cfg['TempDir'] = '/tmp/phpmyadmin';
```


##### Solving Warning - configuration file needs a valid key
**Error Message**:
The configuration file needs a valid key for cookie encryption. A temporary key was automatically generated for you. Please refer to the [documentation](http://localhost/phpmyadmin/doc/html/config.html#cfg_blowfish_secret).

**Solved by**:
blowfish secret 
Go to [phpMyAdmin blowfish secret generator](https://phpsolved.com/phpmyadmin-blowfish-secret-generator/?g=[insert_php]echo%20)
And paste it to `/usr/share/webapps/phpMyAdmin/config.inc.php`



# Extras Logs
========================================= mysql configure user local
mysql -u root -p
## show user
SELECT user FROM mysql.user;
CREATE USER 'sammy'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
ALTER USER 'sammy'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
## add user level privileges
GRANT CREATE, ALTER, DROP, INSERT, UPDATE, DELETE, SELECT, REFERENCES, RELOAD on *.* TO 'sammy'@'localhost' WITH GRANT OPTION;
## add root user $WARNING$
GRANT ALL PRIVILEGES ON *.* TO 'sammy'@'localhost' WITH GRANT OPTION;

## Unsorted Old
==================================================================================================== apache
[fryctze@AI-chan ~]$ sudo pacman -S apache
[sudo] password for fryctze: 
resolving dependencies...
looking for conflicting packages...

Packages (3) apr-1.7.0-3  apr-util-1.6.1-8  apache-2.4.48-1

Total Download Size:   2,10 MiB
Total Installed Size:  8,21 MiB

:: Proceed with installation? [Y/n] 
:: Retrieving packages...
Optional dependencies for apr-util
    gdbm: enable gdbm support [installed]
    libldap: enable ldap support [installed]
    unixodbc: enable odbc support
    mariadb-libs: enable mysql/mariadb support
    postgresql-libs: enable postgres support
    db: enable berkley db support [installed]
    sqlite: enable sqlite support [installed]
    nss: enable nss crypto support [installed]
    openssl: enable openssl crypto support [installed]
(3/3) installing apache
Optional dependencies for apache
    lua: for mod_lua module [installed]
    libxml2: for mod_proxy_html, mod_xml2enc modules [installed]
    curl: for mod_md module [installed]
    jansson: for mod_md module [installed]
    brotli: for mod_brotli module [installed]
    uwsgi: for mod_proxy_uwsgi module
    lynx: apachectl status
    perl: for apxs and dbmmanage [installed]
:: Running post-transaction hooks...
(1/3) Reloading system manager configuration...
(2/3) Creating temporary files...
(3/3) Arming ConditionNeedsUpdate...
[fryctze@AI-chan ~]$ 




==================================================================================================== mysql   
[fryctze@AI-chan ~]$ sudo pacman -S mysql
[sudo] password for fryctze: 
:: There are 2 providers available for mysql:
:: Repository extra
   1) mariadb
:: Repository community
   2) percona-server

Enter a number (default=1): 
resolving dependencies...
looking for conflicting packages...

Packages (4) jemalloc-1:5.2.1-3  mariadb-clients-10.5.10-1  mariadb-libs-10.5.10-1  mariadb-10.5.10-1

Total Download Size:    36,26 MiB
Total Installed Size:  304,88 MiB

:: Proceed with installation? [Y/n] 
:: Retrieving packages...
 mariadb-libs-10.5.10-1-x86_64
 jemalloc-1:5.2.1-3-x86_64
 mariadb-clients-10.5.10-1-x86_64
 mariadb-10.5.10-1-x86_64
(4/4) checking keys in keyring
(4/4) checking package integrity
(4/4) loading package files
(4/4) checking for file conflicts
(4/4) checking available disk space
:: Processing package changes...
(1/4) installing mariadb-libs
Optional dependencies for mariadb-libs
    krb5: for gssapi authentication [installed]
(2/4) installing jemalloc
Optional dependencies for jemalloc
    perl: for jeprof [installed]
(3/4) installing mariadb-clients
(4/4) installing mariadb
:: You need to initialize the MariaDB data directory prior to starting
   the service. This can be done with mariadb-install-db command, e.g.:
   mariadb-install-db --user=mysql --basedir=/usr --datadir=/var/lib/mysql
Optional dependencies for mariadb
    cracklib: for cracklib plugin
    curl: for ha_s3 plugin [installed]
    galera: for MariaDB cluster with Galera WSREP
    python-mysqlclient: for myrocks_hotbackup
    perl-dbd-mariadb: for mariadb-hotcopy, mariadb-convert-table-format and mariadb-setpermission
:: Running post-transaction hooks...
(1/4) Creating system user accounts...
Creating group mysql with gid 966.
Creating user mysql (MariaDB) with uid 966 and gid 966.
(2/4) Reloading system manager configuration...
(3/4) Creating temporary files...
(4/4) Arming ConditionNeedsUpdate...
[fryctze@AI-chan ~]$ 




==================================================================================================== installation mariadb
[fryctze@AI-chan ~]$ sudo mysql_install_db --user=mysql --basedir=/usr --datadir=/var/lib/mysql
[sudo] password for fryctze: 
Installing MariaDB/MySQL system tables in '/var/lib/mysql' ...
OK

To start mysqld at boot time you have to copy
support-files/mysql.server to the right place for your system


Two all-privilege accounts were created.
One is root@localhost, it has no password, but you need to
be system 'root' user to connect. Use, for example, sudo mysql
The second is mysql@localhost, it has no password either, but
you need to be the system 'mysql' user to connect.
After connecting you can set the password, if you would need to be
able to connect as any of these users with a password and without sudo

See the MariaDB Knowledgebase at https://mariadb.com/kb or the
MySQL manual for more instructions.

You can start the MariaDB daemon with:
cd '/usr' ; /usr/bin/mysqld_safe --datadir='/var/lib/mysql'

You can test the MariaDB daemon with mysql-test-run.pl
cd '/usr/mysql-test' ; perl mysql-test-run.pl

Please report any problems at https://mariadb.org/jira

The latest information about MariaDB is available at https://mariadb.org/.
You can find additional information about the MySQL part at:
https://dev.mysql.com
Consider joining MariaDB's strong and vibrant community:
https://mariadb.org/get-involved/

[fryctze@AI-chan ~]$ 




==================================================================================================== init mysql
[fryctze@AI-chan ~]$ sudo systemctl start mariadb
[fryctze@AI-chan ~]$ sudo mysql_secure_installation

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user. If you've just installed MariaDB, and
haven't set the root password yet, you should just press enter here.

Enter current password for root (enter for none): 
OK, successfully used password, moving on...

Setting the root password or using the unix_socket ensures that nobody
can log into the MariaDB root user without the proper authorisation.

You already have your root account protected, so you can safely answer 'n'.

Switch to unix_socket authentication [Y/n] 
Enabled successfully!
Reloading privilege tables..
 ... Success!


You already have your root account protected, so you can safely answer 'n'.

Change the root password? [Y/n] 
New password: 
Re-enter new password: 
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] 
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] 
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] 
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] 
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
[fryctze@AI-chan ~]$ 





==================================================================================================== php
sudo pacman -S php php-fpm php-gd
Packages (3) php-8.0.7-1  php-fpm-8.0.7-1  php-gd-8.0.7-1


==================================================================================================== phpmyadmin
sudo pacman -S phpmyadmin
Packages (1) phpmyadmin-5.1.1-1


==================================================================================================== Configuration

========================================= php.ini
=== required
extension=gd            // image
extension=mysqli        // mariadb
extension=pdo_mysql     // mariadb & laravel
extension=iconv         // by phpmyadmin
=== other
extension=bz2
date.timezone = Asia/Jakarta

========================================= httpd.conf
LoadModule unique_id_module modules/mod_unique_id.so
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_fcgi_module modules/mod_proxy_fcgi.so
Include conf/extra/php-fpm.conf
Include conf/extra/phpmyadmin.conf

========================================= conf php-fpm.conf
DirectoryIndex index.php index.html
<FilesMatch \.php$>
    SetHandler "proxy:unix:/run/php-fpm/php-fpm.sock|fcgi://localhost/"
</FilesMatch>

========================================= conf phpmyadmin.conf
Alias /phpmyadmin "/usr/share/webapps/phpMyAdmin"
<Directory "/usr/share/webapps/phpMyAdmin">
    DirectoryIndex index.php
    AllowOverride All
    Options FollowSymlinks
    Require all granted
</Directory>


==================================================================
