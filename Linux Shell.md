#Linux Shell
## 实用命令
tee
watch
sed
awk/gawk
### sed
替换特定行中某些字符：
sed -i '/snd-soc-wmt-fm34/s/^#//'
## 比较操作
### 整数比较
```
-eq 等于,如:if [ "$a" -eq "$b" ]
-ne 不等于,如:if [ "$a" -ne "$b" ]
-gt 大于,如:if [ "$a" -gt "$b" ]
-ge 大于等于,如:if [ "$a" -ge "$b" ]
-lt 小于,如:if [ "$a" -lt "$b" ]
-le 小于等于,如:if [ "$a" -le "$b" ]

< 小于(需要双括号),如:(("$a" < "$b"))
<= 小于等于(需要双括号),如:(("$a" <= "$b"))
> 大于(需要双括号),如:(("$a" > "$b"))
>= 大于等于(需要双括号),如:(("$a" >= "$b"))
```

### 字符串比较
```
= 等于,如:if [ "$a" = "$b" ]
== 等于,如:if [ "$a" == "$b" ],与=等价
!= 不等于

注意: [[]]和[]中的行为在某些情况下是不同的:

[[ $a == z* ]]           # 如果$a 以"z"开头(模式匹配)那么将为true
[[ $a == "z*" ]]         # 如果$a 等于z*  (字符匹配),那么结果为true

[ $a == z* ]             # File globbing 和word splitting 将会发生
[ "$a" == "z*" ]         # 如果$a 等于z* (字符匹配),那么结果为true

```
>注：File globbing 是一种关于文件的速记法， 比如“*.c”或者"~"， 但其并不是严格的正则表达式。

### 逻辑表达式
[ exp1 -a exp2 ]		# 逻辑与
[ exp1 -o exp2 ]		# 逻辑或
但在[[]]中使用&& 和|| 代替， -a 和-o 一般都是搭配test 命令或者[]
[[ exp1 && exp2 ]]		# 逻辑与
[[ exp1 || exp2 ]]		# 逻辑或

### 文件判断
```
[ ! -x "$myPath" ]       # 是否存在并有可执行权限
[ ! -d "$myPath" ]       # 是否存在
[ ! -f "$myFile" ]       # 是否存在
[ ! -n "$myVar" ]        # 是否有值

其它：
 -a file exists. 
 -b file exists and is a block special file. 
 -c file exists and is a character special file. 
 -d file exists and is a directory. 
 -e file exists (just the same as -a). 
 -f file exists and is a regular file. 
 -g file exists and has its setgid(2) bit set. 
 -G file exists and has the same group ID as this process. 
 -k file exists and has its sticky bit set. 
 -L file exists and is a symbolic link. 
 -n string length is not zero. 
 -o Named option is set on. 
 -O file exists and is owned by the user ID of this process. 
 -p file exists and is a first in, first out (FIFO) special file or 
 named pipe. 
 -r file exists and is readable by the current process. 
 -s file exists and has a size greater than zero. 
 -S file exists and is a socket. 
 -t file descriptor number fildes is open and associated with a 
 terminal device. 
 -u file exists and has its setuid(2) bit set. 
 -w file exists and is writable by the current process. 
 -x file exists and is executable by the current process. 
 -z string length is zero. 
```

## 按行读取文件
写法一
```bash
#!/bin/bash
while read line
do
echo $line
done < filename(待读取的文件)
```
写法二
```bash
#!/bin/bash
cat filename(待读取的文件) | while read line
do
echo $line
done
```
写法三
```bash
for line in `cat filename(待读取的文件)`
do
echo $line
done
```
说明：
for逐行读和while逐行读是有区别的,如:
```bash
$ cat file
1111
2222
3333 4444 555

$ cat file | while read line; do echo $line; done
1111
2222
3333 4444 555

$ for line in $(<file); do echo $line; done
1111
2222
3333
4444
555
```
##正则表达式
[[ $var =~ regex ]]
适用于bash v4.0+， 其他bash或许要给regex套上引号

##整数相加
```
V1=1
V2=2
let V3=$V1+$V2
echo $V3
echo $(($V1+$V2))
echo $[$V1+$V2]
expr $V1 + $V2
echo $V1+$V2 | bc
awk 'BEGIN{print "'$V1'"+"'$V2'"}'
```

##数组
数组长度：
```
${#arrary[@]}
```
遍历数组：
```
for i in ${array[@]} ; do ; echo $i
```
把一个数组的值赋值给另一个：
```bash

array1=("a" "b" "c")
array2=(${array1[*]})
```

[查看Debian的版本信息](http://www.cnblogs.com/lbsx/archive/2010/12/29/1920989.html)

[Shell读取用户输入 ](http://blog.csdn.net/zilong00007/article/details/6681090)

[Shell脚本学习-命令行参数处理 ](http://blog.csdn.net/li385805776/article/details/16981541)

[shell判断文件是否存在 ](http://www.cnblogs.com/sunyubo/archive/2011/10/17/2282047.html)

[linux上swap的查看与调整](http://blog.sina.com.cn/s/blog_6d6e54f701010jxt.html)

[shell下的加法](http://linux.chinaunix.net/techdoc/develop/2008/09/16/1032452.shtml)

[Shell命令：echo 命令详解](http://blog.csdn.net/felix_f/article/details/12433171)

[shell 如何实现字符串不足N位自动补零](http://www.oschina.net/question/2795_29778)

[shell判断文件是否存在](http://www.cnblogs.com/sunyubo/archive/2011/10/17/2282047.html)

[shell编程 for in 循环](http://blog.csdn.net/hainan16/article/details/6667483)

[Linux find 用法示例](http://www.cnblogs.com/wanqieddy/archive/2011/06/09/2076785.html)

[linux之sed用法](http://www.cnblogs.com/dong008259/archive/2011/12/07/2279897.html)

[linux awk命令详解](http://www.cnblogs.com/ggjucheng/archive/2013/01/13/2858470.html)

[对话 UNIX: 掌握强大的命令行](http://www.ibm.com/developerworks/cn/aix/library/es-unix-commandline/index.html)

[bash中的正则表达式](http://huntmind.blog.163.com/blog/static/133672943201038102528992/)

[linux shell bash 比较操作](http://www.2cto.com/os/201202/120793.html)

[shell按行读取文件的3种方法](http://www.jb51.net/article/48830.htm)

[bash shell字符串的截取](http://www.cnblogs.com/liuweijian/archive/2009/12/27/1633661.html)

[linux shell 字符串操作（长度，查找，替换）详解](http://www.cnblogs.com/chengmo/archive/2010/10/02/1841355.html)

[shell脚本----if（数字条件，字符串条件，字符串为空）](http://blog.csdn.net/yf210yf/article/details/9207147)

[bash shell之数组使用](http://www.linuxidc.com/Linux/2012-08/67655.htm)

[Bash函数的参数和返回值](http://blog.chinaunix.net/uid-726813-id-2060120.html)

[Linux shell 之 提取文件名和目录名的一些方法](http://blog.csdn.net/ljianhui/article/details/43128465)

[编写Shell脚本的最佳实践](http://kb.cnblogs.com/page/574767/)