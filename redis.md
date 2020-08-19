## Redis入门

### Linux安装

Redis6.0 以上版本需要gcc5 以上 

1. 安装gcc相关

   ```bash
   # 安装gcc相关
   yum -y install gcc
   yum -y install gcc-c++
   # 升级gcc
   sudo yum install centos-release-scl
   sudo yum install devtoolset-7-gcc*
   scl enable devtoolset-7 bash
   make
   # 查看gcc
   gcc -v
   ```

2. 下载Redis并上传至Linux服务器

   ```bash
   # 官网下载地址下载Redis
   https://redis.io
   # mac中使用scp命令把Redis包上传至Linux服务器
   scp 本地文件路径 Linux服务器用户名@Ip:要上传到的服务器文件路径
   scp /Users/pd/Downloads/redis-6.0.6.tar.gz root@10.211.55.5:/home/pd/Downloads
   # 一般安装文件放在/opt目录下 
   mv redis-6.0.6.tar.gz /opt
   ```

3. 安装Redis

   ```bash
   # 进入Redis压缩包目录 解压Redis
   tar -zxvf redis-6.0.6.tar.gz
   # 进入到解压后的Redis目录
   make
   make install
   # Redis可执行命令在/usr/local/bin 目录下
   ```

   ![截屏2020-08-04 上午10.54.04](/Users/pd/Desktop/截屏2020-08-04 上午10.54.04.png)

   ```bash
   # 在/usr/local/bin 目录下新建myconfig目录用于copy redis.conf文件
   mkdir myconfig
   cp /opt/redis-6.0.6/redis.conf myconfig
   ```

4. 启动Redis  

   ```bash
   cd /usr/local/bin
   redis-server myconfig/redis.conf
   redis-cli -p 6379
   ping
   # 新开terminal窗口查看Redis服务是否开启
   ps -ef|grep redis
   ```

5. 设置后台启动

   ```bash
   vim redis.conf
   # 修改daemonize属性为yes
   ```

   

### 测试性能

 redis-benchmark

![benchmark参数](/Users/pd/Library/Application Support/typora-user-images/benchmark参数.png)

测试性能：

```bash
# 100个并发连接 100000次请求
redis-benchmark -h localhost p 6379 -c 100 -n 100000
```

![benchmark测试](/Users/pd/Library/Application Support/typora-user-images/benchmark测试.png)

### 基础知识

Redis默认有16个数据库

![redis 默认数据库数量](/Users/pd/Library/Application Support/typora-user-images/redis 默认数据库数量.png)

默认使用的是第0个

可以使用select进行切换数据库

```bash
# 切换数据库
127.0.0.1:6379> select 3      
OK
# 查看DB大小
127.0.0.1:6379[3]> DBSIZE
(integer) 0
127.0.0.1:6379[3]> set name pd
OK
127.0.0.1:6379[3]> DBSIZE
(integer) 1
127.0.0.1:6379[3]> select 7
OK
127.0.0.1:6379[7]> get name
(nil)
127.0.0.1:6379[7]> DBSIZE
(integer) 0
127.0.0.1:6379[7]> 
```

查看数据库所有数据 `keys *`

清除当前数据库 `flushdb`

清除全部数据库的内容 `FLUSHALL`

> Redis 为什么是单线程的？

Redis是基于内存操作，CPU不是Redis的性能瓶颈，Redis的瓶颈是机器的内存和网络带宽，既然CPU不是性能瓶颈，那么就使用单线程咯。

## 五大数据类型

**Redis 命令不区分大小写**

### Redis-Key

```bash
# 判断当前的key是否存在  存在返回1。不存在返回0
EXISTS key
# 移动一个key到另一个数据库    成功1 失败0
move key 数据库下标（0-15） 
# 设置key的过期时间，单位秒  如果成功设置过期时间返回1  如果key不存在或者不能设置过期时间返回0
EXPIRE key 秒

# 查看当前key的剩余时间 单位秒
# 在Redis 2.6和之前版本，如果key不存在或者已过期时返回-1。
# 从Redis2.8开始，错误返回值的结果有如下改变：
# 如果key不存在或者已过期，返回 -2
# 如果key存在并且没有设置过期时间（永久有效），返回 -1 。
ttl key

# 查看当前key的剩余时间 单位毫秒 其他和ttl一样
pttl key

# 查看key的类型  存在返回类型，不存在返回none
type key
```

### String(字符串)

```bash
# 设置值
set key value
# 获取值
get key
# 追加字符串，如果当前key不存在，就相当于set key
APPEND key value
# 获取字符串的长度
STRLEN key
```

```bash
# 自增1
incr key
# 自减1
decr key
# 自增 设置步长
INCRBY key step
# 自减 设置步长
DECRBY key step
```

```bash
# 字符串范围 
getrange key 起始下标 终止下标
# 获取全部的字符串 和 get key 一样
getrange key 0 -1
# 替换指定位置开始的字符串
setrange key 替换位置 value
```

```bash
# setex (set with expire)  # 设置过期时间
# setex 相当于 set key value + expire key seconds
# setnx (set if not exist) # 不存在再设置（在分布式锁中会常常使用）
```

