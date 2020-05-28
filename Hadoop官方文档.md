# 第一节 设置单节点集群
## 目的
这个文档描述了如何设置和配置单节点的安装，以便您可以使用Hadoop MapReduce和Hadoop分布式文件系统（HDFS）进行快速的简单操作。
## 准备
### 平台支持
* 支持GNU/Linux作为开发的生产平台。Hadoop已经在具有2000个GNU/Linux集群进行了演示。
* Windows平台同样得到了支持，但是以下的步骤只针对于Linux。如果想在Windows平台设置Hadoop，请参考[wiki页面](http://wiki.apache.org/hadoop/Hadoop2OnWindows)。
### 需要的软件
对于Linux需要的软件包括：
1. 必须安装Java。推荐的Java版本在[HadoopJavaVersions](http://wiki.apache.org/hadoop/HadoopJavaVersions)中有介绍。
2. 如果需要使用可选的启动和停止脚本，必须安装ssh且sshd必须运行用于使用管理远程Hadoop守护进程/系统服务进程的Hadoop脚本。另外，建议安装pdsh以更好地进行资源管理。
### 安装软件
如果您的集群没有安装必需的软件，则需要安装它。
以Ubuntu Linux为例
```
$ sudo apt-get install ssh
$ sudo apt-get install pdsh
```
## 下载
为了获得Hadoop的发行版，从任意一个[Apache Download Mirrors](http://www.apache.org/dyn/closer.cgi/hadoop/common/)下载最新的稳定发行版即可。
## 准备启动Hadoop集群
解压缩下载的Hadoop压缩包。在发行版中，编辑 ```etc/hadoop/hadoop-env.sh``` 文件以定义一些参数如下：
```
# set to the root of your Java installation
export JAVA_HOME=/usr/java/latest
```
尝试下面的命令：
```
$ bin/hadoop
```
这将显示Hadoop脚本的用法文档。
现在，您可以支持的三种模式之一启动Hadoop集群：
* [本地/独立模式](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html#Standalone_Operation) Local (Standalone) Mode
* [伪分布式模式](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html#Pseudo-Distributed_Operation) Pseudo-Distributed Mode
* [全分布式模式](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html#Pseudo-Distributed_Operation) Fully-Distributed Mode
## 独立运行
默认情况下，Hadoop被设置为作为一个单独的Java进程以独立模式运行。这对调试很有用。
下面的例子复制了解压缩后的conf目录作为输入，然后根据给定正则表达式找到并显示每一个匹配项。输出被写入给定的输出目录中。
```
$ mkdir input
$ cp etc/hadoop/*.xml input
$ bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.2.1.jar grep input output 'dfs[a-z.]+'
$ cat output/*
```
## 伪分布式运行
Hadoop可以在单节点中以伪分布式运行，其中每一个Hadoop守护进程作为一个独立的Java进程运行。
### 配置
使用以下内容：

```etc/hadoop/core-site.xml:``` 
```
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
</configuration>
```
```etc/hadoop/hdfs-site.xml:``` 
```
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>
```
### 设置无密码ssh
现在确认您可以无需密码通过ssh连接到本地主机
```
$ ssh localhost
```
如果您不能通过无密码的ssh连接到本地主机，执行下面的命令：
```
$ ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
$ chmod 0600 ~/.ssh/authorized_keys
```
### 执行
以下说明是在本地运行MapReduce作业。如果您需要执行YARN作业，请参考[YARN on Single Node](https://hadoop.apache.org/docs/r3.2.1/hadoop-project-dist/hadoop-common/SingleCluster.html#YARN_on_Single_Node)
1. 格式化文件系统：
```
$ bin/hdfs namenode -format
```
2. 启动NameNode和DataNode守护进程：
```
$ sbin/start-dfs.sh
```
Hadoop守护进程输出日志被写入```$ HADOOP_LOG_DIR``` 目录（默认为```$ HADOOP_HOME / logs``` ）
3. 浏览NameNode的网页界面；默认的获取方式：
* NameNode - http://localhost:9870/
4. 设置执行MapReduce作业所需要的HDFS目录：
```
$ bin/hdfs dfs -mkdir /user
$ bin/hdfs dfs -mkdir /user/<username>
```
5. 将输入文件复制到分布式文件系统中：
```
$ bin/hdfs dfs -mkdir input
$ bin/hdfs dfs -put etc/hadoop/*.xml input
```
6. 运行提供的一些示例：
```
$ bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.2.1.jar grep input output 'dfs[a-z.]+'
```
7. 检查输出文件：从分布式文件系统中复制输出文件到本地文件系统中并检查它们：
```
$ bin/hdfs dfs -get output output
$ cat output/*
```
或者
在分布式文件系统中检查输出文件：
```
$ bin/hdfs dfs -cat output/*
```
8. 完成后，使用以下命令停止守护进程：
```
$ sbin/stop-dfs.sh
```
### 在单节点上的YARN
您可以通过设置一些参数在伪分布式的YARN上运行MapReduce作业，另外可以运行ResourceManager和NodeManager守护进程。
以下指令假定[上述指令](https://hadoop.apache.org/docs/r3.2.1/hadoop-project-dist/hadoop-common/SingleCluster.html#Execution)中的1.~4.已经执行完成。
1. 如下配置参数：
```
etc/hadoop/mapred-site.xml:
```
```
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <property>
        <name>mapreduce.application.classpath</name>
        <value>$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/*:$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/lib/*</value>
    </property>
</configuration>
```

```
etc/hadoop/yarn-site.xml:
```
```
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.env-whitelist</name>
        <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
    </property>
</configuration>
```
2. 启动ResourceManager和NodeManager守护进程：
```
$ sbin/start-yarn.sh
```
3. 浏览ResourceManager的网页接口；默认方式如下：
* ResourceManager - http://localhost:8088/
4. 执行一个MapReduce作业。
5. 完成后，使用以下命令结束守护进程：
```
$ sbin/stop-yarn.sh
```
## 全分布式运行
有关设置全分布式，复杂集群的信息，请参考[集群设置](https://hadoop.apache.org/docs/r3.2.1/hadoop-project-dist/hadoop-common/ClusterSetup.html) 
