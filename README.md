# Apache Hive教程
![](https://programtalk-1256529903.cos.ap-beijing.myqcloud.com/202302241400529.png)

# Hive入门
## 什么是Hive？
Hive是由Facebook开源，基于Hadoop的一个数据仓库工具，可以将结构化的数据文件映射为一张表，并提供类SQL查询功能。



Hive是一个Hadoop客户端，用于将HQL（Hive SQL）转化成MapReduce程序。

1. Hive中每张表的数据存储在HDFS

2. Hive分析数据底层的实现是MapReduce（也可配置为Spark或者Tez）

3. 执行程序运行在Yarn上

# Hive安装部署说明
本文中，hive版本是3.1.2，该版本hive要求，hadoop版本必须是3.x.y，jdk版本8。

## 准备
按照如下几步准备：

1. 准备一台服务器，Linux系统，我使用的是Centos Stream 9。我通过Vagrant + Virtualbox搭建一个虚拟机。

   **Vagrantfile:**

   ```text
   Vagrant.configure("2") do |config|
     config.vm.box = "boxomatic/centos-stream-9"
     config.vm.network "private_network", ip: "192.168.33.11"
     config.vm.provider "virtualbox" do |vb|
       vb.name = "hive"
       vb.gui = true
       vb.memory = "2048"
     end
   end
   ```

   使用`vagrant up`创建虚拟机（要进入Vagrantfile所在文件夹哦）。

   后面需要将下载下来的hadoop、hive等安装包传到虚拟机中，所以要安装vagrant的scp插件。

   ```shell
   vagrant plugin install vagrant-scp
   ```

   

2. 下载Hive安装包，并上传到虚拟机

   [Hive下载](https://dlcdn.apache.org/hive/hive-3.1.3/apache-hive-3.1.3-bin.tar.gz)

   ```shell
   vagrant scp apache-hive-3.1.3-bin.tar.gz :/home/vagrant
   ```

​		

3. 下载Hadoop安装包并上传到虚拟机
   [Hadoop下载](https://www.apache.org/dyn/closer.cgi/hadoop/common/hadoop-3.3.4/hadoop-3.3.4.tar.gz)

   ```shell
   vagrant scp hadoop-3.3.4.tar.gz :/home/vagrant
   ```

   

4. 安装JDK

   ```shell
   [vagrant@centos ~]$ sudo yum install -y java-1.8.0-openjdk*
   ```

   一般使用yum安装不需要配置环境变量（自动配置好了），如果需要参考一下配置JDK的环境变量。

   `sudo vim /etc/profile`

   ```
   export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.362.b08-3.el9.x86_64
   export PATH=$JAVA_HOME/bin:$PATH
   ```

​		执行`source /etc/profile`使其生效。

## Hive安装

## 配置Hadoop
解压并配置hadoop环境变量。

`sudo vim /etc/profile`

```shell
export HADOOP_HOME=/home/vagrant/hadoop-3.3.4
export PATH=$HADOOP_HOME/bin:$PATH
```
执行`source /etc/profile`使其生效。

## 安装Hive

解压并配置Hive环境变量。

配置环境变量

`sudo vim /etc/profile`

```shell
export HIVE_HOME=/home/vagrant/apache-hive-3.1.3-bin
export PATH=$HIVE_HOME/bin:$PATH
```

执行`source /etc/profile`使其生效。

# 配置

hive默认提供了很多配置，在conf目录下。

```shell
[vagrant@centos apache-hive-3.1.3-bin]$ ls conf/
beeline-log4j2.properties.template  hive-env.sh.template                  hive-log4j2.properties.template  llap-cli-log4j2.properties.template
hive-default.xml.template           hive-exec-log4j2.properties.template  ivysettings.xml                  llap-daemon-log4j2.properties.template
```

基本都是以`.template`结尾的文件，其中`hive-default.xml.template`是比较重要的一个配置，这个hive的配置文件（里面都是hive的默认配置，实际上hive根本不使用`hive-default.xml.template `这个文件），如果想覆盖配置，需要创建一个新的文件`hive-site.xml`，并重写配置值。

hive-site.xml

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
  <property>
    <!-- hive的元数据存储目录 -->
    <name>hive.metastore.warehouse.dir</name>
    <!-- 默认在/user/hive/warehouse路径下，我修改为/home/vagrant/hive/warehouse -->
    <value>/home/vagrant/hive/warehouse</value>
    <description>location of default database for the warehouse</description>
  </property>
</configuration>
```



## 启动

首先要初始化元数据。

初始化Hive的元数据，如果不初始化执行命令会出现类似错误提示：

```shell
hive> show databases;
FAILED: HiveException java.lang.RuntimeException: Unable to instantiate org.apache.hadoop.hive.ql.metadata.SessionHiveMetaStoreClient
```

使用`schematool`进行初始化。

```shell
[vagrant@centos ~]$ schematool -dbType derby -initSchema
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/home/vagrant/apache-hive-3.1.3-bin/lib/log4j-slf4j-impl-2.17.1.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/home/vagrant/hadoop-3.3.4/share/hadoop/common/lib/slf4j-reload4j-1.7.36.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]
Metastore connection URL:        jdbc:derby:;databaseName=metastore_db;create=true
Metastore Connection Driver :    org.apache.derby.jdbc.EmbeddedDriver
Metastore connection User:       APP
Starting metastore schema initialization to 3.1.0
Initialization script hive-schema-3.1.0.derby.sql


Error: FUNCTION 'NUCLEUS_ASCII' already exists. (state=X0Y68,code=30000)
org.apache.hadoop.hive.metastore.HiveMetaException: Schema initialization FAILED! Metastore state would be inconsistent !!
Underlying cause: java.io.IOException : Schema script failed, errorcode 2
Use --verbose for detailed stacktrace.
*** schemaTool failed ***
```

失败了！





```shell
[vagrant@centos ~]$ hive
/usr/bin/which: no hbase in (/home/vagrant/apache-hive-3.1.3-bin/bin:/home/vagrant/hadoop-3.3.4/bin:/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.362.b08-3.el9.x86_64/bin:/home/vagrant/apache-hive-3.1.3-bin/bin:/home/vagrant/hadoop-3.3.4/bin:/home/vagrant/hadoop-3.3.4/bin:/home/vagrant/.local/bin:/home/vagrant/bin:/usr/share/Modules/bin:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin)
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/home/vagrant/apache-hive-3.1.3-bin/lib/log4j-slf4j-impl-2.17.1.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/home/vagrant/hadoop-3.3.4/share/hadoop/common/lib/slf4j-reload4j-1.7.36.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]
Hive Session ID = 70d889bc-f4db-41a7-86a1-d0bc342b1666

Logging initialized using configuration in jar:file:/home/vagrant/apache-hive-3.1.3-bin/lib/hive-common-3.1.3.jar!/hive-log4j2.properties Async: true
Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
hive>
```





