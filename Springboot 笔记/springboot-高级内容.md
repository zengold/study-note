# 内容概要

1. [缓存：JSP-107、Spring boot 缓存抽象、整合 Redis](#1 Springboot 与缓存)
2. 消息：JMS、AMQP、RabbitMQ
3. 检索
4. 任务
5. 安全
6. 分布式
7. 监控管理
8. 部署

# 1 缓存

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

Exchange - 转换器。用于接收消息，转发给指定的队列，不只有一个

Queue - 消息队列，容器，等待消息被消费，不只有一个

Binding - Exchange 与 Queue 的绑定，多对多关系

Connection - 网络连接，与消息服务器建立连接

Channel - 信道。多路复用，建一条TCP连接（Connection），在其上开多个信道，减少性能开销。信道是消息通过的通道

Consumer - 消费者，收消息的

Virtual Host - 虚拟主机，也叫 vhost。消息服务器可划分出多个虚拟主机，相互之间独立。连接时需要指定虚拟主机，默认是 / 。虚拟主机通过路径划分，如：/abc, /123 ...

Broker - 消息代理，message Broker，就是消息服务器

#### 2.2.1.1 运行机制

![](.\图片\AMQP主要概念.png)

#### 2.2.2.2 消息路由

对比起 JMS，AMQP 中增加了 Exchange、Binding 

binding - 决定交换器的消息应该发送到哪个队列

Exchange 4种类型：direct、fanout、topic、headers(几乎用不到)

- headers 匹配 AMQP 消息的 header（消息头） 而不是路由键。和 direct 完全一致，但性能差很多，几乎不用

- direct

  - 消息中的路由键与 Binding 中的 binding key 一致，则交换器将消息发送到队列中
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

2. 创建工程整合 RabbitMQ

- 自动配置 - RabbitAUtoConfiguration

  - ConnectionFactory - 自动配置连接工厂
  - RabbiteProperties - 封装 RabbitMQ 的配置
  - RabbiteTemplate - 给 RabbiteMQ 发送和接受消息
  - AmqpAdmin - RabbiteMQ 系统管理功能组件，创建 Exchange、队列等等

- **发送**

  - Message 需要自己构造，可以自定义消息体和消息头

  > rabbiteTemplate 使用自动导入即可

  ```java
  rabbiteTemplate.send(exchange, routeKey, message) 
  ```

  - objtec 默认当成消息体，只需要传入需要发送的对象，将会自动序列化并发送给 rabbitMQ

  > 默认使用 java 序列化 

  ```java
  rabbiteTemplate.converAndSend(exchange, routeKey, object);
  ```

- **接收**

  - 返回 Message 对象

  ```java
  rabbiteTemplate.receive(queueName);
  ```

  - 返回 Object 对象，可强转为对象本身的类

  ```java
  rabbiteTemplate.receiveAndConvert(queueName);
  ```

- **Json 序列化**

  - RabbiteTemplate 中有一个 MessageConverter，它默认使用了 SimpleMessageConverter。这个 SimpleMessageConverter 使用了 Java 默认序列化。
  - 自定义。对 MessageConveter 使用 Ctrl + H，可以获取它的实现。

  ![](.\图片\自定义MessageConverter.png)

- **广播** - 广播时同样使用上面的发送和接受方法，**但是 routeKey 不用写了**，如

```java
rabbiteTemplate.convertAndSend("exchange.news", "", object);
```

###  2.2.3 监听

在一般例子中，如订单系统，新建的订单会进入消息队列，然后库存再从订单队列中获取实时获取订单。这就要求库存需要对消息队列进行监听。

**使用 @RabbiteListener 注解进行监听，使用这个同时需要在 Springboot 启动类中加入 @EnableRabbite 来开启基于注解的 RabbiteMQ。**

```java
@Service
.... {
 	// queues - 监听的队列的名字
    @RabbiteListener(queues = "queuesName")
    public Book receive(Book book) {
        System.out.printl("监听到的消息: " + book);
	}
    
    // 当我们需要额外的数据，如消息头
    @RabbiteListener(queues = "queuesName")
    public Book receive(Message message) {
        // 消息体
        System.out.printl(message.getBody);
        // 消息头
        System.out.printl(message.getMessageProperties());
	}
}
```

### 2.2.4 代码中管理 Queue、Exchange、binding

使用 AmapAdmin 来管理，直接注入就可以使用

- declarexxxx - 都是创建组件

- removexxxx/deletexxxx - 都是删除组件

如：

```java
amqpAdmin.declareExchange(new DirectExchange("exchangeName"));
amqpAdmin.declareQueue(new Queue("queueName"));
// 目的地类 - 可以为 Queue/Exchange
// exchange - 绑定的交换器
// 其他参数 - 不需要就填写 null
amqpAdmin.declareBinding(new Binding("目的地", 目的地类型, exchange, 其他参数))
```

# 3 检索

使用到的时 ElasticSearch，j简称 ES。并与 Springboot 整合。

![](.\图片\ElasticSearch介绍.png)

## 3.1 安装 ES

使用 docker，使用加速

！！注意：ElasticSerach 底层使用 Java 编写，默认状态下启动时会占用 2G 的堆内存，如果不够的话会报错。所以，可以在启动 ElasticSearch 时添加参数，限制它的内存使用：

> ES_JAVA="-Xms256m -Xmx256m" === 限制堆内存使用
>
> 9200 - 默认 web 通信端口
>
> 9300 - 默认各节点通信端口

```sh
docker run -e ES_JAVA="-Xms256m -Xmx256m" -d -p 9200:9200 -p 9300:9300 --name ES01 [镜像ID]
```

然后访问 主机地址:9200 端口，返回 JSON 数据，证明安装成功

## 3.2 ES 入门

ES 区别于传统的数据库，**可以存储整个对象/ 文档，可以对文档内容进行检索**。

它使用 JSON 格式进行数据存储。

### 3.2.1 基本概念

索引 （动词）- 在 ES 中将存储数据的行为叫做索引（这是一个动词）

存储数据的时候，需要先确定数据存到哪里：

先确定要存储的**索引（名词）**，然后所以索引下面可以有多个**类型**，类型下又有多个**文档（数据）**，每个文档里又有多个**属性（JSON字段）**

**类比 MySQL:**

索引（名词）- 确定需要存数据的**数据库**

类型 - 某个数据库中的可以有多张**表**

文档 - 表里面有一行行的**数据**，也就是文档

属性 - 表里有多个**字段**

### 3.2.2 使用

因为 ES 提供 Restful API 接口，所以可以使用 Postman 等工具发送请求来使用数据的储存和搜索等 ES 功能。

#### **存储/更新数据**

发送如下请求到 ES，表示将 Json 格式消息体中的数据进行**存储/更新**

```http
PUT /索引/类型/数据编号
{
	Json格式消息体
}
```

#### **检索文档**

```http
GET /索引/类型/数据编号
```

#### **删除文档（数据）**

```http
Delete /索引/类型/数据编号
```

#### **检查是否存在数据**

可以使用 HEAD 方式发送请求，不过数据有没有都没有返回，需要通过返回状态码（200/404）判断

> 也可以正常使用 GET，这个有返回数据，通过返回数据中字段进行判断数据是否存在

#### **搜索所有**

列出指定的索引、类型下的所有文档

```http
GET /索引/类型/_search
```

#### 带条件搜索

> q - 以**查询字符串**的形式进行查询
>
> last_name:Smith - 字段:值

```http
GET /索引/类型/_search?q=last_name:Smith
```

**使用查询表达式**

```http
POST /索引/类型/_search

{
	查询表达式
}
```

**可以发现，其实后面主要就是在编写查询表达式，通过查询表达式来操作更复杂的查询**

## 3.3 整合 ElasticSearch

Springboot 支持两种整合（连接）Elasticsearch 的方法：

- SpringData 

该方法需要引入 `spring-boot-starter-data-elasticsearch` 依赖。

使用  SpringData ElasticSearch 模块进行操作，该方法是 Springboot 默认的。

- Jest

该方法需要引入第三方的依赖包 `io.searchbox.client.JestClient`。

使用 JestClient 进行操作。该方法默认不生效的，只有在引入包后才启用。

### 3.3.1. Jest

1. 需要配置

```properties
# 默认配置为 http://loacalhost:9200
spring.elastricsearch.jest.uris =http://[ES的地址]:[ES的web通信端口]
```

2. 自动注入 JestClient 即可使用

注意：**实体类中主键需要加上 @JestId 注解，以标明这是主键**

#### 储存 - Index

```java
// 构建一个索引（动词）功能
// 还有 .id() 指定数据ID的方法，但如果实体类中已经设置了 ID 就不需要了
Index index = new Index.Builder(实体类对象).index(数据储存的索引名).type(类型).build();
// 执行
jestClient.execute(index);
```

#### 搜索 - Search

使用**查询表达式**来进行搜索

```java
String json = 查询表达式
Search search = new Search.Builder(json).addIndex(要搜索的索引名).addType(类型).build();
jestClient.execute(search);
```

详细的使用可以去看 github 上面的 Jest 官方文档

### 3.3.2 SpringData ElasticSearch

需要配置 Client 的节点信息，如 clusterNodes、clusterName

```properties
# 集群名
spring.data.elasticsearch.cluster-name = elasticsearch
# 配置节点信息，默认 9300 端口
spring.data.elasticsearch.cluster-nodes = [ES地址]:[ES的节点通讯端口]
```

注意：

springData  与 Elasticsearch 可能存在版本不适配的问题，这样会导致报错**中说 9300 端口无法连接等。**

[版本适配说明](https://github.com/spring-projects/spring-data-elasticsearch)

![](.\图片\ES版本适配.png)

> 现在[20191004]
>
> |                  Spring Data Release Train                   |                  Spring Data Elasticsearch                   |                        Elasticsearch                         |                         Spring Boot                          |
> | :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
> | Moore[[1](https://docs.spring.io/spring-data/elasticsearch/docs/3.2.0.RELEASE/reference/html/#_footnotedef_1)] | 3.2.x[[1](https://docs.spring.io/spring-data/elasticsearch/docs/3.2.0.RELEASE/reference/html/#_footnotedef_1)] | 6.8.1 / 7.x[[2](https://docs.spring.io/spring-data/elasticsearch/docs/3.2.0.RELEASE/reference/html/#_footnotedef_2)] | 2.2.0[[1](https://docs.spring.io/spring-data/elasticsearch/docs/3.2.0.RELEASE/reference/html/#_footnotedef_1)] |
> |                           Lovelace                           |                            3.1.x                             | 6.2.2 / 7.x[[2](https://docs.spring.io/spring-data/elasticsearch/docs/3.2.0.RELEASE/reference/html/#_footnotedef_2)] |                            2.1.x                             |
> | Kay[[3](https://docs.spring.io/spring-data/elasticsearch/docs/3.2.0.RELEASE/reference/html/#_footnotedef_3)] | 3.0.x[[3](https://docs.spring.io/spring-data/elasticsearch/docs/3.2.0.RELEASE/reference/html/#_footnotedef_3)] |                            5.5.0                             | 2.0.x[[3](https://docs.spring.io/spring-data/elasticsearch/docs/3.2.0.RELEASE/reference/html/#_footnotedef_3)] |
> | Ingalls[[3](https://docs.spring.io/spring-data/elasticsearch/docs/3.2.0.RELEASE/reference/html/#_footnotedef_3)] | 2.1.x[[3](https://docs.spring.io/spring-data/elasticsearch/docs/3.2.0.RELEASE/reference/html/#_footnotedef_3)] |                            2.4.0                             | 1.5.x[[3](https://docs.spring.io/spring-data/elasticsearch/docs/3.2.0.RELEASE/reference/html/#_footnotedef_3)] |

如果版本不适配，两个办法：

- 升级 spirngboot 版本
- 安装对应版本的ES

两种用法，可以参考上面出现的文档连接

https://docs.spring.io/spring-data/elasticsearch/docs/3.2.0.RELEASE/reference/html/

#### 编写一个 ElasticsearchRepository 的子接口来操作 ES，类似 JPA 

- 编写后注入 ElasticsearchRepository 的子接口，即可使用

- ElasticsearchRepository 中可以看到有储存和搜索的方法

- 实体类中需要加上注解，以标明该数据储存到哪个索引和类型下

```java
@Document(indexName="索引名", type="类型名")
public class Book {
 ...   
}
```

- 同样支持像 JPA 一样，在 ElasticsearchRepository 的子接口中加入自定义的方法

#### ElasticsearchTemplate 操作 ES

参照文档 https://docs.spring.io/spring-data/elasticsearch/docs/3.2.0.RELEASE/reference/html/

# 4 任务

## 4.1 异步任务

在方法上使用注解 @Async ，可以告诉 Springboot 创建线程去进行异步处理。

要使这个 @Async 注解生效，必须在启动类中添加 @EnableAsync 注解

这个可以很智能，比如，一个 hello 接口，经过睡眠 3 秒后返回字符串 success。那么正常请求这个接口需要等待3 秒之后才可以看到 success 结果，如果使用 @Async 与 @EnableAsync 注解，那么 Springboot 则自动创建另一条线程进行睡眠，直接返回 success，所以可以直接看到 success 接口。

## 4.2 定时任务

凌晨时，分析日志等

Springboot 提供了异步执行任务调度的方式，提供 TaskExecutor、TaskScheduler 接口

![](.\图片\定时任务.png)



使用 @Scheduled 标记在方法上，表示这个方法定时执行。

使用 @EnableScheduling 开启定时任务。

**cron 表达式：**

一共有6位，每位之间使用空格分隔，如`0 * * * * Mon-Fir` ，从左到右，分别表示：

秒、分、时、月中的日期、月份、星期几。24小时制

而该例子则表示：从周一到周五，任意月份、任意日期、任意小时、任意分钟，0 秒时执行定时任务，换句话说就是周一到周五的每分钟的0秒执行一次。

**特殊符号**

， 枚举，意思就是使用逗号分隔各个希望的时间，如秒位的，1,2,3,4 则表示1，2，3，4秒都执行

区间 - 就是区间内以及端点都执行，

步长 - 表示每隔多少时间执行一次，比如秒位，0/4，则表示0秒启动（即不延迟启动），每4秒执行一次

？- 日和星期的冲突匹配，比如，在日位上写了 *，星期里写了 Mon，但实际情况不是任意一天都是 Mon，所以日位上应该写 ？。反过来同理。

## 4.3 邮件任务

![](.\图片\邮件任务.png)

需要引入springboot mail 的启动器。

**了解邮件发送流程**

比如有两个邮箱A、B。A 进行邮件发送，B 进行邮件接收。那么：

1. A 通过自己的账号、密码，登录到 A 邮箱的邮箱服务器，然后通过该邮箱服务器将编写好的邮件发送给 B邮箱的邮箱服务器。
2. 用户想在 B 接收邮件时，通过 B 邮箱的账号、密码登录 B 邮箱，然后 B 邮箱从 B 邮箱服务器上拉取邮件。

所以，如果希望发送邮件，需要知道：

- 用于发送邮件的邮箱账号、密码
- 发送邮件的邮箱服务器地址

使用如下配置在 application 中配置

```properties
spring.mail.username=邮箱账号
spring.mail.password=邮箱密码，通常邮箱会使用授权码代替密码，以避免泄密
spring.mail.host=发送邮件的邮箱服务器地址，如 QQ 邮箱为 smtp.qq.com
# 如果提示邮件发送需要 SSL 连接，则添加如下配置
# mail 额外的配置都在 spring.mail.properties 下
spring.mail.properties.mail.smtp.ssl.enable=true
```

**使用**

自动注入 JavaMailSenderImpl，即可使用，如：

```java
@Autowired
JavaMailSenderImpl mailSender
...

// 简单邮件
SimpleMailMessage message = new SimpleMailMessage();
// 邮件内容的设置，包括目的邮箱地址，以及发送人等等
message.setSubject("...")

mailSender.send(message);
```

以上是发送简单邮件的例子，简单邮件中没有添加附件等功能。如果希望有这些功能，则需要创建复杂邮件 MimeMessage。但这个邮件并不能直接设置邮件内容，需要借助另一个类 MimeMessageHelper。

```java
// 复杂邮件
MimeMessage mimeMessage = mailSender.createMimeMessage();
// true 表示该复杂邮件需要上传附件
MimeMessageHelper helper = new MimeMessageHelper(mimeMessage, true);
// 设置内容
helper.set...
// 添加附件
helper.addAttachment("附件名", 附件源);

mailSender.send(mimeMessage);
```

# 5 安全

















