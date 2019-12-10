**特别说明**

```

//查看客户端 版本
psql --version	

//产看服务器版本 
psql -h 192.168.4.38 -p 5431 -U postgres
select version()

docker ps   //查看id
docker exec -it  id  /bin/bash


sql执行计划
explain SELECT st_distance("WZ",ST_GeomFromText('POINT (50 50)',4326)) AS distance , "DMMC",st_astext("WZ"),*  
FROM "GFGX_Y_DMK_DMSJ" where  ST_DWithin("WZ", ST_GeomFromText('POINT (50 50)',4326), 300000) 
AND "DLBM" < 900000000000000 AND "DLBM" > 400000000000000  ORDER BY distance

```



**序列-sequence 自增id**

```
CREATE SEQUENCE sequencename
	[ INCREMENT increment ]		-- 自增数，默认是 1
	[ MINVALUE minvalue ]		-- 最小值
	[ MAXVALUE maxvalue ]		-- 最大值
	[ START start ]				-- 设置起始值
	[ CACHE cache ]				-- 是否预先缓存
	[ CYCLE ]					-- 是否到达最大值的时候，重新返回到最小值
	
查看索引	
SELECT * FROM test_seq;

查看下一个
SELECT nextval('test_seq');

删
DROP SEQUENCE test_seq;
ERROR:  cannot drop sequence test_seq because other objects depend on it
删除引用了次seq的表，再删seq
DROP TABLE test；
DROP SEQUENCE test_seq;




实战
create sequence test_seq 
INCREMENT 1
MINVALUE 1
MAXVALUE 2000000000
START 1

DROP TABLE IF EXISTS "public"."test";
CREATE TABLE "public"."test" (
  "id" int4 NOT NULL DEFAULT nextval('test_seq'::regclass),
  "name" varchar(255) COLLATE "pg_catalog"."default"
)
;
ALTER TABLE "public"."test" ADD CONSTRAINT "test_pkey" PRIMARY KEY ("id");
如果表已经存在
后期添加序列
alter table test alter column id set default nextval('test_seq');

insert into test (name) VALUES('gg')
当增加到最大值5的时候报错 ERROR:  nextval: reached maximum value of sequence "test_seq" (5)

	
	
```



**导入导出**

```

数据导出：
pg_dump -h 192.168.4.97 -p 5432 -U postgres -d gv_place -F t -f /home/place.tar 
单独导出一张表：
pg_dump -h 192.168.4.17 -p 5431 -U postgres -d new_world -t world -F t -f /data/postgresql/world.tar

导入.sql
psql -h 192.168.4.17 -p 5431 -U postgres -f ./xx.sql


数据导入：
psql -h 192.168.4.38 -p 5431 -U postgres

DROP DATABASE  IF EXISTS  gv_place;

报错：
ERROR:   database "gv_place" is being accessed by other users
DETAIL:  There are 200 other sessions using the database.
错误:  其他用户正在使用源数据库 "template1"
描述:  那里有1个其它会话正在使用数据库.
解决方案：
select pg_terminate_backend(pid) from pg_stat_activity where DATNAME='gv_place';

(创建gv_place数据库)
create database gv_place encoding = 'UTF-8';

退出数据库
\q

将tar包数据导入到gv_place数据库
pg_restore -h 192.168.4.38 -p 5431 -U postgres -d gv_place -v /mnt/mfs/lzz/icenter112/icenterData/manager/lg/data/place.tar

报错：type "public.geometry" does not exist
错误: 缺少插件  
解决方案：
\c database			//进入这个database

\q
重新导入


```



**新_地名数据库创建索引**

