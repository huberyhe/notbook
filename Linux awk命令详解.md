使用passwd文件批量恢复用户
```bash
awk -F: '{ print "groupadd -g",$3,$1 | "/bin/bash" }' testGroup
awk -F: '{ print "useradd -u",$3,"-m -d",$6,"-p defaultPasswd"$1 | "/bin/bash" }' testPasswd
```
注意
无论使用system（）还是print cmd | “/bin/bash”
awk都是新开一个shell，在相应的cmdline参数送回给shell，所以要注意当前shell变量与新开shell变量问题
```bash
root@ubuntu:~# abc=12345567890
root@ubuntu:~# awk 'BEGIN{system("echo $abc")}'

root@ubuntu:~#
root@ubuntu:~# export abc=12345567890
root@ubuntu:~# awk 'BEGIN{system("echo $abc")}'
12345567890
root@ubuntu:~#
```
>参考：
>[linux awk命令详解](http://www.cnblogs.com/ggjucheng/archive/2013/01/13/2858470.html)
>[awk中使用的shell命令-浅记](http://blog.chinaunix.net/uid-280990-id-2449802.html)