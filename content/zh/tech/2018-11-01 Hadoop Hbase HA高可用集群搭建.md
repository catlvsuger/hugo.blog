+++
title = "Hadoop Hbase HA高可用集群搭建"
date = "2018-11-01T08:00:00+08:00"
categories = "Hbase"
tags = ["Java", "Hbase", "Hadoop"]
description = ""
#image = null  # To use: uncomment and replace null with value
"<!-- link" = "https://www.jianshu.com/p/9bd2327230c1 -->"
slug = "Hadoop Hbase HA distributed cluster construction"
+++

<p class="description"></p>

**基础环境准备**
根据前面[hadoop集群搭建](https://mydiscat.cn/category/2018/10/31/Hadoop distributed cluster construction.html)、[hbase集群搭建](https://mydiscat.cn/category/2018/10/31/Hbase distributed cluster construction.html)添加外部zookeeper集群
>下载zookeeper： [zookeeper-3.4.13](http://mirrors.hust.edu.cn/apache/zookeeper/zookeeper-3.4.13/zookeeper-3.4.13.tar.gz)

**zookeeper安装**

1、下载及安装 
解压到/home/zookeeper/目录下：
```
tar -zxvf zookeeper-3.4.13.tar.gz -C /home/zookeeper/
```

<!-- more -->

2、拷贝 zoo_sample.cfg
进入zookeeper的conf目录，拷贝zoo_sample.cfg并重命名为zoo.cfg ：
```
cd zookeeper-3.4.13/conf/
cp zoo_sample.cfg zoo.cfg
```



3、修改 zoo.cfg
```
vi zoo.cfg
```
修改如下，若原文件没有dataDir则直接添加：
```
dataDir=/home/zookeeper/zookeeper-3.4.13/data/zkData
//在最后添加，指定zookeeper集群主机及端口，机器数必须为奇数
server.1=hadoop-master:2888:3888
server.2=hadoop-slave1:2888:3888
server.3=hadoop-slave2:2888:3888
```
4、创建并编辑myid
//在zookeeper根目录下创建zoo.cfg中配置的目录
```
mkdir data/zkData/ -p
//创建并编辑文件
vi myid
//输入1，即表示当前机器为在zoo.cfg中指定的server.1
1
//保存退出
:wq
```
5、拷贝zookeeper到其他机器
上述操作是在hadoop-master机器上进行的，要将zookeeper拷贝到其他zookeeper集群机器上：
```
cd /home/zookeeper
scp -r zookeeper-3.4.13/ hadoop-slave1:/home/zookeeper/
scp -r zookeeper-3.4.13/ hadoop-slave2:/home/zookeeper/
```
6、修改其他机器的myid文件
myid文件是作为当前机器在zookeeper集群的标识，这些标识在zoo.cfg文件中已经配置好了，但是之前在hadoop-master这台机器上配置的myid为1，所以还需要修改其他机器的myid文件：
```
//在hadoop-slave1机器上
echo 2 > /home/zookeeper/zookeeper-3.4.13/data/zkData/myid
//在hadoop-slave2机器上
echo 3 > /home/zookeeper/zookeeper-3.4.13/data/zkData/myid
```
7、配置环境变量 vim  /etc/profile
```
添加：
export ZOOKEEPER_HOME=/home/zookeeper/zookeeper-3.4.13
export PATH=$PATH:$ZOOKEEPER_HOME/bin
其它服务器同样配置
```
配置生效 source /etc/profile
8、启动zookeeper集群
```
cd zookeeper-3.4.11/bin/
//分别在master188、master189、slave190上启动
/home/zookeeper/zookeeper-3.4.13/bin/zkServer.sh start

//查看状态
/home/zookeeper/zookeeper-3.4.13/bin/zkServer.sh  status
三台机器的zookeeper状态必须只有一个leader，其他都是follower。

//查看进程，若有QuorumpeerMain，则启动成功
jps
//停止
/home/zookeeper/zookeeper-3.4.13/bin/zkServer.sh stop
```
**hadoop添加zookeeper**

1、配置core-site.xml
```
添加：
  <!-- 指定ZooKeeper集群的地址和端口。注意，数量一定是奇数，且不少于三个节点-->
  <property>
    <name>ha.zookeeper.quorum</name>
    <value>hadoop-master:2181,hadoop-slave1:2181,hadoop-slave2:2181</value>
  </property>
修改：
    <property>
         <name>fs.defaultFS</name>
         <value>hdfs://hadoop-master:9000</value>
    </property> 
为：
    <property>
         <name>fs.defaultFS</name>
         <value>hdfs://ns1</value>
    </property> 
```
2、配置hdfs-site.xml
```
<configuration>
  <!-- 指定副本数，不能超过机器节点数  -->
  <property>
    <name>dfs.replication</name>
    <value>3</value>
  </property>

  <!-- 为namenode集群定义一个services name -->
  <property>
    <name>dfs.nameservices</name>
    <value>ns1</value>
  </property>

  <!-- nameservice 包含哪些namenode，为各个namenode起名 -->
  <property>
    <name>dfs.ha.namenodes.ns1</name>
    <value>hadoop-master,hadoop-slave1</value>
  </property>

  <!-- 名为hadoop-master的namenode的rpc地址和端口号，rpc用来和datanode通讯 -->
  <property>
    <name>dfs.namenode.rpc-address.ns1.hadoop-master</name>
    <value>hadoop-master:9000</value>
  </property>

  <!-- 名为hadoop-slave1的namenode的rpc地址和端口号，rpc用来和datanode通讯 -->
  <property>
    <name>dfs.namenode.rpc-address.ns1.hadoop-slave1</name>
    <value>hadoop-slave1:9000</value>
  </property>

  <!--名为hadoop-master的namenode的http地址和端口号，用来和web客户端通讯 -->
  <property>
    <name>dfs.namenode.http-address.ns1.hadoop-master</name>
    <value>hadoop-master:50070</value>
  </property>

  <!-- 名为hadoop-slave1的namenode的http地址和端口号，用来和web客户端通讯 -->
  <property>
    <name>dfs.namenode.http-address.ns1.hadoop-slave1</name>
    <value>hadoop-slave1:50070</value>
  </property>
  
  <!-- namenode间用于共享编辑日志的journal节点列表 -->
  <property>
    <name>dfs.namenode.shared.edits.dir</name>
    <value>qjournal://hadoop-master:8485;hadoop-slave1:8485;hadoop-slave2:8485/ns1</value>
  </property>

  <!-- 指定该集群出现故障时，是否自动切换到另一台namenode -->
  <property>
    <name>dfs.ha.automatic-failover.enabled.ns1</name>
    <value>true</value>
  </property>

  <!-- journalnode 上用于存放edits日志的目录 -->
  <property>
    <name>dfs.journalnode.edits.dir</name>
    <value>/home/hadoop/hadoop/tmp/data/dfs/journalnode</value>
  </property>

  <!-- 客户端连接可用状态的NameNode所用的代理类 -->
  <property>
    <name>dfs.client.failover.proxy.provider.ns1</name>
    <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
  </property>

  <!-- 一旦需要NameNode切换，使用ssh方式进行操作 -->
  <property>
    <name>dfs.ha.fencing.methods</name>
    <value>sshfence</value>
  </property>

  <!-- 如果使用ssh进行故障切换，使用ssh通信时用的密钥存储的位置 -->
  <property>
    <name>dfs.ha.fencing.ssh.private-key-files</name>
    <value>/root/.ssh/id_rsa</value>
  </property>

  <!-- connect-timeout超时时间 -->
  <property>
    <name>dfs.ha.fencing.ssh.connect-timeout</name>
    <value>30000</value>
  </property>

  <property>
       <name>dfs.name.dir</name>
       <value>/home/hadoop/hadoop/hdfs/name</value>
  </property>

  <property>
       <name>dfs.data.dir</name>
       <value>/home/hadoop/hadoop/hdfs/data</value>
  </property>
</configuration>

```
3、配置 mapred-site.xml
```
取消：
 <!-- <property>
        <name>mapred.job.tracker</name>
        <value>http://hadoop-master:9001</value>
  </property>-->
```
4、配置 yarn-site.xml
```
<configuration>

  <!-- 启用HA高可用性 -->
  <property>
    <name>yarn.resourcemanager.ha.enabled</name>
    <value>true</value>
  </property>

  <!-- 指定resourcemanager的名字 -->
  <property>
    <name>yarn.resourcemanager.cluster-id</name>
    <value>yrc</value>
  </property>

  <!-- 使用了2个resourcemanager,分别指定Resourcemanager的地址 -->
  <property>
    <name>yarn.resourcemanager.ha.rm-ids</name>
    <value>rm1,rm2</value>
  </property>
  
  <!-- 指定rm1的地址 -->
  <property>
    <name>yarn.resourcemanager.hostname.rm1</name>
    <value>hadoop-master</value>
  </property>
  
  <!-- 指定rm2的地址  -->
  <property>
    <name>yarn.resourcemanager.hostname.rm2</name>
    <value>hadoop-slave1</value>
  </property>
  
  <!-- 指定当前机器hadoop-master作为rm1 -->
  <property>
    <name>yarn.resourcemanager.ha.id</name>
    <value>rm1</value>
  </property>
  
  <!-- 指定zookeeper集群机器 -->
  <property>
    <name>yarn.resourcemanager.zk-address</name>
    <value>hadoop-master:2181,hadoop-slave1:2181,hadoop-slave2:2181</value>
  </property>
  
  <!-- NodeManager上运行的附属服务，默认是mapreduce_shuffle -->
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>

</configuration>
```
5、vi slaves
```
hadoop-master
hadoop-slave1
hadoop-slave2
```
**拷贝hadoop到其他机器**

1、拷贝
```
scp -r /home/hadoop/hadoop hadoop-slave1:/home/hadoop/
scp -r /home/hadoop/hadoop hadoop-slave2:/home/hadoop/
```
2、修改yarn-site.xml
在hadoop-slave1机器，即ResourceManager备用主节点上修改如下属性，表示当前机器作为rm2:：
```
  <property>
    <name>yarn.resourcemanager.ha.id</name>
    <value>rm2</value>
  </property>
```
同时删除hadoop-slave2机器上的该属性对，因为hadoop-slave2机器并不作为ResourceManager。

####启动Hadoop
1、启动zookeeper
```
/home/zookeeper/zookeeper-3.4.13/bin/zkServer.sh start
```
2、启动所有Journalnode
```
/home/hadoop/hadoop/sbin/hadoop-daemon.sh start  journalnode
```
3、格式化master namenode（这里直接复制会有问题，最好手动输入）（第一次启动操作）
```
/home/hadoop/hadoop/bin/hdfs namenode –format

#启动 master namenode 
/home/hadoop/hadoop/sbin/hadoop-daemon.sh start namenode

#master2上同步master namenode元数据 
bin/hdfs namenode -bootstrapStandby
#格式化 zk（在hadoop-master即可）（这里直接复杂会有问题，最好手动输入）

/home/hadoop/hadoop/bin/hdfs zkfc –formatZK
```
4、启动HDFS、YARN、ZookeeperFailoverController
```
/home/hadoop/hadoop/sbin/start-dfs.sh
//jps验证，显示NameNode和DataNode

/home/hadoop/hadoop/sbin/start-yarn.sh
//jps 验证，显示ResourceManager和NodeManager
```
4、启动resourcemanager（hadoop-master、hadoop-slave1）
```
/home/hadoop/hadoop/sbin/yarn-daemon.sh start resourcemanager
```
5、启动zkfc来监控NN状态（在hadoop-master、hadoop-slave1）
```
/home/hadoop/hadoop/sbin/hadoop-daemon.sh start zkfc
```
启动命令：
```
#hadoop-master
/home/hadoop/hadoop/sbin/start-all.sh
/home/hadoop/hadoop/sbin/hadoop-daemon.sh start zkfc

#hadoop-slave1
/home/hadoop/hadoop/sbin/yarn-daemon.sh start resourcemanager
/home/hadoop/hadoop/sbin/hadoop-daemon.sh start zkfc
```
停止命令：
```
#hadoop-master
/home/hadoop/hadoop/sbin/stop-all.sh
/home/hadoop/hadoop/sbin/hadoop-daemon.sh stop zkfc

#hadoop-slave1
/home/hadoop/hadoop/sbin/yarn-daemon.sh stop resourcemanager
/home/hadoop/hadoop/sbin/hadoop-daemon.sh stop zkfc
```
启动所有进程显示：
![启动项](https://upload-images.jianshu.io/upload_images/6273500-862219dd55729e0b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**错误处理：**
1、NameNode is not formatted
```
原因:  Path /home/hadoop/hadoop/hdfs/name should be specified as a URI in configura
tion files.
方法:把dfs.namenode.name.dir、dfs.datanode.data.dir的原路径格式如/usr/mywind/name改成file:/usr/mywind/name，即使用完全路径。
还有个原因：格式化命令复制进去运行报错，手动输入正常
```
**测试**
wordcount程序测试，在本地创建一个测试文件，并上传到hdfs上
```
#https:// 为下面文字加颜色
https://

#创建一个测试文件
 vim test.txt 
#上传到hdfs上
hadoop fs -put test.txt /input
#查询hdfs上面是否存在input文件
hadoop fs -ls /input
#计算
 hadoop jar hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.4.jar wordcount /input /output1
#查看输出结果
hadoop fs -cat /output1/part*

```
![结果](https://upload-images.jianshu.io/upload_images/6273500-fb526b6ff7c7bb7b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**Hbase安装配置**
进入/home/hbase/hbase/conf/目录，修改配置文件：
1、配置 hbase-env.sh
```
//配置JDK
export JAVA_HOME=/usr/java/

//保存pid文件
export HBASE_PID_DIR=/home/hbase/hbase/data/hbase/pids

//修改HBASE_MANAGES_ZK，禁用HBase自带的Zookeeper，因为我们是使用独立的Zookeeper
export HBASE_MANAGES_ZK=false
```
2、配置 hbase-site.xml
```
<configuration>
  <!-- 设置HRegionServers共享目录，请加上端口号 -->
  <property>
    <name>hbase.rootdir</name>
    <value>hdfs://master188:9000/hbase</value>
  </property>

  <!-- 指定HMaster主机 -->
  <property>
    <name>hbase.master</name>
    <value>hdfs://master188:60000</value>
  </property>

  <!-- 启用分布式模式 -->
  <property>
    <name>hbase.cluster.distributed</name>
    <value>true</value>
  </property>

  <!-- 指定Zookeeper集群位置 -->
  <property>
    <name>hbase.zookeeper.quorum</name>
    <value>hadoop-master:2181,hadoop-slave1:2181,hadoop-slave2:2181</value>
  </property>

  <!-- 指定独立Zookeeper安装路径 -->
  <property>
    <name>hbase.zookeeper.property.dataDir</name>
    <value>/home/zookeeper/zookeeper-3.4.13</value>
  </property>

  <!-- 指定ZooKeeper集群端口 -->
  <property>
    <name>hbase.zookeeper.property.clientPort</name>
    <value>2181</value>
  </property>
</configuration>
```
3）vi regionservers
修改regionservers文件，因为当前是使用独立的Zookeeper集群，所以要指定RegionServers所在机器：
```
hadoop-master
hadoop-slave1
hadoop-slave2
```

4）创建pid文件保存目录
在/home/hbase/hbase/目录下：
```
mkdir data/hbase/pids -p
```
3、拷贝HBase到其他机器
```
scp -r /home/hbase/hbase/ hadoop-slave1:/home/hadoop/
scp -r /home/hbase/hbase/ hadoop-slave2:/home/hadoop/
```
4、启动HBase
在主节点上启动HBase（主节点指NameNode状态为active的节点，非指文中的机器声明）：
```
/home/hbase/hbase/bin/start-hbase.sh
```
5、查看HMaster、Regionserver进程是否启动
```
jps
注意：此时Hadoop集群应处于启动状态，并且是在主节点执行start-hbase.sh启动HBase集群，否则HMaster进程将在启动几秒后消失，
而备用的HMaster进程需要在备用主节点单独启动，命令是：./hbase-daemon.sh start master。

在备用主节点启动HMaster进程，作为备用HMaster：
/home/hbase/hbase/bin/hbase-daemon.sh start master
```
5、HA高可用测试
```
在浏览器中输入 ip:16010 ，查看主节点和备用主节点上的HMaster的状态，在备用主节点的web界面中，
可以看到“Current Active Master: master188”，表示当前HBase主节点是master188机器；

主节点--->备用主节点
这里的主节点指使用start-hbase.sh命令启动HBase集群的机器

kill掉主节点的HMaster进程，在浏览器中查看备用主节点的HBase是否切换为active；

若上述操作成功，则在主节点启动被杀死的HMaster进程：

/home/hbase/hbase/bin/hbase-daemon.sh start master

然后，kill掉备用主节点的HMaster进程，在浏览器中查看主节点的HBase是否切换为active，若操作成功，则HBase高可用集群搭建完成；
```
6、HBase基本操作
```
//启动HBase
[root@vnet ~] start-hbase.sh

//进入HBase Shell
[root@vnet ~] hbase shell

//查看当前HBase有哪些表
hbase(main):> list

//创建表t_user，cf1和cf2是列族，列族一般不超过3个
hbase(main):> create 't_user','cf1','cf2'

//获得表t_user的描述信息
hbase(main):> describe 't_user'

//禁用表
hbase(main):> disable 't_user'

//删除表，删除表之前要先把表禁用掉
hbase(main):> drop 't_user'

//查询表是否存在
hbase(main):> exists 't_user'

//查看全表数据
hbase(main):> scan 't_user'

//插入数据，分别是表名、key、列（列族：具体列）、值。HBase是面向列的数据库，列可无限扩充
hbase(main):> put 't_user' ,'001','cf1:name','chenxj'
hbase(main):> put 't_user' ,'001','cf1:age','18'
hbase(main):> put 't_user' ,'001','cf2:sex','man'
hbase(main):> put 't_user' ,'002','cf1:name','chenxj'
hbase(main):> put 't_user' ,'002','cf1:address','fuzhou'
hbase(main):> put 't_user' ,'002','cf2:sex','man'

//获取数据，可根据key、key和列族等进行查询
hbase(main):> get 't_user','001'
hbase(main):> get 't_user','002','cf1'
hbase(main):> get 't_user','001','cf1:age'
```
7、集群启动结果
Hadoop + Zookeeper + HBase 高可用集群启动后，进程状态如下：

|描述	| hadoop-master	| hadoop-slave1 |  hadoop-slave2 | 
|-------------|------------------|--------------------|--------------|
| HDFS主	| NameNode	| NameNode	| |
| HDFS从	| DataNode	| DataNode	| DataNode |
| YARN主	| ResourceManager	| ResourceManager | |
| YARN从	| NodeManager	| NodeManager	| NodeManager |
| HBase主	| HMaster	| HMaster |	|
| HBase从	| HRegionServer	| HRegionServer	| HRegionServer |
| Zookeeper独立进程	| QuorumPeerMain	| QuorumPeerMain	| QuorumPeerMain |
| NameNodes数据同步	| JournalNode|	 JournalNode   | JournalNode |
|主备故障切换	| DFSZKFailoverController	| DFSZKFailoverController	| |

**总结**
需要注意的地方：

1）备用节点上的NameNode、ResourceManager、HMaster均需单独启动；
```
hadoop-daemon.sh start namenode
yarn-daemon.sh start resourcemanager
hbase-daemon.sh start master 
```
2）可以使用-forcemanual参数强制切换主节点与备用主节点，但强制切换后集群的自动故障转移将会失效，需要重新格式化zkfc：hdfs zdfc -formatZK;
```
（这个没有测试）
hdfs haadmin -transitionToActive/transitionToStandby  -forcemanual  hadoop-slave1
yarn rmadmin -transitionToActive/transitionToStandby  -forcemanual  rm2
```
3）在备用主节点同步主节点的元数据时，主节点的HDFS必须已经启动；

4）无法查看standby状态的节点上的hdfs；

5）格式化namenode时要先启动各个JournalNode机器上的journalnode进程：
否则会报journalnode拒绝连接错误
```
hadoop-daemon.sh start journalnode；
```
6）若遇到问题，可以先考虑是哪个组件出现问题，然后查看该组件或与该组件相关的组件的日志信息；若各组件web页面无法访问，或存在其他连接问题，可以从「防火墙是否关闭」、「端口是否被占用」、「SSH」、「集群机器是否处于同一网段」内等角度考虑

**参考： [Hadoop HA高可用集群搭建（Hadoop+Zookeeper+HBase）](https://www.cnblogs.com/sqchen/p/8080952.html)**


<hr />

