##iptables之LOG目标
###问题
在iptables的INPUT链中发现有大量未知包被拦截，这种情况就有两种可能，一是自己的某个服务的iptables端口没有打开，二是服务器正在遭受攻击

###分析
这就要看那些被拦截的包来源于哪里，正在访问哪个端口。于是iptables的LOG目标被派上用场。
- 指定log文件（可选， 默认在`/var/log/messages`）。服务器系统版本为`CentOS release 6.6 (Final)`, 使用了rsyslog代替了syslog，在`/etc/rsyslog.conf`文件末尾添加`kern.=notice                       /var/log/firewall.log`。
- 增加iptables规则。我的INPUT链中最后一条，也就是第二十条为REJECT所有包。所以有`iptables -I INPUT 20 -j LOG --log-level 5 --log-prefix "IPTABLES:"`
- 查看日志。分析哪些包是有用的或危险的，`tail -f /var/log/firewall.log`

###解决
根据实际情况调整防火墙规则

>参考：
>[iptables之LOG目标](http://blog.163.com/leekwen@126/blog/static/33166229200973105543171/)