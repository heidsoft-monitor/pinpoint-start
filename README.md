# start pinpoint 
```
pinpoint 组件说明
web 
agent 
collector 

安装hbase以及pinpoint
10.211.55.3  hbase pinpoint-web  pinpoint-collector  

安装zookeeper 
10.211.55.4  zoo1
10.211.55.5	zoo2

安装软件选用稳定版,本次不使用hbase 自带zookeeper管理功能
hbase-1.2.6-bin.tar.gz
zookeeper-3.4.10.tar.gz
apache-tomcat-8.5.20.tar.gz
pinpoint-agent-1.6.2.tar.gz
pinpoint-collector-1.6.2.war
pinpoint-web-1.6.2.war

```

## hbase 配置
```
<configuration>
	<property>
		<name>hbase.rootdir</name>
		<value>file:///usr/local/hbase-1.2.6/data</value>
	</property>
	
	<property>
		<name>hbase.cluster.distributed</name>
		<value>true</value>
		<description>
		false: 单机模式或者为分布式模式
		true: 完全分布式
		</description>
	</property>
	
	<property>
	  <name>hbase.zookeeper.property.clientPort</name>
	  <value>2181</value>
	</property>
	
	<property>
	  <name>hbase.zookeeper.quorum</name>
	  <value>zoo1,zoo2</value>
	</property>
	
	<property>
	  <name>hbase.zookeeper.property.dataDir</name>
	  <value>/tmp/zookeeper</value>
	</property>
	
	<property>
	    <name>hbase.master.info.port</name>
	    <value>60010</value>
	</property>

</configuration>

# 设置hbase不自动管理zk
export HBASE_MANAGES_ZK=false
```

## hbase 初始化数据库
```
hbase shell hbase-create.hbase
```

## zookeeper 安装
```
从apche 官方下载稳定版，zookeeper-3.4.10.tar.gz 解压到安装目录
个人喜欢放到 /usr/local目录中

可以使用进行解压
tar -xvf  zookeeper-3.4.10.tar.gz -C /usr/local

```

## zookeeper 配置
```
本此基于zookeeper 真实集群，进行配置

集群采用本地dns配置，IP与主机名的映射

heidsoft@heidsoft:~$ cat /etc/hosts
127.0.0.1	localhost
127.0.1.1	heidsoft.localdomain	heidsoft

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

10.211.55.4 zoo1
10.211.55.5 zoo2




zoo.cfg 配置说明
heidsoft@heidsoft:/usr/local/zookeeper-3.4.10/conf$ cat zoo.cfg
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just
# example sakes.
dataDir=/tmp/zookeeper
# the port at which the clients will connect
clientPort=2181
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1

#此处配置,需要注意，server后面的id是根据myid来确认的，创建方式是
echo "1" > /tmp/zookeeper/myid 对应zoo1这台服务器
echo "2" > /tmp/zookeeper/myid 对应zoo2这对服务器

server.1=zoo1:2888:3888
server.2=zoo2:2888:3888

```

## pinpoint配置
```
tomcat 修改端口
sed -i 's/port="8005"/port="18005"/g' server.xml
sed -i 's/port="8080"/port="18080"/g' server.xml
sed -i 's/port="8443"/port="18443"/g' server.xml
sed -i 's/port="8009"/port="18009"/g' server.xml
sed -i 's/redirectPort="8443"/redirectPort="18443"/g' server.xml
sed -i "s/localhost/`ifconfig eth0 | grep 'inet addr' | awk '{print $2}' | awk -F: '{print $2}'`/g" server.xml
```


## 参考链接

[pinpoint安装参考](http://www.cnblogs.com/yyhh/p/6106472.html)