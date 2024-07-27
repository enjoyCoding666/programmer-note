
### 依赖包：
```
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-pool2</artifactId>
        </dependency>

```

### 配置文件
如果是 properties 文件，使用：
```
spring.redis.host=127.0.0.1
spring.redis.port=6379
spring.redis.database=0
##连接超时时间（毫秒）
spring.redis.timeout=1800000
spring.redis.password=123456
##连接池最大阻塞等待时间（使用负值表示没有限制）
spring.redis.lettuce.pool.max-wait=-1
##连接池中的最大空闲连接
spring.redis.lettuce.pool.max-idle=8
##连接池中的最小空闲连接
spring.redis.lettuce.pool.min-idle=0
##连接池最大连接数（使用负值表示没有限制）
spring.redis.lettuce.pool.max-active=20
```

如果用的是 yml文件，则是：
```
spring:
    redis:
        database: 0
        host: 127.0.0.1
        lettuce:
            pool:
                max-active: 20
                max-idle: 8
                max-wait: -1
                min-idle: 0
        password: 123456
        port: 6379
        timeout: 180000
```

### Redis配置：
```
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.RedisSerializer;

@Configuration
public class RedisConfig {
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        // 创建模板
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        // 设置连接工厂
        redisTemplate.setConnectionFactory(redisConnectionFactory);
        // 设置序列化工具
        GenericJackson2JsonRedisSerializer jsonRedisSerializer =
                new GenericJackson2JsonRedisSerializer();
        // key和 hashKey采用 string序列化
        redisTemplate.setKeySerializer(RedisSerializer.string());
        redisTemplate.setHashKeySerializer(RedisSerializer.string());
        // value和 hashValue采用 JSON序列化
        redisTemplate.setValueSerializer(jsonRedisSerializer);
        redisTemplate.setHashValueSerializer(jsonRedisSerializer);
        return redisTemplate;
    }
}
```

### 代码示例
```

@Service
public class RedisTestServiceImpl {

    @Autowired
    private StringRedisTemplate redisTemplate;

    /**
     * 缓存的前缀
     */
    private static final String PREFIX = "PREFIX_";


    public String getData(String id) {
        String key = PREFIX + id;
        String data = redisTemplate.opsForValue().get(key);
        if (StringUtils.isNotBlank(data)) {
            return data;
        }
        //实际中一般是从数据库中查询，有数据就插入缓存。
        String value = "test";
        //缓存5分钟
        redisTemplate.opsForValue().set(key, value, 5L, TimeUnit.MINUTES);
        return value;
    }


}
```


### 参考资料：
https://blog.csdn.net/weixin_59654772/article/details/125692784
