##pt-table-checksum(percona-toolkit)的安装与使用
###CentOS
```bash
sudo yum install -y perl perl-IO-Socket-SSL perl-Time-HiRes perl-TermReadKey
wget percona.com/get/percona-toolkit.rpm
sudo rpm -ivh percona-toolkit.rpm
```
###Debian
```bash
apt-get install percona-toolkit
```
##一致性校验
1.保证从库的同步IO和SQL进程是YES状态
```mysql
mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 46.166.129.214
                  Master_User: dbsync
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-log-bin.000013
          Read_Master_Log_Pos: 69254175
               Relay_Log_File: mysql-relay-log.000002
                Relay_Log_Pos: 546
        Relay_Master_Log_File: mysql-log-bin.000013
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: iksdb,iksdb2,iksdb3,test
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 69254175
              Relay_Log_Space: 701
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
1 row in set (0.00 sec)

```
2.创建用户, 需要同步复制权限和创建权限（pt-table-checksum 会创建一个表用来记录结果）
```mysql
##mysql 1
mysql> GRANT CREATE, INSERT, UPDATE, DELETE, DROP, SELECT, PROCESS, SUPER, REPLICATION SLAVE ON *.* TO 'checksums'@'178.62.234.75' IDENTIFIED BY 'MY-checksums-user';
Query OK, 0 rows affected (0.01 sec)

mysql> GRANT CREATE, INSERT, UPDATE, DELETE, DROP, SELECT, PROCESS, SUPER, REPLICATION SLAVE ON *.* TO 'checksums'@'localhost' IDENTIFIED BY 'MY-checksums-user';
Query OK, 0 rows affected (0.03 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

##mysql 2
mysql> GRANT CREATE, INSERT, UPDATE, DELETE, DROP, SELECT, PROCESS, SUPER, REPLICATION SLAVE ON *.* TO 'checksums'@'46.166.129.214' IDENTIFIED BY 'MY-checksums-user';
Query OK, 0 rows affected (0.00 sec)

mysql> GRANT CREATE, INSERT, UPDATE, DELETE, DROP, SELECT, PROCESS, SUPER, REPLICATION SLAVE ON *.* TO 'checksums'@'localhost' IDENTIFIED BY 'MY-checksums-user';
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```
3.执行检查，双向热备份的可以两边各一次
```bash
[hubery@vvshare ~]$ pt-table-checksum --databases=iksdb3 --tables=recharge_history --replicate-check h=46.166.129.214,u=checksums,p='MY-checksums-user',P=3306 --set-vars innodb_lock_wait_timeout=50 --no-check-binlog-format --no-check-replication-filters --replicate=mydb.checksums
            TS ERRORS  DIFFS     ROWS  CHUNKS SKIPPED    TIME TABLE
09-30T04:12:50      0      1       64       1       0   0.042 iksdb3.recharge_history
```
参数说明：
TS ：完成检查的时间。
ERRORS ：检查时候发生错误和警告的数量。
DIFFS ：0表示一致，1表示不一致。当指定–no-replicate-check时，会一直为0，当指定–replicate-check-only会显示不同的信息。
ROWS ：表的行数。
CHUNKS ：被划分到表中的块的数目。
SKIPPED ：由于错误或警告或过大，则跳过块的数目。
TIME ：执行的时间。
TABLE ：被检查的表名。

参数意义：
–nocheck-replication-filters ：不检查复制过滤器，建议启用。后面可以用–databases来指定需要检查的数据库。
–no-check-binlog-format : 不检查复制的binlog模式，要是binlog模式是ROW，则会报错。
–replicate-check-only :只显示不同步的信息。
–replicate= ：把checksum的信息写入到指定表中，建议直接写到被检查的数据库当中。
–databases= ：指定需要被检查的数据库，多个则用逗号隔开。
–tables= ：指定需要被检查的表，多个用逗号隔开
h=127.0.0.1 ：Master的地址
u=root ：用户名
p=123456 ：密码
P=3306 ：端口