```
-- 创建索引

create index gfgx_dmsj_cjbm_btree on "GFGX_Y_DMK_DMSJ" using btree("DMRD");  
create index gfgx_dmsj_dlbm_btree on "GFGX_Y_DMK_DMSJ" using btree("DLBM");  
create index gfgx_dmsj_dmmc_btree on "GFGX_Y_DMK_DMSJ" using btree("DMMC" collate "C");  
create index gfgx_dmsj_dmjp_btree on "GFGX_Y_DMK_DMSJ" using btree("DMJP" collate "C");	 
create index gfgx_dmsj_dmqp_btree on "GFGX_Y_DMK_DMSJ" using btree("DMQP" collate "C");  
create index concurrently  gfgx_dmsj_wz_rtree on  "GFGX_Y_DMK_DMSJ"  using rtree("WZ");


create index dmsj_area_btree  on "DMSJ"  using btree("Area");  

create index dmsj_qpy_btree  on "DMSJ"  using btree("QPY");  

create index dmsj_name_btree on "DMSJ"  using btree("Name"); 

create index dmsj_spy_btree  on "DMSJ"  using btree("SPY");  

create index concurrently dmsj_center_rtree on  "DMSJ"  using rtree("Center"); 

create index concurrently dmsj_wz_rtree on  "DMSJ"  using rtree("WZ"); 


create index concurrently world_geom_rtree  on "world"  using rtree("geom");  

create index concurrently prov_geom_rtree  on "prov"  using rtree("geom");  

create index concurrently city_geom_rtree  on "city"  using rtree("geom");  

create index concurrently county_geom_rtree  on "county"  using rtree("geom");
```



**老_地名数据库创建索引**

```
-- 创建索引

create index gfgx_dmsj_dmcjbm_btree on "GFGX_Y_DMK_DMSJ" using btree("DMCJBM");  -- 15.906

create index gfgx_dmsj_dlbm_bress on "GFGX_Y_DMK_DMSJ" using btree("DLBM");  -- 16.169

CREATE INDEX gfgx_dmsj_dmmc_gin ON "GFGX_Y_DMK_DMSJ" USING gin ("DMMC" gin_trgm_ops); 

CREATE INDEX gfgx_dmsj_dmjp_gin ON "GFGX_Y_DMK_DMSJ" USING gin ("DMJP" gin_trgm_ops); 

CREATE INDEX gfgx_dmsj_dmqp_gin ON "GFGX_Y_DMK_DMSJ" USING gin ("DMQP" gin_trgm_ops); 

create index gfgx_dmsj_wz_gist on "GFGX_Y_DMK_DMSJ"  using gist("WZ"); 


create index dmsj_area_btree  on "DMSJ"  using btree("Area");  

create index dmsj_qpy_btree  on "DMSJ"  using btree("QPY");  

create index dmsj_name_btree  on "DMSJ"  using btree("Name"); 

create index dmsj_spy_btree  on "DMSJ"  using btree("SPY");  

create index dmsj_center_gist  on "DMSJ"  using gist("Center");  

create index dmsj_wz_gist  on "DMSJ"  using gist("WZ"); 



create index world_geom_gist  on "world"  using gist("geom");  

create index prov_geom_gist  on "prov"  using gist("geom");  

create index city_geom_gist  on "city"  using gist("geom");  

create index county_geom_gist  on "county"  using gist("geom"); 


```



**gin vs btree**

