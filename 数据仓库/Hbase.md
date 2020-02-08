### Base

```
start-dfs.sh
bin/zkServer.sh start
bin/zkCli.sh 
ll

是否在安全模式
bin/hdfs dfsadmin -safemode get
bin/hbase-daemon.sh start master
bin/hbase-daemon.sh start regionserver

start-hbase.sh

bin/hbase shell

help

create_namespace 'my_ns'

create 'psn', 'cf1', 'cf2'

list

describe 'psn'
put 'psn', '001', 'cf1:name', 'xiaoming'
put 'psn', '001', 'cf1:sex', 'boy'

依据rowkey查询，查询速度最快
	get	
	get 'psn', '001', 'cf1:name'
范围查询
	scan
	scan 'user', {COLUMNS =>['info:name', 'info:age']}
	scan 'user', {STARTROW => '1002'}
全表扫描
	scan 'psn'

get 'psn', '001', 'cf1:name'
put 'psn', '001', 'cf1:name', 'lilei'
get 'psn', '001', 'cf1:name'
//只得到最新的版本  version版本为一 这样添加就是修改 合并时会删除就版本


delete 'user', '1001', 'info:name'
scan 'user'

deleteall 'user', 


deleteall 'user',  'info:name'

deleteall 'user', '1001'
scan 'user'

create 'tbl', 'cf'
drop 'tbl'
不成功
disable 'test222'
drop 'tbl'
先禁用在删除


//强制溢写
flush 'psn'


exit

bin/stop-hbase.sh 
bin/zkServer.sh stop

//查看版本
hbase shell	//进入shell 
version		//输入version

```

