#镜像
##构建镜像
`$ docker build -t nginx:v3 .`
```bash
www@v-ubuntu:~$ docker build --help

Usage:	docker build [OPTIONS] PATH | URL | -

Build an image from a Dockerfile

Options:
      --build-arg value         Set build-time variables (default [])
      --cgroup-parent string    Optional parent cgroup for the container
      --cpu-period int          Limit the CPU CFS (Completely Fair Scheduler) period
      --cpu-quota int           Limit the CPU CFS (Completely Fair Scheduler) quota
  -c, --cpu-shares int          CPU shares (relative weight)
      --cpuset-cpus string      CPUs in which to allow execution (0-3, 0,1)
      --cpuset-mems string      MEMs in which to allow execution (0-3, 0,1)
      --disable-content-trust   Skip image verification (default true)
  -f, --file string             Name of the Dockerfile (Default is 'PATH/Dockerfile')
      --force-rm                Always remove intermediate containers
      --help                    Print usage
      --isolation string        Container isolation technology
      --label value             Set metadata for an image (default [])
  -m, --memory string           Memory limit
      --memory-swap string      Swap limit equal to memory plus swap: '-1' to enable unlimited swap
      --no-cache                Do not use cache when building the image
      --pull                    Always attempt to pull a newer version of the image
  -q, --quiet                   Suppress the build output and print image ID on success
      --rm                      Remove intermediate containers after a successful build (default true)
      --shm-size string         Size of /dev/shm, default value is 64MB
  -t, --tag value               Name and optionally a tag in the 'name:tag' format (default [])
      --ulimit value            Ulimit options (default [])
```
##Dockerfile 指令详解
###FROM
###RUN
###COPY 复制文件
格式：

COPY <源路径>... <目标路径>
COPY ["<源路径1>",... "<目标路径>"]
和 RUN 指令一样，也有两种格式，一种类似于命令行，一种类似于函数调用。
###ADD 更高级的复制文件
ADD 指令和 COPY 的格式和性质基本一致。但是在 COPY 基础上增加了一些功能。比如，根据<源路径>url下载文件，将<源路径>压缩包解压
###CMD 容器启动命令
CMD 指令的格式和 RUN 相似，也是两种格式：

shell 格式：CMD <命令>
exec 格式：CMD ["可执行文件", "参数1", "参数2"...]
参数列表格式：CMD ["参数1", "参数2"...]。在指定了 ENTRYPOINT 指令后，用 CMD 指定具体的参数。
CMD 指令就是用于指定默认的容器主进程的启动命令的。

在指令格式上，一般推荐使用 exec 格式，这类格式在解析时会被解析为 JSON 数组，因此一定要使用双引号 "，而不要使用单引号。

Docker 不是虚拟机，容器中的应用都应该以前台执行，而不是像虚拟机、物理机里面那样，用 upstart/systemd 去启动后台服务，容器内没有后台服务的概念。

###ENTRYPOINT 入口点
ENTRYPOINT 的格式和 RUN 指令格式一样，分为 exec 格式和 shell 格式。

ENTRYPOINT 的目的和 CMD 一样，都是在指定容器启动程序及参数。ENTRYPOINT 在运行时也可以替代，不过比 CMD 要略显繁琐，需要通过 docker run 的参数 --entrypoint 来指定。

当指定了 ENTRYPOINT 后，CMD 的含义就发生了改变，不再是直接的运行其命令，而是将 CMD 的内容作为参数传给 ENTRYPOINT 指令，换句话说实际执行时，将变为：

`<ENTRYPOINT> "<CMD>"`
###ENV 设置环境变量
格式有两种：

- `ENV <key> <value>`
- `ENV <key1>=<value1> <key2>=<value2>...`

下列指令可以支持环境变量展开： ADD、COPY、ENV、EXPOSE、LABEL、USER、WORKDIR、VOLUME、STOPSIGNAL、ONBUILD。

###ARG 构建参数
格式：ARG <参数名>[=<默认值>]

Dockerfile 中的 ARG 指令是定义参数名称，以及定义其默认值。该默认值可以在构建命令 docker build 中用 --build-arg <参数名>=<值> 来覆盖。

###VOLUME 定义匿名卷
格式为：

VOLUME ["<路径1>", "<路径2>"...]
VOLUME <路径>

###EXPOSE 声明端口
格式为 EXPOSE <端口1> [<端口2>...]。

EXPOSE 指令是声明运行时容器提供服务端口

###WORKDIR 指定工作目录

格式为 WORKDIR <工作目录路径>。

###USER 指定当前用户

格式：USER <用户名>

USER 指令和 WORKDIR 相似，都是改变环境状态并影响以后的层。WORKDIR 是改变工作目录，USER 则是改变之后层的执行 RUN, CMD 以及 ENTRYPOINT 这类命令的身份。

###HEALTHCHECK 健康检查

格式：

HEALTHCHECK [选项] CMD <命令>：设置检查容器健康状况的命令
HEALTHCHECK NONE：如果基础镜像有健康检查指令，使用这行可以屏蔽掉其健康检查指令

