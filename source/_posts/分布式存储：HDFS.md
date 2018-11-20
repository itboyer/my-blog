---
title: 分布式存储：HDFS
date: 2017-07-10 16:36:26
tags: 
 - 分布式
 - HDFS
top_img: /img/905584.jpg
---

###分布式存储：HDFS

[TOC]



##### NameNode(名称节点)

**一、维护HDFS文件系统,是HDFS的主节点【相当于MVC的控制器,进行任务分发】**

###### #HDFS结构图

![image](img/HDFS structure.jpg)







**二、接受客户端的请求：上传文件、下载文件、创建目录等等**

###### 文件上传流程图

![image](img/HDFS fileup.png)

###### 文件下载流程图

![image](img/HDFS filedown.png)



  

**三、负责元信息(meta data)的维护 fsimage文件**

  >eg: 倒排索引 ---> HDFS的元信息
  >​	(a) NameNode会在内存中，缓存1000M的元信息(大概：一个文件的元信息150字节)
  >​	
  >​	//更改：
  >​	/root/training/hadoop-2.4.1/etc/hadoop/hadoop-env.sh
  >​	44 # The maximum amount of heap to use, in MB. Default is 1000.
  >​	46 #export HADOOP_NAMENODE_INIT_HEAPSIZE="" 
  >​				 
  >​	(b)采用LRU（最近最少使用算法）算法，替换内存中的元信息。【被替换掉的文件保存在fimage文件中】
  >​	
  >​	路径/root/training/hadoop-2.4.1/tmp/dfs/name/current
  >​	(c) fsimage 元信息的保存文件(二进制)

可以使用hdfs oiv -i 命令将二进制日志输出为文本和XML

![image](img/hdfs oiv.png)

**四、维护HDFS的日志(edits文件，二进制)，保存了HDFS最新的状态**

​	Edits文件保存了自最后一次检查点之后所有针对HDFS文件系统的操作，比如：增加文件、重命名文件、删除目录等等

​	保存目录：/root/training/hadoop-2.4.1/tmp/dfs/name/current  ![image](img/edits file.png)

可以使用hdfs oev -i命令将二进制日志文件格式化输出为XML格式文件

![image](img/hdfs oev.png)

##### DateNode（数据节点）

1. 以数据块为单位，保存数据

   ​	1）Hadoop1.x的数据块大小64M

   ​	2）Hadoop2.x的数据块大小128M

2. 在全分布模式下至少两个DateNode节点

3. 数据保存目录：有hadoop.tmp.dir参数指定

   ​	例如：
   ![image](img/Hadoop.tmp.dir.png)


##### Secondary NameNode（第二名称节点）

1. 主要是用于日志合并

2. 日志合并的过程

![image](img/SecondaryNameNode mesh.png)

##### HDFS存在的问题

1）NameNode单点故障，难以应用二在线场景

​	解决方案：Hadoop1.0中，没有解决方案

​			   Hadoop2.0中，使用Zookeeper实现NameNode的HA功能

2）NameNode压力过大，且受内存限制，影响系统的扩展性

​	解决方案：Hadoop1.0中，没有解决方案

​			   Hadoop2.0中，使用NameNode的联盟实现其水平扩展