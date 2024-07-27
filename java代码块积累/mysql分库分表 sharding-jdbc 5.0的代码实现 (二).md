### 分库分表

之前试过了分表不分库，详情见：https://www.cnblogs.com/expiator/p/17524493.html

这次再试下分库分表。



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



由于是分库分表，因此还需要在其他的库，执行一下以上的建表语句sql。



### 配置环境：

在 application.properties 中指定使用 哪个环境的配置文件：

```
server.port=8080

spring.profiles.active=dev
```

当 spring.profiles.active 为 dev 时，会读取 application-dev.properties 的配置。
当 spring.profiles.active 为 test 时，会读取 application-test.properties 的配置。



### sharding-jdbc 配置

新建 application-dev.properties 文件，配置值如下：

分库分表的配置，会比分表不分库稍微多一些。

```
# Sharding Jdbc配置

# dbm为主库
# 配置好数据源的名称，后面的会经常用到这些定义好的名称
spring.shardingsphere.datasource.names=dbm,db0,db1

# 配置主库
# 如果数据库连接使用的是 hikari，那么就换成 com.zaxxer.hikari.HikariDataSource
spring.shardingsphere.datasource.dbm.type=com.alibaba.druid.pool.DruidDataSource
#如果是高版本的mysql，使用驱动路径 com.mysql.cj.jdbc.Driver
spring.shardingsphere.datasource.dbm.driver-class-name=com.mysql.jdbc.Driver
spring.shardingsphere.datasource.ds.url=jdbc:mysql://mysql的ip:端口/库名?useUnicode=true&characterEncoding=utf-8&allowMultiQueries=true
spring.shardingsphere.datasource.ds.username=账号
spring.shardingsphere.datasource.ds.password=密码

# 配置db0
spring.shardingsphere.datasource.db0.type=com.alibaba.druid.pool.DruidDataSource
spring.shardingsphere.datasource.db0.driver-class-name=com.mysql.jdbc.Driver
spring.shardingsphere.datasource.ds.url=jdbc:mysql://mysql的ip:端口/库名?useUnicode=true&characterEncoding=utf-8&allowMultiQueries=true
spring.shardingsphere.datasource.ds.username=账号
spring.shardingsphere.datasource.ds.password=密码

# 配置db1
spring.shardingsphere.datasource.db1.type=com.alibaba.druid.pool.DruidDataSource
spring.shardingsphere.datasource.db1.driver-class-name=com.mysql.jdbc.Driver
spring.shardingsphere.datasource.ds.url=jdbc:mysql://mysql的ip:端口/库名?useUnicode=true&characterEncoding=utf-8&allowMultiQueries=true
spring.shardingsphere.datasource.ds.username=账号
spring.shardingsphere.datasource.ds.password=密码

# 分库字段
spring.shardingsphere.rules.sharding.default-database-strategy.standard.sharding-column=user_id
#分库策略的类，该类必须实现 StandardShardingAlgorithm 接口，且类对应的Component注解名称为 preciseShardingTableAlgorithm.
spring.shardingsphere.rules.sharding.default-database-strategy.standard.sharding-algorithm-name=preciseShardingDatabaseAlgorithm

# 分表配置
# tb_shard_test表配置
spring.shardingsphere.rules.sharding.tables.tb_shard_test.actual-data-nodes=db$->{0..1}.tb_shard_test_$->{0..7}
# 分表字段
spring.shardingsphere.rules.sharding.tables.tb_shard_test.table-strategy.standard.sharding-column=order_id
#分表策略的类，该类必须实现 StandardShardingAlgorithm 接口.且类对应的Component注解名称为 preciseShardingTableAlgorithm.
spring.shardingsphere.rules.sharding.tables.tb_shard_test.table-strategy.standard.sharding-algorithm-name=preciseShardingTableAlgorithm

# 打印分库分表日志
spring.shardingsphere.props.sql-show=true
```



### 自定义分库策略：

在前面的配置中，有一个分库策略相关的配置如下：

```
#分库策略的类，该类必须实现 StandardShardingAlgorithm 接口，且类对应的Component注解名称为 preciseShardingTableAlgorithm.
spring.shardingsphere.rules.sharding.default-database-strategy.standard.sharding-algorithm-name=preciseShardingDatabaseAlgorithm
```

