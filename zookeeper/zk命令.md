## 一、常用命令

### 启动zk服务

 ```properties
[root@localhost bin]# ./zkServer.sh
ZooKeeper JMX enabled by default
Using config: /usr/home/zookeeper-3.4.11/bin/../conf/zoo.cfg
Usage: ./zkServer.sh {start|start-foreground|stop|restart|status|upgrade|print-cmd}
# 提示要以./zkCli.sh start 启动zk
./zkCli.sh start 
 ```

### 查看zk的运行状态

```properties
[root@localhost bin]# ./zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /usr/home/zookeeper-3.4.11/bin/../conf/zoo.cfg
Mode: leader
```

### 客户端连接zk

```properties
[root@localhost bin]# ./zkCli.sh 
......
WatchedEvent state:SyncConnected type:None path:null
[zk: localhost:2181(CONNECTED) 0]
```

### help 查看客户端帮助命令

```properties
[zk: localhost:2181(CONNECTED) 0] help
ZooKeeper -server host:port cmd args
    stat path [watch]
    set path data [version]
    ls path [watch]
    delquota [-n|-b] path
    ls2 path [watch]
    setAcl path acl
    setquota -n|-b val path
    history 
    redo cmdno
    printwatches on|off
    delete path [version]
    sync path
    listquota path
    rmr path
    get path [watch]
    create [-s] [-e] path data acl
    addauth scheme auth
    quit 
    getAcl path
    close 
    connect host:port
[zk: localhost:2181(CONNECTED) 1] 
```

### ls查看

`ls` 查看命令(`niocoder`是我测试集群创建的节点，默认只有`zookeeper`一个节点) 

```properties
[zk: localhost:2181(CONNECTED) 1] ls /
[niocoder, zookeeper]
[zk: localhost:2181(CONNECTED) 2] ls /zookeeper 
[quota]
[zk: localhost:2181(CONNECTED) 4] ls /zookeeper/quota
[]
```

### get获取节点数据和更新信息

- get 内容为空
- cZxid ：创建节点的id
- ctime ： 节点的创建时间
- mZxid ：修改节点的id
- mtime ：修改节点的时间
- pZxid ：子节点的id
- cversion : 子节点的版本
- dataVersion ： 当前节点数据的版本
- aclVersion ：权限的版本
- ephemeralOwner ：判断是否是临时节点
- dataLength ： 数据的长度
- numChildren ：子节点的数量

```properties
[zk: localhost:2181(CONNECTED) 7] get /zookeeper #下面空行说明节点内容为空

cZxid = 0x0
ctime = Thu Jan 01 00:00:00 UTC 1970
mZxid = 0x0
mtime = Thu Jan 01 00:00:00 UTC 1970
pZxid = 0x0
cversion = -1
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 0
numChildren = 1
[zk: localhost:2181(CONNECTED) 8]
```

### stat获得节点的更新信息

```properties
[zk: localhost:2181(CONNECTED) 8] stat /zookeeper
cZxid = 0x0
ctime = Thu Jan 01 00:00:00 UTC 1970
mZxid = 0x0
mtime = Thu Jan 01 00:00:00 UTC 1970
pZxid = 0x0
cversion = -1
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 0
numChildren = 1
```

### ls2  ls命令和stat命令的结合

```properties
[zk: localhost:2181(CONNECTED) 10] ls2 /zookeeper
[quota]
cZxid = 0x0
ctime = Thu Jan 01 00:00:00 UTC 1970
mZxid = 0x0
mtime = Thu Jan 01 00:00:00 UTC 1970
pZxid = 0x0
cversion = -1
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 0
numChildren = 1
[zk: localhost:2181(CONNECTED) 11] 
```

### create创建节点

`create [-s] [-e] path data acl` 可以注意一下各个版本的变化 

