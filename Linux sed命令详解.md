sed是一个很好的文件处理工具，本身是一个管道命令，主要是以行为单位进行处理，可以将数据行进行替换、删除、新增、选取等特定工作，下面先了解一下sed的用法。
sed命令行格式为：
```
Usage: sed [OPTION]... {script-only-if-no-other-script} [input-file]...
```

**常用选项：**
-n ∶使用安静(silent)模式。在一般 sed 的用法中，所有来自 STDIN的资料一般都会被列出到萤幕上。但如果加上 -n 参数后，则只有经过sed 特殊处理的那一行(或者动作)才会被列出来。
-e ∶直接在指令列模式上进行 sed 的动作编辑；
-f ∶直接将 sed 的动作写在一个档案内， -f filename 则可以执行 filename 内的sed 动作；
-r ∶sed 的动作支援的是延伸型正规表示法的语法。(预设是基础正规表示法语法)
-i ∶直接修改读取的档案内容，而不是由萤幕输出。

**常用命令：**
a ∶新增， a 的后面可以接字串，而这些字串会在新的一行出现(目前的下一行)～
c ∶取代， c 的后面可以接字串，这些字串可以取代 n1,n2 之间的行！
d ∶删除，因为是删除啊，所以 d 后面通常不接任何咚咚；
i ∶插入， i 的后面可以接字串，而这些字串会在新的一行出现(目前的上一行)；
p ∶列印，亦即将某个选择的资料印出。通常 p 会与参数 sed -n 一起运作～
s ∶取代，可以直接进行取代的工作哩！通常这个 s 的动作可以搭配正规表示法！例如 1,20s/old/new/g 就是啦！

**举例：**（假设我们有一文件名为ab）
删除某行
```bash
[root@localhost ruby] # sed '1d' ab              #删除第一行
[root@localhost ruby] # sed '$d' ab              #删除最后一行
[root@localhost ruby] # sed '1,2d' ab           #删除第一行到第二行
[root@localhost ruby] # sed '2,$d' ab           #删除第二行到最后一行
```

显示某行
```bash
[root@localhost ruby] # sed -n '1p' ab           #显示第一行
[root@localhost ruby] # sed -n '$p' ab           #显示最后一行
[root@localhost ruby] # sed -n '1,2p' ab        #显示第一行到第二行
[root@localhost ruby] # sed -n '2,$p' ab        #显示第二行到最后一行
```

使用模式进行查询
```bash
[root@localhost ruby] # sed -n '/ruby/p' ab    #查询包括关键字ruby所在所有行
[root@localhost ruby] # sed -n '/\$/p' ab        #查询包括关键字$所在所有行，使用反斜线\屏蔽特殊含义
[root@localhost ruby] # sed -n '/16\/Feb\/2016/,/17\/Feb\/2016/p' access_log>20160216.log #列出nginx在某时间段的日志
```

增加一行或多行字符串
```bash
[root@localhost ruby]# cat ab
Hello!
ruby is me,welcome to my blog.
end
[root@localhost ruby] # sed '1a drink tea' ab  #第一行后增加字符串"drink tea"
Hello!
drink tea
ruby is me,welcome to my blog.
end
[root@localhost ruby] # sed '1,3a drink tea' ab #第一行到第三行后增加字符串"drink tea"
Hello!
drink tea
ruby is me,welcome to my blog.
drink tea
end
drink tea
[root@localhost ruby] # sed '1a drink tea\nor coffee' ab   #第一行后增加多行，使用换行符\n
Hello!
drink tea
or coffee
ruby is me,welcome to my blog.
end
```
代替一行或多行
```bash
[root@localhost ruby] # sed '1c Hi' ab                #第一行代替为Hi
Hi
ruby is me,welcome to my blog.
end
[root@localhost ruby] # sed '1,2c Hi' ab             #第一行到第二行代替为Hi
Hi
end
```

替换一行中的某部分
格式：sed 's/要替换的字符串/新的字符串/g'   （要替换的字符串可以用正则表达式）
```bash
[root@localhost ruby] # sed -n '/ruby/p' ab | sed 's/ruby/bird/g'    #替换ruby为bird
[root@localhost ruby] # sed -n '/ruby/p' ab | sed 's/ruby//g'        #删除ruby

插入
[root@localhost ruby] # sed -i '$a bye' ab         #在文件ab中最后一行直接输入"bye"
[root@localhost ruby]# cat ab
Hello!
ruby is me,welcome to my blog.
end
bye
```

删除匹配行
```bash
sed -i '/匹配字符串/d'  filename  #若匹配字符串是变量，则需要“”，而不是‘’。
```
替换匹配行中的某个字符串
```bash
sed -i '/匹配字符串/s/替换源字符串/替换目标字符串/g' filename
```

### 正则元字符
- ^ 行首定位符， `/^my/` 匹配所有以my开头的行
- $ 行尾定位符， `/my$/` 匹配所有以my结尾的行
- . 匹配除换行符以外的单个字符， `/m...y/` 匹配包含字母m, 后跟两个任意字符，再跟字母y的行
- \* 匹配零个或多个前导字符， `/my*/` 匹配包含字母m，后跟零个或多个y字母的行
- [] 匹配指定字符组内的任一字符， `/[Mn]y/` 匹配包含My或my的行
- [^] 匹配不在指定字符组内的任一字符， `/[^Mm]]y/` 匹配包含y，但y之前的那个字符不是M或m的行
- (...) 匹配已包含的字符， `1,30s/(you)self/\1r` 标记元字符之间的模式，并将其保存为标签1， 之后可以使用\1来引用它
- & 保存查找串以便在替换串中引用， `s/my/&/` 符号&代表查找串。my将被替换为my
- < 词首定位符， `/<my/` 匹配包含以my开头的单词的行
- \> 词尾定位符， `/my>/` 匹配包含以my结尾的单词的行
- x{m} 连续m个x， `/9{5}/` 匹配包含连续5个9的行
- x{m,} 至少m个x， `/9{5,}/` 匹配包含至少连续5个9的行
- x{m,n} 至少m个，但不超过n个x， `,7}/` 匹配包含连续5到7个9的行



