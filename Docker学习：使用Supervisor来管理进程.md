# Docker学习之使用 Supervisor 来管理进程
Docker 容器在启动的时候开启单个进程，比如，一个 ssh 或者 apache 的 daemon 服务。但我们经常需要在一个机器上开启多个服务，这可以有很多方法，最简单的就是把多个启动命令放到一个启动脚本里面，启动的时候直接启动这个脚本，另外就是安装进程管理工具。
这里我们使用进程管理工具 supervisor 来管理容器中的多个进程。使用 Supervisor 可以更好的控制、管理、重启我们希望运行的进程。

## Dockerfile配置文件
```
FROM ubuntu:13.04
MAINTAINER examples@docker.com
RUN echo "deb http://archive.ubuntu.com/ubuntu precise main universe" > /etc/apt/sources.list
RUN apt-get update
RUN apt-get upgrade -y

RUN apt-get install -y --force-yes  perl-base=5.14.2-6ubuntu2
RUN apt-get install -y apache2.2-common
RUN apt-get install -y openssh-server apache2 supervisor
RUN mkdir -p /var/run/sshd
RUN mkdir -p /var/log/supervisor

COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
EXPOSE 22 80
CMD ["/usr/bin/supervisord"]
```

## supervisor配置文件
```
[supervisord]
nodaemon=true
[program:sshd]
command=/usr/sbin/sshd -D

[program:apache2]
command=/bin/bash -c "source /etc/apache2/envvars && exec /usr/sbin/apache2 -DFOREGROUND"
```
配置文件包含目录和进程，第一段 supervsord 配置软件本身，**使用 nodaemon 参数来运行**。第二段包含要控制的 2 个服务。每一段包含一个服务的目录和启动这个服务的命令。

## 创建镜像
`$ sudo docker build -t test/supervisord .`

## 启动容器
`$ sudo docker run -p 22 -p 80 -t -i test/supervisord`

## 重点
最重要的是supervisor默认是以守护进程启动的，如果把一个守护进程当作容器启动命令，则容器在启动这个daemon后会随着命令执行完立马退出。因此如上我们需要在`supervisor.conf`中加上：
```
[supervisord]
nodaemon=true
```
或者在启动时带上参数`--nodaemon`，即：
```
CMD ["/usr/bin/supervisord", "--nodaemon ", "-c", "/etc/supervisor/supervisord.conf"]
```
>参考：
>《Docker —— 从入门到实践》
>[使用supervisord管理容器进程](http://debugo.com/docker-supervisord/)
>[Docker Supervisor 进程管理/启动](https://onebox.site/archives/76.html)