```properties
#创建merryyou节点，节点的内容为merryyou
[zk: localhost:2181(CONNECTED) 1] create /merryyou merryyou
Created /merryyou
#获得merryyou节点内容
[zk: localhost:2181(CONNECTED) 3] get /merryyou
merryyou
cZxid = 0x200000004
ctime = Sat Jun 02 14:20:06 UTC 2018
mZxid = 0x200000004
mtime = Sat Jun 02 14:20:06 UTC 2018
pZxid = 0x200000004
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 8
numChildren = 0
```

#### create -e 创建临时节点

```properties
#创建临时节点
[zk: localhost:2181(CONNECTED) 4] create -e  /merryyou/temp merryyou
Created /merryyou/temp
[zk: localhost:2181(CONNECTED) 5] get /merryyou
merryyou
cZxid = 0x200000004
ctime = Sat Jun 02 14:20:06 UTC 2018
mZxid = 0x200000004
mtime = Sat Jun 02 14:20:06 UTC 2018
pZxid = 0x200000005
cversion = 1
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 8
numChildren = 1
[zk: localhost:2181(CONNECTED) 6] get /merryyou/temp
merryyou
cZxid = 0x200000005
ctime = Sat Jun 02 14:22:24 UTC 2018
mZxid = 0x200000005
mtime = Sat Jun 02 14:22:24 UTC 2018
pZxid = 0x200000005
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x2000000d4500000
dataLength = 8
numChildren = 0
[zk: localhost:2181(CONNECTED) 7] 
#断开重连之后，临时节点自动消失
WATCHER::

WatchedEvent state:SyncConnected type:None path:null
#因为默认的心跳机制，此时查询临时节点还存在
[zk: localhost:2181(CONNECTED) 0] ls /merryyou
[temp]
#再次查询，临时节点消失
[zk: localhost:2181(CONNECTED) 1] ls /merryyou
[]
[zk: localhost:2181(CONNECTED) 2] 
```

#### create -s创建顺序节点 自动累加

```properties
# 创建顺序节点，顺序节点会自动累加
[zk: localhost:2181(CONNECTED) 2] create -s /merryyou/sec seq
Created /merryyou/sec0000000001
[zk: localhost:2181(CONNECTED) 3] create -s /merryyou/sec seq
Created /merryyou/sec0000000002 
```

### set path data [version] 修改节点

```properties
# 修改节点内容为new-merryyou
[zk: localhost:2181(CONNECTED) 7] set /merryyou new-merryyou
cZxid = 0x200000004
ctime = Sat Jun 02 14:20:06 UTC 2018
mZxid = 0x20000000a
mtime = Sat Jun 02 14:29:23 UTC 2018
pZxid = 0x200000009
cversion = 4
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 12
numChildren = 2
#再次查询，节点内容已经修改
[zk: localhost:2181(CONNECTED) 8] get /merryyou
new-merryyou
cZxid = 0x200000004
ctime = Sat Jun 02 14:20:06 UTC 2018
mZxid = 0x20000000a
mtime = Sat Jun 02 14:29:23 UTC 2018
pZxid = 0x200000009
cversion = 4
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 12
numChildren = 2
#set 根据版本号更新 dataVersion 乐观锁
[zk: localhost:2181(CONNECTED) 9] set /merryyou test-merryyou 1
cZxid = 0x200000004
ctime = Sat Jun 02 14:20:06 UTC 2018
mZxid = 0x20000000b
mtime = Sat Jun 02 14:31:30 UTC 2018
pZxid = 0x200000009
cversion = 4
dataVersion = 2
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 13
numChildren = 2
#因为数据的版本号已经修改为2 再次使用版本号1修改节点提交错误
[zk: localhost:2181(CONNECTED) 10] set /merryyou test-merryyou 1
version No is not valid : /merryyou
```

###  delete path [version] 删除节点

