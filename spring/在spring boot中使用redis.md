## 在spring boot中使用redis

### 一、添加依赖

```xml
		<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-pool2</artifactId>
            <version>2.6.2</version>
        </dependency>
```

因为Springboot 2.0 中redis客户端使用了Lettue, 其依赖于commons, 所以加入commons-pool，这其实是个连接池(似乎Jedis依然可以使用）

### 二、添加配置

```properties
#redis
# Redis数据库索引（默认为0）
spring.redis.database=0
# Redis服务器地址
spring.redis.host=192.168.10.134
# Redis服务器连接端口
spring.redis.port=6379
# Redis服务器连接密码（默认为空）
spring.redis.password=
# 连接池最大连接数（使用负值表示没有限制）
spring.redis.lettuce.pool.max-active=8
# 连接池最大连接数（使用负值表示没有限制）
spring.redis.lettuce.pool.max-wait=-1
# 连接池中的最大空闲连接
spring.redis.lettuce.pool.max-idle=8
# 连接池中的最小空闲连接
spring.redis.lettuce.pool.min-idle=0
# 连接超时时间（毫秒）
spring.redis.timeout=5000
```

### 三、添加redis配置类

+ 原理：一个 Spring Boot 项目中，我们只需要维护一个 `RedisTemplate` 对象和一个 `StringRedisTemplate` 对象就可以了。所以我们需要通过一个 `Configuration` 类来初始化这两个对象并且交由的 `BeanFactory` 管理。我们在 `config` 包下面新建了一个 `RedisConfig` 类

+ 实现：为了redis客户端查看操作数据, redisTemplate需要进行序列化设置, 默认配置的jdk序列化会导致在客户端查看不了数据(仍可使用内在函数存取修改, 只是查看不了), 为避免这种情况发生, 使用**StringRedisTemplate**或自行配置序列化, 自行配置可参考如下代码，设置完成后，客户端也能正确显示键值对了。

+ 只用配置redisTemplate就行

  ```java
  package edu.qingtai.pubandcollect.config;
  
  import org.springframework.context.annotation.Bean;
  import org.springframework.context.annotation.Configuration;
  import org.springframework.data.redis.connection.RedisConnectionFactory;
  import org.springframework.data.redis.core.RedisTemplate;
  import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
  import org.springframework.data.redis.serializer.RedisSerializer;
  import org.springframework.data.redis.serializer.StringRedisSerializer;
  
  @Configuration
  public class RedisConfig {
      @Bean(name = "redisTemplate")
      public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory){
  
          RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
          //参照StringRedisTemplate内部实现指定序列化器
          redisTemplate.setConnectionFactory(redisConnectionFactory);
          redisTemplate.setKeySerializer(keySerializer());
          redisTemplate.setHashKeySerializer(keySerializer());
          redisTemplate.setValueSerializer(valueSerializer());
          redisTemplate.setHashValueSerializer(valueSerializer());
          return redisTemplate;
      }
  
      private RedisSerializer<String> keySerializer(){
          return new StringRedisSerializer();
      }
  
      //使用Jackson序列化器
      private RedisSerializer<Object> valueSerializer(){
          return new GenericJackson2JsonRedisSerializer();
      }
  }
  ```

### 四、操作数据

Spring Boot 的 spring-boot-starter-data-redis 为 Redis 的相关操作提供了一个高度封装的 RedisTemplate 类，而且对每种类型的数据结构都进行了归类，将同一类型操作封装为 operation 接口。RedisTemplate 对五种数据结构分别定义了操作，如下所示：

+ 操作字符串：redisTemplate.opsForValue()
+ 操作 Hash：redisTemplate.opsForHash()
+ 操作 List：redisTemplate.opsForList()
+ 操作 Set：redisTemplate.opsForSet()
+ 操作 ZSet：redisTemplate.opsForZSet()

但是对于 string 类型的数据，Spring Boot 还专门提供了 StringRedisTemplate 类，而且官方也建议使用该类来操作 String 类型的数据。那么它和 RedisTemplate 又有啥区别呢？

+ RedisTemplate 是一个泛型类，而 StringRedisTemplate 不是，后者只能对键和值都为 String 类型的数据进行操作，而前者则可以操作任何类型。
+ 两者的数据是不共通的，StringRedisTemplate 只能管理 StringRedisTemplate 里面的数据，RedisTemplate 只能管理 RedisTemplate 中 的数据。

``` java
//增删改查，操作字符串
stringRedisTemplate.opsForValue().set("test-string-value", "Hello Redis");
stringRedisTemplate.opsForValue().get("test-string-value");
stringRedisTemplate.opsForValue().set("test-string-key-time-out", "Hello Redis", 3, TimeUnit.HOURS);
}
stringRedisTemplate.opsForValue().getAndSet(key, value);
stringRedisTemplate.delete("test-string-value");
//还有很多操作在IDE中体现
//还可以操作数组、Hash、集合、有序集合
```

[这里](https://www.ibm.com/developerworks/cn/java/know-redis-and-use-it-in-springboot-projects/index.html)有其他详情，[这里](https://www.cnblogs.com/kingstar718/p/10941958.html)也有很多操作举例

### 五、可视化工具

Redis Desktop Manager是redis的可视化工具，可以查看数据是否插入成功

### 六、Redis操作工具类

如果很多地方都要用到RedisTemplate或StringRedisTemplate，那我们可以将其包装成一个工具类，便于对redis进行数据操作，代码会简洁许多

![image-20200313153036083](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20200313153036083.png)

```java
package edu.qingtai.pubandcollect.util;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.stereotype.Component;

import java.util.concurrent.TimeUnit;

@Component
public class RedisUtils {
    private StringRedisTemplate stringRedisTemplate;

    @Autowired
    public RedisUtils(final StringRedisTemplate stringRedisTemplate){
        this.stringRedisTemplate=stringRedisTemplate;
    }

    public String get(final String key){
        return stringRedisTemplate.opsForValue().get(key);
    }

    public void set(final String key, final String value, long i, TimeUnit timeUnit){
        stringRedisTemplate.opsForValue().set(key, value, i, timeUnit);
    }

    public boolean delete(final String key){
        return stringRedisTemplate.delete(key);
    }
}
```

不能通过以下使用

```java
RedisUtils.get(rd3session)
```

因为方法并不是静态方法，而且如果要变成静态方法，就不能使用非静态对象了，所以要在使用它的地方进行依赖注入。