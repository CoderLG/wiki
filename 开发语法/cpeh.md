// 相关博客
https://blog.csdn.net/Jmilk/article/details/89419873
https://www.cnblogs.com/flytor/p/11384518.html
https://blog.csdn.net/uxiAD7442KMy1X86DtM3/article/details/81059215



**shell**

```java
//健康状况
ceph -s
ceph health detail


//pg-num pgp-num
https://www.jianshu.com/p/ae96ee24ef6c

//PGP是为了管理placement而存在的专门的PG，它和PG的数量应该保持一致。
//如果你增加pool的pg_num，就需要同时增加pgp_num，保持它们大小一致，这样集群才能正常rebalancing。

//检查rbd这个pool里已存在的PG和PGP数量：
ceph osd pool get rbd pg_num
ceph osd pool get rbd pgp_num

//检查pool的复制size，执行如下命令：
ceph osd dump |grep size|grep rbd

//计算公式
Total PGs = ((Total_number_of_OSD * 100) / max_replication_count) / pool_count
算出来取最接近其值的2的整幂次

//变更rbd的pg_num和pgp_num为256：
ceph osd pool set rbd pg_num 256
ceph osd pool set rbd pgp_num 256


//集群的使用情况
ceph df 


// ceph rgw：s3基本概念在rados中的组织方式
// https://www.jianshu.com/p/61be74aaa983m
radosgw-admin metadata list
radosgw-admin metadata list bucket
radosgw-admin metadata list bucket.instance
radosgw-admin metadata list user

radosgw-admin metadata get bucket:<bucket>
radosgw-admin metadata get bucket.instance:<bucket>:<marker>
radosgw-admin metadata get user:<user>   # get or set

# 创建桶 并加入
s3cmd mb s3://first-bucket

s3cmd ls
s3cmd put -r /etc/hosts s3://first-bucket

s3cmd ls s3://first-bucket

s3cmd get -r s3://first-bucket/  /home

s3cmd del s3://first-bucket/*
s3cmd rb s3://first-bucket
s3cmd ls

```





**灾备**

```
灾备 数据迁移
1.当某个节点出现事故是可以自动恢复的
	ceph具备数据隔离级别  磁盘 机器  机架  机房等

2.整个集群全部瘫痪
	ceph有快照，理论上可恢复到快照时的状态
	
3.为损坏下的数据迁移
	a.对于某些格式的数据可以通过某些特定的软件
	b.将新的节点加入 旧集群，利用ceph自己的crush-rule 慢慢将数据同步过去
```