```bash
# 同时设置多个值
mset key1 value1 key2 value2 ...
# 同时获取多个值
mget key1 key2 ...
# msetnx 如果不存在就设置 是一个原子性的操作，要么一起成功，要么一起失败
msetnx key1 value1 key2 value2
```

```bash
# redis存储对象 设置一个user:1 对象值为json字符串来存储一个对象
set user:1 {name:zhangsan,age:3}

# 巧妙的设置key  user:{id}:{field}
127.0.0.1:6379> mset user:1:name zhangsan user:1:age 20
OK
127.0.0.1:6379> mget user:1:name user:1:age
1) "zhangsan"
2) "20"
127.0.0.1:6379> 
```

```bash
# 如果不存在值，则返回nil
127.0.0.1:6379> getset db redis
(nil)
# 如果存在值，获取原来的值，并设置新的值
127.0.0.1:6379> getset db redis
(nil)
127.0.0.1:6379> get db
"redis"
127.0.0.1:6379> getset db mongodb
"redis"
127.0.0.1:6379> get db
"mongodb"
127.0.0.1:6379> 
```

String的使用呢场景：value除了是字符串也可以是数字

- 计数器
- 统计多单位的数量
- 粉丝数
- 对象缓存存储

### List(列表)

![redis List图解](/Users/pd/Library/Application Support/typora-user-images/redis List图解.png)

在redis中，可以把list玩成栈、队列、阻塞队列

所有的List命令都是用`l`开头的 

```bash
# 将一个值或多个值，插入到列表头部（左）
lpush key value
# 将一个值或者多个值，插入到列表尾部（右）
rpush key value
# 查看所有元素
lrange key 0 -1
```

```bash
# 移除列表头部第一个元素（左）
lpop key
# 移除列表尾部第一个元素（右）
rpop key
```

```bash
# 通过下标获取list中的某一个值
lindex key index
# 返回列表的长度
llen key 
# 移除list中指定个数的value，精确匹配   
lrem key num value   # num 表示要移除的个数
# 通过下标截取指定的长度
ltrim key 起始下标 终止下标 
```

```bash
# 移除列表的最后一个元素，将其添加到新的列表中
rpoplpush key newKey
# 将列表中指定下标的值替换为另外一个值，更新操作，如果列表不存在或者要替换的下标不存在 就报错
lset key index value
# 在列表的某个元素的前面或者后面插入元素
linsert key before|after value newValue
```

> 小结

- List实际上是一个链表，before Node after ，左右都可以插入值
- 如果key不存在，创建新的链表
- 如果key存在，新增内容
- 如果移除了所有值，空链表也表示不存在
- 在两边插入或者修改值，效率最高

### Set(集合)

Set中的值是不能重复的！

```bash
# set集合中添加元素
sadd key value
# 查看set的所有值
smembers key
# 判断某一个值是不是在set集合中 在返回1 不在返回0
sismember key value
# 获取set集合中元素个数
scard key
# 移除集合中某个元素
srem key value
# 随机抽选出一个元素
srandmember key
# 随机抽选出指定个数的元素 num 为指定个数
srandmember key num
# 随机删除set中的元素  num 为删除的元素个数 可选项
spop key [num]
# 移动一个set中的某个元素到另一个set中
smove 原key 新key value
```

```bash
# 差集 前一个set和后一个set的差集
sdiff key1 key2
# 交集
sinter key1 key2 
# 并集
sunion key1 key2
```

应用场景：

- 共同关注
- 共同爱好
- 二度好友
- 推荐好友

### Hash(哈希)

`key-map Hash的value是一个map集合！`

```bash
# 设置一个值
hset key key1 value1
# 取值
hget key key1
# 设置多个值  如果key1存在相同则替换
hmset key key1 value1 key2 value2
# 获取多个字段值
hmget key key1 key2
# 获取全部的字段和值
hgetall key
# 删除指定的值 hdel myhash key1 value1
hdel key value
# 获取hash表的字段数量
hlen key

# 判断某一个字段是否存在
hexists key key1
# 获取所有的key
hkeys key
# 获取所有的value
hvals key
```

```bash
# 自增 step 增量
hincrby key key1 step
# 如果不存在则设置
hsetnx key key1 value1
```

应用场景：

- 变更数据
- 用户信息 hmset user:1 name zhangshan age 20 

### Zset(有序集合)

`在set的基础上，增加了一个值。set key value.  zset key score value`

```bash
# 添加一个值 zadd myset 1 one
zadd key score value
# 添加多个值 zadd myset 29 two 13 three
zadd key score1 value1 score2 value2
# 查询所有元素  zrevrange 降序
zrange key 0 -1
```

> 排序 

`ZRANGEBYSCOREkey min max [WITHSCORES] [LIMIT offset count] 升序`

`ZREVRANGEBYSCOREkey max min [WITHSCORES] [LIMIT offset count] 降序`

min和max可以是-inf和+inf