##根据校验结果执行同步
1.先查看执行同步需要的操作，不会真正执行
```bash
[hubery@vvshare ~]$ pt-table-sync --print --replicate=mydb.checksums h=127.0.0.1,u=checksums,p='MY-checksums-user',P=3306 h=46.166.129.214,u=checksums,p='MY-checksums-user',P=3306
DELETE FROM `iksdb3`.`recharge_history` WHERE `id`='96' LIMIT 1 /*percona-toolkit src_db:iksdb3 src_tbl:recharge_history src_dsn:P=3306,h=127.0.0.1,p=...,u=checksums dst_db:iksdb3 dst_tbl:recharge_history dst_dsn:P=3306,h=46.166.129.214,p=...,u=checksums lock:1 transaction:1 changing_src:mydb.checksums replicate:mydb.checksums bidirectional:0 pid:11036 user:hubery host:vvshare.info*/;
DELETE FROM `iksdb3`.`recharge_history` WHERE `id`='112' LIMIT 1 /*percona-toolkit src_db:iksdb3 src_tbl:recharge_history src_dsn:P=3306,h=127.0.0.1,p=...,u=checksums dst_db:iksdb3 dst_tbl:recharge_history dst_dsn:P=3306,h=46.166.129.214,p=...,u=checksums lock:1 transaction:1 changing_src:mydb.checksums replicate:mydb.checksums bidirectional:0 pid:11036 user:hubery host:vvshare.info*/;
DELETE FROM `iksdb3`.`recharge_history` WHERE `id`='114' LIMIT 1 /*percona-toolkit src_db:iksdb3 src_tbl:recharge_history src_dsn:P=3306,h=127.0.0.1,p=...,u=checksums dst_db:iksdb3 dst_tbl:recharge_history dst_dsn:P=3306,h=46.166.129.214,p=...,u=checksums lock:1 transaction:1 changing_src:mydb.checksums replicate:mydb.checksums bidirectional:0 pid:11036 user:hubery host:vvshare.info*/;
DELETE FROM `iksdb3`.`recharge_history` WHERE `id`='116' LIMIT 1 /*percona-toolkit src_db:iksdb3 src_tbl:recharge_history src_dsn:P=3306,h=127.0.0.1,p=...,u=checksums dst_db:iksdb3 dst_tbl:recharge_history dst_dsn:P=3306,h=46.166.129.214,p=...,u=checksums lock:1 transaction:1 changing_src:mydb.checksums replicate:mydb.checksums bidirectional:0 pid:11036 user:hubery host:vvshare.info*/;
DELETE FROM `iksdb3`.`recharge_history` WHERE `id`='118' LIMIT 1 /*percona-toolkit src_db:iksdb3 src_tbl:recharge_history src_dsn:P=3306,h=127.0.0.1,p=...,u=checksums dst_db:iksdb3 dst_tbl:recharge_history dst_dsn:P=3306,h=46.166.129.214,p=...,u=checksums lock:1 transaction:1 changing_src:mydb.checksums replicate:mydb.checksums bidirectional:0 pid:11036 user:hubery host:vvshare.info*/;
DELETE FROM `iksdb3`.`recharge_history` WHERE `id`='120' LIMIT 1 /*percona-toolkit src_db:iksdb3 src_tbl:recharge_history src_dsn:P=3306,h=127.0.0.1,p=...,u=checksums dst_db:iksdb3 dst_tbl:recharge_history dst_dsn:P=3306,h=46.166.129.214,p=...,u=checksums lock:1 transaction:1 changing_src:mydb.checksums replicate:mydb.checksums bidirectional:0 pid:11036 user:hubery host:vvshare.info*/;
DELETE FROM `iksdb3`.`recharge_history` WHERE `id`='122' LIMIT 1 /*percona-toolkit src_db:iksdb3 src_tbl:recharge_history src_dsn:P=3306,h=127.0.0.1,p=...,u=checksums dst_db:iksdb3 dst_tbl:recharge_history dst_dsn:P=3306,h=46.166.129.214,p=...,u=checksums lock:1 transaction:1 changing_src:mydb.checksums replicate:mydb.checksums bidirectional:0 pid:11036 user:hubery host:vvshare.info*/;
REPLACE INTO `iksdb3`.`recharge_history`(`id`, `card_id`, `user_id`, `recharge_time`, `ip`) VALUES ('115', '2215', '2201079', '2016-09-05 03:58:38', '/220.162.97.38:50739') /*percona-toolkit src_db:iksdb3 src_tbl:recharge_history src_dsn:P=3306,h=127.0.0.1,p=...,u=checksums dst_db:iksdb3 dst_tbl:recharge_history dst_dsn:P=3306,h=46.166.129.214,p=...,u=checksums lock:1 transaction:1 changing_src:mydb.checksums replicate:mydb.checksums bidirectional:0 pid:11036 user:hubery host:vvshare.info*/;
```
先MASTER的IP，再SLAVE的IP；同步可以是单向的，也可以是双向的，这里是单向

 参数的意义：