```properties
[zk: localhost:2181(CONNECTED) 13] delete /merryyou/sec000000000

sec0000000001   sec0000000002
[zk: localhost:2181(CONNECTED) 13] delete /merryyou/sec0000000001
[zk: localhost:2181(CONNECTED) 14] ls /merryyou
[sec0000000002]
[zk: localhost:2181(CONNECTED) 15] 
#版本号操作与set类似 version
```



### watcher通知机制

关于watcher机制大体的理解可以为，当每个节点发生变化，都会触发watcher事件，类似于mysql的触发器。zk中 watcher是一次性的，触发后立即销毁。可以参考https://blog.csdn.net/hohoo1990/article/details/78617336

- stat path [watch] 设置watch事件
- get path [watch]设置watch事件
- 子节点创建和删除时触发watch事件，子节点修改不会触发该事件

##### stat path [watch] 设置watch事件

```properties
#添加watch 事件
[zk: localhost:2181(CONNECTED) 18] stat /longfei watch
Node does not exist: /longfei
#创建longfei节点时触发watcher事件
[zk: localhost:2181(CONNECTED) 19] create /longfei test

WATCHER::

WatchedEvent state:SyncConnected type:NodeCreated path:/longfei
Created /longfei
```

##### get path [watch] 设置watch事件

```properties
#使用get命令添加watch事件
[zk: localhost:2181(CONNECTED) 20] get /longfei watch
test
cZxid = 0x20000000e
ctime = Sat Jun 02 14:43:15 UTC 2018
mZxid = 0x20000000e
mtime = Sat Jun 02 14:43:15 UTC 2018
pZxid = 0x20000000e
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 4
numChildren = 0
#修改节点触发watcher事件
[zk: localhost:2181(CONNECTED) 21] set /longfei new_test

WATCHER::

WatchedEvent state:SyncConnected type:NodeDataChanged path:/longfei
cZxid = 0x20000000e
ctime = Sat Jun 02 14:43:15 UTC 2018
mZxid = 0x20000000f
mtime = Sat Jun 02 14:45:06 UTC 2018
pZxid = 0x20000000e
cversion = 0
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 8
numChildren = 0
[zk: localhost:2181(CONNECTED) 22] 
#删除触发watcher事件
[zk: localhost:2181(CONNECTED) 23] get /longfei watch
new_test
cZxid = 0x20000000e
ctime = Sat Jun 02 14:43:15 UTC 2018
mZxid = 0x20000000f
mtime = Sat Jun 02 14:45:06 UTC 2018
pZxid = 0x20000000e
cversion = 0
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 8
numChildren = 0
[zk: localhost:2181(CONNECTED) 24] delete /longfei

WATCHER::

WatchedEvent state:SyncConnected type:NodeDeleted path:/longfei
[zk: localhost:2181(CONNECTED) 25] 
```



## 二、四字命令

使用四字命令需要安装`nc`命令,(`yum install nc`) 

### stat 查看状态信息

```properties
[root@localhost bin]# echo stat | nc 192.168.0.68 2181
Zookeeper version: 3.4.11-37e277162d567b55a07d1755f0b31c32e93c01a0, built on 11/01/2017 18:06 GMT
Clients:
 /192.168.0.68:493460
Latency min/avg/max: 0/0/4
Received: 62
Sent: 61
Connections: 1
Outstanding: 0
Zxid: 0x50000000a
Mode: follower
Node count: 10
[root@localhost bin]# 
```

### ruok 查看zookeeper是否启动

```properties
[root@localhost bin]# echo ruok | nc 192.168.0.68 2181
imok[root@localhost bin]# 
```

### dump 列出没有处理的节点，临时节点

```properties
imok[root@localhost bin]# echo dump | nc 192.168.0.68 2181
SessionTracker dump:
org.apache.zookeeper.server.quorum.LearnerSessionTracker@29805957
ephemeral nodes dump:
Sessions with Ephemerals (0):
[root@localhost bin]# 
```



### conf 查看服务器配置

