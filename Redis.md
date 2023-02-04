# 数据类型
# 数据结构
# 使用场景
# 持久化 AOF
# 持久化 RDB
# 主从复制
# 哨兵模式
# 分片集群
# Spring 与 Redis总结
Spring也提供了Spring-data-redis包来整合Jedis的操作，另外Spring也单独封装了Jedis

## spring-data-redis的设计
spring-data是高层次的抽象，而springdata redis是基于底层的具体实现，因此springdata redis也提供了与springdata完全一致的操作
- DAO接口只需继承CrudRepository，Spring Data Redis能为DAO组件提供实现类
- Spring Data Redis 支持DAO组件体检自定义的查询方法？？？
- Spring Data Redis同样支持Example查询？？？
详细可参考 https://toutiao.io/posts/m6aeqc6/preview
SpringDataRedis支持的方法名关键字查询不如JPA强大，这是由Redis底层决定，Redis不支持任何查询；
SpringData 操作与Redis之间的映射关系：
- @Id 设置为redisHas的id 即key的名称
- @RedisHash 注解指定数据类型映射到Redis的Hash对象
- @TimeToLive 注解修饰数值类型的属性，指定对象的超时时间
- @Indexed 注解在pojo属性上, 实现原理：在redis内部创建Set的二级索引，为每一个@Indexed会对应到7个set,  用于索引查询，Java代码上findAll(比如通过非主键字段查询-不是通过@ID字段查询)内部实现则是通过Set取交集，拿到所有的元素之后再每个再HGETALL  时间复杂度 SINTER 是O(N*M) HGETALL 是O(N) N 是hashsize的长度
- （注意和org.springframework.stereotype下的@Indexed区分，stereotype下的是模式化注解的方法，它解决的是@ComponentScan的包的类越来越多的时候，Spring启动时模式注解解析时间就会变得很长的问题--- 通过@Indexed将注释的包类在项目编译打包时生成META-INF/spring.components文件， 以后别的项目引用时扫描这个文件提高组件扫描效率，减少启动时间）

## Spring Cache的使用
Spring Cache是作用在方法上的，其核心思想是这样的：当我们在调用一个缓存方法时会把该方法参数和返回结果作为一个键值对存放在缓存中，等到下次利用同样的参数来调用该方法时将不再执行该方法，而是直接从缓存中获取结果进行返回。所以在使用Spring Cache的时候我们要保证我们缓存的方法对于相同的方法参数要有相同的返回结果。

@EnableCaching 开启SpringCache缓存，能够在服务类上标注@Cacheable，使用CacheManager管理Cache对象
Spring3.1引入缓存技术，本质上不是一个缓存实现方案，是一个对缓存使用的抽象，可以接入Redis
什么时候使用？
优缺点？
解决什么问题

@Cacheable:把数据保存到缓存中
@CacheEvict: 将数据从缓存删除
@CachePut: 不影响方法执行更新缓存
@Caching:组合多个缓存操作
@CacheConfig:  一个类上共享缓存相同配置



@Cacheable(value=“sampleCache”)，这个注释的意思是，当调用这个方法的时候，会从一个名叫 sampleCache 的缓存(缓存本质是一个map)中查询key为id的值，如果不存在，则执行实际的方法（即查询数据库等服务逻辑），并将执行的结果存入缓存中，否则返回缓存中的对象。这里的缓存中的 key 就是参数 id，value 就是 返回的String 对象

## spring-data-redis的使用总结
spring-data-redis在springboot2以后默认使用的是Lettuce，不是Jedis
使用Jedis：当多线程使用同一个链接时，是线程不安全的，所以需要使用连接池，为每个Jedis实例分配一个链接
使用lettuce：多线程使用同一个链接时，是线程安全的。采用netty链接Redis server，实例在多个线程间共享，不存在线程不安全的情况，这样可以减少线程数量
spring-data-redis提供了统一的操作模版(RedisTemplate)封装了Jedis Lettuce 的API操作；

## Redisson
Redisson是由redission-spring-data来提供；

## Jedis
参考 https://www.jianshu.com/p/a1038eed6d44
Jedis  官方出的连接开发工具
JedisPool 基于apache-commons pool2实现
高可用 支持Redis sentinel 主从切换时，通知应用，切换到新的master
客户端分片

## Redisson
https://juejin.cn/post/7158256053350744095
https://www.jianshu.com/p/e9b26c743cae


代码中遇到的问题
1.lus脚本debug
redis-cli --ldb --eval  fuck.lua  参数
./src/redis-cli -a Lym@12345 --ldb --eval iplimit.lua ip1 2 10 , 20 30 调试出ARGV参数是用空格隔开
2.key生成了\xac\xed\x00\x05t\x00\x0c前缀乱码问题

keyvalue序列化问题
Spring Redis 默认使用 Byte数据类型存储Key，在redis-cli中会看到 "\xac\xed\x00\x05t\x00\x04" 前缀不方便get操作，所以我们会设置使用字符串，通过 redisTemplate.setKeySerializer(new StringRedisSerializer()); 实现


3. 使用spring-data-redis
   https://blog.csdn.net/Frankenstein_/article/details/104994925
   @RedisHash注解存储实体到redis
   实体中设置一个属性标识为唯一ID
   @TimeToLive注解设置失效时间
   通过Dao层方法进行save和get


4. Jedis与Lettuce的比较
   谁更容易
   谁坑多一些

@EnableRedisRepositories 与 @Repository 关系？
spring-data中对数据库的增删改的库，他的实现为CrudRepository
通过@RedisHash实例化操作Hash实例
RedisTemplate用户Redis继承的CRUD repository

@EnableCaching Java中的门面模式(外观模式)




redis 命令
hset
hmset 同时设置一个Key的多filed-value
hgetall
sinter 获取给定集合的交集
