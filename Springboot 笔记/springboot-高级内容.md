# 内容概要

1. 缓存：JSP-107、Spring boot 缓存抽象、整合 Redis
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

![](F:\study-note\Springboot 笔记\图片\JSR-107.png)

![](F:\study-note\Springboot 笔记\图片\JSP-107-接口.png)

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
> ![](F:\study-note\Springboot 笔记\图片\springboot缓存抽象-key.png)

- keyGenerator：指定 key 生成器，**与 key 二选一**
- cacheManager / cacheResolver : 相当于指定不同的缓存实现。

> 如图，一边使用 ConcurrentHashMap，另一边使用 Redis
>
> ![](F:\study-note\Springboot 笔记\图片\使用多个cacheManager.png)

- condition : 符合条件的情况下才进行缓存，可用 SpEL.
- unless：否定缓存。**当 unless 条件满足时，不缓存**。可获取结果作为判断，**#result**
- sync : 是否使用异步模式

### 1.2.2 原理

Springboot 的缓存自动配置类 - CacheAutoConfiguration。我们从中可以看到加载了 11 个缓存配置类，如：

- JCacheCacheConfiguration
- EhCache~
- Infinispan~
- ...

然后当我们只使用 spring boot 默认的缓存组件，可以看到**默认生效**的是 **SimpleCacheConfiguration**。

这个类的作用就是将**缓存管理器 - ConcurrentMapCacheManager** 加入到组件中。

缓存管理器的作用就是**获取和创建对应类型的缓存组件（Cache）**

然后数据就保存到这些缓存组件中。

### 1.2.3 运行流程

以被 @Cacheable 标注的方法为例。

1. 被标注缓存注解的方法运行前，先去查询 Cache.getCahce(cacheName)。第一次获取缓存（表示还没有这个 Cache），会创建一个 Cache。





















