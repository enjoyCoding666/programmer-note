
### 依赖包：
不要使用太低的 版本，低版本有内存泄露的问题。可以使用 3.18 及以上的版本。

```
        <dependency>
            <groupId>org.redisson</groupId>
            <artifactId>redisson</artifactId>
            <version>3.18.1</version>
        </dependency>
```



### Redis配置文件：

可以在 application.properties 中添加。

```
spring.redis.host=127.0.0.1
spring.redis.port=6379
spring.redis.database=0
##连接超时时间（毫秒）
spring.redis.timeout=1800000
spring.redis.password=
##连接池最大阻塞等待时间（使用负值表示没有限制）
spring.redis.lettuce.pool.max-wait=-1
##连接池中的最大空闲连接
spring.redis.lettuce.pool.max-idle=8
##连接池中的最小空闲连接
spring.redis.lettuce.pool.min-idle=0
##连接池最大连接数（使用负值表示没有限制）
spring.redis.lettuce.pool.max-active=20
```



### Redis的第一种配置方式：

```
@Configuration
@ConditionalOnMissingBean(RedissonClient.class)
@Import(RedissonClientConfig.RedissonConfig.class)
public class RedissonClientConfig {

    private final Config config;

    RedissonClientConfig(Config config) {
        this.config = config;
    }

    @Bean
    public RedissonClient redissonClient() {
        return Redisson.create(config);
    }

    @ConditionalOnMissingBean(Config.class)
    @EnableConfigurationProperties({RedisProperties.class})
    static class RedissonConfig {

        @Autowired
        private RedisProperties redisProperties;

        @Bean
        public Config redissonConfig() {
            Config config = new Config();

            //哨兵模式
            if (redisProperties.getSentinel() != null) { //sentinel
                SentinelServersConfig sentinelServersConfig = config.useSentinelServers();
                org.springframework.boot.autoconfigure.data.redis.RedisProperties.Sentinel sentinel = redisProperties.getSentinel();
                sentinelServersConfig.setMasterName(sentinel.getMaster());
                sentinelServersConfig.addSentinelAddress(sentinel.getNodes().toArray(new String[sentinel.getNodes().size()]));
                sentinelServersConfig.setDatabase(redisProperties.getDatabase());
                baseConfig(sentinelServersConfig, redisProperties);

                //集群模式
            } else if (redisProperties.getCluster() != null) { //cluster
                ClusterServersConfig clusterServersConfig = config.useClusterServers();
                org.springframework.boot.autoconfigure.data.redis.RedisProperties.Cluster cluster = redisProperties.getCluster();
                clusterServersConfig.addNodeAddress(cluster.getNodes().toArray(new String[cluster.getNodes().size()]));
                clusterServersConfig.setFailedSlaveReconnectionInterval(cluster.getMaxRedirects());
                baseConfig(clusterServersConfig, redisProperties);

                //普通模式
            } else { //single server
                SingleServerConfig singleServerConfig = config.useSingleServer();
                // format as redis://127.0.0.1:7181 or rediss://127.0.0.1:7181 for SSL
                String schema = redisProperties.isSsl() ? "rediss://" : "redis://";
                singleServerConfig.setAddress(schema + redisProperties.getHost() + ":" + redisProperties.getPort());
                singleServerConfig.setDatabase(redisProperties.getDatabase());
                baseConfig(singleServerConfig, redisProperties);
            }
            config.setCodec(new JsonJacksonCodec());
            return config;
        }

        private void baseConfig(BaseConfig config, RedisProperties properties) {
            if (!StringUtils.isBlank(properties.getPassword())) {
                config.setPassword(properties.getPassword());
            }
            if (properties.getTimeout() != null) {
                config.setTimeout(Long.valueOf(properties.getTimeout().getSeconds() * 1000).intValue());
            }
            if (!StringUtils.isBlank(properties.getClientName())) {
                config.setClientName(properties.getClientName());
            }
        }
    }

}

```



### Reids 的第二种配置方式：

如果只需要普通的模式，也可以直接配置如下。

```
@Configuration
public class RedissonConfig {
	/**
	  *  冒号后面是默认值.如果配置文件中没有配置，就会取默认值。
	  */
    @Value("${spring.redis.host:127.0.0.1}")
    private String host;

    @Value("${spring.redis.port:6379}")
    private int port;

    @Value("${spring.redis.password:abc}")
    private String password;

    @Value("${spring.redis.database:0}")
    private int database;

    @Value("${spring.redis.pool.pingInternal:0}")
    private int pingInternal;

    @Bean(name = "redisson")
    public RedissonClient getRedisson() {
        Config config = new Config();
        config.useSingleServer()
                .setAddress(StringUtils.join("redis://", host, ":", port))
                .setDatabase(database)
                .setPassword(password)
                .setPingConnectionInterval(pingInternal);

        return Redisson.create(config);
    }
}
```









### Redisson分布式锁 示例：

```
@Service
public class RedissonServiceImpl {
	/**
	  * 引入 redisson
	  */
    @Autowired
    private RedissonClient redissonClient;

    public void getRedissonLock() {
        RLock lock = redissonClient.getLock("myLock");
        lock.lock();
        System.out.println("已加锁，在此处执行逻辑.");
        lock.unlock();
    }

}
```

### Redisson延时队列示例：
[https://blog.csdn.net/sinat_32502451/article/details/133800469](https://blog.csdn.net/sinat_32502451/article/details/133800469)




### 参考资料：

https://huaweicloud.csdn.net/637eeec0df016f70ae4c9de7.html
https://blog.csdn.net/weixin_53922163/article/details/127482085


