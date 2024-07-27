



### RedisTemplate

如果想要在java中使用Redis相关的数据结构，要先注入RedisTemplate。

```
    @Autowired
    private RedisTemplate<K,V>  redisTemplate;
```

其中K，V类型，可以使用具体的类型，比如String或者其他具体类。

```
    @Autowired
    private RedisTemplate<String,Integer>  redisTemplate;
```

### StringRedisTemplate

如果key和value是String类型的，可以用StringRedisTemplate 。
其他对象，也可以转换为String后使用StringRedisTemplate.

```
    @Autowired
    private StringRedisTemplate stringRedisTemplate;
```

### String(字符串)

Redis缓存，设置key和value。

value可以是普通的字符串，也可以将对象(或者List )转换为Json字符串，设置为value。

对象转换为Json字符串的资料，详情见：[https://blog.csdn.net/sinat_32502451/article/details/132819550](https://blog.csdn.net/sinat_32502451/article/details/132819550)


* value为对象的Json字符串
  示例：

```
    public Person getPerson() {
        String cacheKey = 缓存key的前缀 + 字段组成的唯一id;
        String valueStr = stringRedisTemplate.opsForValue().get(cacheKey);
        if (StringUtil.isNotBlank(valueStr)) {
            return JSONObject.parseObject(valueStr, Person.class);
        }
        Person Person = 此处为业务逻辑;
        if (Person != null) {
            stringRedisTemplate.opsForValue().set(cacheKey, JSONObject.toJSONString(person), 1, TimeUnit.HOURS);
        }
        return Person;
    }
```

* value为泛型对象( 比如泛型对象的List )的Json字符串
  value为泛型对象，需要用TypeReference转换为json字符串。

```
JSON.parseObject(valueStr, new TypeReference<List<Person>>(){})
```

如下所示：

```
    public List<Person> getIdList(Dto dto) {
        String cacheKey = 缓存key的前缀 + 字段组成的唯一id;
        String valueStr = stringRedisTemplate.opsForValue().get(cacheKey);
        if (StringUtil.isNotBlank(valueStr)) {
            return JSON.parseObject(valueStr, new TypeReference<List<Person>>(){});
        }
        List<Person> list = 此处为业务逻辑;
        stringRedisTemplate.opsForValue().set(cacheKey, JSONObject.toJSONString(list), 1, TimeUnit.HOURS);

        return list;
    }
```

* 特殊的对象、数据结构。
  有些对象的数据结构比较特殊，用普通的json字符串缓存，在转化json的过程可能会有问题。
  比如guava包的 Table。缓存此类特殊的对象，可以用Jedis连接，再用byte[]作为key和value。

```
import org.apache.commons.lang3.SerializationUtils;

    public Person getValueCache() {
        Jedis jedis = jedisPool.getResource();
        String key = 缓存前缀+字段组成的唯一id;
        byte[] bytes = jedis.get(key.getBytes());
        if (bytes != null) {
         //key和value均为byte[]
            return SerializationUtils.deserialize(bytes);
        }

        Person obj = 业务逻辑取值;
        byte[] byteArray = getByteArray(obj);
        if (byteArray != null) {
            jedis.set(key.getBytes(), byteArray, 60*60*2);
        }
        return obj;
    }

    public byte[] getByteArray(Object obj) {
        byte[] byteArray = null;
        try {
            ByteArrayOutputStream os = new ByteArrayOutputStream();
            ObjectOutputStream oos = new ObjectOutputStream(os);
            oos.writeObject(obj);
            byteArray = os.toByteArray();
            oos.close();
            os.close();
        } catch (Exception e) {
            log.error("get getByteArray error.obj:{}", obj, e);
        }
        return byteArray;
    }
```

* 查看key是否存在：

```
stringRedisTemplate.opsForValue().set("name", "feng");
//检查key是否存在，存在就返回boolean值 true
Boolean hasKey = stringRedisTemplate.hasKey("name");
System.out.println(hasKey);
```

* redis 删除指定前缀的键：
  比如 前缀 prefix  为 userId123，那么会删除所有 userId123 开头的缓存。*表示其他任意字符。
  使用时要注意些，有些团队不允许使用 keys 这个命令。

```
Set<String> keys = stringRedisTemplate.keys(prefix + "*");
 if (CollectionUtils.isNotEmpty(keys)) {
     stringRedisTemplate.delete(keys);
 }
```


* redis设置超时时间，第三个参数 TimeUnit 是超时时间的单位。
```
stringRedisTemplate.expire(key, 30, TimeUnit.MINUTES);
```

### 计数器

* increment 做计数器：

```
//计数器，每次调用加1
Long count = stringRedisTemplate.opsForValue().increment("count12345");
//计数器，每次调用加10
Long countMore = stringRedisTemplate.opsForValue().increment("count45678", 10);
System.out.println(count);
System.out.println(countMore);

//获取计数器的值，可以使用 apache commons-lang3包的 NumberUtils 工具类转换
String value =  stringRedisTemplate.opsForValue().get("count12345");
long valueCount = NumberUtils.toLong(value);

```

### List(队列)

Redis队列通过redisTemplate.opsForList()来操作。
常用api如下：

```
redisTemplate.opsForList().rightPush(K  key ,V  value);  //表示从队列的右侧放入新的值 ，其中key为队列名，value为入列的值

redisTemplate.opsForList().leftPop(K  key);            //取出队列最右侧的值

redisTemplate.opsForList().range(K key, long start, long end);         //遍历队列
```

示例如下：
从队列的最右侧放入新的数据。从队列的最左侧取出已有的数据。

```
@Autowired
private StringRedisTemplate stringRedisTemplate;

redisTemplate.opsForList().rightPush("list","python");

redisTemplate.opsForList().leftPop("list");
```

### Hash(散列)

Redis的散列可以让将多个键值对存储到一个Redis键里面，
hash 特别适合用于存储对象。

常用api：

```
void put(H key, HK hashKey, HV value);      //设置散列hashKey的值
HV get(H key, Object hashKey);      //从键中的哈希获取给定hashKey的值。返回类型HV表示HashValue
Set<HK> keys(H key);            //获取key所对应的散列表的key
Set<K> keys(K pattern) ;        //按照给定的pattern查找key。*表示所有的key，关键字加*表示模糊查询，比如user*表示所有带user的key。
Map<HK, HV> entries(H var1);	//获取指定hash的所有的map
Long increment(H key, HK hashKey, long delta);     //通过给定的delta增加散列hashKey的值（整型）
Boolean hasKey(H key, Object hashKey);     //确定哈希hashKey是否存在
Long size(H key);      //获取key所对应的散列表的大小个数
Cursor<Map.Entry<HK, HV>> scan(H key, ScanOptions options);       //使用Cursor在key的hash中迭代，相当于迭代器。
```

示例如下：

* 用一个Hash存储某个用户数据。

```
stringRedisTemplate.opsForHash().put("userId123", "name", "wu");
stringRedisTemplate.opsForHash().put("userId123", "age", "26");
stringRedisTemplate.opsForHash().put("userId123", "height", "110");
```

获取数据，做类型转换

```
String name = (String)stringRedisTemplate.opsForHash().get("userId123", "name");
```

* 获取一个hash的所有key：

```
//获取hash所有的key
Set<Object> keys = stringRedisTemplate.opsForHash().keys("userId123");
//遍历hash所有的key
stringRedisTemplate.opsForHash().keys("userId123").forEach(System.out::println);
```

* 获取一个hash所有的map：

```
stringRedisTemplate.opsForHash().entries("userId123")
```

示例：

```
stringRedisTemplate.opsForHash().put("userId123", "name", "wu");
stringRedisTemplate.opsForHash().put("userId123", "age", "26");
stringRedisTemplate.opsForHash().put("userId123", "height", "110");

//获取key对应的map
Map<Object, Object> map = stringRedisTemplate.opsForHash().entries("userId123");
//遍历map
stringRedisTemplate.opsForHash().entries("userId123").forEach((key,value)-> {
		System.out.println("key:"+ key +",value:"+ value);
	}
);
```

* 获取一个hash中多个key对应的所有values：

```
stringRedisTemplate.opsForHash().put("userId123", "name", "wu");
stringRedisTemplate.opsForHash().put("userId123", "age", "26");
stringRedisTemplate.opsForHash().put("userId123", "height", "110");

//获取一个hash中多个key对应的所有values
List<Object> values = stringRedisTemplate.opsForHash().multiGet("userId123", Arrays.asList(new String[]{"name", "height", "age"}));
values.forEach(System.out::println);
```


* 获取所有key包含的数据。
  通过keys(pattern)方式查找key，以下的pattern为user加上*表示所有包含user的key。

```
Set<String> setKeys = stringRedisTemplate.keys("user"+"*");
List<Object> list = new ArrayList<>();
for(String key:setKeys) {
	//取出所有数据
	List<Object> hashList = stringRedisTemplate.opsForHash().values(key);
	for(int i=0;i<hashList.size();i++) {
		Object object=hashList.get(i);
	}
}
```

* 使用Cursor遍历：

```
Cursor<Map.Entry<Object, Object>> curosr = template.opsForHash().scan("userId123", ScanOptions.NONE);
        while(curosr.hasNext()){
            Map.Entry<Object, Object> entry = curosr.next();
            System.out.println(entry.getKey()+":"+entry.getValue());
        }
```



### Set(无序集合)

Redis的Set是string类型的无序集合。集合成员是唯一的，这就意味着集合中不能出现重复的数据。
Redis 中 集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是O(1)。
常用api如下:

```
Long add(K key, V... values);
Long remove(K key, Object... values);
Cursor<V> scan(K key, ScanOptions options);            //遍历set

```

### ZSET(有序集合)

* ZSet是有序的集合，且不允许重复的成员。
  可以用ZSet做排行榜

* ZSet新增元素：

```
stringRedisTemplate.opsForZSet().add(KEY, VALUE, SCORE);
```

add()方法，最后一个参数是分数，ZSet的排行，就是根据分数来排的。

* ZSet查看分数：

```
Double score = stringRedisTemplate.opsForZSet().score(KEY, VALUE);
```

- ZSet返回分数最低的前十名：

```vbnet
Set<String> rangeSet = stringRedisTemplate.opsForZSet().range(KEY, 0, 9);
```

- ZSet返回分数最高的前十名：

```vbnet
Set<String> values = stringRedisTemplate.opsForZSet().reverseRange(KEY, 0, 9);
```

- ZSet返回分数在范围内的数据，按照**分值从小到大**的顺序返回：

```vbnet
Set<String> values = stringRedisTemplate.opsForZSet().rangeByScore(KEY, 1.0, 2.0);
```

- ZSet返回分数在范围内的数据，按照**分值从大到小**的顺序返回：

```vbnet
Set<String> values = stringRedisTemplate.opsForZSet().reverseRangeByScore(KEY, 1.0, 2.0);
```

- ZSet移除元素：

```
stringRedisTemplate.opsForZSet().remove(KEY,VALUE);
```


### 其他

Redis工具类，如下：

```
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.ListOperations;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.ValueOperations;
import org.springframework.data.redis.core.ZSetOperations;
import org.springframework.stereotype.Component;

import java.io.Serializable;
import java.util.List;
import java.util.Map;
import java.util.Set;
import java.util.concurrent.TimeUnit;

/**
 * redis缓存工具类
 *
 */
@Component
public class  RedisUtil {
    private static final Logger LOGGER = LoggerFactory.getLogger(RedisUtil.class);

    @Autowired
    private RedisTemplate redisTemplate;

    // redis 失效時間 15分鐘
    public static final Long REDIS_INVALID_COMMAND = (long) 60 * 15;
    // redis 失效時間 5分鐘
    public static final Long REDIS_INVALID_SECONDS = (long) 60 * 5;

    // redis 失效時間 2分鐘
    public static final Long REDIS_INVALID_2SECONDS = (long) 60 * 2;

    // redis 失效時間 1分鐘
    public static final Long REDIS_TIMEOUT_SECONDS = (long) 60;

    // redis 失效時間 3分鐘
    public static final Long WAREHOUSE_DATA_INVALID_SECONDS = (long) 60 * 3;

    // redis 失效時間 5秒钟
    public static final Long REDIS_INVALID_SECONDS2 = (long) 5;

    // redis 失效時間 30秒钟
    public static final Long REDIS_INVALID_SECONDS3 = (long) 30;

  // redis 失效時間 一周
    public static final Long REDIS_INVALID_WEEK = (long) 60*60*24*7;
    // redis 失效時間 一天
    public static final Long REDIS_INVALID_DAY = (long) 60*60*24;

    /**
     * 批量删除对应的value
     *
     * @param keys
     */
    public void remove(final String... keys) {
        for (String key : keys) {
            remove(key);
        }
    }

    /**
     * 批量删除key
     *
     * @param pattern
     */
    public void removePattern(final String pattern) {
        Set<Serializable> keys = redisTemplate.keys(pattern);
        if (keys.size() > 0) {
            redisTemplate.delete(keys);
        }
    }

    /**
     * 删除对应的value
     *
     * @param key
     */
    public void remove(final String key) {
        if (exists(key)) {
            redisTemplate.delete(key);
        }
    }

    /**
     * 判断缓存中是否有对应的value
     *
     * @param key
     * @return
     */
    public boolean exists(final String key) {
        return redisTemplate.hasKey(key);
    }

    /**
     * 读取缓存
     *
     * @param key
     * @return
     */
    public Object get(final String key) {
        Object result = null;
        ValueOperations<Serializable, Object> operations = redisTemplate.opsForValue();
        result = operations.get(key);
        return result;
    }

    /**
     * 读取缓存
     *
     * @param key
     * @return
     */
    public <T> T get(final String key, Class<T> clazz) {
        T result = null;
        ValueOperations<Serializable, Object> operations = redisTemplate.opsForValue();
        result = (T) operations.get(key);
        return result;
    }

    /**
     * 读取缓存list
     *
     * @param key
     * @return
     */
    public <T> List<T> getList(final String key, Class<T> clazz) {
        List<T> result = null;
        ListOperations operations = redisTemplate.opsForList();
        result = operations.range(key, 0, -1);
        return result;
    }

    /**
     * 写入缓存
     *
     * @param key
     * @param value
     * @return
     */
    public boolean set(final String key, Object value) {
        boolean result = false;
        try {
            ValueOperations<Serializable, Object> operations = redisTemplate.opsForValue();
            operations.set(key, value);
            result = true;
        } catch (Exception e) {
            LOGGER.error("operations error.",e);
        }
        return result;
    }

    /**
     * 写入缓存
     *
     * @param key
     * @param value
     * @return
     */
    public boolean set(final String key, Object value, Long expireTime) {
        boolean result = false;
        try {
            ValueOperations<Serializable, Object> operations = redisTemplate.opsForValue();
            operations.set(key, value);
            //EXPIRE key seconds为给定 key 设置生存时间,当 key 过期时(生存时间为 0 ),它会被自动删除。
            redisTemplate.expire(key, expireTime, TimeUnit.SECONDS);
            result = true;
        } catch (Exception e) {
            LOGGER.error("redisTemplate operation error.",e);
        }
        return result;
    }

    /**
     * 写入缓存list
     *
     * @param key
     * @param value
     * @return
     */
    public <T> boolean setList(final String key, List<T> value, Long expireTime, Class<T> clazz) {
        boolean result = false;
        try {
            ListOperations operations = redisTemplate.opsForList();
            operations.rightPushAll(key, value);
            //EXPIRE key seconds为给定 key 设置生存时间,当 key 过期时(生存时间为 0 ),它会被自动删除。
            redisTemplate.expire(key, expireTime, TimeUnit.SECONDS);
            result = true;
        } catch (Exception e) {
            LOGGER.error("setList error.",e);
        }
        return result;
    }

	
    /**
     * @param hashKey hash表的key
     * @param key     要保存的key
     * @param val     要保存的val
     */
    public boolean setHash(String hashKey, String key, Object val) {
        try {
            redisTemplate.opsForHash().put(hashKey, key, val);
            return true;
        } catch (Exception e) {
            LOGGER.error("RedisUtil setHash exception", e);
            return false;
        }

    }
    public boolean setHashAndExpire(String hashKey, String key, Object val,long expireTime) {
        try {
            boolean useExpire=false;
            if(!redisTemplate.hasKey(hashKey)){
                useExpire=true;
            }
            redisTemplate.opsForHash().put(hashKey, key, val);
            if(useExpire)
            redisTemplate.boundHashOps(hashKey).expire(expireTime,TimeUnit.SECONDS);
            return true;
        } catch (Exception e) {
            LOGGER.error("RedisUtil setHash exception", e);
            return false;
        }

    }


    /**
     * 从hash表中查询所有的数据
     *
     * @param hashKey hash表的key
     */
    public <T> Map<String, T> getHash(String hashKey) {
        try {
            return (Map<String, T>) (redisTemplate.boundHashOps(hashKey).entries());
        } catch (Exception e) {
            LOGGER.error("RedisUtil getHash exception", e);
            return null;
        }
    }

    public long getHashSize(String hashKey) {
        try {
            return redisTemplate.boundHashOps(hashKey).size();
        } catch (Exception e) {
            LOGGER.error("RedisUtil getHash exception", e);
            return 0;
        }
    }
    /**
     * 根据key从hash表中查询所有的数据
     *
     * @param hashKey hash表的key
     */
    public <T> T getHash(String hashKey, String key) {
        try {
            return (T) redisTemplate.boundHashOps(hashKey).get(key);
        } catch (Exception e) {
            LOGGER.error("RedisUtil getHash exception {}", e);
            return null;
        }
    }

    public boolean setIfAbsent(String key, Object value,long expireTime) {
        try {
            boolean result = redisTemplate.boundValueOps(key).setIfAbsent(value);
            redisTemplate.boundValueOps(key).expire(expireTime,TimeUnit.SECONDS);
            return result;
        } catch (Exception e) {
            LOGGER.error("RedisUtil getHash exception {}", e);
            return false;
        }
    }
    public boolean hasHash(String hashKey, String key,long expireTime) {
        try {
            if(!redisTemplate.hasKey(hashKey)){
                redisTemplate.boundHashOps(hashKey).put("test","1");
                redisTemplate.boundHashOps(hashKey).expire(expireTime,TimeUnit.SECONDS);
                return false;
            }
            return  redisTemplate.boundHashOps(hashKey).hasKey(key);
        } catch (Exception e) {
            LOGGER.error("RedisUtil getHash exception {}", e);
            return false;
        }
    }
    /**
     * 从hash表中删除数据
     *
     * @param hashKey
     * @param key
     */
    public void delHash(String hashKey, String key) {
        try {
            redisTemplate.boundHashOps(hashKey).delete(key);
        } catch (Exception e) {
            LOGGER.error("RedisUtil delHash exception", e);
        }
    }

    /**
     * 删除整个hash表
     *
     * @param hashKey
     */
    public void delHash(String hashKey) {
        try {
            redisTemplate.expire(hashKey, 0, TimeUnit.SECONDS);
        } catch (Exception e) {
            LOGGER.error("RedisUtil delHash exception", e);
        }
    }

    public <T> void addToSet(String key, List<T> values, Long expire) {
        ZSetOperations<String, T> set = redisTemplate.opsForZSet();
        for (int i = 0; i < values.size(); i++) {
            set.add(key, values.get(i), i);
        }
        if(expire!=null)
        redisTemplate.expire(key, expire, TimeUnit.SECONDS);
    }

    public <T> Set<T> getDatasFormSet(String key, int start, int end) {
        ZSetOperations<String, T> set = redisTemplate.opsForZSet();
        Set<T> results = set.range(key, start, end);
        return results;
    }

    public long getCountFormSet(String key) {
        ZSetOperations<String, String> set = redisTemplate.opsForZSet();
        return set.size(key);
    }

    public Set<String> getKeys(String cacheKeyPre) {
        return redisTemplate.keys(cacheKeyPre);
    }


}


```

### 更详细的讲解请见：

https://www.jianshu.com/p/7bf5dc61ca06
https://blog.csdn.net/lydms/article/details/105224210