--replicate=  ：指定通过pt-table-checksum得到的表，这2个工具差不多都会一直用。
--databases=  : 指定执行同步的数据库，多个用逗号隔开。
--tables=     ：指定执行同步的表，多个用逗号隔开。
--sync-to-master ：指定一个DSN，即从的IP，他会通过show processlist或show slave status 去自动的找主。
h=127.0.0.1   ：服务器地址，命令里有2个ip，第一次出现的是M的地址，第2次是Slave的地址。
u=root        ：帐号。
p=123456      ：密码。
--print       ：打印，但不执行命令。
--execute     ：执行命令。


2.确认正确后执行
```bash
[hubery@vvshare ~]$ pt-table-sync --print --replicate=mydb.checksums h=127.0.0.1,u=checksums,p='MY-checksums-user',P=3306 h=46.166.129.214,u=checksums,p='MY-checksums-user',P=3306 --execute
DELETE FROM `iksdb3`.`recharge_history` WHERE `id`='96' LIMIT 1 /*percona-toolkit src_db:iksdb3 src_tbl:recharge_history src_dsn:P=3306,h=127.0.0.1,p=...,u=checksums dst_db:iksdb3 dst_tbl:recharge_history dst_dsn:P=3306,h=46.166.129.214,p=...,u=checksums lock:1 transaction:1 changing_src:mydb.checksums replicate:mydb.checksums bidirectional:0 pid:11046 user:hubery host:vvshare.info*/;
DELETE FROM `iksdb3`.`recharge_history` WHERE `id`='112' LIMIT 1 /*percona-toolkit src_db:iksdb3 src_tbl:recharge_history src_dsn:P=3306,h=127.0.0.1,p=...,u=checksums dst_db:iksdb3 dst_tbl:recharge_history dst_dsn:P=3306,h=46.166.129.214,p=...,u=checksums lock:1 transaction:1 changing_src:mydb.checksums replicate:mydb.checksums bidirectional:0 pid:11046 user:hubery host:vvshare.info*/;
DELETE FROM `iksdb3`.`recharge_history` WHERE `id`='114' LIMIT 1 /*percona-toolkit src_db:iksdb3 src_tbl:recharge_history src_dsn:P=3306,h=127.0.0.1,p=...,u=checksums dst_db:iksdb3 dst_tbl:recharge_history dst_dsn:P=3306,h=46.166.129.214,p=...,u=checksums lock:1 transaction:1 changing_src:mydb.checksums replicate:mydb.checksums bidirectional:0 pid:11046 user:hubery host:vvshare.info*/;
DELETE FROM `iksdb3`.`recharge_history` WHERE `id`='116' LIMIT 1 /*percona-toolkit src_db:iksdb3 src_tbl:recharge_history src_dsn:P=3306,h=127.0.0.1,p=...,u=checksums dst_db:iksdb3 dst_tbl:recharge_history dst_dsn:P=3306,h=46.166.129.214,p=...,u=checksums lock:1 transaction:1 changing_src:mydb.checksums replicate:mydb.checksums bidirectional:0 pid:11046 user:hubery host:vvshare.info*/;
DELETE FROM `iksdb3`.`recharge_history` WHERE `id`='118' LIMIT 1 /*percona-toolkit src_db:iksdb3 src_tbl:recharge_history src_dsn:P=3306,h=127.0.0.1,p=...,u=checksums dst_db:iksdb3 dst_tbl:recharge_history dst_dsn:P=3306,h=46.166.129.214,p=...,u=checksums lock:1 transaction:1 changing_src:mydb.checksums replicate:mydb.checksums bidirectional:0 pid:11046 user:hubery host:vvshare.info*/;
DELETE FROM `iksdb3`.`recharge_history` WHERE `id`='120' LIMIT 1 /*percona-toolkit src_db:iksdb3 src_tbl:recharge_history src_dsn:P=3306,h=127.0.0.1,p=...,u=checksums dst_db:iksdb3 dst_tbl:recharge_history dst_dsn:P=3306,h=46.166.129.214,p=...,u=checksums lock:1 transaction:1 changing_src:mydb.checksums replicate:mydb.checksums bidirectional:0 pid:11046 user:hubery host:vvshare.info*/;
DELETE FROM `iksdb3`.`recharge_history` WHERE `id`='122' LIMIT 1 /*percona-toolkit src_db:iksdb3 src_tbl:recharge_history src_dsn:P=3306,h=127.0.0.1,p=...,u=checksums dst_db:iksdb3 dst_tbl:recharge_history dst_dsn:P=3306,h=46.166.129.214,p=...,u=checksums lock:1 transaction:1 changing_src:mydb.checksums replicate:mydb.checksums bidirectional:0 pid:11046 user:hubery host:vvshare.info*/;
REPLACE INTO `iksdb3`.`recharge_history`(`id`, `card_id`, `user_id`, `recharge_time`, `ip`) VALUES ('115', '2215', '2201079', '2016-09-05 03:58:38', '/220.162.97.38:50739') /*percona-toolkit src_db:iksdb3 src_tbl:recharge_history src_dsn:P=3306,h=127.0.0.1,p=...,u=checksums dst_db:iksdb3 dst_tbl:recharge_history dst_dsn:P=3306,h=46.166.129.214,p=...,u=checksums lock:1 transaction:1 changing_src:mydb.checksums replicate:mydb.checksums bidirectional:0 pid:11046 user:hubery host:vvshare.info*/;
```

