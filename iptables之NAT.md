### 1.NAT
将源地址改为公网IP：
`iptables -t nat -A POSTROUTING -o eth0 -s 192.168.0.0/24 -j SNAT --to $PUBLIC_IP`
如果PUBLIC_IP不是固定的：
`iptables -t nat -A POSTROUTING -o eth0 -s 192.168.0.0/24 -j MASQUERADE`
### 2.NAPT
端口映射DMZ
`iptables -t nat -A  PREROUTING -i eth0 -p tcp --dport 80 -j DNAT --to 192.168.0.1:80`