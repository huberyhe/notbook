##实例
###配置文件
```ini
[Unit]
Description=uWSGI Emperor
After=syslog.target

[Service]
ExecStart=/root/uwsgi/uwsgi --ini /etc/uwsgi/emperor.ini
# Requires systemd version 211 or newer
RuntimeDirectory=uwsgi
Restart=always
KillSignal=SIGQUIT
Type=notify
StandardError=syslog
NotifyAccess=all

[Install]
WantedBy=multi-user.target
```
保存为 /etc/systemd/system/emperor.uwsgi.service； 或保存为/usr/lib/systemd/system/emperor.uwsgi.service，然后 `systemctl enable emperor.uwsgi.service`，这将创建同等软链接。
###启动与关闭、重启动
```bash
systemctl start emperor.uwsgi.service
systemctl stop emperor.uwsgi.service
systemctl restart emperor.uwsgi.service
systemctl status emperor.uwsgi.service
```
>参考：
[Linux运维笔记>>systemd详解](https://blog.linuxeye.com/400.html)
[阮一峰的网络日志>>Systemd 入门教程：命令篇](http://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-commands.html)
[systemd详解](https://blog.linuxeye.com/400.html)
[systemd.service](https://www.freedesktop.org/software/systemd/man/systemd.service.html#)
[RuntimeDirectory= creates directory with wrong group if Group= is set](https://github.com/systemd/systemd/issues/1231)
