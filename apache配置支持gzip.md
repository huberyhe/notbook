## apache 配置支持gzip
apache使用gzip压缩能够大幅提高网站访问速度并节省网络流量，在网页响应头信息中可以判断是否支持压缩。

---
HTTP/1.1 200 OK
Date: Wed, 14 Dec 2016 04:33:08 GMT
Content-Type: text/html; charset=utf-8
Transfer-Encoding: chunked
Connection: keep-alive
Vary: Accept-Encoding
Cache-Control: private, max-age=10
Expires: Wed, 14 Dec 2016 04:33:18 GMT
Last-Modified: Wed, 14 Dec 2016 04:33:08 GMT
X-UA-Compatible: IE=10
**Content-Encoding: gzip**

---

### 1.安装动态库
gzip需要mod_deflate.so和mod_headers.so， 但是不像系统自带的apache，后来安装的apache是没有带这些库的。如果自带库，请直接跳到第二步。
```bash
[hubery@localhost ~]$ ls -l /etc/httpd/modules/
total 5140
-rwxr-xr-x 1 root root 3699056 8月  12 04:36 libphp5.so
...
-rwxr-xr-x 1 root root   27056 11月 19 07:49 mod_deflate.so
-rwxr-xr-x 1 root root   10416 11月 19 07:49 mod_dir.so
-rwxr-xr-x 1 root root   22992 11月 19 07:49 mod_disk_cache.so
-rwxr-xr-x 1 root root   10448 11月 19 07:49 mod_dumpio.so
-rwxr-xr-x 1 root root   10416 11月 19 07:49 mod_env.so
-rwxr-xr-x 1 root root   10512 11月 19 07:49 mod_expires.so
-rwxr-xr-x 1 root root   23120 11月 19 07:49 mod_ext_filter.so
-rwxr-xr-x 1 root root   18736 11月 19 07:49 mod_filter.so
-rwxr-xr-x 1 root root   22992 11月 19 07:49 mod_headers.so
...

[hubery@localhost ~]$ ls /usr/local/apache2/modules/
httpd.exp  libphp5.so  mod_deflate.so  mod_headers.so

```
##### 安装mod_headers
```bash
[root@localhost metadata]# pwd
/home/hubery/httpd-2.2.29/modules/metadata
[root@localhost metadata]# /usr/local/apache2/bin/apxs -i -a -c mod_headers.c
/usr/local/apache2/build/libtool --silent --mode=compile gcc -prefer-pic   -DLINUX -D_REENTRANT -D_GNU_SOURCE -g -O2 -pthread -I/usr/local/apache2/include  -I/usr/local/apache2/include   -I/usr/local/apache2/include   -c -o mod_headers.lo mod_headers.c && touch mod_headers.slo
/usr/local/apache2/build/libtool --silent --mode=link gcc -o mod_headers.la  -rpath /usr/local/apache2/modules -module -avoid-version    mod_headers.lo
/usr/local/apache2/build/instdso.sh SH_LIBTOOL='/usr/local/apache2/build/libtool' mod_headers.la /usr/local/apache2/modules
/usr/local/apache2/build/libtool --mode=install cp mod_headers.la /usr/local/apache2/modules/
cp .libs/mod_headers.so /usr/local/apache2/modules/mod_headers.so
cp .libs/mod_headers.lai /usr/local/apache2/modules/mod_headers.la
cp .libs/mod_headers.a /usr/local/apache2/modules/mod_headers.a
chmod 644 /usr/local/apache2/modules/mod_headers.a
ranlib /usr/local/apache2/modules/mod_headers.a
PATH="$PATH:/sbin" ldconfig -n /usr/local/apache2/modules
----------------------------------------------------------------------
Libraries have been installed in:
   /usr/local/apache2/modules

If you ever happen to want to link against installed libraries
in a given directory, LIBDIR, you must either use libtool, and
specify the full pathname of the library, or use the `-LLIBDIR'
flag during linking and do at least one of the following:
   - add LIBDIR to the `LD_LIBRARY_PATH' environment variable
     during execution
   - add LIBDIR to the `LD_RUN_PATH' environment variable
     during linking
   - use the `-Wl,--rpath -Wl,LIBDIR' linker flag
   - have your system administrator add LIBDIR to `/etc/ld.so.conf'