```properties
[root@localhost bin]# echo conf | nc 192.168.0.68 2181
clientPort=2181
dataDir=/usr/home/zookeeper-3.4.11/data/version-2
dataLogDir=/usr/home/zookeeper-3.4.11/data/version-2
tickTime=2000
maxClientCnxns=60
minSessionTimeout=4000
maxSessionTimeout=40000
serverId=2
initLimit=10
syncLimit=5
electionAlg=3
electionPort=3888
quorumPort=2888
peerType=0
[root@localhost bin]# 
```



### cons 显示连接到服务端的信息

```properties
[root@localhost bin]# echo cons | nc 192.168.0.68 2181
 /192.168.0.68:493540
 
 [root@localhost bin]# 
```

### envi 显示环境变量信息

```properties
[root@localhost bin]# echo envi | nc 192.168.0.68 2181
Environment:
zookeeper.version=3.4.11-37e277162d567b55a07d1755f0b31c32e93c01a0, built on 11/01/2017 18:06 GMT
host.name=localhost
java.version=1.8.0_111
java.vendor=Oracle Corporation
java.home=/usr/local/jdk1.8.0_111/jre
java.class.path=/usr/home/zookeeper-3.4.11/bin/../build/classes:/usr/home/zookeeper-3.4.11/bin/../build/lib/.jar:/usr/home/zookeeper-3.4.11/bin/../lib/slf4j-log4j12-1.6.1.jar:/usr/home/zookeeper-3.4.11/bin/../lib/slf4j-api-1.6.1.jar:/usr/home/zookeeper-3.4.11/bin/../lib/netty-3.10.5.Final.jar:/usr/home/zookeeper-3.4.11/bin/../lib/log4j-1.2.16.jar:/usr/home/zookeeper-3.4.11/bin/../lib/jline-0.9.94.jar:/usr/home/zookeeper-3.4.11/bin/../lib/audience-annotations-0.5.0.jar:/usr/home/zookeeper-3.4.11/bin/../zookeeper-3.4.11.jar:/usr/home/zookeeper-3.4.11/bin/../src/java/lib/.jar:/usr/home/zookeeper-3.4.11/bin/../conf:
java.library.path=/usr/java/packages/lib/amd64:/usr/lib64:/lib64:/lib:/usr/lib
java.io.tmpdir=/tmp
java.compiler=<NA>
os.name=Linux
os.arch=amd64
os.version=3.10.0-514.10.2.el7.x86_64
user.name=root
user.home=/root
user.dir=/usr/home/zookeeper-3.4.11/bin
[root@localhost bin]#
```



### mntr 查看zk的健康信息

```properties
[root@localhost bin]# echo mntr | nc 192.168.0.68 2181
zk_version  3.4.11-37e277162d567b55a07d1755f0b31c32e93c01a0, built on 11/01/2017 18:06 GMT
zk_avg_latency  0
zk_max_latency  4
zk_min_latency  0
zk_packets_received 68
zk_packets_sent 67
zk_num_alive_connections    1
zk_outstanding_requests 0
zk_server_state follower
zk_znode_count  10
zk_watch_count  0
zk_ephemerals_count 0
zk_approximate_data_size    124
zk_open_file_descriptor_count   32
zk_max_file_descriptor_count    4096
[root@localhost bin]# 
wchs 展示watch的信息
[root@localhost bin]# echo wchs | nc 192.168.0.68 2181
0 connections watching 0 paths
Total watches:0
[root@localhost bin]# 
```



### wchc和wchp 显示session的watch信息 path的watch信息

需要在 配置zoo.cfg文件中添加 4lw.commands.whitelist=*

```properties
[root@localhost bin]# echo wchc | nc 192.168.0.68 2181
wchc is not executed because it is not in the whitelist.
[root@localhost bin]# echo wchp | nc 192.168.0.68 2181
wchp is not executed because it is not in the whitelist.
```

