在使用centos7的软件包管理程序yum安装python-pip的时候会报一下错误：

```
No package python-pip available.

Error: Nothing to do
```

说没有python-pip软件包可以安装。

这是因为像centos这类衍生出来的发行版，他们的源有时候内容更新的比较滞后，或者说有时候一些扩展的源根本就没有。

所以在使用yum来search python-pip的时候，会说没有找到该软件包。因此为了能够安装这些包，需要先安装扩展源EPEL。

EPEL(http://fedoraproject.org/wiki/EPEL) 是由 Fedora 社区打造，为 RHEL 及衍生发行版如 CentOS、Scientific Linux 等提供高质量软件包的项目。

首先安装epel扩展源：

```bash
sudo yum -y install epel-release
```

然后安装python-pip

```bash
sudo yum -y install python-pip
```

安装完之后别忘了清除一下cache

```bash
sudo yum clean all
```

搞定，收工！

原文链接：[Ailab.cn](http://www.ailab.cn/article/2015082028905.html)