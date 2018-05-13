# Hadoop 配置文件解析

###**hadoop官网上有各个配置文件所有标签的定义解释，查阅很不方便，重点是解析各个配置文件中常用标签所指代的内容**

### 1.core-site.xml
在docker上部署简单hadoop集群所用core-site.xml.
```xml
<configuration>
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://Master:9000</value>
  </property>
  <property>
    <name>hadoop.tmp.dir</name>
    <value>/hadoop/tmp</value>
  </property>
</configuration>
```
文件中Master为主机名，也可以是服务器地址
``<value>``格式常为hdfs://主机名（主机ip):端口

fs.defaultFS: 这是一个**描述集群中NameNode结点的URI**（包括协议、主机名称、端口号），集群里面的每一台机器都需要知道NameNode的地址。DataNode结点会先在NameNode上注册，这样它们的数据才可以被使用。独立的客户端程序通过这个URI跟DataNode交互，以取得文件的块列表。HDFS和MapReduce组件都需要它，这就是它出现在core-site.xml文件中而不是hdfs-site.xml文件中的原因。

hadoop.tmp.dir: 这里的路径默认是NameNode、DataNode、JournalNode等存放数据的公共目录。用户也可以自己单独指定这三类节点的目录。

### 2.hdfs-site.xml
```xml
<configuration>
  <property>
    <name>dfs.replication</name>
    <value>1</value>
  </property>
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>/hadoop/data</value>
  </property>
  <property>
    <name>dfs.namenode.name.dir</name>
    <value>/hadoop/name</value>
  </property>
</configuration>

```

dfs.replication:决定着系统里面的**文件块的数据备份个数**。对于一个实际的应用，它 应该被设为3（不大于datanode的数目即可）。少于三个的备份，可能会影响到数据的可靠性（系统故障时，也许会造成数据丢失）。在单机和单机伪分布模式下，将此值修改为1。

dfs.datanode.data.dir:是DataNode结点被指定要存储数据的本地文件系统路径
dfs.namenode.name.dir:是NameNode结点存储hadoop文件系统信息的本地系统路径

**相关目录会在格式化的时候会自动创建，hdfs-site.xml中的有关目录都是以core-site.xml中的hadoop.tmp.dir为主进行配置的，所以可以先创建hadoop.tmp.dir的目录**

该文件是核心文件，里面还有很多内容，大部分都还没有尝试过，这里补充一个网上的资料:

```xml
<configuration>  
    <!--【指定DataNode存储block的副本数量。默认值是3个，我们现在有4个DataNode，该值不大于4即可。】-->    
    <property>    
      <name>dfs.replication</name>    
       <value>2</value>    
    </property>          
    <property>    
       <name>dfs.permissions</name>    
      <value>false</value>    
   </property>    
   <property>    
      <name>dfs.permissions.enabled</name>    
      <value>false</value>    
   </property>  
   <!--【给hdfs集群起名字】 -->     
   <property>        
      <name>dfs.nameservices</name>      
      <value>cluster1</value>        
   </property>  
   <!--【指定NameService是cluster1时的namenode有哪些，这里的值也是逻辑名称，名字随便起，相互不重复即可】-->               
  <property>    
     <name>dfs.ha.namenodes.cluster1</name>    
     <value>hadoop1,hadoop2</value>    
  </property>    
  <!--【指定hadoop101的RPC地址】 -->    
  <property>    
     <name>dfs.namenode.rpc-address.cluster1.hadoop1</name>    
     <value>hadoop1:9000</value>    
  </property>      
  <!--【指定hadoop101的http地址】-->   
  <property>                    
    <name>dfs.namenode.http-address.cluster1.hadoop1</name>        
    <value>hadoop1:50070</value>        
  </property>         
  <property>        
    <name>dfs.namenode.rpc-address.cluster1.hadoop2</name>        
    <value>hadoop2:9000</value>        
  </property>    
  <property>        
    <name>dfs.namenode.http-address.cluster1.hadoop2</name>        
    <value>hadoop2:50070</value>       
  </property>    
  <property>    
    <name>dfs.namenode.servicerpc-address.cluster1.hadoop1</name>    
    <value>hadoop1:53310</value>    
  </property>    
  <property>    
    <name>dfs.namenode.servicerpc-address.cluster1.hadoop2</name>    
    <value>hadoop2:53310</value>    
  </property>  
  <!--【指定cluster1是否启动自动故障恢复，即当NameNode出故障时，是否自动切换到另一台NameNode】-->      
  <property>      
    <name>dfs.ha.automatic-failover.enabled.cluster1</name>      
    <value>true</value>      
  </property>             
    <!--指定JournalNode -->    
    <!--【指定cluster1的两个NameNode共享edits文件目录时，使用的JournalNode集群信息】  -->  
    <property>    
        <name>dfs.namenode.shared.edits.dir</name>         
        <value>qjournal://hadoop1:8485;hadoop2:8485;hadoop3:8485;hadoop4:8485;hadoop5:8485/cluster1</value>    
    </property>    
    <!--【指定cluster1出故障时，哪个实现类负责执行故障切换】-->  
    <property>    
        <name>dfs.client.failover.proxy.provider.cluster1</name>         
        <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>    
    </property>    
    <!--【指定JournalNode集群在对NameNode的目录进行共享时，自己存储数据的磁盘路径。tmp路径是自己创建，journal是启动journalnode自动生成】        -->  
    <property>        
       <name>dfs.journalnode.edits.dir</name>        
       <value>/home/tom/yarn/yarn_data/tmp/journal</value>        
    </property>    
    <!--【一旦需要NameNode切换，使用ssh方式进行操作】-->  
    <property>        
       <name>dfs.ha.fencing.methods</name>        
       <value>sshfence</value>        
    </property>  
    <!--【如果使用ssh进行故障切换，使用ssh通信时用的密钥存储的位置】-->    
   <property>        
        <name>dfs.ha.fencing.ssh.private-key-files</name>        
        <value>/home/tom/.ssh/id_rsa</value>        
    </property>      
    <property>    
        <name>dfs.ha.fencing.ssh.connect-timeout</name>    
        <value>10000</value>    
    </property>    
    <property>    
        <name>dfs.namenode.handler.count</name>    
        <value>100</value>    
   </property>    
</configuration>  

```

