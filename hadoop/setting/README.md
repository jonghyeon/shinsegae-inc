# hadoop setting
## 설정파일





###### $HADOOP_HOME/etc/hadoop/core-site.xml
```
<configuration>
   <property>
        <name>fs.defaultFS</name>
        <value>hdfs://hadoop01:9000</value>
   </property>
   <property>
        <name>hadoop.tmp.dir</name>
        <value>/data/hadoop/tmp</value>
   </property>
</configuration>
```

###### $HADOOP_HOME/etc/hadoop/hdfs-site.xml
```
<configuration>
    <property>
         <name>dfs.replication</name>
         <value>1</value>
    </property>
    <property>
         <name>dfs.namenode.name.dir</name>
         <value>/data/hadoop/dfs/name</value>
         <final>true</final>
    </property>
    <property>
         <name>dfs.namenode.checkpoint.dir</name>
         <value>/data/hadoop/dfs/namesecondary</value>
         <final>true</final>
    </property>
    <property>
         <name>dfs.http.address</name>
         <value>hadoop01:50070</value>
    </property>
    <property>
         <name>dfs.secondary.http.address</name>
         <value>hadoop01:50090</value>
    </property>
    <property>
         <name>dfs.datanode.data.dir</name>
         <value>/data/hadoop/dfs/data</value>
         <final>true</final>
    </property>
    <property>
         <name>dfs.permissions</name>
         <value>false</value>
    </property>
</configuration>
```
###### $HADOOP_HOME/etc/hadoop/mapred-site.xml
```
<configuration>
    <property>
         <name>mapreduce.framework.name</name>
         <value>yarn</value>
    </property>
    <property>
         <name>mapred.local.dir</name>
         <value>/data/hadoop/hdfs/mapred</value>
    </property>
    <property>
         <name>mapred.system.dir</name>
         <value>/data/hadoop/hdfs/mapred</value>
    </property>
</configuration>
```
###### $HADOOP_HOME/etc/hadoop/yarn-site.xml
```
<configuration>
     <property>
           <name>yarn.nodemanager.aux-services</name>
           <value>mapreduce_shuffle</value>
     </property>
     <property>
           <name>yarn.nodemanager.aux-services.mapreduce_shuffle.class</name>
           <value>org.apache.hadoop.mapred.ShuffleHandler</value>
     </property>
     <property>
            <name>yarn.resourcemanager.resource-tracker.address</name>
            <value>hadoop01:8025</value>
     </property>
     <property>
            <name>yarn.resourcemanager.scheduler.address</name>
            <value>hadoop01:8030</value>
     </property>
     <property>
            <name>yarn.resourcemanager.address</name>
            <value>hadoop01:8035</value>
     </property>
</configuration>
```
