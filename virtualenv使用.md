###1.安装Virtualenv
```bash
wget https://bootstrap.pypa.io/ez_setup.py -O - | sudo python
sudo easy_install pip
sudo pip install virtualenv
```
###2.使用Virtualenv
```bash
www@v-ubuntu:~$ virtualenv -h
Usage: virtualenv [OPTIONS] DEST_DIR

Options:
  --version             show program's version number and exit
  -h, --help            show this help message and exit
  -v, --verbose         Increase verbosity.
  -q, --quiet           Decrease verbosity.
  -p PYTHON_EXE, --python=PYTHON_EXE
                        The Python interpreter to use, e.g.,
                        --python=python2.5 will use the python2.5 interpreter
                        to create the new environment.  The default is the
                        interpreter that virtualenv was installed with
                        (/usr/bin/python)
  --clear               Clear out the non-root install and start from scratch.
  --no-site-packages    DEPRECATED. Retained only for backward compatibility.
                        Not having access to global site-packages is now the
                        default behavior.
  --system-site-packages
                        Give the virtual environment access to the global
                        site-packages.
  --always-copy         Always copy files rather than symlinking.
  --unzip-setuptools    Unzip Setuptools when installing it.
  --relocatable         Make an EXISTING virtualenv environment relocatable.
                        This fixes up scripts and makes all .pth files
                        relative.
  --no-setuptools       Do not install setuptools in the new virtualenv.
  --no-pip              Do not install pip in the new virtualenv.
  --no-wheel            Do not install wheel in the new virtualenv.
  --extra-search-dir=DIR
                        Directory to look for setuptools/pip distributions in.
                        This option can be used multiple times.
  --download            Download preinstalled packages from PyPI.
  --no-download, --never-download
                        Do not download preinstalled packages from PyPI.
  --prompt=PROMPT       Provides an alternative prompt prefix for this
                        environment.
  --setuptools          DEPRECATED. Retained only for backward compatibility.
                        This option has no effect.
  --distribute          DEPRECATED. Retained only for backward compatibility.
                        This option has no effect.
```
**创建virtualenv**
`virtualenv venv`
**使用特定python版本创建virtualenv**
`virtualenv -p /usr/bin/python2.7 ENV2.7`
**激活virtualenv**
`source venv/bin/activate`
**关闭virtualenv**
`deactivate`
**生成可打包环境**
某些特殊需求下,可能没有网络, 我们期望直接打包一个ENV, 可以解压后直接使用, 这时候可以使用virtualenv -relocatable指令将ENV修改为可更改位置的ENV
```bash
#对当前已经创建的虚拟环境更改为可迁移
➜ ENV3.4 git:(master) ✗ virtualenv --relocatable ./
Making script ./bin/easy_install relative
Making script ./bin/easy_install-3.4 relative
Making script ./bin/pip relative
Making script ./bin/pip3 relative
Making script ./bin/pip3.4 relative
```

>参考：
>[python virtualenv使用 解决依赖、版本等问题](http://www.ttlsa.com/python/python-virtualenv/)
>[使用 pyenv + virtualenv 打造多版本 Python 开发环境](https://segmentfault.com/a/1190000005859547)
>[setuptools 31.0.0](https://pypi.python.org/pypi/setuptools/#using-setuptools-and-easyinstall)
>[Install Python Setuptools/Distribute for Python 2.x and Python 3.x](http://yoyzhou.github.io/blog/2012/08/12/install-python-setuptools-slash-distribute-for-both-python2-and-python3/)