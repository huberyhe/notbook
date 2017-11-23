**按时间**
iptables -I FORWARD -m time --timestart 8:15 --timestop 12:30 -j DROP
**按mac地址**
iptables -I FORWARD -m mac --mac-source c0:64:c6:cc:3e:a0 -j DROP