这个配置，对应类的Component注解名称 preciseShardingDatabaseAlgorithm，代码如下：

```
import com.alibaba.fastjson.JSON;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.lang3.StringUtils;
import org.apache.shardingsphere.sharding.api.sharding.standard.PreciseShardingValue;
import org.apache.shardingsphere.sharding.api.sharding.standard.RangeShardingValue;
import org.apache.shardingsphere.sharding.api.sharding.standard.StandardShardingAlgorithm;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

import java.util.Collection;
import java.util.Iterator;

/**
 *
 * 分库策略
 * StandardShardingAlgorithm<Integer> 后面的泛型，需要跟分库字段保持一样的类型
 *
 **/
@Slf4j
@Component(value = "preciseShardingDatabaseAlgorithm")
public class PreciseShardingDatabaseAlgorithm implements StandardShardingAlgorithm<Long> {

    /**
     *  主库别名
     */
    private static final String DBM = "dbm";

    @Value("${order.dataBase.size:2}")
    private int dataBaseSize;


    /**
     * 分库策略，按用户编号最后一位数字对数据库数量取模
     * PreciseShardingValue<Long>，后面的泛型，需要跟分库字段保持一样的类型
     *
     * @param dbNames 所有库名
     * @param preciseShardingValue 精确分片值，包括（columnName，logicTableName，value）
     * @return 表名
     *
     */
    @Override
    public String doSharding(Collection<String> dbNames, PreciseShardingValue<Long> preciseShardingValue) {
        log.info("Database PreciseShardingAlgorithm dbNames:{} ,preciseShardingValue: {}.", JSON.toJSONString(dbNames),
                JSON.toJSONString(preciseShardingValue));

        // 若走主库，直接返回主库
        if (dbNames.size() == 1) {
            Iterator<String> iterator = dbNames.iterator();
            String dbName = iterator.next();
            if (DBM.equals(dbName)) {
                return DBM;
            }
        }

        int mod = preciseShardingValue.getValue().intValue() % dataBaseSize;
        for (String dbName : dbNames) {
            // 分库的规则
            if (dbName.endsWith(String.valueOf(mod))) {
                return dbName;
            }
        }
        throw new UnsupportedOperationException();
    }

    /**
     * RangeShardingValue<Long>，后面的泛型，需要跟分库字段保持一样的类型
     * @param collection
     * @param rangeShardingValue
     * @return
     */
    @Override
    public Collection<String> doSharding(Collection<String> collection, RangeShardingValue<Long> rangeShardingValue) {
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

运行单元测试，可以看到：

先是根据分库字段，找到对应的库，

然后根据分表字段，找到对应的表。

```
PreciseShardingDatabaseAlgorithm : Database PreciseShardingAlgorithm dbNames:["db0","db1"] ,preciseShardingValue: {"columnName":"user_id","logicTableName":"tb_shard_test","value":12345}.
2023-07-04 23:59:46.663  INFO 1124 --- [           main] c.e.d.c.PreciseShardingTableAlgorithm    : doSharding tableNames:["tb_shard_test_0","tb_shard_test_1","tb_shard_test_2","tb_shard_test_3","tb_shard_test_4","tb_shard_test_5","tb_shard_test_6","tb_shard_test_7"] ,preciseShardingValue: {"columnName":"order_id","logicTableName":"tb_shard_test","value":"123456789"}.
2023-07-04 23:59:46.794  INFO 1124 --- [           main] ShardingSphere-SQL                       : SQLStatement: MySQLInsertStatement(setAssignment=Optional.empty, onDuplicateKeyColumns=Optional.empty)
2023-07-04 23:59:46.795  INFO 1124 --- [           main] ShardingSphere-SQL                       : Actual SQL: db1 ::: INSERT INTO tb_shard_test_3  ( order_id,user_id,create_time )  VALUES  (?, ?, ?) ::: [123456789, 12345, null]
<==    Updates: 1
```



### 参考资料：

https://blog.csdn.net/m0_47503416/article/details/124189469
