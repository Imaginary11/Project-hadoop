# Apache Flume 
-- 
* 解压到hadoop 目录下 
* 修改flume/bin/conf 目录下的 flume-env.sh，加入java环境变量，修改内存使用
JAVA_OPTS="-Xms8192m -Xmx8192m -Xss256k -Xmn2g -XX:+UseParNewGC -XX:+UseConcMarkSweepGC -XX:-UseGCOverheadLimit"

* 在conf 目录下创建测试文件



## flume-ng 启动命令
flume-ng agent -n $agent_name -c conf -f conf/{filename}


## 例子

###1、监听telnet 

####配置文件
-- 
Agent名称定义为agent.   
Source:可以理解为输入端，定义名称为s1  
channel：传输频道，定义为c1，设置为内存模式  
sinks：可以理解为输出端，定义为sk1,  
  
agent.sources = s1    
agent.channels = c1  
agent.sinks = sk1  
  
设置Source的内省为netcat 端口为5678，使用的channel为c1  
agent.sources.s1.type = netcat  
agent.sources.s1.bind = localhost  
agent.sources.s1.port = 5678  
agent.sources.s1.channels = c1  
  
设置Sink为logger模式，使用的channel为c1  
agent.sinks.sk1.type = logger  
agent.sinks.sk1.channel = c1  
设置channel信息  
agent.channels.c1.type = memory #内存模式  
agent.channels.c1.capacity = 1000     
agent.channels.c1.transactionCapacity = 100 #传输参数设置。  

#####启动命令
-- 
flume-ng agent -c conf -f conf/simple.conf --name agent -Dflume.root.logger=INFO,console 


####发送测试数据
-- 
telnet localhost 5678  
hello，world. 

-- 
###2、write to hdfs

####配置文件
-- 
####### Define a memory channel called ch1 on agent1
* agent1.channels.ch1.type = memory
* agent1.channels.ch1.capacity = 100000
* agent1.channels.ch1.transactionCapacity = 100000
* agent1.channels.ch1.keep-alive = 30
 
 
#######define source monitor a file
* agent1.sources.avro-source1.type = exec
* agent1.sources.avro-source1.shell = /bin/bash -c
* agent1.sources.avro-source1.command = tail -n +0 -F /data/flumetmp/a
* agent1.sources.avro-source1.channels = ch1
* agent1.sources.avro-source1.threads = 5
 
####### Define a logger sink that simply logs all events it receives
####### and connect it to the other end of the same channel
* agent1.sinks.log-sink1.channel = ch1
* agent1.sinks.log-sink1.type = hdfs
* agent1.sinks.log-sink1.hdfs.path = hdfs://alex:9000/flumeTest
* agent1.sinks.log-sink1.hdfs.writeFormat = Text
* agent1.sinks.log-sink1.hdfs.fileType = DataStream
* agent1.sinks.log-sink1.hdfs.rollInterval = 0
* agent1.sinks.log-sink1.hdfs.rollSize = 1000000
* agent1.sinks.log-sink1.hdfs.rollCount = 0
* agent1.sinks.log-sink1.hdfs.batchSize = 1000
* agent1.sinks.log-sink1.hdfs.txnEventMax = 1000
* agent1.sinks.log-sink1.hdfs.callTimeout = 60000
* agent1.sinks.log-sink1.hdfs.appendTimeout = 60000
 
####### Finally, now that we've defined all of our components, tell
####### agent1 which ones we want to activate.
* agent1.channels = ch1
* agent1.sources = avro-source1
* agent1.sinks = log-sink1


###执行命令
flume-ng agent --conf ../conf/ -f flume_directHDFS.conf -n agent1 -Dflume.root.logger=INFO,console



###3、avro2log

#### 添加如下配置文件
-- 
###avro2log.conf

* agent1.sources = source1
* agent1.sinks = sink1
* agent1.channels = channel1
 
* Describe/configure source1
* agent1.sources.source1.type = avro
* agent1.sources.source1.bind = localhost
* agent1.sources.source1.port = 44444

* agent1.sinks.sink1.type = logger
* agent1.channels.channel1.type = memory
* agent1.channels.channel1.capacity = 1000
* agent1.channels.channel1.transactionCapactiy = 100
* agent1.sources.source1.channels = channel1
* agent1.sinks.sink1.channel = channel1

现在我们可以启动看看我们的flume-ng是否成功。
在flume-ng的home目录下

$ bin/flume-ng agent --conf-file avro2log.conf --name agent1 -Dflume.root.logger=INFO,console --conf = conf

打开另外一个consloe:
在目录/data/flumetmp下创建文件a
执行echo 11111111 >>a
 
我们再打开另外一个console：执行 
bin/flume-ng avro-client --conf conf -H localhost -p 44444 -F /data/flumetmp/a
显示：