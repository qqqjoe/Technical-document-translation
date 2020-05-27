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

