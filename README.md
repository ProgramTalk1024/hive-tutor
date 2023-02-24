# Apache Hive教程
![](https://programtalk-1256529903.cos.ap-beijing.myqcloud.com/202302241400529.png)

# Hive入门
## 什么是Hive？
Hive是由Facebook开源，基于Hadoop的一个数据仓库工具，可以将结构化的数据文件映射为一张表，并提供类SQL查询功能。



Hive是一个Hadoop客户端，用于将HQL（Hive SQL）转化成MapReduce程序。

1. Hive中每张表的数据存储在HDFS

2. Hive分析数据底层的实现是MapReduce（也可配置为Spark或者Tez）

3. 执行程序运行在Yarn上

