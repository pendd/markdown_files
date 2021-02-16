### springBoot 和 springCloud 版本选择

```java
https://start.spring.io/actuator/info
```

### Cloud组件停用

- 服务注册中心
  - ❌ Eureka
  - ✅ Zookeeper
  - ✅ Consul
  - ✅ Nacos
- 服务调用
  - ✅ Ribbon
  - ✅ LoadBalancer
- 服务调用2
  - ❌ Feign
  - ✅ OpenFeign
- 服务降级
  - ❌ Hystrix
  - ✅ Resilience4j
  - ✅ Sentienl
- 服务网关
  - ❌ Zuul
  - ⚠️ Zuul2
  - ✅ Gateway
- 服务配置
  - ❌ Config
  - ✅ Nacos
- 服务总线
  - ❌ Bus
  - ✅ Nacos

### ZooKeeper

1. 下载

   ```java
   http://archive.apache.org/dist/zookeeper/
   ```

2. 安装

   上传到`/tools` 目录下并解压

   ```shell
   [root@pd tools]# ll
   总用量 4
   drwxr-xr-x 7 root root 4096 1月  19 22:06 apache-zookeeper-3.5.8-bin
   ```

3. 配置

   ```shell
   # 进入安装目录下的conf目录，
   # 复制zoo_sample.cfg,命名为zoo.cfg
   [root@pd conf]# pwd
   /tools/apache-zookeeper-3.5.8-bin/conf
   [root@pd conf]# ll
   总用量 16
   -rw-r--r-- 1 root root  535 5月   4 2020 configuration.xsl
   -rw-r--r-- 1 root root 2712 5月   4 2020 log4j.properties
   -rw-r--r-- 1 root root  923 1月  19 21:51 zoo.cfg
   -rw-r--r-- 1 root root  922 5月   4 2020 zoo_sample.cfg
   
   # vim zoo.cfg 里面有两个重要配置
   # dataDir = /data/zookeeper
   # clientPort = 2181
   ```

   ```shell
   # 配置环境变量
   [root@pd conf]# vim /etc/profile
   # 加入下面两行代码
   # export ZOOKEEPER_HOME=/tools/apache-zookeeper-3.5.8-bin
   #export PATH=$ZOOKEEPER_HOME/bin:$PATH
   [root@pd conf]# source /etc/profile
   ```

4. 启动

   ```shell
   [root@pd /]# zkServer.sh start
   ZooKeeper JMX enabled by default
   Using config: /tools/apache-zookeeper-3.5.8-bin/bin/../conf/zoo.cfg
   Starting zookeeper ... STARTED
   ```

   ```shell
   # 查看ZooKeeper启动日志
   [root@pd conf]# cd /tools/apache-zookeeper-3.5.8-bin/logs/
   [root@pd logs]# grep -E -i "((Exception)|(erroe)) *
   >    # 这表示正常 没有错误
   ```

   

5. 查看服务是否启动成功

```shell
# 在新版本中 zookeeper使用netty作为内嵌服务器，所以默认使用8080端口，在zoo.cfg 配置文件中添加admin.serverPort=8018 (不一定是8018 只要是没有被占用的端口都可以)
[root@pd logs]# grep -E -i "((Exception)|(error))" *
2021-01-19 22:06:25,055 [myid:] - ERROR [main:ZooKeeperServerMain@79] - Unable to start AdminServer, exiting abnormally
org.apache.zookeeper.server.admin.AdminServer$AdminServerException: Problem starting AdminServer on address 0.0.0.0, port 8080 and command URL /commands
Caused by: java.io.IOException: Failed to bind to /0.0.0.0:8080
Caused by: java.net.BindException: 地址已在使用
```

```shell
root@pd logs]# cd /data/zookeeper/
[root@pd zookeeper]# ls
version-2  zookeeper_server.pid

# tree 命令是以树方式展示目录 默认没有tree命令，需要安装
# sudo yum install tree -y
[root@pd zookeeper]# tree
.
├── version-2
│   └── snapshot.0   # 启动成功会有一个快照版本
└── zookeeper_server.pid

1 directory, 2 files

[root@pd zookeeper]# netstat -an | grep 2181
tcp        0      0 0.0.0.0:2181            0.0.0.0:*               LISTEN
```



#### zkCli