```
创建时间上btree 非常优秀
create index btree_dmmc on "GFGX_Y_DMK_DMSJ" using btree("DMMC");  -- 172.158  中文
create index btree_dmmc on "GFGX_Y_DMK_DMSJ" using btree("DMMC" collate "C");  -- 172.158  中文


create index btree_dmjp on "GFGX_Y_DMK_DMSJ" using btree("DMJP" );  -- 49.247  英文
create index btree_dmjp on "GFGX_Y_DMK_DMSJ" using btree("DMJP" collate "C");  -- 英文19.784

create index btree_dmqp on "GFGX_Y_DMK_DMSJ" using btree("DMQP");  -- 13.528 多为 ‘’
create index btree_dmqp on "GFGX_Y_DMK_DMSJ" using btree("DMQP" collate "C");  -- 17.104 多为 ‘’


CREATE INDEX gin_trgm_dmmc ON "GFGX_Y_DMK_DMSJ" USING gin ("DMMC" gin_trgm_ops); -- 887.014  中文

CREATE INDEX gin_trgm_dmjp ON "GFGX_Y_DMK_DMSJ" USING gin ("DMJP" gin_trgm_ops); -- 63.422 英文

CREATE INDEX gin_trgm_dmqp ON "GFGX_Y_DMK_DMSJ" USING gin ("DMQP" gin_trgm_ops); -- 5.856 多为 ‘’

项目中的后模糊查询， 非常优秀

-- 无索
select * from "GFGX_Y_DMK_DMSJ" where "DMMC" like '白%'  -- 1.669

select * from "GFGX_Y_DMK_DMSJ" where "DMMC" like '%县'  -- 1.867


-- btree
select * from "GFGX_Y_DMK_DMSJ" where "DMMC" like '白%'  -- 1.651 //索引没有指定c
select * from "GFGX_Y_DMK_DMSJ" where "DMMC" like '白%'  -- 0.682 //索引指定c


select * from "GFGX_Y_DMK_DMSJ" where "DMMC" like '%县'  -- 1.863

-- gin
select * from "GFGX_Y_DMK_DMSJ" where "DMMC" like '白%'  -- 0.486

select * from "GFGX_Y_DMK_DMSJ" where "DMMC" like '%县'  -- 1.865

select * from "GFGX_Y_DMK_DMSJ" where "DMMC" like '%乌县'  -- 0.051

select * from "GFGX_Y_DMK_DMSJ" where "DMMC" like '%比乌县'  -- 0.048


----------------------------

-- 无索
select * from "GFGX_Y_DMK_DMSJ" where "DMJP" like 'nbl%'  -- 1.504

select * from "GFGX_Y_DMK_DMSJ" where "DMJP" like '%bl'  -- 1.743


-- btree
select * from "GFGX_Y_DMK_DMSJ" where "DMJP" like 'nbl%'  -- 1.565  //索引没有指定c

select * from "GFGX_Y_DMK_DMSJ" where "DMJP" like 'nbl%'  -- 0.059  //索引指定c

select * from "GFGX_Y_DMK_DMSJ" where "DMJP" like '%bl'  --  1.821

-- gin
select * from "GFGX_Y_DMK_DMSJ" where "DMJP" like 'nbl%'  -- 0.071

select * from "GFGX_Y_DMK_DMSJ" where "DMJP" like '%l'  -- 3.495

select * from "GFGX_Y_DMK_DMSJ" where "DMJP" like '%bl'  -- 0.399


----------------------------

-- 无索
select * from "GFGX_Y_DMK_DMSJ" where "DMMC" like '白%' or "DMJP" like '白%'  or "DMQP" like '白%'  -- 2.069

select * from "GFGX_Y_DMK_DMSJ" where "DMMC" like '%县' or "DMJP" like '%县'  or "DMQP" like '%县'  --  2.371


-- btree
select * from "GFGX_Y_DMK_DMSJ" where "DMMC" like '白%' or "DMJP" like '白%'  or "DMQP" like '白%'   -- 2.114  //索引没有指定c

select * from "GFGX_Y_DMK_DMSJ" where "DMMC" like '白%' or "DMJP" like '白%'  or "DMQP" like '白%'  -- 0.769 //索引没有指定c

select * from "GFGX_Y_DMK_DMSJ" where "DMMC" like '%县' or "DMJP" like '%县'  or "DMQP" like '%县'  --  2.347

-- gin
select * from "GFGX_Y_DMK_DMSJ" where "DMMC" like '白%' or "DMJP" like '白%'  or "DMQP" like '白%'   -- 0.579

select * from "GFGX_Y_DMK_DMSJ" where "DMMC" like '%县' or "DMJP" like '%县'  or "DMQP" like '%县'  -- 2.365

select * from "GFGX_Y_DMK_DMSJ" where "DMMC" like '%乌县' or "DMJP" like '%乌县'  or "DMQP" like '%乌县'  -- 0.052

```

-----------------------------------------

