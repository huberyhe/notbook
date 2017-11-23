Supervisor 是一个 CS 模式系统，允许用户在类 UNIX 系统上监测和管理一定数量的进程，还支持 Web 界面管理进程。它的目标与 launchd, daemontools 和runit 有些相似, 但是与它们不一样的是, 它不是作为 init (进程号pid是1)运行. 它是被用来控制进程, 并且它在启动的时候和一般程序并无二致。
supervisord 还要求管理的程序是非 daemon 程序，supervisord 会帮你把它转成 daemon 程序, Monit 和 Supervisord 的一个比较大的差异是 Supervisord 管理的进程必须由 Supervisord来启动，Monit 可以管理已经在运行的程序


###一、安装 Supervisor
Supervisor 是由 python 编写的，先安装 python 和 pip (python 包管理工具)
```bash
yum install epel* python python-pip -y
```

用 pip 安装 supervisor
```bash
pip install supervisor
```

安装后，会出现2个可执行文件：
/usr/bin/supervisord -- supervisor 服务守护进程
/usr/bin/supervisorctl -- supervisor 控制台进程


生成默认配置文件
```bash
echo_supervisord_conf > /etc/supervisord.conf
```

###二、配置 Supervisor
这里以 Nginx 为例介绍怎么管理应用进程

1、安装 nginx
```bash
yum -y install nginx      # 安装 nginx
```
设置 nginx 不以后台模式运行
```bash
sed -i.bak '/worker_processes/a daemon off;' /etc/nginx/nginx.conf
```

2、设置 supervisord
主要通过修改其主配置文件实现
```bash
vim /etc/supervisord.conf
```
```
[inet_http_server]             ; 开启TCP/IP http 服务器
port=192.168.18.10:9001        ; 侦听端口
username=user                  ; 认证用户名
password=123                   ; 密码

[supervisord]
logfile=/tmp/supervisord.log    ; 日志输出文件
logfile_maxbytes=50MB           ; 日志最大空间
logfile_backups=10              ; 日志轮转保留数
loglevel=info                   ; (日志等级; default info; others: debug,warn,trace)
pidfile=/tmp/supervisord.pid    ; (pid文件; default supervisord.pid)
nodaemon=false                  ; (前台运行 ;default false)
minfds=1024                     ; (最小文件描述符数 ;default 1024)
minprocs=200                    ; (最小进程数  ;default 200)

[rpcinterface:supervisor]       ; rpc 接口
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

[supervisorctl]    ; 控制台设置
erverurl=unix:///tmp/supervisor.sock    ; socket 访问路径
serverurl=http://192.168.18.10:9001     ; URL 访问路径
username=user                           ; 使用的认证用户名 （上同）
password=123                            ; 密码
prompt=mysupervisor                     ; cmd line prompt (default "supervisor")
history_file=~/.sc_history              ; use readline history if available


[program:nginx]
command=/usr/local/nginx/sbin/nginx -c /etc/nginx/nginx.conf         ; 程序路径
autorestart=true                                      ; 自动重启
priority=999                                          ; 优先级
startsecs=1                                           ; 重启前等待时间
startretries=100                                      ; 最大重启次数 
stdout_logfile=/tmp/supervisor/nginx/nginx.log        ; 日志文件
stdout_logfile_maxbytes=10MB                          ; 日志文件最大容量
stderr_logfile=/tmp/supervisor/nginx/nginx_err.log    ; 错误日志文件
stderr_logfile_maxbytes=1MB                           ; 错误日志文件最大容量

```
创建日志目录，设置权限
```bash
mkdir -p /tmp/supervisor/nginx/
chown -R nginx:nginx /tmp/supervisor/nginx/
```

启动 supervisord 的后台进程：
```bash
supervisord -c /etc/supervisord.conf
```

加入开机启动项
```bash
echo "supervisord -c /etc/supervisord.conf" >> /etc/rc.local
```

防火墙上放行 Web 端口
```bash
iptables -I INPUT -m state --state NEW -p tcp --dport 9001 -j ACCEPT
service iptables save
```


###三、管理界面介绍
```
supervisorctl start nginx  # 启动 nginx
supervisorctl start all    # 启动所有进程
supervisorctl status       # 查看进程状态
supervisorctl tail -f ssserver stderr	# debug查看日志

```

也可使用交互模式管理进程：
```bash
supervisorctl
nginx                            RUNNING   pid 4883, uptime 0:02:03
mysupervisor>
mysupervisor> help
default commands (type help &lt;topic&gt;):
=====================================
add    clear  fg        open  quit    remove  restart   start   stop  update
avail  exit   maintail  pid   reload  reread  shutdown  status  tail  version

```
还可使用 Web 界面管理
![Web 界面管理](http://7xk1pm.com1.z0.glb.clouddn.com/wp-content/uploads/2015/09/Supervisor-01.png)
测试：
```bash
lsof -i:80 #　查看到此时的 PID 为 1677
COMMAND  PID  USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
nginx   1677  root    6u  IPv4  29067      0t0  TCP *:http (LISTEN)
nginx   1678 nginx    6u  IPv4  29067      0t0  TCP *:http (LISTEN)

[root@Srv-A meld3-1.0.2]# killall nginx #关闭所有 nginx 进程：
[root@Srv-A meld3-1.0.2]# lsof -i:80 # 此时，已经生成了新的 PID 1681
```
####Debug:
生成配置文件时
```bash
supervisord -c /etc/supervisord.conf
```
出现：
```bash
Traceback (most recent call last):
  File "/usr/bin/echo_supervisord_conf", line 5, in <module>
    from pkg_resources import load_entry_point
  File "/usr/lib/python2.6/site-packages/setuptools-0.6c11-py2.6.egg/pkg_resources.py", line 2603, in <module>
  File "/usr/lib/python2.6/site-packages/setuptools-0.6c11-py2.6.egg/pkg_resources.py", line 666, in require
  File "/usr/lib/python2.6/site-packages/setuptools-0.6c11-py2.6.egg/pkg_resources.py", line 565, in resolve
pkg_resources.DistributionNotFound: meld3>=0.6.5

```
解决方法：pip安装的meld3不可用，手动安装。
```bash
wget https://pypi.python.org/packages/source/m/meld3/meld3-1.0.2.tar.gz
tar -zxf meld3-1.0.2.tar.gz
cd meld3-1.0.2
python setup.py install
```

> 参考：
http://supervisord.org/
http://supervisord.org/installing.html