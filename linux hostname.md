在我们需要维护较多的服务器时，有意义的Hostname将时刻提醒我们这台服务器的功能。
***

##1.Debian
```bash
echo '127.0.1.1 git-server' >> /etc/hosts
echo 'git-server' > /etc/hostname
hostname git-server
```
##2.CentOS
```bash
echo '127.0.1.1 git-server' >> /etc/hosts
echo 'HOSTNAME=git-server' > /etc/sysconfig/network
hostname git-server
```
完成后重新login，可以看到，用户名后面的hostname已经改成git-server，以上操作重启机器也有效。