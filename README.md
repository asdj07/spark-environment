---
layout: post
title: hadoop-spark-hbase-hive
date: 2018-04-09 11:22:43
tags:
---

## 软件版本

* JDK7  
* hadoop2.5.2 
* hbase0.98.24   
* spark1.5.2  
* Hive2.1.1 

## 服务器(一体机)

|    内网ip        | 外网ip | hostname |
| --------------- | -----  |  -----   |
| 192.168.195.15  |  10.0.12.105 | master |
| 192.168.195.14  |  10.0.12.106 | slave1 |
| 192.168.195.13  |  10.0.12.107 | slave2 |
| 192.168.195.12  |  10.0.12.108 | slave3 |


## 部署
#### 1、jdk安装
```sh
cd /usr/local && curl -O http://mirrors.d.com/software/jdk/jdk-8u161-linux-x64.tar.gz
tar -zxvf ./jdk-8u161-linux-x64.tar.gz
ln -s jdk1.8.0_161  jdk
rm -rf ./jdk-8u161-linux-x64.tar.gz

vim /etc/profile
export JAVA_HOME=/usr/local/jdk
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin
##source /etc/profile

java -version

```
#### 2、ntpdate时间同步

#### 3、挂载硬盘
```sh
fdisk -l
fdisk /dev/vdb
#n p  w
mkfs.ext4  /dev/vdb1
mount /dev/vdb1 /data
vim /etc/fstab
 # /dev/vdb1               /data                   ext4    defaults        0 0

```
#### 4、hosts
* 设置hostname，master slave1 slave2 slave3
* /etc/hosts
	
	```sh
		#127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4 master.novalocal
		192.168.195.15 master 
        192.168.195.12 slave1
        192.168.195.13 slave2
        192.168.195.14 slave3
	```

#### 5、ssh免密登录
```sh
cd /root/.ssh
ssh-keygen -t rsa 
```

#### 6、hadoop

* 配置

```sh
cd /usr/local && curl -O http://mirrors.d.com/software/hadoop/2.5.2/hadoop-2.5.2.tar.gz
tar -zxvf ./hadoop252.tar.gz
ln -s /usr/local/hadoop252 /usr/local/hadoop

# /etc/profile   PATH
export HADOOP_HOME=/usr/local/hadoop
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin

source /etc/profile
hadoop version

#/usr/local/hadoop/etc/hadoop
hadoop-env.sh  yarn-env.sh

core-site.xml
hdfs-site.xml
mapred-site.xml
yarn-site.xml
slaves

#格式化NameNode
hadoop namenode -format 

scp -r /usr/local/hadoop root@slave1:/usr/local

start-dfs.sh
mr-jobhistory-daemon.sh start historyserver
start-yarn.sh
hadoop-daemon.sh start secondarynamenode

```

* webUI
	* NameNode - http://10.0.12.105:50070
	* ResourceManager  - http://10.0.12.105:8088
	* MapReduce JobHistory - http://10.0.12.105:19888/jobhistory
* demo 
	
	```sh
	hadoop fs -ls /
	hadoop fs -put ./xxx /test
	hadoop jar /usr/local/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.5.2.jar wordcount /test /testout
	```



#### 7、hbase

```sh
cd /usr/local && curl -O http://mirrors.d.com/software/hbase/hbase-0.98.24-hadoop2-bin.tar.gz
tar -zxvf hbase-0.98.24-hadoop2-bin.tar.gz
ln -s hbase-0.98.24-hadoop2-bin.tar.gz hbase

# /etc/profile   PATH
export HBASE_HOME=/usr/local/hbase
export PATH=$HBASE_HOME/bin:$PATH

source /etc/profile
hbase version

# /usr/local/hbase/conf
hbase-env.sh
hbase-site.xml
regionservers
 
start-hbase.sh

```

#### 8、Spark

* scala

```
cd /usr/local &&curl -O http://mirrors.d.com/software/scala/2.11.7/scala-2.11.7.tgz
tar -zxvf scala-2.11.7.tgz
ln -s scala-2.11.7 scala
vim /etc/profile
#export SCALA_HOME=/usr/local/scala
#export PATH=$JAVA_HOME/bin:$HADOOP_HOME/bin:$SCALA_HOME/bin:$PATH
source /etc/profile

scala -version

```

* 部署,spark用的scala版本是2.10.4，好像没有用装的2.11.7的scala

```
 cd /usr/local && curl -O http://mirrors.d.com/software/spark/1.5.2/spark-1.5.2-bin-hadoop2.4.tgz
 tar -zxvf spark-1.5.2-bin-hadoop2.4.tgz
 ln -s ./spark-1.5.2-bin-hadoop2.4  ./spark
 #/etc/profile 

 #配置
 /usr/local/spark/conf/spark-env.sh
 /usr/local/spark/conf/spark-env.sh/slaves
 #注意修改对应的SPARK_LOCAL_IP

 /usr/local/spark/sbin/start-all.sh
```

* sbt安装

* 运行demo

```
#master
/usr/local/spark/bin/spark-submit --class "SimpleApp" --master spark://master:7077  /usr/local/sparkapp/target/scala-2.10/simple-project_2.10-1.0.jar

#yarn
/usr/local/spark/bin/spark-submit --class "SimpleApp" --master yarn   /usr/local/sparkapp/target/scala-2.10/simple-project_2.10-1.0.jar
```
* webUI - http://10.0.12.105:8080


#### 9、Hive

```
  #hdfs上创建目录
  $ $HADOOP_HOME/bin/hadoop fs -mkdir       /tmp
  $ $HADOOP_HOME/bin/hadoop fs -mkdir       /user/hive/warehouse
  $ $HADOOP_HOME/bin/hadoop fs -chmod g+w   /tmp
  $ $HADOOP_HOME/bin/hadoop fs -chmod g+w   /user/hive/warehouse
  
  cd /usr/local && curl -O http://mirrors.d.com/software/hive/2.1.1/apache-hive-2.1.1-bin.tar.gz
  tar -zxvf ./apache-hive-2.1.1-bin.tar.gz
  ln -s apache-hive-2.1.1-bin ./hive
  
  # /etc/profile  PATH
  
  #配置文件
  hive/conf/hive-env.sh  
  cp hive/conf/hive-default.xml.template   hive/conf/hive-site.xml
  hive/conf/hive-site.xml
  
  
  #jdbc
  cd hive/lib && curl -O http://mirrors.d.com/software/jdbc/mysql-connector-java-5.1.46-bin.jar
  
  
  hive/bin/schematool -initSchema -dbType mysql
  hive/bin/hive
  
```