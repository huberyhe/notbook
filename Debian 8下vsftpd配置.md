##Debian 8下vsftpd安装与配置
###0.环境
```bash
root@remote:/# uname -r
3.16.0-4-amd64
root@remote:/e# lsb_release 
No LSB modules are available.
root@remote:/# lsb_release -a
No LSB modules are available.
Distributor ID:	Debian
Description:	Debian GNU/Linux 8.6 (jessie)
Release:	8.6
Codename:	jessie
```
###1.安装
`hubery@remote:~$ sudo apt-get install vsftpd ftp -y`
###2.配置
重启vsftpd后才能使得更改的配置生效`sudo systemctl restart vsftpd`
默认配置：
```bash
hubery@remote:~$ cat /etc/vsftpd.conf | grep -v "^#" | grep -v "^$"
listen=NO
listen_ipv6=YES
anonymous_enable=NO
local_enable=YES
dirmessage_enable=YES
use_localtime=YES
xferlog_enable=YES
connect_from_port_20=YES
secure_chroot_dir=/var/run/vsftpd/empty
pam_service_name=vsftpd
rsa_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem
rsa_private_key_file=/etc/ssl/private/ssl-cert-snakeoil.key
ssl_enable=NO
```
此时vsftpd已经在运行：
```bash
hubery@remote:~$ sudo systemctl status vsftpd.service
● vsftpd.service - vsftpd FTP server
   Loaded: loaded (/lib/systemd/system/vsftpd.service; enabled)
   Active: active (running) since 四 2016-12-29 02:55:14 EST; 5min ago
 Main PID: 17653 (vsftpd)
   CGroup: /system.slice/vsftpd.service
           └─17653 /usr/sbin/vsftpd /etc/vsftpd.conf

12月 29 02:55:14 remote systemd[1]: Started vsftpd FTP server.
hubery@remote:~$ sudo netstat -lpn | grep vsftpd
tcp6       0      0 :::21                   :::*                    LISTEN      17653/vsftpd
```
####(1)新建用户
vsftp使用系统用户来登陆，任何可以登录系统的用户都可以登陆vsftpd
```bash
hubery@remote:~$ ftp localhost
Connected to localhost.
220 (vsFTPd 3.0.2)
Name (localhost:hubery): 
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> 221 Goodbye.
```
但是为了安全，我们创建普通用户，这个用户可以登录vsfptd，但是不能SSH登陆系统
```bash
hubery@remote:~$ sudo bash -c "echo '/bin/false' >> /etc/shells"
hubery@remote:~$ sudo useradd -m jack -s /bin/false
hubery@remote:~$ sudo passwd jack
Enter new UNIX password: 
Retype new UNIX password: 
passwd: password updated successfully
hubery@remote:~$ ls ~jack
hubery@remote:~$ sudo dd if=/dev/zero of=~jack/file.64MB bs=1M count=64
64+0 records in
64+0 records out
67108864 bytes (67 MB) copied, 0.0278935 s, 2.4 GB/s
hubery@remote:~$ ls -lh ~jack
total 64M
-rw-r--r-- 1 root root 64M 12月 29 03:14 file.64MB
hubery@remote:~$ su jack
Password: 
hubery@remote:~$ echo $?
1
```
####(2)配置：500 OOPS: vsftpd: refusing to run with writable root inside chroot()

```bash
hubery@remote:~$ ftp -d -v localhost
Connected to localhost.
220 (vsFTPd 3.0.2)
ftp: setsockopt: Bad file descriptor
Name (localhost:hubery): jack
---> USER jack
331 Please specify the password.
Password:
---> PASS XXXX
500 OOPS: vsftpd: refusing to run with writable root inside chroot()
Login failed.
---> SYST
421 Service not available, remote server has closed connection
ftp> ---> QUIT
221 Goodbye.
```
配置文件/etc/vsftpd.conf添加一行`allow_writeable_chroot=YES`，重启vsftpd
```bash
hubery@remote:~$ cd tmp/
hubery@remote:~/tmp$ dd if=/dev/zero of=file.32MB bs=1M count=16
16+0 records in
16+0 records out
16777216 bytes (17 MB) copied, 0.0147825 s, 1.1 GB/s
hubery@remote:~/tmp$ ls
file.32MB
hubery@remote:~/tmp$ ftp -d -v localhost
Connected to localhost.
220 (vsFTPd 3.0.2)
ftp: setsockopt: Bad file descriptor
Name (localhost:hubery): jack
---> USER jack
331 Please specify the password.
Password:
---> PASS XXXX
230 Login successful.
---> SYST
215 UNIX Type: L8
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
ftp: setsockopt (ignored): Permission denied
---> EPRT |2|::1|59080|
200 EPRT command successful. Consider using EPSV.
---> LIST
150 Here comes the directory listing.
-rw-r--r--    1 0        0        67108864 Dec 29 03:14 file.64MB
226 Directory send OK.
ftp> !ls
/bin/bash
file.32M
ftp> ---> QUIT
221 Goodbye.
```
####(3)PORT（主动）模式、PASV（被动）模式
**FTP两种模式的区别：**
（1）PORT（主动）模式

所谓主动模式，指的是FTP服务器“主动”去连接客户端的数据端口来传输数据，其过程具体来说就是：客户端从一个任意的非特权端口N（N>1024）连接到FTP服务器的命令端口（即tcp 21端口），紧接着客户端开始监听端口N+1，并发送FTP命令“port N+1”到FTP服务器。然后服务器会从它自己的数据端口（20）“主动”连接到客户端指定的数据端口（N+1），这样客户端就可以和ftp服务器建立数据传输通道了。