**索引的探索**

<https://www.cnblogs.com/jiaoyiping/p/7191300.html> 

<https://www.jianshu.com/p/0a657fddd53e> 

<https://www.jianshu.com/p/98a33ee17b94> 

<https://www.jianshu.com/p/ee9119505a03> 

<https://www.jianshu.com/p/72ea45696811> 

<https://blog.csdn.net/qingyafan/article/details/86694121> 



<https://yq.aliyun.com/articles/69250> 

<https://my.oschina.net/u/3938953/blog/1923575> 

<https://github.com/digoal/blog/blob/master/201704/20170426_01.md> 

```
-- 简单索引的创建
CREATE INDEX city_geom_idx ON city (code);  -- 4.065

create index trgm_idx_gin on "GFGX_Y_DMK_DMSJ" using gin("DMMC")
> ERROR:  data type text has no default operator class for access method "gin"
HINT:  You must specify an operator class for the index or define a default operator class for the data type.

-- 参考 https://www.postgresql.org/message-id/F937451D-EA55-46E7-96B8-C47091997D40%40hp.com
create index trgm_idx_gin on "GFGX_Y_DMK_DMSJ"  using  gin(to_tsvector('english', "DMMC")); -- 566.056

-- 参考 https://www.cnblogs.com/carey-he/p/11187103.html
CREATE INDEX trgm_idx_gin ON "GFGX_Y_DMK_DMSJ" USING gin ("DMMC" gin_trgm_ops);
```



**深入索引**

```
聚集索引/聚簇索引 索引 值存的不是地址，而是正行数据
非聚集索引 索引 值存的是正行数据的地址
单值索引 常见索引
联合索引 多个常见索引 多列比较

select * from test1 where name = 'sdf';
没有索引的时候，系统将不得不逐行扫描整个test1表，以查找所有匹配的条目。如果test1中有很多行，并且这样的查询仅仅返回几行（可能是零或一行），这显然是一种低效的方法。但是如果系统已被指示在列上维护索引，则可以使用更有效的方法来定位匹配的行。例如，它可能只需要在搜索树中深入几层。

在大型表上创建索引可能需要很长时间。默认情况下，PostgreSQL允许读表（SELECT语句）与索引创建并行发生，但写入（INSERT，UPDATE，DELETE）将被阻止，直到索引生成完成。在生产环境中，这通常是不能接受的。可以设置允许写入与索引创建并行发生。

PostgreSQL提供了几种索引类型：B-tree，Hash，GiST，SP-GiST，GIN和BRIN。每个索引类型使用不同的算法，适合不同种类的查询。默认情况下，CREATE INDEX命令创建B-tree索引，这符合最常见的情况。

GiST索引适用于二维地理位置数据等。
SP-GiST 适用于二维点坐标数据的索引等。
GIN索引适用于数组等。
BRIN(Block Range INdexes)适用于块中查找最大值最小值。

```

```
B-tree 适用
<,<=,=,>=,>,
between
in
is null
is not null
like 'foo' 
~ '^foo'  --正则匹配
like 'foo%'

B-tree 不适用
like '%foo'

B-tree索引也可用于按排序顺序检索数据。这并不总是比简单的扫描和排序更快，但它通常是有益的。
支持并发创建索引，其他索引 pg11均不支持 
```

```
gin 索引 、pg_trgm插件

首先，pg_trgm将字符串的前端添加2个空格，末尾添加1个空格。
然后，每连续的3个字符为一个TOKEN，拆开。
最后，对TOKEN建立GIN倒排索引。
查看字符串的TOKEN，可以使用如下方法。
test=# select show_trgm('123');      
        show_trgm              
-------------------------      
 {"  1"," 12",123,"23 "}      
(1 row)     


使用pg_trgm时，如果要获得最好的效果，最好满足这些条件。
1. 有前缀的模糊查询，例如a%，至少需要提供1个字符。( 搜索的是token=' a' )
2. 有后缀的模糊查询，例如%ab，至少需要提供2个字符。( 搜索的是token='ab ' )
3. 前后模糊查询，例如%abcd%，至少需要提供3个字符。( 这个使用数组搜索，搜索的是token(s) 包含 {" a"," ab",abc,bcd,"cd "} )
原因是什么呢？
因为pg_trgm生成的TOKEN是三个字符，只有在以上三个条件下，才能匹配到对应的TOKEN。
test=# select show_trgm('123');      
        show_trgm              
-------------------------      
 {"  1"," 12",123,"23 "}      
(1 row)      
```

