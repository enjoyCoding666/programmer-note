### shardingsphere-jdbc 5.0 有什么优点？

5.0之前的版本，不支持CASE WHEN、HAVING、UNION(ALL),有限支持子查询。

5.0支持这些特性，开发起来会更方便些。



### 依赖包

SpringBoot 用的是 2.6.13 版本。

```
<dependency>
    <groupId>org.apache.shardingsphere</groupId>
    <artifactId>shardingsphere-jdbc-core-spring-boot-starter</artifactId>
    <version>5.0.0</version>
</dependency>

<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.22</version>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
</dependency>

<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>

<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.0.6</version>
</dependency>

<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.70</version>
</dependency>

```

注意，sharding-jdbc 不同版本的差异较大，如果引入 其他版本的 sharding-jdbc，以下的配置有可能不兼容。

### mysql 建表

```
CREATE TABLE tb_shard_test
(
    id          INT         NOT NULL AUTO_INCREMENT COMMENT '主键,自增id',
    order_id    VARCHAR(25) NOT NULL UNIQUE COMMENT '订单号,唯一',
    pay_status  INT UNSIGNED DEFAULT 0 COMMENT '10：未支付,20：支付成功,30：支付失败, 40：已下单,50：申请退款,60：退款成功,70：退款失败 ',
    user_id     BIGINT(20)  NOT NULL COMMENT '用户id',
    total_price DECIMAL(25, 2)   DEFAULT 0.00 COMMENT '交易金额',
    result      TEXT COMMENT '结果',
    order_desc  VARCHAR(128)     DEFAULT '' COMMENT '订单描述',
    order_date  DATE             DEFAULT NULL COMMENT '订单日期',
    create_time DATETIME         DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间，默认当前时间',
    update_time DATETIME         DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间，更新时默认当前时间',
    is_delete   TINYINT(1)       DEFAULT 0 COMMENT '是否删除,0表示否,1表示是',
    PRIMARY KEY (id),
    INDEX idx_order (order_id)
) ENGINE = INNODB
  DEFAULT CHARSET = utf8
  AUTO_INCREMENT = 1 COMMENT ='示例表';
```

建立多个分表：

```
CREATE TABLE tb_shard_test_0 LIKE tb_shard_test;
CREATE TABLE tb_shard_test_1 LIKE tb_shard_test;
CREATE TABLE tb_shard_test_2 LIKE tb_shard_test;
CREATE TABLE tb_shard_test_3 LIKE tb_shard_test;
CREATE TABLE tb_shard_test_4 LIKE tb_shard_test;
CREATE TABLE tb_shard_test_5 LIKE tb_shard_test;
CREATE TABLE tb_shard_test_6 LIKE tb_shard_test;
CREATE TABLE tb_shard_test_7 LIKE tb_shard_test;
```

插入数据：
```
INSERT INTO tb_shard_test_2 (id, order_id, pay_status, user_id, total_price, result, order_desc, order_date, create_time, update_time, is_delete) VALUES (1, '1234', 1, 666666666, 12.82, null, '', null, '2023-06-28 23:18:52', '2023-06-28 23:18:52', 0);
```



### 配置环境：
在 application.properties 中指定使用 哪个环境的配置文件：

```
server.port=8080

spring.profiles.active=dev
```
当 spring.profiles.active 为 dev 时，会读取 application-dev.properties 的配置。
当 spring.profiles.active 为 test 时，会读取 application-test.properties 的配置。

直接在 application.properties 中指定 spring.profiles.active，
如果没有配置中心，会比较麻烦。
可以通过 maven 的 pom.xml 配置profile指定环境，
详情见：https://www.cnblogs.com/expiator/p/17544134.html

### sharding-jdbc 配置
新建 application-dev.properties 文件，配置值如下：

```
# Sharding Jdbc配置
# 配置数据源的名称为 ds，后面的会经常用到这个定义好的名称 ds
spring.shardingsphere.datasource.names=ds

# 配置ds
# 如果数据库连接使用的是 hikari，那么就换成 com.zaxxer.hikari.HikariDataSource
spring.shardingsphere.datasource.ds.type=com.alibaba.druid.pool.DruidDataSource
spring.shardingsphere.datasource.ds.driver-class-name=com.mysql.jdbc.Driver
#如果是高版本的mysql，使用驱动路径 com.mysql.cj.jdbc.Driver
#spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.shardingsphere.datasource.ds.url=jdbc:mysql://mysql的ip:端口/库名?useUnicode=true&characterEncoding=utf-8&allowMultiQueries=true
spring.shardingsphere.datasource.ds.username=账号
spring.shardingsphere.datasource.ds.password=密码


# 分表配置
# 分为8张表， tb_shard_test 是表名。
spring.shardingsphere.rules.sharding.tables.tb_shard_test.actual-data-nodes=ds.tb_shard_test_$->{0..7}
# 分表字段
spring.shardingsphere.rules.sharding.tables.tb_shard_test.table-strategy.standard.sharding-column=order_id
#分表策略的类，该类必须实现 StandardShardingAlgorithm 接口.且类对应的Component注解名称为 preciseShardingTableAlgorithm.
spring.shardingsphere.rules.sharding.tables.tb_shard_test.table-strategy.standard.sharding-algorithm-name=preciseShardingTableAlgorithm

# 打印分库分表日志
spring.shardingsphere.props.sql-show=true


#以下配置可选
#spring.shardingsphere.datasource.ds.initialSize=5
#spring.shardingsphere.datasource.ds.minIdle=5
#spring.shardingsphere.datasource.ds.maxIdle=100
#spring.shardingsphere.datasource.ds.maxActive=20
#spring.shardingsphere.datasource.ds.maxWait=60000
#spring.shardingsphere.datasource.ds.validationQuery=SELECT 1 FROM DUAL
#spring.shardingsphere.datasource.ds.timeBetweenEvictionRunsMillis=60000
#spring.shardingsphere.datasource.ds.minEvictableIdleTimeMillis=300000

```



