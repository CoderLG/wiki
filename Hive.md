### Base

```
后台的端口
netstat -npl | grep java

==========================================================
北风网		北风网		北风网		北风网		看看
==========================================================
start-daf.sh
start-yarn.sh
bin/hive


bin/hive -help
bin/hive -e "select * ......"
set ....; 
set ....

desc function extended upper;
select id ,upper(name) uname from db_hive.student;

vi hives.sql
	select * .... ;	
bin/hive -f /opt/datas/hived.sql(自己写的脚本)

bin/hive -f /opt/datas/hived.sql(自己写的脚本) 
> /opt/datas/hivef-res.txt（所写到的文件）

在外执行直接生成可读文件
bin/hive -e "select * from default.emp;" > /opt/datas/exp_res.txt

直接操作hdfs
dfs -ls /
直接操作本地文件系统
!ls /opt/datas


直接操作hdfs无需加''!!!
dfs -put /opt/datas/emp.ext /user/jack/hive/warehouse/test/

show tables;
show databases;
show databases like 'db_hive*';

msck repair table dept_part;
alter table dept_part add partition(day='20150313');

//查看表的分区数
show partitions dept_part;

truncate table dept_cats;

//重新命名一个表的名称
alter table dept_like rename to dept_like_rename;

drop table if exists dept_like_rename;

desc fotmatted dept;

desc database db_hive_03;
// 更详细的信息
desc database extended db_hive_03;

drop database db_hive_03;
//如果有表的话
drop database db_hive_03 cascade;
drop database if exists db_hive_03;

===============================================================

create external table if not exists dept(
id int,
name string,
local string
)
partitioned by (week string)
row format delimited
fields terminated by '\t';
location '/user/jack/hive/warehouse/test/emp';


create external table if not exists emp(
id int,
name string,
job string,
likes array<string>,
address map<string,string>,
hiredate string,
sal double,
mgr int,
deptid int
)
partitioned by (week string)
row format delimited
fields terminated by '\t'
collection items terminated by '-'
map keys terminated by ':'
location '/user/jack/hive/warehouse/test/emp';

//增加分区
ALTER TABLE psn5 ADD PARTITION (sex='weizhi');

//删除分区  外部表的内容还在
ALTER TABLE psn5 DROP PARTITION (sex='weizhi');
ALTER TABLE psn5 DROP IF EXISTS PARTITION(year = 2015, month = 10, day = 1);

load data [local] inpath 'filepath' [overwrote] into table lename [partition (..=..,...)];

create table join_table as
select e.id empif, e.name empname, e.sal empsal, d.id deptid from 
emp e
join
dept d
on e.deptid = d.id;

insert into table emp3s_like select * from emp3;

求每个部门的平均薪水大于5000的部门

order by
对全局数据的一个排序,仅仅只有一个reduce


放在hdfs系统上 上级目录必须有 最后一级目录可自动创建
hive中“ 和 ‘ 通用，就目前所知

insert overwrite local directory '/opt/datas/leo_emp_res' 
select * from emp distribute by deptid sort by empid asc;

insert overwrite local directory '/opt/datas/hive_tes.txt'
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
select * from test.emp3;

=========大写改小写UDF编程===============
show functions;
desc function extended split;

jar
Export 打包成jar

添加jar包
add jar /opt/datas/hiveudf.jar;
创建函数  名字中带有大写字母也会改成小写
create temporary function my_Lower as "com.bf.hive.udf.Demo";

com.my.udf.Lower
com.bf.hive.udf.TimeTransform

show function
select ename, my_lower(ename) lowername from emp limit 5;

上传到hdfs长期使用
dfs -mkdir -p /user/beifeng/hive/jars;
dfs -put /opt/datas/hiveu.jar /user/beifeng/hive/jars/;

create  function selflowers as 'com.bf.hive.udf.Demo' using jar 'hdfs://hadoop.ldedu.com:8020/user/jack/jars/myHiveHDF.jar';
//show functions  找不到该函数但是能用

create temporary function myselflowers as 'com.bf.hive.udf.Demo' using jar 'hdfs://hadoop.ldedu.com:8020/user/jack/jars/myHiveHDF.jar';
//temporary 同样能用

select ename, myself_lower(ename) lowername from emp limit 5;

=========大写改小写UDF编程======END=========



========hive的常见优化============
	1.数据的存储格式
	2.数据的压缩
	3.优化sql语句
	4.外部表 分区表
	5.大表拆小表 从小表中查 耗费的资源会减少
		set parquet.compression = snappy;
		create table if not exists default.dept_cats as select * from dept;
	6.fetch Task 设置不跑mapreduce  直接抓取显示
	7.设置reduce的数目
	8.设置并行执行
		job1  a join b ----> cc
		job2  d join e ----> ff
		job3  cc join ff --> gg
		job1 和 job2 可以并行执行
	9.jvm 重用
		一个jvm 可运行多个map/reduce 任务
	10.	关闭推测执行
		每个任务都会有多套mepreduce 那个想执行完用那个结果
	11.动态分区
		可设置每天自动分区

	join

	12.优化hql执行计划（只是知道可以这样做）
		ecplain sql语句 ---->可查看 执行过程
	13.严格模式
		对我们的sql语句进行检查，若分区表 where查询的时候没有指定patition（分区的话会报错）
========hive的常见优化==结束==========


============数据 分析实战==================
	1.创建原表    ..内部表就行
	2.针对不同的业务穿件不同的子表
	3.数据的存储格式 orc
	4.数据的压缩 snappy
	5.map output 数据压缩snappy
	6.外部表、分区表
127.0.0.1 - - [31/Mar/2018:15:09:34 +0800] "GET /JspLibrary/login.jsp HTTP/1.1" 404 991
127.0.0.1 - - [31/Mar/2018:15:09:36 +0800] "GET /JspLibrary/login.jsp HTTP/1.1" 404 991

create table log(
host string,
identity string,
t_user string,
time string,
request string,
referer string,
agent string
)
row format serde 'org.apache.hadoop.hive.serde2.RegexSerDe'
with serdeproperties(
"input.regex" = "([^ ]*) ([^ ]*) ([^ ]*) \\[(.*)\\] \"(.*)\" (-|[0-9]*) (-|[0-9]*)"
);

load data local inpath '/opt/datas/hive/log.txt' into table log;

create external table log_part(
host string,
time string,
request string,
referer string,
agent string
)
row format delimited
fields terminated by '\t'
stored as orc;

tblproperties("orc.compress" = "snappy");

insert into table log_part select host, time, request, referer, agent from log;


show function extended substring

select t.hour, count(*) cnt from 
(select substring(time,9,2) hour from tab) t
group by t.hour order by cnt desc;


============数据 分析实战===结束============
exit;

```