###3.mapred.site.xml

```xml
<configuration>
  <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
  </property>
</configuration>

```


mapreduce.framework.name:指定运行mapreduce的环境,这里指定为yarn.
官网解释:The runtime framework for executing MapReduce jobs. Can be one of local, classic or yarn

这个配置文件过于简洁，不适合学习。
实际上该配置文件涉及了mapreduce上的很多配置，对jobtracker和tasktracker还完全不懂，这里继续贴网上的资料先作补充

```xml
 1 <configuration>
 2     <property>
 3         <name> mapreduce.framework.name</name>
 4         <value>yarn</value>
 5         <description>执行框架设置为Hadoop YARN</description>
 6     </property>
 7     <property>
 8         <name>mapreduce.map.memory.mb</name>
 9         <value>1536</value>
10         <description>对maps更大的资源限制的</description>
11     </property>
12     <property>
13         <name>mapreduce.map.java.opts</name>
14         <value>-Xmx2014M</value>
15         <description>maps中对jvm child设置更大的堆大小</description>
16     </property>
17     <property>
18         <name>mapreduce.reduce.memory.mb</name>
19         <value>3072</value>
20         <description>设置 reduces对于较大的资源限制</description>
21     </property>
22     <property>
23         <name>mapreduce.reduce.java.opts</name>
24         <value>-Xmx2560M</value>
25         <description>reduces对 jvm child设置更大的堆大小</description>
26     </property>
27     <property>
28         <name>mapreduce.task.io.sort</name>
29         <value>512</value>
30         <description>更高的内存限制，而对数据进行排序的效率</description>
31     </property>
32     <property>
33         <name>mapreduce.task.io.sort.factor</name>
34         <value>100</value>
35         <description>在文件排序中更多的流合并为一次</description>
36     </property>
37     <property>
38         <name>mapreduce.reduce.shuffle.parallelcopies</name>
39         <value>50</value>
40         <description>通过reduces从很多的map中读取较多的平行副本</description>
41     </property>
42 </configuration>

```

配置jobhistroy服务器(JobHistory用来记录已经finished的mapreduce运行日志，日志信息存放于HDFS目录中)

```xml
 1 <configuration>
 2     <property>
 3         <name> mapreduce.jobhistory.address</name>
 4         <value>192.168.1.100:10200</value>
 5         <description>IP地址192.168.1.100可替换为主机名</description>
 6     </property>
 7     <property>
 8         <name>mapreduce.jobhistory.webapp.address</name>
 9         <value>192.168.1.100:19888</value>
10         <description>IP地址192.168.1.100可替换为主机名</description>
11     </property>
12     <property>
13         <name>mapreduce.jobhistory.intermediate-done-dir</name>
14         <value>/usr/local/hadoop/mr­history/tmp</value>
15         <description>在历史文件被写入由MapReduce作业</description>
16     </property>
17     <property>
18         <name>mapreduce.jobhistory.done-dir</name>
19         <value>/usr/local/hadoop/mr­history/done</value>
20         <description>目录中的历史文件是由MR JobHistoryServer管理</description>
21     </property>
22 </configuration>

```

###4.yarn-site.xml
```xml
<configuration>
    <property>
        <name>yarn.resourcemanager.address</name>
        <value>Master:8032</value>
    </property>
    <property>
        <name>yarn.resourcemanager.scheduler.address</name>
        <value>Master:8030</value> </property> <property>
        <name>yarn.resourcemanager.resource-tracker.address</name>
        <value>Master:8031</value>
    </property>
    <property>
        <name>yarn.resourcemanager.admin.address</name>
        <value>Master:8033</value>
    </property>
    <property>
        <name>yarn.resourcemanager.webapp.address</name>
        <value>Master:8088</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
     <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
    <value>org.apache.hadoop.mapred.ShuffleHandler</value>
    </property>
</configuration>

```
内容繁多，主要是配置**ResourceManger和NodeManager**，个配置文件里仅仅只是部分配置信息。


yarn.resourcemanager.address:客户端对ResourceManager主机通过 host:port 提交作业
yarn.resourcemanager.scheduler.address:ApplicationMasters通过ResourceManager主机访问host:port跟踪调度程序获取资源
yarn.resourcemanager.resource-tracker.address:NodeManagers通过ResourceManager主机访问host:port
yarn.resourcemanager.admin.address:管理命令通过ResourceManager主机访问host:port
yarn.resourcemanager.webapp.address:ResourceManager web页面host:port.
yarn.nodemanager.aux-services:Shuffle service 需要加以设置的Map Reduce的应用程序服务

###5.slaves
不多赘述，添加各个slave节点的主机名即可。
