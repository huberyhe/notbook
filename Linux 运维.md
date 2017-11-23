## centos6.8升级python版本以及其引起的uwsgi部署问题

问题：

```bash

# cat /var/log/carder/uwsgi_youtube.log

*** Starting uWSGI 2.0.13.1 (64bit) on [Wed Sep 21 16:15:23 2016] ***

compiled with version: 4.4.7 20120313 (Red Hat 4.4.7-17) on 02 August 2016 21:07:31

os: Linux-2.6.32-642.4.2.el6.x86_64 #1 SMP Tue Aug 23 19:58:13 UTC 2016

nodename: vultr.guest

machine: x86_64

clock source: unix

pcre jit disabled

detected number of CPU cores: 1

current working directory: /root/uwsgitest

writing pidfile to /var/run/carder/uwsgi_youtube.pid

detected binary path: /usr/sbin/uwsgi

uWSGI running as root, you can use --uid/--gid/--chroot options

*** WARNING: you are running uWSGI as root !!! (use the --uid flag) ***

chdir() to /opt/youtube/youtube

your processes number limit is 3899

your memory page size is 4096 bytes

detected max file descriptor number: 1024

lock engine: pthread robust mutexes

thunder lock: disabled (you can enable it with --thunder-lock)

uwsgi socket 0 bound to TCP address 127.0.0.1:7000 fd 3

your server socket listen backlog is limited to 100 connections

your mercy for graceful operations on workers is 60 seconds

mapped 415360 bytes (405 KB) for 8 cores

*** Operational MODE: preforking+threaded ***

*** no app loaded. going in full dynamic mode ***

*** uWSGI is running in multiple interpreter mode ***

spawned uWSGI master process (pid: 8744)

spawned uWSGI worker 1 (pid: 8745, cores: 2)

spawned uWSGI worker 2 (pid: 8746, cores: 2)

spawned uWSGI worker 3 (pid: 8747, cores: 2)

spawned uWSGI worker 4 (pid: 8748, cores: 2)

Wed Sep 21 16:15:58 2016 - uWSGI worker 1 screams: UAAAAAAH my master disconnected: i will kill myself !!!

Wed Sep 21 16:15:58 2016 - uWSGI worker 2 screams: UAAAAAAH my master disconnected: i will kill myself !!!

Wed Sep 21 16:15:58 2016 - uWSGI worker 3 screams: UAAAAAAH my master disconnected: i will kill myself !!!

Wed Sep 21 16:15:58 2016 - uWSGI worker 4 screams: UAAAAAAH my master disconnected: i will kill myself !!!



```

1.编译安装python2.7

2.安装easy_insall: `python2.7 ez_setup.py`

3.移除旧版本pip和uwsgi： `yum remove uwsgi python-pip`

4.安装新版本pip和uwsgi： `easy_install pip && pip2.7 install uwsgi`



>参考：

>[uwsgi docs](http://uwsgi-docs.readthedocs.io/en/latest/Options.html)

>[bottle docs](http://bottlepy.org/docs/dev/deployment.html)

>[Bottle + UWSGI + Nginx Quickstart](https://michael.lustfield.net/nginx/bottle-uwsgi-nginx-quickstart)

>[经过各种坑之后centos+ uwsgi + nginx +django 终于配好了](http://www.cnblogs.com/liujianzuo888/p/5016228.html)

>[Running your Python Bottle application on Nginx using uWSGI](https://community.runabove.com/kb/en/development/how-to-run-bottle-uwsgi-nginx.html)

>[pip-installed uWSGI ./python_plugin.so error](http://blog.csdn.net/zouyee/article/details/38419531)



## Linux版本查看

1.内核版本： `uname -a`

2.发行版本（*）： `lsb_release -a`

3.发行版本（CentOS）： `cat /etc/redhat-release`

>注：CentOS安装lsb_realease, `yum install redhat-lsb -y`

4.发行版本（Debian）: `cat /etc/debian_version`

## 识别vps虚拟技术
1、通过系统上的相关目录或文件判断
执行：ls /proc/ ，一般Xen的VPS，/proc目录下面会有xen的目录，openvz的会有vz目录。
2、执行：free -m 看内存，openvz的没有swap，当然也有xen的没有swap，但是xen的是可以加的，openvz不行。
3、执行：uname -a  有些xen的VPS里面会显示有xen。
4、执行：ifconfig 查看网卡，openvz的一般都是venet0:* ，xen的一般都是eth*。
5、通过VPS控制面板查看，像SolusVM、vePortal控制面板上都显示虚拟技术。
6、使用专门的软件：virt-what ，virt-what是一个判断当前环境所使用的虚拟技术的脚本，常见的虚拟技术基本上都能正常识别出来。
>[OpenVZ不能更改内核参数](https://superuser.com/questions/659236/permission-denied-when-setting-values-in-sysctl-on-ubuntu-12-04)

##Debian 和CentOS 安装开发组件

1.CentOS: `yum groupinstall "Development Tools"`

2.Debian: `apt-get install build-essential`



## 运维



[使用psacct监控linux用户行为 ](http://blog.163.com/sjt_linux/blog/static/1993103192012111510543761/)

## 检测服务器实际带宽
```bash
pip install speedtest-cli
speedtest-cli
```
[speedtest -- github](https://github.com/sivel/speedtest-cli)
[教您如何在Linux环境下使用命令行测试VPS上传/下载网速的几种方法](http://www.weeiy.com/linux-vps-net-u.html)

##iptables配置

>参考

>[iptables设置安全策略](http://www.centoscn.com/CentosSecurity/CentosSafe/2013/1015/1843.html)

>[使用iptables配置防火墙后本机无法访问外部网络](http://www.netingcn.com/iptables-localhost-not-access-internet.html)

>[Linux服务器安全配置之防火墙篇](http://www.linuxidc.com/Linux/2008-10/16353.htm)


## Debian 8升级内核开启BBR
BBR是`linux-kernel-4.9`起自带的 TCP 拥塞控制算法，可用于优化 TCP 连接
```bash
#下载安装4.9内核
wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.9/linux-image-4.9.0-040900-generic_4.9.0-040900.201612111631_amd64.deb
dpkg -i linux-image-4.9.0*.deb
#删除旧的内核(可选)
dpkg -l|grep linux-image
apt-get remove linux-image-amd64 linux-image-3.16.0-4-amd64
#更新grub，重新启动
update-grub
reboot
apt-get -f install
#设定bbr
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
sysctl -p
sysctl net.ipv4.tcp_available_congestion_control
lsmod | grep bbr
#
```

## 查看gzip/gz格式日志
`zcat access_log.gz`或者`	zmore access_log.gz`

## 查看nginx在某时间段内的日志
`sed -n '/16\/Feb\/2016/,/17\/Feb\/2016/p' access_log>20160216.log`

>参考：
>[CAC的Debian-8-64bit安装BBR正确打开方式](http://www.cnblogs.com/Q2881064156/p/6183611.html)
>[Vultr CentOS7安装Google BBR加速工具方法](https://www.vultrclub.com/174.html)
>[Ubuntu,debian安装谷歌BBR,顺带centos](https://www.whhack.com/bbr.html)
>[Debian 8 X64 升级内核并开启BBR TCP加速](http://www.th7.cn/system/lin/201701/200745.shtml)