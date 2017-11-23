##安装
[Docker Community Edition for Debian](https://store.docker.com/editions/community/docker-ce-server-debian?tab=description)
##问题
升级linux kernel版本后docker无法启动
```bash
hubery@gfw4:~$ sudo systemctl status docker.service
● docker.service - Docker Application Container Engine
   Loaded: loaded (/lib/systemd/system/docker.service; enabled)
   Active: failed (Result: exit-code) since 三 2017-03-15 09:00:59 UTC; 41s ago
     Docs: https://docs.docker.com
  Process: 21253 ExecStart=/usr/bin/dockerd -H fd:// (code=exited, status=1/FAILURE)
 Main PID: 21253 (code=exited, status=1/FAILURE)

3月 15 09:00:59 gfw4 systemd[1]: Starting Docker Application Container Engine...
3月 15 09:00:59 gfw4 dockerd[21253]: time="2017-03-15T09:00:59.625188518Z" level=info msg="libcontainerd: new containerd process, pid: 21256"
3月 15 09:00:59 gfw4 dockerd[21253]: time="2017-03-15T09:00:59.712787611Z" level=error msg="[graphdriver] prior storage driver aufs failed: driver not supported"
3月 15 09:00:59 gfw4 dockerd[21253]: Error starting daemon: error initializing graphdriver: driver not supported
3月 15 09:00:59 gfw4 systemd[1]: docker.service: main process exited, code=exited, status=1/FAILURE
3月 15 09:00:59 gfw4 systemd[1]: Failed to start Docker Application Container Engine.
3月 15 09:00:59 gfw4 systemd[1]: Unit docker.service entered failed state.
```
解决方法
```bash
hubery@gfw4:~$ sudo rm -rf /var/lib/docker/aufs
```
>参考:[Error starting daemon: error initializing graphdriver: driver not supported](https://github.com/docker/docker/issues/15651)