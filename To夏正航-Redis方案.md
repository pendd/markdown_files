> 存数据

redis是key-value形式，上传的数据可以一天插入一次数据库，那一天的数据全部存到一个key中，使用redis的list集合， key的规则可以是data0805。 然后每秒就往这个key中存数据，可以定时任务 设置每天晚上12:00操作这个key 把所有数据全取出来 分批存入数据库

```bash
# 使用lists 集合 这样存储
127.0.0.1:6379> lpush data0805 one
(integer) 1
127.0.0.1:6379> lpush data0805 two
(integer) 2
127.0.0.1:6379> lpush data0805 three
(integer) 3
127.0.0.1:6379> lpush data0805 four
(integer) 4
# 查出所有的数据
127.0.0.1:6379> lrange data0805 0 -1
1) "four"
2) "three"
3) "two"
4) "one"
```

> 取数据

在方法上加缓存注解@Cacheable，当第一次查询的时候会去读取数据库把所有数据加载到缓存中使用，这时的数据不是全部的消息数据，还有当天的数据在redis的 data当天日期的key中存在 ，要把这两部分的数据加起来才是完整的 消息数据，展示给前端。

> 删除缓存

这里要考虑定时删除缓存数据，比如我今天查询方法上的缓存是第一次查数据库出来的，但是明天的数据还是从缓存中取，但是这个数据就不准确了，因为数据库的数据已经是昨天方法查询缓存中的数据+ 昨天data日期的key中数据之和，所以，要在每天24点把缓存中数据持久化到数据库的同时，删除掉方法缓存中的数据。

> 小问题

因为你的数据是1s传一次到redis中，我晚上12点持久化到数据库时，会获取当天日期的key。去持久化到数据库，但是比如今晚12点我准时获取当天日期是不是一定是8.6号，如果是那就没问题，然后你1s来的消息又会存到明天的key中也就是data0807中去 这样消息就不会丢失 ，但就怕12.00整获取的日期不确定是前一天还是后一天的。







> 实现

通过工具类中的lpush把数据添加到redis缓存中会吧

```java
/**
     * 往List中存入数据
     *
     * @param key Redis键
     * @param value 数据
     * @return 存入的个数
     */
    public static long lPush(final String key, final Object value) {
        Long count = redisTemplate.opsForList().rightPush(key, value);
        return count == null ? 0 : count;
    }

```

