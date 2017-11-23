### 1.起因
是升级到ubuntu 16之后使用hexo遇到错误：
**(node:15317) [DEP0061] DeprecationWarning: fs.SyncWriteStream is deprecated.**

**分析：**升级系统之后node版本也有升级, 然后`fs.SyncWriteStream`这个方法被废弃了，我们只要升级hexo博客同时保留数据就可以了。在查找资料的时候我发现一个很好的管理hexo博客的方法（[使用hexo，如果换了电脑怎么更新博客？ - 回答作者: CrazyMilk](https://zhihu.com/question/21193762/answer/79109280)），将hexo博客的源码放到了github page的仓库里，于是我索性升级和迁移一起做了。

**OLD OS:**
```bash
www@v-ubuntu:~/git_repo/hexo_blog$ hexo version
hexo: 3.3.7
hexo-cli: 1.0.3
os: Linux 4.4.0-87-generic linux x64
http_parser: 2.5.0
node: 4.2.4
v8: 4.5.103.35
uv: 1.7.5
zlib: 1.2.8
ares: 1.10.1-DEV
modules: 46
openssl: 1.0.2e
```
**NEW OS:**
```bash
hubery@hubery-VirtualBox:~/Workspace/hexo_blog$ hexo version
(node:17263) [DEP0061] DeprecationWarning: fs.SyncWriteStream is deprecated.
hexo: 3.3.7
hexo-cli: 1.0.3
os: Linux 4.10.0-35-generic linux x64
http_parser: 2.7.0
node: 8.6.0
v8: 6.0.287.53
uv: 1.14.1
zlib: 1.2.11
ares: 1.10.1-DEV
modules: 57
nghttp2: 1.25.0
openssl: 1.0.2l
icu: 59.1
unicode: 9.0
cldr: 31.0.1
tz: 2017b
```
### 2.更新流程
1.github page仓库：[huberyhe.github.io](https://github.com/huberyhe/huberyhe.github.io)
2.hexo blog源码仓库：~/Workspace/hexo_blog
3.hexo安装：
```bash
$ curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.2/install.sh | bash
$ nvm install stable
$ npm install -g hexo-cli
```
4.克隆github page仓库, 创建hexo空白分支用于存放hexo源码
```bash
$ cd ~/Workspace
$ git clone github:huberyhe/huberyhe.github.io.git
$ cd huberyhe.github.io/
$ git checkout --orphan gh-pages
```
5.初始化hexo博客，拷贝旧仓库的数据部分
```bash
$ hexo init
$ rm _config.yml package* scaffolds/ source/ themes/ -rf
$ cd ../hexo_blog
$ cp _config.yml package.json scaffolds source themes new_from_md.sh ../huberyhe.github.io/ -Ra
$ cd -
$ git add -A
$ git commit -m "add branch hexo with hexo original files"
$ git push origin hexo
```
5.发布博客，提交博客静态文件
```bash
$ npm install hexo-deployer-git
$ hexo -g d
```
5.在[huberyhe.github.io](https://github.com/huberyhe/huberyhe.github.io)上，将hexo设为默认分支
6.在本地对博客进行修改（添加新博文、修改样式等等）后，通过下面的流程进行管理
1. 依次执行`git add .`、`git commit -m "..."`、`git push origin hexo`指令将改动推送到GitHub（此时当前分支应为hexo）
2. 然后才执行`hexo g -d`发布网站到master分支上。

>参考：
>[Hexo报错，DEP0061](http://www.abrocks.com/2017/06/17/node8.0方法弃用处理/#more)
>[将 HEXO 从 Ubuntu 迁移到 windows](http://www.cnblogs.com/JinyaoLi/p/4672376.html)
>[使用hexo，如果换了电脑怎么更新博客？ - 回答作者: CrazyMilk](https://zhihu.com/question/21193762/answer/79109280)
>[Hexo Getting Started](https://hexo.io/docs/index.html)
>[git中创建新的空白分支](http://blog.csdn.net/playboyanta123/article/details/48975175)