```
rtree
gist的底层实现是rtree，
测试结果表明，两者的性能相差不大，当然还是R-tree更快，而且在
PostgreSQL中的R-tree实现有缺陷：
不能处理超过8k的要素，GiST可以；
PostgreSQL中实现的R-tree不是null-safe的，如果geometry有null值，则索引构建会失败。
```



```
Hash索引只能处理简单的相等比较， 但只对 = 起作用。
```

**复合索引**

```
B-tree类型的多列索引可以用于索引里声明的列的各种组合。但是包含主列（声明时最左端的那一列）的查询组合查询效率更高。即各种组合用于查询时，如果组合中包含最声明时最左端的那个列，那么这个查询组合比没有包含主列效率更高。还有一条规则时当主列相等，紧接着的查询条件时任何一列是不相等，第三列任何操作，会导致所有查询表时索引无效。例如我们建立的索引为 ON (a,b,c) ,我们的查询条件是：a = 5 AND b >= 42 AND c < 77，如果数据库引擎使用索引查询，索引扫描规则是a=5的行的所有索引值都会被 a = 5 and b >= 42 的查询条件扫描，c<77 的条件会被忽略，所以还要执行一次从结果里的顺序查询，效率很低。这种情况下我们应建立的索引应该是ON (b,c),即在b和c上建立索引，不包括a。如果还是设置3列索引的话，在查询规划器会帮我们优先选择全表索引。另外如果主列用的不是“=”操作，三列索引还是有效的。

GiST可以用于索引里声明的列的各种组合，有主列副列区分，GiST索引在主列只有很少的值的时候效率会比较低。

GIN索引可以用于索引里声明的列的各种组合，但没有主列副列之分。

BRIN可以用于索引里声明的列的各种组合，也没有主列副列之分。

多列索引应该谨慎使用，多列索引的开销比单列索引大，而且索引列一旦超过3列以上查询条件不慎，会引起索引作用不起作用。

--------------------------例子-------------------------
假设我们有2列，我们的索引组合有3种
1、ON (a)
2、ON (b)
3、ON (a,b)
我们表上索引就是7种选择。
1、不加索引。执行全表扫描。在数据量比较大的时候效率很低。
2、只加ON (a)。查询条件不包含a时全表扫描；包含a时，先查找符合条件a的行，再在结果上过滤其他条件。
3、只加ON (b)。类似a。
4、只加ON (a,b)。查询条件为a and b时效率最高；查询条件仅为a时次之；查询条件为b时，较差，但比全盘扫描效果好。
5、加ON (a)，ON (b)。查询会根据查询计划有所不同，但基本有2种情况（and条件）
（1）分别在各自的列上按索引取值然后汇总，
（2）按其中一个条件查询，另一个条件在结果里过滤。
6、加ON (a)，ON (a,b)。累赘了，建议不使用。
7、ON (b)，ON (a,b)。这种在查询频率大大超过插入频率的情况下使用，并且b，和ab的查询量大体相当。毕竟插入1条件数据要维护和修改2条索引，开销很大。
8、ON (a)，ON (b)，ON (a,b)。累赘了，建议不使用。

```

---------------------------------

**开发调优**

```
提交慢： 多条提交	根据实际情况修改pg的配置

队列： add push 各有各的用处

均匀分桶： 对质数取余


//连接池
//查看连接
select count(1) from pg_stat_activity;
//最大连接
show max_connections;
//出现问题可以修改最大连接，和释放连接的时间

```

