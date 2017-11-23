#搭建PHP服务器环境综合篇
###一般安装
###PHP添加模块
###HTTPS支持
###CentOS
###Debian 8
```bash
apt -y install mariadb-server mariadb-client
apt -y install apache2
apt -y install php5 libapache2-mod-php5 php5-mysql
/apt -y install php libapache2-mod-php php-mysql
a2enmod rewrite
sed -i "/s/AllowOverride None/AllowOverride ALL/g" /etc/apache2/apache2.conf
service apache2 restart
```
>参考：
>[在CentOS上搭建PHP服务器环境](http://www.cnblogs.com/liulun/p/3535346.html)
>[Debian 8 Jessie 安装 LAMP 服务器教程](http://www.linuxidc.com/Linux/2015-09/123472.htm)
>[Debian/Ubuntu下安装Apache的Mod_Rewrite模块的步骤分享](http://www.jb51.net/os/Ubuntu/73036.html)