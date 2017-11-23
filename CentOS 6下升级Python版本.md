CentOS6.8默认的python版本是2.6，而现在好多python组件开始只支持2.7以上的版本，比如说我今天遇到的`pip install pysqlite`，升级python版本是一个痛苦但又常见的工作，为什么会痛苦？就是当你安装新版本的同时删掉老版本时，你会一脸懵逼，发现yum/apt都不能用了。

## 0.环境
```bash
[hubery@vultr ~]$ uname -a
Linux vultr.guest 2.6.32-642.4.2.el6.x86_64 #1 SMP Tue Aug 23 19:58:13 UTC 2016 x86_64 x86_64 x86_64 GNU/Linux
[hubery@vultr ~]$ cat /etc/redhat-release
CentOS release 6.9 (Final)
[hubery@vultr ~]$ python -V
Python 2.6.6

```
## 1.编译安装新版本
当前Python 2.7.x的最新版本是[Python 2.7.13](https://www.python.org/ftp/python/2.7.13/Python-2.7.13.tgz)
```bash
sudo yum -y update
sudo yum groupinstall -y 'development tools'
sudo yum install -y zlib-devel bzip2-devel openssl-devel xz-libs wget sqlite-devel

wget https://www.python.org/ftp/python/2.7.13/Python-2.7.13.tgz
tar xvzf Python-2.7.13.tgz
cd Python-2.7.13
./configure
make
sudo make install
```
## 2.安装pip
```bash
wget https://pypi.python.org/packages/11/b6/abcb525026a4be042b486df43905d6893fb04f05aac21c32c638e939e447/pip-9.0.1.tar.gz#md5=35f01da33009719497f01a4ba69d63c9
tar xvzf pip-9.0.1.tar.gz
cd pip-9.0.1
sudo /usr/bin/python2.7 setup.py install
```
## 3.替换老版本成为默认版本
```bash
sudo mv /usr/bin/python /usr/bin/python2.6.6
sudo mv /usr/bin/pip /usr/bin/pip7.1.0
sudo ln -s /usr/local/bin/python /usr/bin/python
sudo ln -s /usr/local/bin/pip /usr/bin/pip
```
## 4.解决yum对于老版本的依赖
yum 会依赖系统默认版本的python，升级之后yum无法使用
所以修改yum的配置
修改文件： vim /usr/bin/yum
修改头#!/usr/bin/python  => #!/usr/bin/python2.6.6

>参考
>[Installing Python 2.7.8 on CentOS 6.5](http://bicofino.io/2014/01/16/installing-python-2-dot-7-6-on-centos-6-dot-5/)
>[Django - No module named _sqlite3](http://stackoverflow.com/questions/10784132/django-no-module-named-sqlite3)
>[No module named yum错误的解决办法](http://www.weste.net/2013/4-10/90297.html)