```shell
# 启动zookeeper客户端
[root@pd zookeeper]# zkCli.sh
Connecting to localhost:2181
# 查看所有的znode
[zk: localhost:2181(CONNECTED) 1] ls -R /
/
/zookeeper
/zookeeper/config
/zookeeper/quota
# 创建znode
[zk: localhost:2181(CONNECTED) 2] create /app2
Created /app2
[zk: localhost:2181(CONNECTED) 3] create /app1
Created /app1
[zk: localhost:2181(CONNECTED) 4] create /app1/p_1
Created /app1/p_1
[zk: localhost:2181(CONNECTED) 5] create /app1/p_2
Created /app1/p_2
[zk: localhost:2181(CONNECTED) 6] create /app1/p_3
Created /app1/p_3
[zk: localhost:2181(CONNECTED) 7] ls -R /
/
/app1
/app2
/zookeeper
/app1/p_1
/app1/p_2
/app1/p_3
/zookeeper/config
/zookeeper/quota
[zk: localhost:2181(CONNECTED) 8]
```

#### 实现一个锁

分布式锁要求如果锁的持有者宕了，锁可以被释放。

ZooKeeper的ephemeral节点恰好具备这样的特性。

```shell
# 终端1:
# 开启客户端1
[root@pd zookeeper]# zkCli.sh
# 创建加锁lock节点
[zk: localhost:2181(CONNECTED) 8] create -e /lock
Created /lock

# 新开一个终端2:
# 开启客户端2
[root@pd zookeeper]# zkCli.sh
# 在客户端1已经创建加锁lock节点后 客户端2尝试再次创建
[zk: localhost:2181(CONNECTED) 0] create -e /lock
Node already exists: /lock
# 提示已存在，监控lock节点
[zk: localhost:2181(CONNECTED) 1] stat -w /lock
cZxid = 0x9
ctime = Wed Jan 20 15:50:58 CST 2021
mZxid = 0x9
mtime = Wed Jan 20 15:50:58 CST 2021
pZxid = 0x9
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x100240c82ad0001
dataLength = 0
numChildren = 0

# 终端1的客户端1上退出
[zk: localhost:2181(CONNECTED) 9] quit

WATCHER::

WatchedEvent state:Closed type:None path:null
2021-01-20 15:53:28,174 [myid:] - INFO  [main:ZooKeeper@1422] - Session: 0x100240c82ad0001 closed
2021-01-20 15:53:28,174 [myid:] - INFO  [main-EventThread:ClientCnxn$EventThread@524] - EventThread shut down for session: 0x100240c82ad0001

# 终端2上的客户端2可以看到下面信息
[zk: localhost:2181(CONNECTED) 2]
WATCHER::

WatchedEvent state:SyncConnected type:NodeDeleted path:/lock
c
# 终端2上的客户端2上再次创建加锁lock节点 ，可以创建成功
[zk: localhost:2181(CONNECTED) 2] create -e /lock
Created /lock
```

#### 协同服务说明

*设计一个master-worker的组成员管理系统，要求系统中只能有一个master，master能实时获取系统中worker的情况*

```shell
# 终端1
# 创建master节点
[zk: localhost:2181(CONNECTED) 0] create -e /master "m1:2223"
Created /master
# 监控workers节点下znode
[zk: localhost:2181(CONNECTED) 1] ls -w /workers
[]

# 终端2
# 创建 /workers/w1
[zk: localhost:2181(CONNECTED) 0] create -e /workers/w1 "w1:2224"
Created /workers/w1

# 在终端2 创建/workers/w1 znode后，终端1 监控得到如下
[zk: localhost:2181(CONNECTED) 2]
WATCHER::

WatchedEvent state:SyncConnected type:NodeChildrenChanged path:/workers

# 此时，在终端1上查看监控/workers
[zk: localhost:2181(CONNECTED) 3] ls -w /workers
[w1]

# 终端3
# 创建 /workers/w2
[zk: localhost:2181(CONNECTED) 0] create -e /workers/w2 "w2:2224"

# 在终端3 创建/workers/w2 znode后，终端1 监控得到如下
# 前提是必须使用命令 ls -w /workers 监控
[zk: localhost:2181(CONNECTED) 2]
WATCHER::

WatchedEvent state:SyncConnected type:NodeChildrenChanged path:/workers

# 终端3 宕机
[zk: localhost:2181(CONNECTED) 1]quit

# 终端1
[zk: localhost:2181(CONNECTED) 5]
WATCHER::

WatchedEvent state:SyncConnected type:NodeChildrenChanged path:/workers
```

