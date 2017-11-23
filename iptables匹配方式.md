## 1 内建的匹配方式
### 1.1 Interface的匹配方式
`iptables -t filter -A FORWARD -i eth0 -o eth1 -j DROP`
### 1.2 Source Destination Address的匹配方式
`iptables -A FORWARD -p tcp -i eth1 -o eth0 -d www.playboy.com -j DROP`
`iptables -A FORWARD -i eth0 -o eth1 -p tcp -s 192.168.10.1 -d $WEB_SRV --dport 80 -j DROP`

```bash
#能ping出，不能被ping到
iptables -A INPUT -p icmp -j DROP
iptables -A INPUT -p icmp --icmp -type 8 -j DROP
```

## 2 使用模块延伸出来的匹配方式
### 2.1 TCP / UDP协议的高级匹配
--dport和--sport一次最多可以匹配两个port，例如：--dport 53：69
### 2.2 MAC Address的匹配
模块：xt_mac.ko
特定mac地址机器才能访问mysql
`iptables -A INPUT -p tcp --dport 3306 -m mac --mac-source 00:02:B3:0C:23:1B -j ACCEPT`
### 2.3 Multiport的匹配
模块：xt_multiport.ko
参数：
- --dports: 匹配的目标为Destination Port， 如--dports 21,22,25
- --sports: 匹配的目标为Source Port
- --ports: 匹配的目标为Destination / Source Port
### 2.4 匹配封包的MARK值
模块：xt_mark.ko
把符合条件的封包标志为80
`iptables -t mangle -A PREROUTING -p tcp --dport 80 -j MARK --set-mark 80`
如果进到FORWARD Chain的封包，其MARK值为80的话，就将其丢掉
`iptables -A FORWARD -p all -m mark --mark 80 -j DROP`
### 2.5 Owner的匹配
模块：ipt_owner.ko
参数：
- --uid-owner userid
- --gid-owner groupid
- --pid-owner processid
- --sid-owner sessionid: 我们可以使用ps指令再加上“j“参数，即可看到Process Group ID
- --cmd-owner processname
### 2.5 IP范围的匹配
模块：ipt_iprange.ko
参数：
- --src-range: 匹配来源地址的范围
- --dst-range: 匹配目的地址的范围
### 2.6 TTL值的匹配
模块：ipt_ttl.ko
参数：
- --ttl-eq 等于
- --ttl-lt 小于
- --ttl-gt 大于
### 封包的状态匹配
模块：xt_state.ko匹配方式，ip_conntrack.ko连接追踪功能
