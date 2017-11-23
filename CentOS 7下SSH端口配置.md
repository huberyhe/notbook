注意！这里的Centos版本是7

###step1 修改/etc/ssh/sshd_config
```bash
vi /etc/ssh/sshd_config
#Port 22         //这行去掉#号
Port 20000      //下面添加这一行
```

###step2 修改SELinux
使用以下命令查看当前SElinux 允许的ssh端口：
```bash
semanage port -l | grep ssh
```
>semanage安装: yum -y install policycoreutils-python

添加20000端口到 SELinux
```bash
semanage port -a -t ssh_port_t -p tcp 20000
```
然后确认一下是否添加进去
```bash
semanage port -l | grep ssh
```
如果成功会输出
```bash
ssh_port_t                    tcp    20000, 22
```
step3 重启ssh
```bash
systemctl restart sshd.service
```
step3 防火墙配置
CentOS 7只用firewall代替了原来的iptables，开启端口并重启firewall：
```bash
firewall-cmd --zone=public --add-port=20000/tcp --permanent
firewall-cmd --reload
```