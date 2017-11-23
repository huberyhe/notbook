##搭建带认证的私有Docker仓库
###环境
Registry域名
```
root@remote:~# nslookup registry.0x01.party
Server:		2001:4860:4860::8844
Address:	2001:4860:4860::8844#53

Non-authoritative answer:
Name:	registry.0x01.party
Address: 128.199.58.37
```
Registry服务器（128.199.58.37）
```
root@remote:~# cat /etc/os-release
PRETTY_NAME="Debian GNU/Linux 8 (jessie)"
NAME="Debian GNU/Linux"
VERSION_ID="8"
VERSION="8 (jessie)"
ID=debian
HOME_URL="http://www.debian.org/"
SUPPORT_URL="http://www.debian.org/support"
BUG_REPORT_URL="https://bugs.debian.org/"
```
本地（Registry用户）
```
www@v-ubuntu:~/docker_repo$ cat /etc/os-release 
NAME="Ubuntu"
VERSION="14.04.5 LTS, Trusty Tahr"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 14.04.5 LTS"
VERSION_ID="14.04"
HOME_URL="http://www.ubuntu.com/"
SUPPORT_URL="http://help.ubuntu.com/"
BUG_REPORT_URL="http://bugs.launchpad.net/ubuntu/"
```
###方案
- 源码安装Docker Registry，使用Nginx做反向代理和Nginx Basic Authentication
- 根据[官方文档](https://docs.docker.com/registry/deploying/)，现有版本的Docker Registry是自带基础验证功能的。

###使用官方 registry 镜像搭建私有registry
####生成证书
```
root@remote:~# wget https://dl.eff.org/certbot-auto
root@remote:~# chmod a+x certbot-auto
root@remote:~# ./certbot-auto
root@remote:~# cp /etc/letsencrypt/live/registry.0x01.party ./ certs-certbot -Ra
root@remote:~# ls certs-certbot/
cert.pem  chain.pem  fullchain.pem  privkey.pem  README

```
####创建基础验证文件
```
root@remote:~# mkdir auth
root@remote:~# docker run --entrypoint htpasswd registry:2 -Bbn admin testpassword > auth/htpasswd
```
####安装Docker
参考[官方文档](https://docs.docker.com/engine/installation/linux/debian/)添加源和安装。
```
root@remote:~# apt-get install docker-engine
root@remote:~# service docker start
```
####运行Docker Registry
```
root@remote:~# docker run -d -p 443:5000 --restart=always --name registry \
  -v /data/registry:/tmp/registry \
  -v `pwd`/auth:/auth \
  -e "REGISTRY_AUTH=htpasswd" \
  -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
  -e "REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd" \
  -v `pwd`/certs-certbot:/certs \
  -e "REGISTRY_HTTP_TLS_CERTIFICATE=/certs/fullchain.pem" \
  -e "REGISTRY_HTTP_TLS_KEY=/certs/privkey.pem" \
  registry:2
```
仓库会被创建在容器的 /tmp/registry 下。可以通过 -v 参数来将镜像文件存放在本地的指定路径。 例如上面的例子将上传的镜像放到 /data/registry 目录。
TLS验证文件使用的是容器中 /certs/ 文件夹下，也就是主机 `pwd`/certs-certbot 文件夹下的 fullchain.pem 和 privkey.pem。
####测试上传
```
www@v-ubuntu:~/docker_repo$ docker tag hello-world registry.0x01.party:443/test
www@v-ubuntu:~/docker_repo$ docker images
REPOSITORY                     TAG                 IMAGE ID            CREATED             SIZE
hello-world                    latest              690ed74de00f        15 months ago       960 B
registry.0x01.party:443/test   latest              690ed74de00f        15 months ago       960 B
learn/tutorial                 latest              a7876479f1aa        3 years ago         128 MB
www@v-ubuntu:~/docker_repo$ docker login registry.0x01.party:443
Username (admin):      
Password: 
Login Succeeded
www@v-ubuntu:~/docker_repo$ docker push registry.0x01.party:443/test
The push refers to a repository [registry.0x01.party:443/test]
Get https://registry.0x01.party:443/v1/_ping: x509: certificate signed by unknown authority
www@v-ubuntu:~/docker_repo$ cat fullchain.pem | sudo tee -a /etc/ssl/certs/ca-certificates.crt
www@v-ubuntu:~/docker_repo$ docker push registry.0x01.party:443/test
The push refers to a repository [registry.0x01.party:443/test]
5f70bf18a086: Pushed 
b652ec3a27e7: Pushed 
latest: digest: sha256:4296e66dfc9390cd3fd39d56baa66a6f70014e36a01304fcb3183f2c52ee3050 size: 730
```
上传完成后，可以通过浏览器打开`https://registry.0x01.party/v2/_catalog`查看仓库中的镜像，或者
```
curl -k -u admin:testpassword https://registry.0x01.party/v2/_catalog
```
在安装了 Docker 后，可以通过获取官方 registry 镜像来运行。
>参考：
>[Docker入门到实践 -- 私有仓库](https://yeasy.gitbooks.io/docker_practice/content/repository/local_repo.html)
>[Docker搭建带认证的私有仓库](http://ju.outofmemory.cn/entry/209746)
>[Docker私有仓库用户认证代理配置](http://www.aichengxu.com/view/6610008)
>[Authentication still available in Registry 2.0.0 ?](https://github.com/docker/distribution/issues/397)
>[Docker Doc -- Deploying a registry server](https://docs.docker.com/registry/deploying/)
>[在Docker容器环境中用Let's Encrypt部署HTTPS](https://www.solaluna.cn/2016/07/05/1720/)
>[Certbot Start](https://certbot.eff.org/#debianwheezy-nginx)
>[letsencrypt -- How It Works](https://letsencrypt.org/how-it-works/)