HEALTHCHECK 支持下列选项：

--interval=<间隔>：两次健康检查的间隔，默认为 30 秒；
--timeout=<时长>：健康检查命令运行超时时间，如果超过这个时间，本次健康检查就被视为失败，默认 30 秒；
--retries=<次数>：当连续失败指定次数后，则将容器状态视为 unhealthy，默认 3 次。
和 CMD, ENTRYPOINT 一样，HEALTHCHECK 只可以出现一次，如果写了多个，只有最后一个生效。

###ONBUILD 为他人做嫁衣裳

格式：ONBUILD <其它指令>。

ONBUILD 是一个特殊的指令，它后面跟的是其它指令，比如 RUN, COPY 等，而这些指令，在当前镜像构建时并不会被执行。只有当以当前镜像为基础镜像，去构建下一级镜像的时候才会被执行。

###docker save 和 docker load
比如我们希望保存这个 alpine 镜像。
```
$ docker images alpine
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
alpine              latest              baa5d63471ea        5 weeks ago         4.803 MB
```
保存镜像的命令为：
```
$ docker save alpine | gzip > alpine-latest.tar.gz
```
然后我们将 alpine-latest.tar.gz 文件复制到了到了另一个机器上，可以用下面这个命令加载镜像：
```
$ docker load -i alpine-latest.tar.gz
Loaded image: alpine:latest
```
果我们结合这两个命令以及 ssh 甚至 pv 的话，利用 Linux 强大的管道，我们可以写一个命令完成从一个机器将镜像迁移到另一个机器，并且带进度条的功能：
```
docker save <镜像名> | bzip2 | pv | ssh <用户名>@<主机名> 'cat | docker load'
```

###删除本地镜像
如果要删除本地的镜像，可以使用 docker rmi 命令，其格式为：

docker rmi [选项] <镜像1> [<镜像2> ...]

其中，<镜像> 可以是 镜像短 ID、镜像长 ID、镜像名 或者 镜像摘要。

当然，更精确的是使用 镜像摘要 删除镜像。
```
$ docker images --digests
```
用 docker images 命令来配合
```
$ docker rmi $(docker images -q -f dangling=true)
```
```
$ docker rmi $(docker images -q redis)
```
```
$ docker rmi $(docker images -q -f before=mongo:3.2)
```

#容器
##启动容器
###新建并启动
当利用 `docker run` 来创建容器时，Docker 在后台运行的标准操作包括：

- 检查本地是否存在指定的镜像，不存在就从公有仓库下载
- 利用镜像创建并启动一个容器
- 分配一个文件系统，并在只读的镜像层外面挂载一层可读写层
- 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去
- 从地址池配置一个 ip 地址给容器
- 执行用户指定的应用程序
- 执行完毕后容器被终止

###启动已终止容器
可以利用 `docker start` 命令，直接将一个已经终止的容器启动运行。

###后台(background)运行
通过添加 -d 参数来实现让Docker在后台运行。
此时容器会在后台运行并不会把输出的结果(STDOUT)打印到宿主机上面(输出结果可以用docker logs 查看)。
```
$ sudo docker logs [container ID or NAMES]
```
##终止容器
可以使用 `docker stop` 来终止一个运行中的容器。
终止状态的容器可以用 `docker ps -a` 命令看到。
可以通过 `docker start` 命令来重新启动。
`docker restart` 命令会将一个运行态的容器终止，然后再重新启动它。

##列出容器
·docker ps`查看容器列表
###用法
`docker ps [OPTIONS]`
###选项
```bash
Usage:	docker ps [OPTIONS]

List containers

Options:
  -a, --all             Show all containers (default shows just running)
  -f, --filter value    Filter output based on conditions provided (default [])
      --format string   Pretty-print containers using a Go template
      --help            Print usage
  -n, --last int        Show n last created containers (includes all states) (default -1)
  -l, --latest          Show the latest created container (includes all states)
      --no-trunc        Don't truncate output
  -q, --quiet           Only display numeric IDs
  -s, --size            Display total file sizes

```
###docker filter
[docker filter](https://docs.docker.com/engine/reference/commandline/ps/#/filtering)
格式：--filter "foo=bar"

	- id (container’s id)
	- label (label=<key> or label=<key>=<value>)
	- name (container’s name)
	- exited (int - the code of exited containers. Only useful with --all)
	- status (created 	restarting 	running 	removing 	paused 	exited 	dead)
	- ancestor (<image-name>[:<tag>], <image id> or <image@digest>) - filters containers that were created from the given image or a descendant.
	- before (container’s id or name) - filters containers created before given id or name
	- since (container’s id or name) - filters containers created since given id or name
	- isolation (default 	process 	hyperv) (Windows daemon only)
	- volume (volume name or mount point) - filters containers that mount volumes.
	- network (network id or name) - filters containers connected to the provided network
	- health (starting 	healthy 	unhealthy 	none) - filters containers based on healthcheck status
	- publish=(container’s published port) - filters published ports by containers
	- expose=(container’s exposed port) - filters exposed ports by containers

例子1：列出使用`ubuntu:lastest`镜像的容器：
```bash
$ docker ps --filter ancestor=ubuntu

CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS              PORTS               NAMES
919e1179bdb8        ubuntu-c1           "top"               About a minute ago   Up About a minute                       admiring_lovelace
5d1e4a540723        ubuntu-c2           "top"               About a minute ago   Up About a minute                       admiring_sammet
82a598284012        ubuntu              "top"               3 minutes ago        Up 3 minutes                            sleepy_bose
bab2a34ba363        ubuntu              "top"               3 minutes ago        Up 3 minutes                            focused_yonath
```

##进入容器
###attach 命令
```
$ sudo docker run -idt ubuntu
243c32535da7d142fb0e6df616a3c3ada0b8ab417937c853a9e1c251f499f550
$ sudo docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
243c32535da7        ubuntu:latest       "/bin/bash"         18 seconds ago      Up 17 seconds                           nostalgic_hypatia
$sudo docker attach nostalgic_hypatia
root@243c32535da7:/#
```
###nsenter 命令
```
$ sudo docker run -idt ubuntu
243c32535da7d142fb0e6df616a3c3ada0b8ab417937c853a9e1c251f499f550
$ sudo docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
243c32535da7        ubuntu:latest       "/bin/bash"         18 seconds ago      Up 17 seconds                           nostalgic_hypatia
$ PID=$(docker-pid 243c32535da7)
10981
$ sudo nsenter --target 10981 --mount --uts --ipc --net --pid
root@243c32535da7:/#
```
##导出和导入容器
如果要导出本地某个容器，可以使用 docker export 命令。
可以使用 docker import 从容器快照文件中再导入为镜像。

*注：用户既可以使用 docker load 来导入镜像存储文件到本地镜像库，也可以使用 docker import 来导入一个容器快照到本地镜像库。这两者的区别在于容器快照文件将丢弃所有的历史记录和元数据信息（即仅保存容器当时的快照状态），而镜像存储文件将保存完整记录，体积也要大。此外，从容器快照文件导入时可以重新指定标签等元数据信息。
###删除容器
可以使用 docker rm 来删除一个处于终止状态的容器。

#仓库
#数据管理
##数据卷
###创建一个数据卷
```
$ sudo docker run -d -P --name web -v /webapp training/webapp python app.py
```
###删除数据卷
可以在删除容器的时候使用 `docker rm -v` 这个命令
###挂载一个主机目录作为数据卷
```
$ sudo docker run -d -P --name web -v /src/webapp:/opt/webapp training/webapp python app.py
```
Docker 挂载数据卷的默认权限是读写，用户也可以通过 :ro 指定为只读。
```
$ sudo docker run -d -P --name web -v /src/webapp:/opt/webapp:ro training/webapp python app.py
```
###查看数据卷的具体信息
```
$ docker inspect web
...
```
###挂载一个本地主机文件作为数据卷
##数据卷容器
如果你有一些持续更新的数据需要在容器之间共享，最好创建数据卷容器。
数据卷容器，其实就是一个正常的容器，专门用来提供数据卷供其它容器挂载的。
在其他容器中使用 --volumes-from 来挂载 dbdata 容器中的数据卷。

##其他资料拷贝方式
[如何在Docker容器内外互相拷贝数据](http://www.jb51.net/article/101510.htm)
`docker cp <container_id>:from_file_in container to_file_in_local`

#使用网络
##外部访问容器
当使用 -P 标记时，Docker 会随机映射一个 49000~49900 的端口到内部容器开放的网络端口。
-p（小写的）则可以指定要映射的端口，并且，在一个指定端口上只可以绑定一个容器。支持的格式有 ip:hostPort:containerPort | ip::containerPort | hostPort:containerPort。
###查看映射端口配置
```
$ docker port nostalgic_morse 5000
127.0.0.1:49155.
```
##容器互联
###自定义容器命名
使用 --name 标记可以为容器自定义命名。
使用 docker ps 来验证设定的命名。
也可以使用 docker inspect 来查看容器的名字
```
$ sudo docker inspect -f "{{ .Name }}" aed84ee21bde
/web
```
###容器互联

使用 --link 参数可以让容器之间安全的进行交互。

下面先创建一个新的数据库容器。
```
$ sudo docker run -d --name db training/postgres
```
然后创建一个新的 web 容器，并将它连接到 db 容器
```
$ sudo docker run -d -P --name web --link db:db training/webapp python app.py
```
此时，db 容器和 web 容器建立互联关系。

--link 参数的格式为 --link name:alias，其中 name 是要链接的容器的名称，alias 是这个连接的别名。

#高级网络配置
##快速配置指南
##配置 DNS
##容器访问控制
##配置 docker0 网桥
##编辑网络配置文件
Docker 1.2.0 开始支持在运行中的容器里编辑 /etc/hosts, /etc/hostname 和 /etc/resolve.conf 文件。
但是这些修改是临时的，只在运行的容器中保留，容器终止或重启后并不会被保存下来。也不会被 docker commit 提交。