### 安装：
```bash
#centos 7
yum install -y mariadb mariadb-server mariadb-devel
#debian 8
apt-get install nginx mariadb-server mariadb-client libmariadbclient-dev python-dev libssl-dev
```
### 启动与开机启动：
```bash
systemctl start mysqld
systemctl enable mariadb
```
### 配置
初始状态下，在终端中输入mysql就可以直接进入数据库，进而可以更改root密码。
```bash
[root@DptServer2 hubery]# mysql
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 2
Server version: 5.5.47-MariaDB MariaDB Server

Copyright (c) 2000, 2015, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> 
MariaDB [(none)]> 
MariaDB [(none)]> 
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| test               |
+--------------------+
4 rows in set (0.00 sec)

MariaDB [(none)]> use mysql;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [mysql]> exit
Bye

```
mysql提供了一套安全的初始化操作，使用/usr/bin/mysql_secure_installation命令。
```bash
[root@DptServer2 hubery]# mysql_secure_installation
/bin/mysql_secure_installation: line 379: find_mysql_client: command not found

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none): 
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

Set root password? [Y/n] Y
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

```

>参考：
>[Data Matters](http://data-matters.blogspot.jp/2013/08/install-mysql-python-with-mariadb.html)
>[MariaDB Setting up](https://downloads.mariadb.org/mariadb/repositories/#mirror=yamagata-university&distro=Debian&distro_release=jessie--jessie&version=10.1)
>[openssl-dev on Ubuntu](http://serverfault.com/questions/249340/install-openssl-dev-on-ubuntu-server)
>[Installing MariaDB 10.1 in Debian Jessie and Running Various MariaDB Queries](http://www.tecmint.com/install-mariadb-in-debian/)