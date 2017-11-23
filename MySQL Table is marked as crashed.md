MYSQL数据表出现问题，提示：
*ERROR 144 (HY000): Table './dpt/dpt_production' is marked as crashed and last (automatic?) repair failed*

修复数据表操作：
```bash
service mysqld stop;
cd /var/lib/mysql/dpt
myisamchk -r dpt_production.MYI
mysqlcheck -o -r dpt -uroot -p
```

> 注意：操作第三步前一定要把mysql服务停掉。
