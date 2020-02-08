### Submit 命令提交给yarn过程入门

Spark-Submit分析
https://www.cnblogs.com/liangjf/p/8134645.html

抛开spark-submit脚本提交spark程序
https://blog.csdn.net/yhb315279058/article/details/51713309

使用JAVA代码实现编程式提交Spark任务
https://blog.csdn.net/gx304419380/article/details/79361645

**yarn-cluster	与yarn-client对比图**
yarn-cluster
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190321205710775.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L20wXzM3OTg3NDAy,size_16,color_FFFFFF,t_70)

yarn-client![在这里插入图片描述](https://img-blog.csdnimg.cn/20190321205343471.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L20wXzM3OTg3NDAy,size_16,color_FFFFFF,t_70)

YARN-client提交任务处理过程细则
https://blog.csdn.net/hahachenchen789/article/details/80586505

http://www.zhangrenhua.com/2015/12/30/hadoop-Yarn%E4%BB%BB%E5%8A%A1%E6%8F%90%E4%BA%A4%E6%B5%81%E7%A8%8B/





### Submit 命令

```
spark-submit --master yarn --deploy-mode cluster --class didlib.operator.io.HbaseReader --jars scopt.jar,didlib-core.jar,guava-12.0.1.jar,protobuf-java-2.5.0.jar,hbase-server-1.2.1.jar,hbase-client-1.2.1.jar,hbase-common-1.2.1.jar,htrace-core-3.1.0-incubating.jar,hbase-protocol-1.2.1.jar,metrics-core-2.2.0.jar didlib2.2.jar 


spark-submit --master yarn-cluster --class BoatSpeedParse --jars didlib-core.jar,scopt.jar boat.jar

```





### Base

```

spark块的原因:
	基于内存的计算
	有DAG引擎 将有向无环图进行切割
	
RDD 文档:
	http://spark.apache.org/docs/1.6.3/programming-guide.html

归并排序  插入排序

RDD五大特性:
	一些列的分区
	分布式计算 每个分片上都会有一个函数去迭代
	依赖关系
	   知道自己的父亲

	可重新分区
	移动计算

	checkpoint
		将rdd落地到磁盘

	offHeap
	tachyon
		分布式内存文件系统

	容错
		重算
		catch
		checkpoint

	宽窄依赖
		shuffle

map  mapPartition  mapPartitionsWithIndex

filter	 coalesce
sample	takeSample
distinct
intersection	取交集
cartesian
countbykey
cogroup		join
二次排序

collect.sort

分组去topN
二次排序

=======================
实战:
	表
	电话	日期	基站名	type
	基站 	精读 	纬度

	1.求在基站停留的总时间
	2.求在一段时间里某个基站停留时间最长的前两个人
	3.在一定的时间段内,用户停留最长的两个基站

	时间	url

	访问学科最多的top3



"hello".intersect("word")
"hello word".distinct
accumulator
repartition
coalesce

trimEnd
remove

for( i <- (0 until (b.length,2)).reverse) println(b(i))

b.sum
b.max

scala.util.Sorting.quickSort(a2)

sample(true,0.5,1L) 放回  1L 种子
takeSample
union
distinct
intersect
cartesian  笛卡尔基
mapPartitions
mapPartitionsWithIndex
countBykey  action
broadcast
cogroup
join


python 
lambda

```