See any operating system documentation about shared libraries for
more information, such as the ld(1) and ld.so(8) manual pages.
----------------------------------------------------------------------
chmod 755 /usr/local/apache2/modules/mod_headers.so
[activating module `headers' in /usr/local/apache2/conf/httpd.conf]
```
##### 安装mod_deflate
```bash
[root@localhost filters]# pwd
/home/hubery/httpd-2.2.29/modules/filters
[root@localhost filters]# /usr/local/apache2/bin/apxs -i -c -a mod_deflate.c
/usr/local/apache2/build/libtool --silent --mode=compile gcc -prefer-pic   -DLINUX -D_REENTRANT -D_GNU_SOURCE -g -O2 -pthread -I/usr/local/apache2/include  -I/usr/local/apache2/include   -I/usr/local/apache2/include   -c -o mod_deflate.lo mod_deflate.c && touch mod_deflate.slo
/usr/local/apache2/build/libtool --silent --mode=link gcc -o mod_deflate.la  -rpath /usr/local/apache2/modules -module -avoid-version    mod_deflate.lo
/usr/local/apache2/build/instdso.sh SH_LIBTOOL='/usr/local/apache2/build/libtool' mod_deflate.la /usr/local/apache2/modules
/usr/local/apache2/build/libtool --mode=install cp mod_deflate.la /usr/local/apache2/modules/
cp .libs/mod_deflate.so /usr/local/apache2/modules/mod_deflate.so
cp .libs/mod_deflate.lai /usr/local/apache2/modules/mod_deflate.la
cp .libs/mod_deflate.a /usr/local/apache2/modules/mod_deflate.a
chmod 644 /usr/local/apache2/modules/mod_deflate.a
ranlib /usr/local/apache2/modules/mod_deflate.a
PATH="$PATH:/sbin" ldconfig -n /usr/local/apache2/modules
----------------------------------------------------------------------
Libraries have been installed in:
   /usr/local/apache2/modules

If you ever happen to want to link against installed libraries
in a given directory, LIBDIR, you must either use libtool, and
specify the full pathname of the library, or use the `-LLIBDIR'
flag during linking and do at least one of the following:
   - add LIBDIR to the `LD_LIBRARY_PATH' environment variable
     during execution
   - add LIBDIR to the `LD_RUN_PATH' environment variable
     during linking
   - use the `-Wl,--rpath -Wl,LIBDIR' linker flag
   - have your system administrator add LIBDIR to `/etc/ld.so.conf'

See any operating system documentation about shared libraries for
more information, such as the ld(1) and ld.so(8) manual pages.
----------------------------------------------------------------------
chmod 755 /usr/local/apache2/modules/mod_deflate.so
[activating module `deflate' in /usr/local/apache2/conf/httpd.conf]
```
### 2.配置支持
#####去掉注释或添加下面两段。
```bash
[hubery@localhost ~]$ cat /usr/local/apache2/conf/httpd.conf
#...
LoadModule deflate_module     modules/mod_deflate.so
LoadModule headers_module     modules/mod_headers.so
#...
<Location />
AddOutputFilterByType DEFLATE text/plain
AddOutputFilterByType DEFLATE text/css
AddOutputFilterByType DEFLATE application/xml
AddOutputFilterByType DEFLATE application/rss+xml
AddOutputFilterByType DEFLATE application/atom_xml
AddOutputFilterByType DEFLATE application/x-javascript
AddOutputFilterByType DEFLATE application/x-httpd-php
AddOutputFilterByType DEFLATE text/html
</Location>
#...
```
##### 重启apache2
`[hubery@localhost ~]$ sudo /usr/local/apache2/bin/httpd -k restart`
##### 检查模块是否加载
```bash
[hubery@localhost ~]$ /usr/local/apache2/bin/apachectl -t -D DUMP_MODULES
Loaded Modules:
 core_module (static)
 authn_file_module (static)
 authn_default_module (static)
 authz_host_module (static)
 authz_groupfile_module (static)
 authz_user_module (static)
 authz_default_module (static)
 auth_basic_module (static)
 include_module (static)
 filter_module (static)
 log_config_module (static)
 env_module (static)
 setenvif_module (static)
 version_module (static)
 ssl_module (static)
 mpm_prefork_module (static)
 http_module (static)
 mime_module (static)
 status_module (static)
 autoindex_module (static)
 asis_module (static)
 cgi_module (static)
 negotiation_module (static)
 dir_module (static)
 actions_module (static)
 userdir_module (static)
 alias_module (static)
 rewrite_module (static)
 so_module (static)
 php5_module (shared)
 deflate_module (shared)
 headers_module (shared)
Syntax OK
```

>参考：
>[CentOS Apache 如何开启 Gzip 开启](http://linux.it.net.cn/CentOS/config/2014/1030/7468.html)
>[linux下apache安装gzip压缩(Deflate模块)](http://www.cnblogs.com/wrmfw/archive/2011/09/05/2166945.html)