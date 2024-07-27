
### 配置表

通过配置表，灵活的配置。

开发中某些经常变更的参数值，加上配置。比如 订单30分钟后失效，需求变更，要改为15分钟，那么直接改配置表就行了，不用发版。

某些关键的容易出错的逻辑，加上一个开关，也就是 config_value 为 0或1，为1表示打开，为0表示关掉。

不需要的逻辑，可以及时用开关关掉。

或者是逻辑复杂，开发环境造数据麻烦时，也可以用配置表配置开关，把前置条件关掉，方便验证数据。



### 建表语句：

config_key 唯一索引，保证配置的 key 唯一。

config_value，如果有多个，可以用逗号隔开。

is_delete 表示是否删除：0-否；1-是。

```sql
CREATE TABLE `tb_system_config` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT '主键',
  `config_key` varchar(128) NOT NULL COMMENT '配置的KEY',
  `config_value` varchar(2000) DEFAULT '' COMMENT '配置的值。如果有多个，用逗号隔开',
  `description` varchar(100) DEFAULT '' COMMENT '描述',
  `create_time` datetime DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` datetime DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  `is_delete` tinyint(1) NOT NULL DEFAULT '0' COMMENT '是否删除：0-否；1-是',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_config_key` (`config_key`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4  COMMENT='系统配置表';
```



### 查询系统配置表：

插入配置数据后，查询：

找出 config_key 为 config_test 的配置值。

```
SELECT config_value FROM tb_system_config_test WHERE config_key='config_test' AND is_delete=0;
```



### 依赖包：

采用 mybatisPlus ，也可以自己用 mybatis 处理。

```
    <properties>
        <mybatis.plus.version>3.4.0</mybatis.plus.version>
    </properties>
	
    <dependencies>
	
        <!--mybatis-plus下面这两个依赖必须加-->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>${mybatis.plus.version}</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
		
        <!--mybatis-plus以下依赖是拓展，比如分页插件-->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-extension</artifactId>
            <version>${mybatis.plus.version}</version>
        </dependency>
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-annotation</artifactId>
            <version>${mybatis.plus.version}</version>
        </dependency>
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-core</artifactId>
            <version>${mybatis.plus.version}</version>
        </dependency>
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-generator</artifactId>
            <version>${mybatis.plus.version}</version>
        </dependency>
		
        <!--单元测试依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        
        
    </dependencies>
```



### 实体类：

```
@Data
@EqualsAndHashCode(callSuper = false)
@TableName("tb_system_config")
public class SystemConfigEntity implements Serializable {

    private static final long serialVersionUID = 1L;

    /**
     * 主键
     */
    @TableId(value = "id", type = IdType.AUTO)
    private Integer id;

    /**
     * 配置的KEY
     */
    private String configKey;

    /**
     * 配置的值
     */
    private String configValue;

    /**
     * 描述
     */
    private String description;

    /**
     * 创建时间
     */
    @TableField(fill = FieldFill.INSERT)
    private Date createTime;

    /**
     * 更新时间
     */
    @TableField(fill = FieldFill.UPDATE)
    private Date updateTime;

    /**
     * 是否删除：0-否；1-是
     */
    @TableLogic
    private Boolean isDelete;


}

```



### Mapper ：

```
public interface SystemConfigMapper extends BaseMapper<SystemConfigEntity> {

}
```



### Mapper.xml：

namespace 和 type 的路径，自行修改。。

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.demo.dao.SystemConfigMapper">

    <!-- 通用查询映射结果 -->
    <resultMap id="BaseResultMap" type="com.example.demo.domain.SystemConfigEntity">
        <id column="id" property="id" />
        <result column="config_key" property="configKey" />
        <result column="config_value" property="configValue" />
        <result column="description" property="description" />
        <result column="create_time" property="createTime" />
        <result column="update_time" property="updateTime" />
        <result column="is_delete" property="isDelete" />
    </resultMap>

    <!-- 通用查询结果列 -->
    <sql id="Base_Column_List">
        id, config_key, config_value, description, create_time, update_time, is_delete
    </sql>

</mapper>

```



### Service 服务类：

如果系统接入了 缓存，也可以先从缓存中获取数据。

插入/更新数据后，记得删掉缓存，保持一致性。

系统配置表的逻辑如下：

```
@Service
public class SystemConfigServiceImpl extends ServiceImpl<SystemConfigMapper, SystemConfigEntity> implements SystemConfigService {

    /**
     * 根据 key 获取配置的 value
     * @param key
     * @return
     */
    public String getValueByKey(String key) {
        LambdaQueryWrapper<SystemConfigEntity> queryWrapper = new LambdaQueryWrapper<>();
        queryWrapper.eq(SystemConfigEntity::getConfigKey, key);
        //未删除的数据
        queryWrapper.eq(SystemConfigEntity::getIsDelete, false);
        SystemConfigEntity systemConfigEntity = getOne(queryWrapper);
        if (systemConfigEntity == null) {
            return "";
        }
        return systemConfigEntity.getConfigValue();
    }

    /**
     * 获取所有的配置。
     * 需要多次查询时使用，不用反复查数据表。
     *
     * @return
     */
    public Map<String, String> getValueMap() {
        LambdaQueryWrapper<SystemConfigEntity> queryWrapper = new LambdaQueryWrapper<>();
        //未删除的数据
        queryWrapper.eq(SystemConfigEntity::getIsDelete, false);
        List<SystemConfigEntity> list = list(queryWrapper);
        Map<String, String> map = new HashMap<>();
        if (CollectionUtils.isEmpty( list)) {
            return map;
        }
        map = list.stream().collect(
                Collectors.toMap(SystemConfigEntity::getConfigKey, SystemConfigEntity::getConfigValue, (key1, key2) -> key2));

        return map;

    }


}

```

