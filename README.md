# hadoop集群配置

## 软件列表
*  hadoop2.7

## 搭建环境
* CentOS-7-x86_64-Minimal
* VMware
* master  192.168.75.128
* slave1  192.168.75.129
* slave2  192.168.75.130

## 配置
	搭建好master,然后克隆slave1,slave2。以下所有操作均在root用户下操作，创建hadoop专门用户。

### VMWare设置

####  VMware中centos静态IP配置

##### 关闭DHCP
    Edit->Virtual Network Editor
	选择VMnet8，去掉Use local DHCP service to distribute IP address to VMs选项。点击NAT Settings查看一下GATEWAY地址192.168.75.2。

##### 设置centos静态IP
	涉及到的配置文件，分别是：
	/etc/sysconfig/network
	/etc/sysconfig/network-scripts/ifcfg-eth0

	首先修改/etc/sysconfig/network如下：
	NETWORKING=yes
	HOSTNAME=master
	GATEWAY=192.168.75.2
	指定网关地址

	然后修改/etc/sysconfig/network-scripts/ifcfg-eth0：
	修改如下四项
	BOOTPROTO="static"
	IPADDR=192.168.75.128
	NETMASK=255.255.255.0
	DNS1=192.168.75.2

### linux设置

#### 修改host
	设置主机名hostname,在/etc/sysconfig/network设置。如HOSTNAME=master。
	
	设置主机名和IP地址的映射关系。在/etc/hosts中设置。
	注释掉127.0.1.1     ubuntu.localdomain 
	增加ip和hostname对应关系
	192.168.75.128 master
	192.168.75.129 slave1
	192.168.75.130 slave2

	修改/etc/hostname为master
#### 配置SSH
		确保系统已经安装好相关的SSHD服务，并启动（CentOS默认已经安装好）。
	需要实现 Master到所有的Slave以及所有Slave 到Master的SSH无密码登录（集群模式），伪分布式一样也要设置。
	Master作为客户端，与服务端slave匹配。
		“公私钥“认证方式简单的解释:首先在客户端上创建一对公私钥 （公钥文件：~/.ssh/id_rsa.pub； 
	私钥文件：~/.ssh/id_rsa）。然后把公钥放到服务器上（~/.ssh/authorized_keys）, 自己保留好私钥.在使用ssh登录时，
	ssh程序会发送私钥去和服务器上的公钥做匹配.如果匹配成功就可以登录了。
		在hadoop启动后，Namenode是通过SSH（Secure Shell）来启动和停止各个datanode上的各种守护进程的，

	这就需要在几点之间执行指令的时候是不需要输入密码的，故我们需要配置SSH运用无密码公钥认证的行事
	查看是否安装 rpm -qa | grep ssh
	使用root登录修改配置文件：/ect/ssh/sshd_config，将其中的三行注视去掉
	RSAAuthentication yes
	PubkeyAuthentication yes
	AuthorizedKeysFile	.ssh/authorized_keys
	重启ssh服务：service sshd restart。最后退出root。
	一下操作都在hadoop专门的用户下进行。
##### 本机无密码登录
	ssh-keygen -t rsa
	-P表示密码，-P '' 就表示空密码，也可以不用-P参数，这样就要三车回车，用-P就一次回车。
	密钥文件按照默认方式，在主目录/home/hanmeng下的隐藏目录.ssh中生成，
	分别为id_dsa和id_dsa.pub（公钥）
	根据配置文件/etc/ssh/sshd_config中的AuthorizedKeysFile项的取值：.ssh/authorized_keys，公钥需要导入到该文件中才能实现校验，如下：
 	cat id_dsa.pub >> authorized_keys
	将id_rsa.pub的内容追加到authorized_keys 中
##### 修改权限
	chmod 644 authorized_keys  #一定是644 不是777

##### 远程SSH
		设置完本机无密码ssh登陆后，使用ssh-copy-id命令将公钥传送到远程主机上
	ssh-copy-id user@hostname
### JDK
	jkd一定是1.7以上
### 配置环境变量
	JAVA_HOME
	HADOOP_HOME
	source /etc/profile  让环境变量生效
### 禁用IPV6

	打开/etc/sysctl.conf，添加如下信息
	#disable ipv6 
	net.ipv6.conf.all.disable_ipv6 = 1 
	net.ipv6.conf.default.disable_ipv6 = 1 
	net.ipv6.conf.lo.disable_ipv6 = 1

	重启机器
	用这个命令查看是否成功，为1是禁用了
	$ cat /proc/sys/net/ipv6/conf/all/disable_ipv6

# hadoop2.7
	见配置文件夹hadoop。


	
	







