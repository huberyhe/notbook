```bash
sudo apt-get install ppp pptpd -y
sudo vim /etc/pptpd.conf
sudo vim /etc/ppp/options
sudo vim /etc/ppp/chap-secrets
sudo vim /etc/sysctl.conf
sudo sysctl -p
sudo iptables -t nat -A POSTROUTING -s 192.168.0.0/24 -o eth0 -j MASQUERADE
sudo iptables -I INPUT 5 -m state --state NEW -p tcp --dport 1723 -j ACCEPT
sudo bash -c "iptables-save > /etc/iptables.rule"
...
Nov 14 09:11:08 gfw3 pptpd[19287]: CTRL: Client 113.116.74.221 control connection started
Nov 14 09:11:09 gfw3 pptpd[19287]: CTRL: Starting call (launching pppd, opening GRE)
Nov 14 09:11:09 gfw3 pppd[19288]: Plugin /usr/lib/pptpd/pptpd-logwtmp.so loaded.
Nov 14 09:11:09 gfw3 pppd[19288]: pppd 2.4.6 started by root, uid 0
Nov 14 09:11:09 gfw3 pppd[19288]: Using interface ppp0
Nov 14 09:11:09 gfw3 pppd[19288]: Connect: ppp0 <--> /dev/pts/1
Nov 14 09:11:38 gfw3 pptpd[19287]: CTRL: Reaping child PPP[19288]
Nov 14 09:11:38 gfw3 pppd[19288]: Modem hangup
Nov 14 09:11:38 gfw3 pppd[19288]: Connection terminated.
Nov 14 09:11:38 gfw3 pppd[19288]: Exit.
Nov 14 09:11:38 gfw3 pptpd[19287]: CTRL: Client 113.116.74.221 control connection finished
...
sudo modprobe nf_conntrack_pptp
sudo tail -f /var/log/pptpd.log
...
Plugin /usr/lib/pptpd/pptpd-logwtmp.so loaded.
Using interface ppp0
Connect: ppp0 <--> /dev/pts/1
peer from calling number 113.116.74.221 authorized
MPPE required but peer negotiation failed
Connection terminated.
Connect time 0.0 minutes.
Sent 10 bytes, received 53 bytes.
...
```
client select to use mppe
客户端（Ubuntu 14连接VPN高级选项中选中“Use Porint-to-Point encryption(MPPE)”
```bash
sudo tail -f /var/log/pptpd.log
...
Plugin /usr/lib/pptpd/pptpd-logwtmp.so loaded.
Using interface ppp0
Connect: ppp0 <--> /dev/pts/1
peer from calling number 113.116.74.221 authorized
CCP: timeout sending Config-Requests
LCP terminated by peer (User request)
Modem hangup
Connection terminated.
Connect time 0.7 minutes.
Sent 100 bytes, received 0 bytes.
...
sudo modprobe ip_nat_pptp

sudo iptables -I INPUT 4 -m state --state NEW -p gre -j ACCEPT
```
can connect vpn but not access to Internet.
能够连接VPN，但不能访问因特网
```bash
sudo iptables -I FORWARD -s 192.168.0.0/24 -o eth0 -j ACCEPT
```

>参考
>[Centos 7搭建VPN（PPTP）服务器方法](http://www.wanghailin.cn/centos-7-vpn/)
>[Linux（VPS+Debian）搭建配置VPN（PPTP）服务](http://blog.bossma.cn/server/linux-vps-debian-vpn-server-pptp/)
>[通过加载ip_nat_pptp模块使iptables支持PPTP穿透](通过加载ip_nat_pptp模块使iptables支持PPTP穿透)
>[Ubuntu VPN连接失败解决方案 ](http://blog.163.com/sg_liao/blog/static/295770832010111812110294/)
>[Failed to Connect to PPTP VPN Server on Ubuntu](http://askubuntu.com/questions/269399/failed-to-connect-to-pptp-vpn-server-on-ubuntu)
>[Linux下搭建VPN服务器（CentOS、pptp）](Linux下搭建VPN服务器（CentOS、pptp）)
>