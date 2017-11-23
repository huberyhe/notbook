#Debian设置IP地址、网关、DNS
**说明：**
系统：debian 6.0.4
IP地址：192.168.21.166
子网掩码：255.255.255.0
网关：192.168.21.2
DNS：8.8.8.8
8.8.4.4
**操作：**
系统运维 温馨提醒：qihang01原创内容版权所有,转载请注明出处及原文链接
####1、设置IP地址、网关
```bash
nano /etc/network/interfaces /etc/network/interfacesbak #备份原有配置文件
nano /etc/network/interfaces #编辑网网卡配置文件
auto lo
auto eth0 #开机自动连接网络
iface lo inet loopback
allow-hotplug eth0
iface eth0 inet static #static表示使用固定ip，dhcp表述使用动态ip
address 192.168.21.166 #设置ip地址
netmask 255.255.255.0 #设置子网掩码
gateway192.168.21.2 #设置网关
ctrl+o #保存配置
ctrl+x #退出
```
####2、设置dns
```bash
cp /etc/resolv.conf /etc/resolv.confbak #备份原有dns配置文件
nano /etc/resolv.conf #编辑配置文件
nameserver 8.8.8.8 #设置首选dns
nameserver 8.8.4.4 #设置备用dns
ctrl+o #保存配置
ctrl+x #退出
```
系统运维 温馨提醒：qihang01原创内容版权所有,转载请注明出处及原文链接
####3、重启网络
```bash
service networking restart #重启网络
```
至此，IP地址、网关、DNS配置完成，现在系统已经可以上网了。