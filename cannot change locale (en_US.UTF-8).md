#bash: warning: setlocale: LC_ALL: cannot change locale (en_US.UTF-8)

###Q:
```bash
hubery@roaster:~$ locale
locale: Cannot set LC_ALL to default locale: No such file or directory
LANG=en_US.UTF-8
LANGUAGE=
LC_CTYPE="en_US.UTF-8"
LC_NUMERIC=zh_CN.UTF-8
LC_TIME=zh_CN.UTF-8
LC_COLLATE="en_US.UTF-8"
LC_MONETARY=zh_CN.UTF-8
LC_MESSAGES="en_US.UTF-8"
LC_PAPER=zh_CN.UTF-8
LC_NAME=zh_CN.UTF-8
LC_ADDRESS=zh_CN.UTF-8
LC_TELEPHONE=zh_CN.UTF-8
LC_MEASUREMENT=zh_CN.UTF-8
LC_IDENTIFICATION=zh_CN.UTF-8
LC_ALL=
```
###A:
```bash
sudo apt-get update
sudo apt-get install locales -y
sudo locale-gen en_US.UTF-8
sudo dpkg-reconfigure locales

sudo update-locale LC_ALL=en_US.UTF-8
#或者手动编辑
hubery@roaster:~$ cat /etc/default/locale
#LANG="en_IN"(old locale)
#LANGUAGE="en_IN:en"(old setting)
LC_ALL=en_US.UTF-8
LANG=en_US.UTF-8
```

>参考：
>[bash: warning: setlocale: LC_ALL: cannot change locale (en_US) ](https://blogs.oracle.com/sakshijain/entry/the_problem_of_setting_locale)