#  Hadoop HA集群配置

## HA架构
![Alt text](../img/framework.png)

## 集群规划

<table>
  <thead>
    <tr>
      <th>主机名</th>
      <th>安装的软件</th>
      <th>进程</th>
	  <th>角色</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>master</td>
      <td>hadoop、zookeeper</td>
      <td>NameNode、ResourceManager、QuorumPeerMain、JournalNode、DFSZKFailoverController、JobHistoryServer</td>
      <td>NameNode(主)、ResourceManager(主)</td>
    </tr>
    <tr>
      <td>slave1</td>
      <td>hadoop、zookeeper</td>
      <td>NameNode、ResourceManager、QuorumPeerMain、JournalNode、DFSZKFailoverController、DataNode、NodeManager、JobHistoryServer</td>
      <td>NameNode(备)、ResourceManager(备)</td>
    </tr>
    <tr>
      <td>slave2</td>
      <td>hadoop、zookeeper</td>
      <td>QuorumPeerMain、JournalNode、DataNode、NodeManager</td>
      <td>DataNode</td>
    </tr>
  </tbody>
</table>

## 初始化
1. 启动zookeeper集群，分别在安装了zookeeper的master、slave1、slave2上执行 zkServer.sh start
2. 格式化ZKFC，在zookeeper中生成HA节点，在master执行 hdfs zkfc -formatZK
3. 启动journalnode，分别在master、slave1、slave2上执行 hadoop-daemon.sh start journalnode
4. 格式化namenode,在主namenode master执行 hdfs namenode -format
5. 将格式化后的主namenode节点master的hadoop工作目录中的元数据目录复制到备namenode slave1上。scp -r /home/grid/hadoop-2.7.2/hdfs/name  grid@slave1:/home/grid/hadoop-2.7.2/hdfs/name
6. 初始化完毕后可关闭journalnode，分别在master、slave1、slave2 hadoop-daemon.sh stop journalnode

## 启动
1. 启动zookeeper集群，分别在master、slave1、slave2上执行 zkServer.sh start
2. 启动HDFS,主namenode(master)上执行 start-dfs.sh。此命令分别在主namenode(master)/备namenode(slave1)上启动NameNode和DFSZKFailoverController，分别在master、slave1、slave2启动JournalNode,在slave1和slave2启动DataNode
3. 启动YARN，在主YARN节点master执行start-yarn.sh，此时在master节点启动ResourceManager，在slave1和slave2节点启动NodeManager
4. 启动备YARN，yarn-daemon.sh start resourcemanager，slave1节点启动ResourceManager
5. 启动YARN安全代理(可以省略)，proxyserver充当防火墙的角色，可以提高访问集群的安全性，根据yarn-site.xml中的配置在相应节点执行 yarn-daemon.sh start proxyserver
6. 在master和slave1上执行 mr-jobhistory-daemon.sh start historyserver，启动历史任务服务

## 恢复
#### 恢复NameNode
1. 启动Zookeeper zkServer.sh start
2. hadoop-daemon.sh start journalnode
3. 在备NameNode上同步主NameNode的元数据，在备NameNode执行hdfs namenode -bootstrapStandby
4. hadoop-daemon.sh start zkfc
5. hadoop-daemon.sh start namenode
6. yarn-daemon.sh start resourcemanager

#### 恢复备YARN
1. yarn-daemon.sh start resourcemanager


## 测试
1. hdfs dfs  -mkdir /in
2. hdfs dfs -put a.txt /in
3. hadoop jar /home/grid/hadoop-2.7.2/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar wordcount /in /out1
	







