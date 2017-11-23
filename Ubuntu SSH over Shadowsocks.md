由于国内的网络环境，时常出现服务器连接不上或连接缓慢的情况，或者为了安全性，服务器防火墙需要配置成只有某个或某几个IP地址才能登陆，于是我们可以选择一台网路状况较好的服务器，把这台服务器当作跳板机登陆其他服务器。
##跳板机安装Shadowsocks服务端
简要配置过程如下，参考[[github]shadowsocks/shadowsocks-go](https://github.com/shadowsocks/shadowsocks-go)
```bash
root@m1:~# apt-get install golang
root@m1:~# export GOPATH=/root/GOPATH
root@m1:~# go get github.com/shadowsocks/shadowsocks-go/cmd/shadowsocks-server
root@m1:~# cat /etc/shadowsocks.json
{
	"server":"192.168.0.133",
	"server_port":8388,
	"local_address":"127.0.0.1",
	"local_port":1080,
	"password":"password",
	"timeout":600,
	"method":"rc4-md5"
}

root@m1:~# cat /etc/supervisor/conf.d/ssserver.conf
[program:ssserver]
command = /root/GOPATH/bin/shadowsocks-server -c /etc/shadowsocks.json
autostart = true
autorestart = true
startsecs = 3
root@m1:~# supervisorctl reload
Restarted supervisord
root@m1:~# supervisorctl restart all
ssserver: stopped
ssserver: started
root@m1:~# iptables -I INPUT 5 -m state --state NEW -p tcp --dport 8388 -j ACCEPT

```
##本地开发机安装Shadowsocks客户端
SS客户端配置大同小异
```bash
root@localhost:~# apt-get install golang
root@localhost:~# export GOPATH=/root/GOPATH
root@localhost:~# go get github.com/shadowsocks/shadowsocks-go/cmd/shadowsocks-local
root@localhost:~# cat /etc/shadowsocks.json
{
	"server":"192.168.0.133",
	"server_port":8388,
	"local_address":"127.0.0.1",
	"local_port":1080,
	"password":"password",
	"timeout":600,
	"method":"rc4-md5"
}

root@localhost:~# cat /etc/supervisor/conf.d/ssserver.conf
[program:ssserver]
command = /root/GOPATH/bin/shadowsocks-local -c /etc/shadowsocks.json
autostart = true
autorestart = true
startsecs = 3
root@localhost:~# supervisorctl reload
Restarted supervisord
root@localhost:~# supervisorctl restart all
ssserver: stopped
ssserver: started
```
##SSH配置通过本地SS代理来登陆服务器
安装代理工具
```bash
sudo apt install proxychains connect-proxy -y
```
先测试一下SS服务是否可用
```bash
root@localhost:~# ssh www@192.168.0.197 -i .ssh/id_rsa -o "ProxyCommand connect -S 127.0.0.1:1080 %h %p"
Last login: Wed Dec  7 09:09:32 2016 from 128.199.58.37

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
www@roaster:~$ sudo tail /var/log/auth.log
```
>`sudo tail /var/log/auth.log`可以看到登陆来源是跳板机的IP而非开发机的地址

配置SSH
```bash
root@localhost:~# cat .ssh/config
#1.某个服务器（可选）
Host server01
     HostName 192.168.0.197
     Port 22
     User www
     PubkeyAuthentication yes
     IdentityFile ~/.ssh/id_rsa
     ProxyCommand connect -S 127.0.0.1:1080 %h %p

#2.一组/几组服务器（建议）
Host vserver01
     HostName 192.168.0.151
     Port 22
     User www
     PubkeyAuthentication yes
     IdentityFile ~/.ssh/id_rsa

Host vserver02
     HostName 192.168.0.152
     Port 22
     User www
     PubkeyAuthentication yes
     IdentityFile ~/.ssh/id_rsa

Host hserver01
     HostName 192.168.0.153
     Port 22
     User www
     PubkeyAuthentication yes
     IdentityFile ~/.ssh/id_rsa

Host oserver01
     HostName 192.168.0.153
     Port 22
     User www
     PubkeyAuthentication yes
     IdentityFile ~/.ssh/id_rsa

Host vserver*, hserver*
    ProxyCommand connect -S 127.0.0.1:1080 %h %p

#3.所有服务器（可选）
Host *
    ProxyCommand connect -S 127.0.0.1:1080 %h %p
```

>参考：[通过 Socks5 代理进行 SSH 连接](https://prinzeugen.net/ssh-over-proxy/), [ssh通过代理连接服务器](https://www.52os.net/articles/ssh-over-proxies.html)