3.验证结果
```bash
[hubery@vvshare ~]$ pt-table-checksum --databases=iksdb3 --tables=recharge_history --replicate-check h=46.166.129.214,u=checksums,p='MY-checksums-user',P=3306 --set-vars innodb_lock_wait_timeout=50 --no-check-binlog-format --no-check-replication-filters --replicate=mydb.checksums
            TS ERRORS  DIFFS     ROWS  CHUNKS SKIPPED    TIME TABLE
09-30T04:14:43      0      0       58       1       0   0.038 iksdb3.recharge_history
```
>参考：
>[percona-toolkit工具安装](http://www.cnblogs.com/adba/p/5279551.html)
>[Percona Toolkit for MySQL](https://www.percona.com/software/database-tools/percona-toolkit)
>[[linux] CentOS 下安装 percona-toolkit](https://mozillazg.com/2014/03/centos-how-to-install-percona-toolkit.html)
>[MySQL主从一致性检查和修复工具-----pt-tables-sum pt-table-sync](http://blog.csdn.net/cug_jiang126com/article/details/41327181)
>[MySQL备份与恢复之保证数据一致性(5)](www.aspku.com/database/mysql/60867.html)
>[用pt-table-checksum校验数据一致性](http://www.linuxidc.com/Linux/2014-11/109851p2.htm)
>[mysql_tools_pt_percona_tookits](http://www.cnblogs.com/wyeat/p/mysql_tools_percona_toolkits.html)
>[MySQL主从失败 错误Got fatal error 1236解决方法](http://www.linuxidc.com/Linux/2012-02/54729.htm)
>[mysql slave不能同步Last_SQL_Error: Error ‘Duplicate entry ‘](www.th7.cn/db/mysql/201503/95961.shtml)
>[mysql主备 Last_Errno: 1146错误](http://bbs.chinaunix.net/archiver/tid-1251290.html)
