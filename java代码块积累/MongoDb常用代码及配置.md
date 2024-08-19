### MongoDb常用的语句：
详情见：[ https://blog.csdn.net/sinat_32502451/article/details/134342559]( https://blog.csdn.net/sinat_32502451/article/details/134342559)

### MongoDb建表：

MongoDB不需要建表，直接插入数据就会建表。
日期用 ISODate() 转换。

```
db.getCollection("mongoDbTest").insert({userId:"dxcefg", status:1,  price:1.23, updateTime : ISODate("2022-02-13T07:06:25.371Z")})
```



### 添加 maven 依赖：

```
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-mongodb</artifactId>
        </dependency>
```



### 添加 application.yml 配置

```
#数据库配置
spring:
  data:
    mongodb:
      # uri格式为：mongodb://用户名:密码@IP地址:端口/数据库
      uri: mongodb://xx:xxx@xx.xx.xx.xx:xx/xx

#mongodb打印日志
logging:
  level:
    org.springframework.data.mongodb.core.MongoTemplate: DEBUG
```

如果使用 properties文件，则是：
```
spring.data.mongodb.uri=mongodb://xx:xxx@xx.xx.xx.xx:xx/xx
#mongodb打印日志
logging.level.org.springframework.data.mongodb.core.MongoTemplate=DEBUG
```



### 数据表对应的Bean：
@Document 指定表名。

```
@Data
@Document(collection = "mongoDbTest")
public class MongoDbTest {

    private String userId;

    private int status;

    private BigDecimal price;

    private Date updateTime;

	//忽略 getter()、setter()
}
```



### 查询

```
@Service
public class MongoDbService {

    @Resource
    private MongoTemplate mongoTemplate;

    public void testMongoDb() {

        Criteria criteria = new Criteria();
        Query query = new Query();
        criteria.and("status").is(1);
        query.addCriteria(criteria);
		
	//排序
        List<Sort.Order> orders = new ArrayList<Sort.Order>();
        orders.add(new Sort.Order(Sort.Direction.DESC, "updateTime"));
        query.with(Sort.by(orders));
        query.limit(1000);
	//查询
        List<MongoDbTest> list = mongoTemplate.find(query, MongoDbTest.class);
        System.out.println("MongoDbTest list:" + JSON.toJSONString(list));

    }
}
```


### 插入

```
    public void testMongoDbInsert() {
        MongoDbTest mongoDbTest = new MongoDbTest();
        mongoDbTest.setUserId("081914");
        mongoDbTest.setPrice(BigDecimal.TEN);
        mongoDbTest.setStatus(2);
        mongoDbTest.setUpdateTime(new Date());

        //表名
        String collectionName = "mongoDbTest";
        //插入
        mongoTemplate.insert(mongoDbTest, collectionName);

    }
```

### 更新/修改

```
    public void testMongoDbUpdate() {
        //要更新的数据
        Update update = Update.update("status", 789);
        //查询条件
        Query query = new Query();
        query.addCriteria(Criteria.where("userId").is("abc"));
        //表名
        String collectionName = "mongoDbTest";

        //更新符合条件的所有数据
        mongoTemplate.updateMulti(query, update, collectionName);

        //只更新一条数据
//        mongoTemplate.updateFirst(query, update, collectionName);

    }
```


### 删除

```
    public void testMongoDbDelete() {

        //查询条件
        Query query = new Query();
        query.addCriteria(Criteria.where("userId").is("abcdefg"));
        //表名
        String collectionName = "mongoDbTest";
        //删除
        mongoTemplate.remove(query, collectionName);

    }
```
