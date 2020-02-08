**安装**

<https://redis.io/> 

![1574062165984](assets/1574062165984.png)





```
tar -zxvf redis-5.0.6

cd ...

 make PREFIX=/opt/redis5.0.6 install


修改配置文件
vi [redis-xxx]/7001/redis.conf 
    bind 127.0.0.1 x.x.x.x #添加ip，或者注释掉即谁都能访问
    port 7001  #修改端口对应
    requirepass redis6380 配置密码!!
    daemonize yes

    cluster-enabled yes
    cluster-config-file nodes.conf 
    cluster-node-timeout 5000
    appendonly yes
    
    

后台启动
cp /usr/local/redis-5.0.6/redis.conf  /opt/redis5.0.6/bin
cd ...
vi redis.conf
	daemonize yes

启动server/关闭
./redis-server ./redis.conf
kill ... / pkill redis-server

启动客户端/关闭客户端
./redis-cli 
./redis-cli shutdown
```









学习重点：Redis各种存储类型 分布式锁

String

![1572837357641](assets/1572837357641.png)



![1572837468882](assets/1572837468882.png)



![1572837871404](assets/1572837871404.png)



![1572837980875](assets/1572837980875.png)



![1572838057921](assets/1572838057921.png)





![1572838427422](assets/1572838427422.png)



![1572846144983](assets/1572846144983.png)



![1572846597129](assets/1572846597129.png)



![1572846577438](assets/1572846577438.png)





![1572846612704](assets/1572846612704.png)





![1572846667545](assets/1572846667545.png)



![1572846910521](assets/1572846910521.png)



![1572848959393](assets/1572849306405.png)



![1572849397482](assets/1572849397482.png)



![1572849408849](assets/1572849408849.png)



![1572852239179](assets/1572852239179.png)





![1572852574960](assets/1572852574960.png)



![1572852702592](assets/1572852702592.png)





![1572853100801](assets/1572853100801.png)