### 自定义分表策略：

在之前的配置中，有一个配置是：

```
spring.shardingsphere.rules.sharding.tables.tb_shard_test.table-strategy.standard.sharding-algorithm-name=preciseShardingTableAlgorithm
```

它表示的是 分表策略的类，该类必须实现 StandardShardingAlgorithm 接口.且类对应的Component注解名称为 preciseShardingTableAlgorithm.

代码如下：

```
import com.alibaba.fastjson.JSON;
import lombok.extern.slf4j.Slf4j;
import org.apache.shardingsphere.sharding.api.sharding.standard.PreciseShardingValue;
import org.apache.shardingsphere.sharding.api.sharding.standard.RangeShardingValue;
import org.apache.shardingsphere.sharding.api.sharding.standard.StandardShardingAlgorithm;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;
import java.util.Collection;

/**
 *
 * 分表策略
 *
 **/
@Slf4j
@Component(value = "preciseShardingTableAlgorithm")
public class PreciseShardingTableAlgorithm implements StandardShardingAlgorithm<String> {

    /**
     * 分表数量
     */
    @Value("${order.table.size:8}")
    private int tableSize;


    /**
     * 分表策略
     *
     * @param tableNames 所有表名
     * @param preciseShardingValue 精确分片值，包括（columnName，logicTableName，value）
     * @return 表名
     */
    @Override
    public String doSharding(Collection<String> tableNames, PreciseShardingValue<String> preciseShardingValue) {
        log.info("doSharding tableNames:{} ,preciseShardingValue: {}.",
                JSON.toJSONString(tableNames), JSON.toJSONString(preciseShardingValue));
        //分表策略：根据分表字段求出 hashCode的绝对值, 再取模
        String shardingValue = preciseShardingValue.getValue();
        int mod = Math.abs(shardingValue.hashCode()) % tableSize;

        for (String tableName : tableNames) {
            // 分表的规则
            if (tableName.endsWith(String.valueOf(mod))) {
                return tableName;
            }
        }
        throw new UnsupportedOperationException();
    }

    @Override
    public Collection<String> doSharding(Collection<String> collection, RangeShardingValue<String> rangeShardingValue) {
        return null;
    }


    @Override
    public void init() {

    }

    @Override
    public String getType() {
        return null;
    }
}

```



### Service层代码

此处用的是 Mybatis-Plus，相关讲解见：https://www.cnblogs.com/expiator/p/17125880.html

也可以直接用 mybatis 写sql。

增删改查时，不需要特别指定是哪一张分表。sharding-jdbc 会进行处理。

```
@Service
public class ShardTestServiceImpl extends ServiceImpl<ShardTestMapper, ShardTestEntity> implements ShardTestService {

    /**
     * 根据 orderId 查询结果
     */
    public List<ShardTestEntity> getByOrderId(String orderId) {
        LambdaQueryWrapper<ShardTestEntity> queryWrapper = new LambdaQueryWrapper<>();
        queryWrapper.eq(ShardTestEntity::getOrderId, orderId);
        return list(queryWrapper);

    }


}

```



### 测试代码：

由于是根据  orderId的 hashCode 对8取余进行分表。

orderId为 12345 时， 计算结果为 3，保存/查询就都在 tb_shard_test_3 表。
保存数据后，可以在 tb_shard_test_3 查到。

orderId为 1234 时， 计算结果为 2，保存/查询就都 tb_shard_test_2 表。

如下：

```
    @Test
    public void testSave() throws Exception {
        String orderId = "12345";
        long userId = 123L;
        //根据 hashCode 对8取余，可以知道数据存在哪一张分表上。
        int partition = Math.abs(orderId.hashCode()) % 8;
        System.out.println("partition:"+ partition);

        ShardTestEntity shardTestEntity = new ShardTestEntity();
        shardTestEntity.setOrderId(orderId);
        shardTestEntity.setUserId(userId);
        //保存
        shardTestService.save(shardTestEntity);

    }

 	@Test
    public void testGetByOrderId() throws Exception {
        String orderId = "1234";
        //根据 hashCode 对8取余，可以知道数据存在哪一张分表上。
        int partition = Math.abs(orderId.hashCode()) % 8;
        System.out.println("partition:"+ partition);

        List<ShardTestEntity> entities = shardTestService.getByOrderId(orderId);
        System.out.println(JSON.toJSONString(entities));
    }
```

运行单元测试，可以看到：
查询的时候，"orderId"为 "1234"时，自动选择了对应的分表  tb_shard_test_2。

```
ShardingSphere-SQL: Actual SQL: ds ::: SELECT  id,order_id,pay_status,user_id,total_price,result,order_desc,order_date,create_time,update_time,is_delete  FROM tb_shard_test_2  
 WHERE order_id = ? ::: [1234]
[{"createTime":1687965532000,"id":1,"isDelete":false,"orderDesc":"","orderId":"1234","payStatus":1,"totalPrice":12.82,"updateTime":1687965532000,"userId":666666666}]

```

### 分库分表
本文主要是分表，没有分库。
分库分表，详情见： https://www.cnblogs.com/expiator/p/17530648.html

### 参考资料：

https://blog.csdn.net/m0_47503416/article/details/124189469

### 官方文档 ：

https://shardingsphere.apache.org/document/