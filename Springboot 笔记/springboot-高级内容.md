# 内容概要

1. [缓存：JSP-107、Spring boot 缓存抽象、整合 Redis](#1 Springboot 与缓存)
2. 消息：JMS、AMQP、RabbitMQ
3. 检索
4. 任务
5. 安全
6. 分布式
7. 监控管理
8. 部署

# 1 Springboot 与缓存

临时性数据也可以存缓存中

## 1.1 JSP-107 

JSP-107 是一个缓存规范。由 JAVA 官方规定的，从下面图可以看到，有 5 个接口，提供不同的功能。

![](.\图片\JSR-107.png)

![](.\图片\JSP-107-接口.png)

**JSP-107 提供的接口，是需要实现的，同时也提供了方便使用缓存的注解。**但是，并不是所有缓存框架都有实现 JSP-107 规范，所以有些麻烦，比较少用。

所以，为了简化缓存使用与开发，**spring boot 定义了缓存抽象**，同样提供了类似的注解

## 1.2 Springboot 缓存抽象

spring boot 缓存抽象中只有三个核心接口：

- CacheManager
- Cache - 不同缓存组件用不同的 Cache,如 RedisCache、EhCacheCache...
- 注解
  - @EnableCaching - 放到启动类中，表示开启缓存
  - @Cacheable - 如无法通过 key 找到对应缓存(value)，则将返回值存放到缓存中，否则直接返回value
  - @CacheEvict - 缓存驱逐。手动删去指定 key 的缓存，也可以整个缓存空间驱逐
  - @CachePut - 更新缓存，会将返回值覆盖已有缓存

缓存保存都是以 key : value 的形式，所以涉及到两点：

1. key 的生成
2. value 的序列化

### 1.2.1 @Cacheable

以 @Cacheable 为例，@Cacheable 有一些属性

- cachename / value : 指定缓存组件(缓存空间)的名字，可以是数组：value = {"xxx", "xxx", ...}

> CacheManager管理多个 Cache 组件。
>
> **Cache 组件是对缓存的真正 CRUD 操作的地方**。
>
> 在 Cache 组件中，每个缓存组件都有自己唯一的名字。

- key : 默认使用方法参数的值

> 缓存中保存的是 key : 方法的返回值。
>
> #id - 表示拿到方法参数 id 的值作为 key
>
> 其他方法: 
>
> ![](.\图片\springboot缓存抽象-key.png)

- keyGenerator：指定 key 生成器，**与 key 二选一**
- cacheManager / cacheResolver : 相当于指定不同的缓存实现。

> 如图，一边使用 ConcurrentHashMap，另一边使用 Redis
>
> ![](.\图片\使用多个cacheManager.png)

- condition : 符合条件的情况下才进行缓存，可用 SpEL.
- unless：否定缓存。**当 unless 条件满足时，不缓存**。可获取结果作为判断，**#result**
- sync : 是否使用异步模式

#### 1.2.1.1 原理

Springboot 的缓存自动配置类 - CacheAutoConfiguration。我们从中可以看到加载了 11 个缓存配置类，如：

- JCacheCacheConfiguration
- EhCache~
- Infinispan~
- ...

然后当我们只使用 spring boot 默认的缓存组件，可以看到**默认生效**的是 **SimpleCacheConfiguration**。

这个类的作用就是将**缓存管理器 - ConcurrentMapCacheManager** 加入到组件中。

缓存管理器的作用就是**获取和创建对应类型的缓存组件（Cache）**

然后数据就保存到这些缓存组件中。

#### 1.2.1.2 运行流程

以被 @Cacheable 标注的方法为例。

1. 被标注缓存注解的方法运行前，先去查询 Cache.getCahce(cacheName)。第一次获取缓存（表示还没有这个 Cache），会创建一个 Cache。
2. 使用 Cache zhong  lookup(key) 方法，查询缓存，这个 key 是由 keyCenerator 生成的。默认使用 SimpleKeyGenerator 生成 key。默认生成策略：
   - 没有参数：key = new SimpleKey();
   - 只有一个参数：key = 参数值
   - 多个参数：key = new SimpleKey(params);
3. 如果没有查到缓存，则调用目标方法，即被标注的方法
4. 将目标方法结果放进缓存

#### 1.2.1.3 @Cacheable 的其他属性

1. cacheName / value - 数组，可指定多个
2. key 拼接：key = "#root.methodName + '[' + #id + ']'"
3. keyGenerator:

![](G:\woods\study-note\Springboot 笔记\图片\keyGenetator.png)

4. sync：默认是目标执行完，将结果**同步**保存到缓存中。这个属性可以开启异步模式，开启异步模式后不支持 unless

### 1.2.2 @CachePut

既调用方法，又更新缓存

**运行流程**：

1. 先运行目标方法
2. 再将方法结果保存到缓存中，若缓存中已有该缓存，则更新它

注意：该注解可以使用 `#result`，而 @Cacheable 是不能的，这是运行流程的问题

### 1.2.3 @CacheEvict

用于清除 / 驱逐缓存

1. 可以通过 key 指定需要清楚的缓存
2. allEntries 属性，表示是否清楚该缓存（Cache）中所有数据
3. beforeInvocation - 是否在目标方法前执行清楚操作，默认是 false。这两个的区别，可以考虑目标方法出错时的情况，若先执行清楚缓存操作，则目标方法出错时，缓存已经被清除。

### 1.2.4 @Caching

这个注解用于组合其他三种注解，可以自由搭配，同一个注解可以出现多次。如

```java
@Caching(
    cacheable={
        @Cacheable(value = xx, key = xx),
        @Cacheable(...),
        ...
    },
    put = {
        @CachePut(...)
    },
    evict = {
        @CacheEvict(...)
    }
)
```

### 1.2.5 @CahceConfig

标注在类上，可以将重复的配置统一配置在该注解上，如，key、keyGenerator 等配置

## 1.3 整合 Redis

在实际开发中，常常使用到缓存中间件，如：redis / memcached / ehcache / infinispan /hazelcast ...，springboot 的缓存抽象支持了它们。

这里我们学习怎么整合 redis 到 spring boot 中。

Redis 不仅仅用作缓存中间件，还有其他用途，有兴趣可以去了解

### 1.3.1 步骤

1. （使用 docker）安装 Redis，最好下载带管理界面的。

> PS:
>
> 1.  可以使用 docker-cn 来加速镜像的下载。每次拉去镜像时添加如下地址：`docker pull registry.docker-cn.com/library/xxx`
> 2. 可以使用 `RedisDestopManager` (这是一个工具) 来连接 Redis
> 3. Redis 并不是简单的嵌入式的缓存中间件，有自己一套语法，可以去 “redis中国” 查看语法。

2. 引入 Redis 的 starter：`spring-boot-starter-data-redis`

3. 配置 redis: `spring.redis.host = [安装了 redis 的主机地址]`

4. redis 自动配置类生效，我们从中可以发现，它给 IOC 容器添加了两个 bean：

   - `RedisTemplate<Object, Object>` - 用于操作 k-v，两者都是对象
   - `StringRedisTemplate` - 用于操作字符串

   这两个 bean 就是用来简化 spring boot 的 redis 操作的组件，需要使用时直接注入（`@Autowired`）即可


### 1.3.2 使用

Redis 中存在**五大常用基本数据类型**：

- String - 字符串
- List - 列表
- Set - 集合
- Hash - 散列
- ZSet - 有序集合

**使用例子**

```java
@Autowired
StringRedisTemplate stringRedisTemplate

/** 其中，opsForxxx 有：
  * opsForValue - 用于操作 String
  * opsForList - 用于操作 List
  * ...
  * 同样，redisTemplate 也有相同的方法
 **/
stringRedisTemplate.opsForxxx ...

 
// 在 redis 中的语法，基本都可以在 stringRedisTemplate 中找到对应的方法，如：
// msg - key, hello - value
stringRedisTemplate.opsForValue().append("msg", "hello");
~.get("msg");
stringRedisTemplate.opsForList().leftPush("mylist", "1");
...
```

**保存对象**

需要保存对象时，我们使用 `redisTemplate.opsForValue.set("emp-01", emp);`
**注意，此时 emp 需要序列化才可以使用，否则报错！！！**

**但如果使用 JDK 默认的序列化接口，在 Redis 的图形界面管理中会将对象显示为 Unicode 编码的形式！！！**

所以我们可以将数据以 JSON 方法序列化，有两种方法：
1. 自己手动将对象转为 JSON 再返回给缓存进行保存
2. **修改 redisTemplate 中默认的序列化策略（该策略默认使用 JDK 序列化接口），将其改成 JSON**

![](.\图片\修改序列化策略.png)

> 其中，
>
> 1. `RedisTemplate<Object, Employee>` 这个泛型就是需要转换 JSON 的类
> 2. 需要补充代码： `new Jackson2JsonRedisSerializer<Employee>(Employee.class)` 

加入组件后：

```java
@Autowired
RedisTemplate<Object, Employee> redisTemplate
// 即可在图行界面上看到经过 JSON 化的 emp 对象
redisTemplate.opsForValue.set("emp-01", emp);
```

### 1.3.3 Redis 缓存流程

1. 引入 redis 的 starter 后，容器中保存的是 RedisManager

2. RedisManager 帮我们创建了一个 RedisCache 缓存组件

3. **上面说的其实只是手动操作 redis 时的操作，还是可以使用注解来简化操作的，但是保存对象时，依然是使用默认 JDK 的序列化策略**

4. 与上面的手动调用不同，使用注解方法时，主要使用到 RedisCacheManager，而该类中自动加入了 默认的 RedisTemplate。这个默认的 RedisTemplate 就使用默认的 JDK 序列化策略

5. 所以我们如果需要使用注解的方式，那么我们需要自己编写一下 RedisCahceManager：

   ![](.\图片\自定义RedisCacheManager.png)

   我们可以看到，这个 RedisCacheManager 传入了一个关于 Employee 的参数，这表明这个 RedisCacheManager 是 Employee 专用的。

   这导致了一个问题，有其他实体类时的序列化问题。

   如我们还有一个实体类 dept。按照上面配置，dept 是 可以经过 JSON 序列化进入缓存的。但是当 dept 从缓存中取出时则会报错，这是因为 JSON 字段映射失败，即反序列化失败，dept 的字段与 employee 的字段不同所以无法反序列化成 employee。

   这就是因为配置中我们将泛型配成 Employee 的后果。

   

   **解决办法: 再编写一个属于 dept 的 RedisTemplate、ReidsCacheManager，然后在注解上指定使用的 cacheManager**

   

   **注意：**

   1. 当有多个 CacheManager 时，我们需要将其中一个标注为**主**，即**默认的**。使用 `@Primary` 来标注。建议将 spring boot 默认初始化的相应 CacheManager 作为默认的。

      ```
      @Primary
      @Bean
      ```

   2. 有多个 CacheManager 时，当需要手动注入，则最好指定注入

      ```java
      // @QualiFier 用于明确指定注入的 RedisCacheManager 的 ID,即名字
      @QualiFier("deptCacheManager")
      @Autowired
      RedisCacheManager deptCacheManager
      ```

# 2 消息

消息服务中间件，消息队列

**为什么需要消息队列呢？**

主要应用于异步任务、应用解耦、流量削峰（秒杀活动，利用限定容量的消息队列限制流量）等应用场景

## 2.1 基本概念

### 2.1.1 两个概念

- 消息代理 - 接收存放消息的服务器
- 目的地 - 消息服务器需要将消息发往的地方

### 2.1.2 消息队列主要有两种目的地

- 队列 - 点对点 - 消费后删去消息，一个消息只能被一个消费者消费
  - A -> 队列 <- B/C/D
- 主题 - 发布订阅 - 多个消费者同时消费一个消息
  - A -> 主题 <- B & C & D, 同时收到

### 2.1.3 两种协议

JMS - Java 消息服务，基于 JVM消息代理规范

- ActiveMQ、HornetMQ 都是它的实现

AMQP - 高级消息队列协议 / 消息代理规范，兼容 JMS

- RabbitMQ 是它的实现

![](.\图片\JMS与AMQP对比.png)

springboot 两个都支持：

![](.\图片\springboot支持JMS与AMQP.png)

## 2.2 整合 RabbitMQ

### 2.2.1 RabbitMQ 简介

Message

- 消息头 - 有一些属性
- 消息体 - 不透明的

Publisher - 消息生产者，简称 P

Exchange - 接收消息，转发给指定的队列，不只有一个

Queue - 消息队列，容器，等待消息被消费，不只有一个

Binding - Exchange 与 Queue 的绑定，多对多关系

Connection - 网络连接，与消息服务器建立连接

Channel - 信道。多路复用，建一条TCP连接，在其上开多个信道，减少性能开销。信道是消息通过的通道

Consumer - 消费者，收消息的

Virtual Host - 虚拟主机，vhost。消息服务器可划分出多个虚拟主机，相互之间独立。连接时需要指定虚拟主机，默认是 / 。虚拟主机通过路径划分，如：/abc, /123 ...

Broker - 消息代理，message Broker，就是消息服务器

#### 2.2.1.1 运行机制

![](.\图片\AMQP主要概念.png)

#### 2.2.2.2 消息路由

对比起 JMS，AMQP 中增加了 Exchange、Binding 

binding - 决定交换器的消息应该发送到哪个队列

Exchange 4种类型：direct、fanout、topic、headers(几乎用不到)

- headers 匹配 AMQP 消息的 header 而不是路由键。和 direct 完全一致，但性能差很多，几乎不用

- direct

  - 消息中路由键与 Binding 中的 binding key 一致，则交换器将消息发送到队列中
  - 点对点通信模型，单播
  - 必须完全一致

- fanout

  - 将接收到的信息发给**全部**绑定的队列
  - 广播模式，速度最快。主题/订阅模型

- topic

  - 允许模糊匹配，通配符 `#` 、`*` ，前者匹配0或多个**单词**，后者匹配一个**单词**

  ![](.\图片\topci exchange.png)

### 2.2.2 整合

1. docker 安装 RabbitMQ，安装带 management 标签的，带web的管理界面

> 注意
>
> 1. 带管理界面的有两个端口，5672（ 通信端口） 和 15672（管理界面访问端口）
> 2. 管理界面用默认的 guest/guest 的用户/密码登陆
>
> 3. 管理界面可以直接创建交换器、队列、bingding等
>    - Durability - 选择是否是持久化的，即下次重启后数据还在不在
>    - Ack Mode - 可以告诉队列获取后删去消息

2. 



