默认情况下，区间的取值使用闭区间(小于等于或大于等于)，你也可以通过给参数前增加(符号来使用可选的开区间(小于或大于)。

```bash
redis> ZADD myzset 1 "one"
(integer) 1
redis> ZADD myzset 2 "two"
(integer) 1
redis> ZADD myzset 3 "three"
(integer) 1
redis> ZRANGEBYSCORE myzset -inf +inf
1) "one"
2) "two"
3) "three"
redis> ZRANGEBYSCORE myzset 1 2
1) "one"
2) "two"
redis> ZRANGEBYSCORE myzset (1 2
1) "two"
redis> ZRANGEBYSCORE myzset (1 (2
(empty list or set)
redis> 
```

```bash
# 移除元素
zrem key value
# 获取集合大小
zcard key
# 获取指定区间的元素数量
ZCOUNT key min max
```

## 三种特殊数据类型

### Geospatial(地理位置)

> GEO 底层的实现原理其实就是Zset ！我们可以使用Zset命令来操作GEO

### Hyperloglog

### Bitmap

## 事务

Redis 事务本质：一组命令的集合，一个事务中的所有命令都会被序列化，在事务执行过程中，会按照顺序从上到下执行。

`一致性、顺序性、排他性`

`Redis的事务没有隔离级别的概念`

`Redis单条命令保证原子性，但是事务不保证原子性`

Redis事务：

- 开启事务（multi）
- 命令入队
- 执行事务（exec）

> 正常执行事务！

```bash
# 开启事务
127.0.0.1:6379> multi     
OK
# 命令入队
127.0.0.1:6379> set k1 v1
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> get k1
QUEUED
127.0.0.1:6379> set k3 v3
QUEUED
127.0.0.1:6379> exec
1) OK
2) OK
3) "v1"
4) OK
127.0.0.1:6379> get k1
"v1"
127.0.0.1:6379> get k2
"v2"
127.0.0.1:6379> get k3
"v3"
127.0.0.1:6379> 
# 执行事务
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set k1 v1
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> get k1
QUEUED
127.0.0.1:6379> set k3 v3
QUEUED
127.0.0.1:6379> exec
1) OK
2) OK
3) "v1"
4) OK
127.0.0.1:6379> get k1
"v1"
127.0.0.1:6379> get k2
"v2"
127.0.0.1:6379> get k3
"v3"
```

> 放弃事务

```bash
# 开启事务
127.0.0.1:6379> multi
OK
# 命令入队
127.0.0.1:6379> set k1 v1
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
# 取消事务
127.0.0.1:6379> discard
OK
# 事务已经不在了 所以exec命令没有作用
127.0.0.1:6379> exec
(error) ERR EXEC without MULTI
# 取消事务，事务里的命令是不会执行的
127.0.0.1:6379> get k1
(nil)
```

> 编译型异常（代码有问题！命令有错！），事务中的所有命令都不会执行

```bash
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set k1 v1
QUEUED
# 执行了错误命令
127.0.0.1:6379> getset k2
(error) ERR wrong number of arguments for 'getset' command
127.0.0.1:6379> set k3 v3
QUEUED
# 整个事务都会被取消
127.0.0.1:6379> exec
(error) EXECABORT Transaction discarded because of previous errors.
127.0.0.1:6379> get k1
(nil)

```

> 运行时异常（I/O），如果事务队列中入队命令存在语法型错误，那么执行事务时，其他命令时可以正常执行的，错误命令抛出异常

```bash
127.0.0.1:6379> set k1 v1
OK
127.0.0.1:6379> multi
OK
# 存在语法型错误，字符串不能自增
127.0.0.1:6379> incr k1
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> exec
1) (error) ERR value is not an integer or out of range
2) OK
```

> 监控！Watch

**悲观锁：**

- 很悲观，认为什么时候都会出问题，无论做什么都会加锁！

**乐观锁：**

- 很乐观，认为什么时候都不会出问题，所以不会上锁！更新数据的时候去判断一下，在此期间是否有人修改过这个过程
- 获取version
- 更新的时候比较version

> Redis 使用watch实现乐观锁

> 正常执行

```bash
127.0.0.1:6379> set money 100
OK
127.0.0.1:6379> set out 0
OK
127.0.0.1:6379> watch money # 监视 money 对象
OK
127.0.0.1:6379> multi  
OK
127.0.0.1:6379> decrby money 20
QUEUED
127.0.0.1:6379> incrby out 20
QUEUED
127.0.0.1:6379> exec  # 事务正常结束，事务期间数据没有被其他线程修改
1) (integer) 80
2) (integer) 20
```

> 测试多线程修改值

```bash
127.0.0.1:6379> watch money
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> decrby money 10
QUEUED
127.0.0.1:6379> incrby out 10
QUEUED
127.0.0.1:6379> exec   # 在exec执行之前。其他线程修改了money的值
(nil)


# 执行失败后 需要unwatch 一下
127.0.0.1:6379> unwatch
OK
127.0.0.1:6379> watch
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> decrby money 10 
QUEUED
127.0.0.1:6379> incrby out 10
QUEUED
127.0.0.1:6379> exec
1) (integer) 170
2) (integer) 30
```

## Jedis

**什么是Jedis**

> 是Redis官方推荐的java连接开发工具

## SpringBoot整合Redis

## Redis.conf详解