（2）PASV（被动）模式

所谓被动模式，指的是FTP服务器“被动”等待客户端来连接自己的数据端口，其过程具体是：当开启一个FTP连接时，客户端打开两个任意的非特权本地端口（N >1024和N+1）。第一个端口连接服务器的21端口，但与主动方式的FTP不同，客户端不会提交PORT命令并允许服务器来回连它的数据端口，而是提交PASV命令。这样做的结果是服务器会开启一个任意的非特权端口（P > 1024），并发送PORT P命令给客户端。然后客户端发起从本地端口N+1到服务器的端口P的连接用来传送数据。（注意此模式下的FTP服务器不需要开启tcp 20端口了）

**两种模式的比较：**
（1）PORT（主动）模式模式只要开启服务器的21和20端口，而PASV（被动）模式需要开启服务器大于1024所有tcp端口和21端口。

（2）从网络安全的角度来看的话似乎ftp PORT模式更安全，而ftp PASV更不安全，那么为什么RFC要在ftp PORT基础再制定一个ftp PASV模式呢？其实RFC制定ftp PASV模式的主要目的是为了数据传输安全角度出发的，因为ftp port使用固定20端口进行传输数据，那么作为黑客很容使用sniffer等探嗅器抓取ftp数据，这样一来通过ftp PORT模式传输数据很容易被黑客窃取，因此使用PASV方式来架设ftp server是最安全绝佳方案。

因此：如果只是简单的为了文件共享，完全可以禁用PASV模式，解除开放大量端口的威胁，同时也为防火墙的设置带来便利。

不幸的是，FTP工具或者浏览器默认使用的都是PASV模式连接FTP服务器，因此，必须要使vsftpd在开启了防火墙的情况下，也能够支持PASV模式进行数据访问。

####(4)配置PASV主动模式
iptables配置: 主动模式使用20端口来传输数据，需要同时打开
```bash
hubery@remote:~$ sudo iptalbes -I INPUT 4 -m state --state NEW -p tcp --dport 21 -j ACCEPT
hubery@remote:~$ sudo iptalbes -I INPUT 4 -m state --state NEW -p tcp --dport 20 -j ACCEPT
```
尽管如此，由于vsftpd需要主动连接ftp客户端监听的端口，导致部分操作失败。这样就需要关闭客户端所在机器的防火墙`www@localhost:~$ sudo iptables -F`
```bash
ftp> ls
ftp: setsockopt (ignored): Permission denied
---> PORT 192,168,0,156,86,3
200 PORT command successful. Consider using PASV.
---> LIST
425 Failed to establish connection.
```

####(5)配置PASV被动模式
因为主动模式下需要关闭客户机的防火墙，加上使用固定端口20传输数据易被抓取，被动模式才是被广泛采用的模式。默认状态下，vsftpd使用任意非特权端口传输数据，这里给它限定一个范围7000~800，在配置文件后添加下面两句。
```
pasv_enable=yes
pasv_min_port=7000
pasv_max_port=8000
```
并配置防火墙`sudo iptables -I INPUT 4 -p tcp --dport 7000:8000 -j ACCEPT'
加载内核模块(可能不是必选项)，
```bash
root@remote:/# modprobe ip_conntrack_ftp
root@remote:/# modprobe ip_nat_ftp
```
永久加载需要在`/etc/modules-load.d/modules.conf`文件尾添加
```
ip_conntrack_ftp
ip_nat_ftp
```
ftp被动模式连接`ftp -p remoteIP`
####(6)配置SSL安全连接
编辑`/etc/vsftpd.conf`去注释或添加
```bash
pam_service_name=vsftpd
rsa_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem
rsa_private_key_file=/etc/ssl/private/ssl-cert-snakeoil.key
ssl_enable=YES
force_local_data_ssl=NO
force_local_logins_ssl=NO
require_cert=NO
```

配置之后：
```bash
root@remote:/# cat /etc/vsftpd.conf | grep -v "^#" | grep -v "^$"
listen=NO
listen_ipv6=YES
anonymous_enable=NO
local_enable=YES
write_enable=YES
local_umask=022
dirmessage_enable=YES
use_localtime=YES
xferlog_enable=YES
connect_from_port_20=YES
ascii_upload_enable=YES
ascii_download_enable=YES
ftpd_banner=Welcome to blah FTP service.
chroot_local_user=YES
secure_chroot_dir=/var/run/vsftpd/empty
pam_service_name=vsftpd
rsa_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem
rsa_private_key_file=/etc/ssl/private/ssl-cert-snakeoil.key
ssl_enable=YES
force_local_data_ssl=NO
force_local_logins_ssl=NO
require_cert=NO
allow_writeable_chroot=YES
pasv_enable=yes
pasv_min_port=7000
pasv_max_port=8000
```
>参考：
>[vsftpd doc](https://security.appspot.com/vsftpd/vsftpd_conf.html)
>[vsftpd的主动模式与被动模式 ](http://www.cnblogs.com/kuliuheng/p/3209744.html)
>[vsftpd被动模式配置](http://linux008.blog.51cto.com/2837805/611474)
>[vsftpd：500 OOPS: vsftpd: refusing to run with writable root inside chroot ()错误的解决方法  ](http://blog.csdn.net/bluishglc/article/details/42399439)
>[Installing and configuring FTP server vsftpd.](https://wiki.debian.org/vsftpd)
>[vsftpd与/bin/false、/sbin/nologin ](http://blog.chinaunix.net/uid-7354272-id-2643621.html)