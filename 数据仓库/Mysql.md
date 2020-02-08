**Linux下数据库导入**

```
导出数据和表结构：
mysqldump -u用户名 -p 数据库名 > 数据库名.sql
/usr/local/mysql/bin/mysqldump -uroot -p abc > abc.sql
敲回车后会提示输入密码

只导出表结构
mysqldump -u用户名 -p -d 数据库名 > 数据库名.sql
/usr/local/mysql/bin/mysqldump -uroot -p -d abc > abc.sql



导入数据库
1、首先建空数据库
mysql>create database abc;

2、导入数据库
（1）选择数据库
mysql>use abc;
（2）设置数据库编码
mysql>set names utf8;
（3）导入数据（注意sql文件的路径）
mysql>source /home/abc/abc.sql;

或者
mysql -u用户名 -p 数据库名 < 数据库名.sql
mysql -uabc_f -p abc < abc.sql

```

**1045错误**

```
my.init 远程访问端口没有配置

```

