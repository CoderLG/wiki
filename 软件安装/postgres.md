**离线安装ic任务输出中寻找**

**在线安装postgres教程如下：**

```shell
#Install the repository RPM
yum install https://download.postgresql.org/pub/repos/yum/11/redhat/rhel-7.8-x86_64/pgdg-centos11-11-2.noarch.rpm
 
#Install the client packages
yum install postgresql11-11.2
 
#Optionally install the server packages
yum install postgresql11-server-11.2

#安装postgis的依赖（防止安装postgis时pg被yum升级）
yum install postgresql11-contrib-11.2

##若想安装自定义插件（编译安装）
#yum install postgresql11-devel-11.2

#为postgres用户赋权（安装时，自动生成用户）
chown –R postgres. /var/lib/
#修改服务（数据库数据及其配置保存路径）
systemctl edit postgresql-11   
#输入/修改：   
[Service] 
Environment=PGDATA=/var/lib/pgsql-11/data    #数据库保存路径,默认路径为/var/lib/pgsql/11/data  
systemctl daemon-reload   #重新加载
#初始化数据库
#Optionally initialize the database and enable automatic start
/usr/pgsql-11/bin/postgresql-11-setup initdb 
systemctl enable postgresql-11
systemctl start postgresql-11

#使用postgres账户(数据库用户默认为该名称)进入控制台(现在密码应该是空) 
# cd /var/lib/pgsql-11/  #防止提示权限不足
su postgres -c psql
\password   #修改数据库密码
\q  ##退出

#设置监听
#修改数据库存储目录下的pg_hba.conf(默认在$PGDATA= /var/lib/pgsql/11/data/下)
#下面用$PGDATA代替
#修改pg_hba.conf
vi $PGDATA/pg_hba.conf

# 修改IPv4 一行内容如下:
# "local" is for Unix domain socket connections only 本地信任
local   all             all                                     trust
# IPv4 local connections: 需要密码 md5加密
host    all             all            0.0.0.0/0          md5
# IPv6 local connections: 需要密码 md5加密
host    all             all             ::1/128                md5
# Allow replication connections from localhost, by a user with the
# replication privilege.
#注释其他 按需设置
#local   replication     all                                     trust
#host    replication     all             127.0.0.1/32            trust
#host    replication     all             ::1/128                 trust

#修改postgresql.conf
vi $PGDATA/postgresql.conf

#修改监听一节如下:
# - Connection Settings -
listen_addresses = '*' #可以指定地址，这里为所有地址可访问
port = 5432

#重启pg服务生效
systemctl restart postgresql-11
```



**安装插件**

```shell
#1.先安装几个工具包
yum install wget net-tools epel-release –y
yum install openssl –y
systemctl restart postgresql-11
#然后安装postgis
yum install postgis25_11 postgis25_11-client -y
#安装拓展工具
yum install ogr_fdw11 -y
yum install pgrouting_11 -y

#2.创建数据库postgis
CREATE DATABASE postgis OWNER postgres;
#切换新创建的database
\c postgis
#安装PostGis扩展
CREATE EXTENSION postgis;
CREATE EXTENSION postgis_topology;
CREATE EXTENSION ogr_fdw;

#然后可以验证是否安装成功
SELECT postgis_full_version();

#另外，下面是官方网站点列的一份扩展：
-- Enable PostGIS (includes raster)
CREATE EXTENSION postgis;
-- Enable Topology
CREATE EXTENSION postgis_topology;
-- Enable PostGIS Advanced 3D
-- and other geoprocessing algorithms
-- sfcgal not available with all distributions
CREATE EXTENSION postgis_sfcgal;
-- fuzzy matching needed for Tiger
CREATE EXTENSION fuzzystrmatch;
-- rule based standardizer
CREATE EXTENSION address_standardizer;
-- example rule data set
CREATE EXTENSION address_standardizer_data_us;
-- Enable US Tiger Geocoder
CREATE EXTENSION postgis_tiger_geocoder;
```







