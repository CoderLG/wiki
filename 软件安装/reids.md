





```
下载
wget http://download.redis.io/releases/redis-4.0.6.tar.gz 

解压
tar -zxvf redis-4.0.6.tar.gz

安装依赖
yum install gcc
 
编译
cd redis-4.0.6
make MALLOC=libc
cd src && make install

修改配置文件 reids.conf
后台运行
daemonize yes
解除保护模式
protected-mode no
注释掉
#bind 127.0.0.1

启动服务
./redis-server ../reids.conf

连接
./redis-cli
```

