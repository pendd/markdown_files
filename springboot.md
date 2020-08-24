## 一、spring缓存抽象

```
1.spring缓存抽象默认使用的是ConcurrentMapCacheManager = ConcurrentMapCache，将数据保存在ConcurrentMap<Object,Object>中 
2.原理：
        1)自动配置类；CacheAutoConfiguration
        2)缓存的配置类
        org.springframework.boot.autoconfigure.cache.GenericCacheConfiguration
        org.springframework.boot.autoconfigure.cache.JCacheCacheConfiguration
        org.springframework.boot.autoconfigure.cache.EhCacheCacheConfiguration
        org.springframework.boot.autoconfigure.cache.HazelcastCacheConfiguration
        org.springframework.boot.autoconfigure.cache.InfinispanCacheConfiguration
        org.springframework.boot.autoconfigure.cache.CouchbaseCacheConfiguration
        org.springframework.boot.autoconfigure.cache.RedisCacheConfiguration
        org.springframework.boot.autoconfigure.cache.CaffeineCacheConfiguration
        org.springframework.boot.autoconfigure.cache.GuavaCacheConfiguration
        org.springframework.boot.autoconfigure.cache.SimpleCacheConfiguration【默认】
        org.springframework.boot.autoconfigure.cache.NoOpCacheConfiguration
       3)哪个配置类默认生效：SimpleCacheConfiguration；
       4)给容器中注册了一个CacheManager：ConcurrentMapCacheManager
       5)可以获取和创建ConcurrentMapCache类型的缓存组件；他的作用将数据保存在ConcurrentMap中；
```

### 1. Cache  

```java
缓存接口，定义缓存操作。实现有：RedisCache、EhCacheCache、ConcurrentMapCache等。
```

### 2. CacheManager

```
缓存管理器，管理各种缓存（Cache) 组件
CacheManager管理多个Cache组件的，对缓存的真正CRUD操作在Cache组件中，每一个缓存组件有自己唯一一个名字；
```

### 3. @Cacheable

```
主要针对方法配置，能够根据方法的请求参数对其结果进行缓存，
例如：get方法，查询的结果缓存起来
运行流程：
     1、方法运行之前，先去查询Cache（缓存组件），按照cacheNames指定的名字获取；
        （CacheManager先获取相应的缓存），第一次获取缓存如果没有Cache组件会自动创建。
     2、去Cache中查找缓存的内容，使用一个key，默认就是方法的参数；
        key是按照某种策略生成的；默认是使用keyGenerator生成的，默认使用SimpleKeyGenerator生成key；
          SimpleKeyGenerator生成key的默认策略；
                  如果没有参数；key=new SimpleKey()；
                  如果有一个参数：key=参数的值
                  如果有多个参数：key=new SimpleKey(params)；
     3、没有查到缓存就调用目标方法；
     4、将目标方法返回的结果，放进缓存中
     
@Cacheable标注的方法执行之前先来检查缓存中有没有这个数据，默认按照参数的值作为key去查询缓存，
     如果没有就运行方法并将结果放入缓存；以后再来调用就可以直接使用缓存中的数据；
     
 核心：
     1、使用CacheManager【ConcurrentMapCacheManager】按照名字得到Cache【ConcurrentMapCache】组件
     2、key使用keyGenerator生成的，默认是SimpleKeyGenerator
```

```
几个属性：
     cacheNames/value：指定缓存组件的名字;将方法的返回结果放在哪个缓存中，是数组的方式，可以指定多个缓存；
     key：缓存数据使用的key；可以用它来指定。默认是使用方法参数的值  参数值-方法的返回值
             编写SpEL； #i d;参数id的值   #a0  #p0  #root.args[0]
    keyGenerator：key的生成器；可以自己指定key的生成器的组件id
             key/keyGenerator：二选一使用;
    cacheManager：指定缓存管理器；或者cacheResolver指定获取解析器
    condition：指定符合条件的情况下才缓存；
             condition = "#id>0"
             condition = "#a0>1"：第一个参数的值 > 1的时候才进行缓存   
    unless:否定缓存；当unless指定的条件为true，方法的返回值就不会被缓存；可以获取到结果进行判断
              unless = "#result == null"
              unless = "#a0==2":如果第一个参数的值是2，结果不缓存；
    sync：是否使用异步模式  当sync=true时，unless属性不支持使用
```



### 4. @CacheEvit

```
清空缓存

例如：delete方法，把删除的对象的get方法的缓存清空
```

```
几个属性：
    key：指定要清除的数据的键值
    allEntries = true：指定清除这个缓存中所有的数据
    beforeInvocation = false：缓存的清除是否在方法之前执行
         默认代表缓存清除操作是在方法执行之后执行;如果出现异常缓存就不会清除
    beforeInvocation = true：
         代表清除缓存操作是在方法运行之前执行，无论方法是否出现异常，缓存都清除
```

### 5. @CachePut

```
保证方法被调用，结果也被缓存。

例如：update方法，更新数据，并把更新的数据缓存中更新
运行时机：
     1、先调用目标方法
     2、将目标方法的结果缓存起来
```



### 6. @EnableCaching

```
开启基于注解的缓存
```



### 7. keyGenerator

```
缓存数据时key生成策略
可以使用默认的，也可以自定义
```

```java
import java.util.Arrays;
import org.springframework.cache.interceptor.KeyGenerator;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class MyCacheConfig {

    @Bean
    public KeyGenerator keyGenerator() {
    return (target, method, params) -> method.getName() + "[" + Arrays.asList(params).toString() + "]";
    }
}
```



### 8. serialize

```
缓存数据时value序列化策略
```

### 9.@Caching

```java
定义复杂的缓存规则
例如：通过lastName查询员工属性，并缓存lastName查询，id查询，email查询的值，
     但是在执行该方法时，即使缓存中有值，还是会查询数据库，因为@CachePut里面还要缓存其他key值
@Caching(
         cacheable = {
             @Cacheable(value="emp",key = "#lastName")
         },
         put = {
             @CachePut(value="emp",key = "#result.id"),
             @CachePut(value="emp",key = "#result.email")
         }
    )
public Employee getEmpByLastName(String lastName){
    return employeeMapper.getEmpByLastName(lastName);
}
```

### 10.@CacheConfig

```
抽取缓存的公共配置
例如：放在类上面，整个类都公用CacheConfig中的属性值
@CacheConfig(cacheNames="emp",cacheManager = "employeeCacheManager") 
```

```
几个属性：
	cacheNames
	keyGenerator
	cacheManager
	cacheResolver
```

### 11.Cache SpEL 

​	cache支持的spel表达式

![image-20200717100551911](C:\Users\StupidBoy\AppData\Roaming\Typora\typora-user-images\image-20200717100551911.png)