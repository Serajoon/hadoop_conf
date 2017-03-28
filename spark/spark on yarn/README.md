#  spark on yarn集群配置

## 软件列表
*  jdk1.8
*  scala2.11.8
*  hadoop2.7.2
*  hive1.2.1
*  spark2.1.0

## 软件规划

<table>
  <thead>
    <tr>
      <th>主机名</th>
      <th>操作系统</th>
      <th>IP地址</th>
      <th>角色</th>
      <th>安装软件</th>
      <th>服务</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>master</td>
      <td>CentOS-7-x86_64-Minimal</td>
      <td>210.25.24.68</td>
      <td>master</td>
      <td>hadoop、hive、spark</td>
      <td>NameNode、ResourceManager、RunJar(hive)、Master</td>
    </tr>
    <tr>
      <td>slave1</td>
      <td>CentOS-7-x86_64-Minimal</td>
      <td>210.25.24.85</td>
      <td>slave</td>
      <td>hadoop、spark</td>
      <td>DataNode、NodeManager、Worker</td>
    </tr>
    <tr>
      <td>slave2</td>
      <td>CentOS-7-x86_64-Minimal</td>
      <td>210.25.24.86</td>
      <td>slave</td>
      <td>hadoop、spark</td>
      <td>DataNode、NodeManager、Worker</td>
    </tr>
  </tbody>
</table>

## 配置
    搭建hadoop环境。以下所有操作均在root用户下操作，创建hadoop专门用户。

#### 环境变量
    配置scala环境变量，在每台机器上安装配置scala(依赖jdk)
    #scala
    export SCALA_HOME=/home/grid/scala-2.11.8
    export PATH=$SCALA_HOME/bin:$PATH
	
    #spark
    export SPARK_HOME=/home/grid/spark-2.1.0
    export PATH=$PATH:$SPARK_HOME/bin

# master

## spark

#### spark-env.sh
	cp spark-env.sh.template spark-env.sh
	#增加
	export JAVA_HOME=/home/grid/java
	export SCALA_HOME=/home/grid/scala-2.11.8
	export SPARK_MASTER_IP=210.25.24.68
	export SPARK_LOG_DIR=/home/grid/spark-2.1.0/datafile/logs
	export SPARK_PID_DIR=/home/grid/spark-2.1.0/datafile/tmp
	
	export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
	#由于hive只安装在了master上，所以只有在master的spark配置中才增加hive相关的配置
	export HIVE_HOME=/home/grid/hive
	export HIVE_CONF_DIR=$HIVE_HOME/conf
	export SPARK_CLASSPATH=$SPARK_CLASSPATH:$HIVE_HOME/lib/mysql-connector-java-5.1.36-bin.jar
#### hive-site.xml
	将hive-site.xml复制到$SPARK_HOME/conf/
#### slaves
	slave1
	slave2

# slave1/slave2

## spark

#### spark-env.sh
	cp spark-env.sh.template spark-env.sh
	export JAVA_HOME=/home/grid/java
	export SCALA_HOME=/home/grid/scala-2.11.8
	export SPARK_MASTER_IP=210.25.24.68
	export SPARK_LOG_DIR=/home/grid/spark-2.1.0/datafile/logs
	export SPARK_PID_DIR=/home/grid/spark-2.1.0/datafile/tmp
	
	export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
	#由于slave1和slave2上没有安装hive,不用配置hive

#### slaves
	slave1
	slave2

	







