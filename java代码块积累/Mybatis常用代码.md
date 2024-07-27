以下使用的数据库是Mysql。
### Mybatis字段类型映射
在resultMap 中定义数据库字段对应的字段类型。
```
  <resultMap id="BaseResultMap" type="com.model.Order" >
    <constructor >
      <idArg column="id" jdbcType="INTEGER" javaType="java.lang.Integer" />
      <arg column="order_id" jdbcType="VARCHAR" javaType="java.lang.String" />
      <arg column="create_date" jdbcType="DATE" javaType="java.util.Date" />
      <arg column="create_time" jdbcType="TIMESTAMP" javaType="java.util.Date" />
      <arg column="deleted" jdbcType="TINYINT" javaType="java.lang.Byte" />
      <arg column="total_price" jdbcType="DECIMAL" javaType="java.math.BigDecimal" />
    </constructor>
  </resultMap>
```
TINYINT 类型，可以直接用 布尔类型去映射。命名时，避免使用is开头，可以用 type，或者status 结尾。

也可以使用property，如下所示
```
    <resultMap id="BaseResultMap" type="com.model.Bill">
        <result property="serialNo" column="fserial_no" jdbcType="VARCHAR"/>
        <result property="type" column="ftype" jdbcType="INTEGER" />
        <result property="invoiceAmount" column="finvoice_amount" jdbcType="DECIMAL" />
        <result property="createTime" column="fcreate_time" jdbcType="TIMESTAMP"/>
       <result property="invoiceDate" column="finvoice_date" jdbcType="DATE" />
    </resultMap>
```
###Mybatis动态Sql：
Mapper.xml如下：
```
<select id="selectOrderList" resultMap="BaseResultMap"
		parameterType="com.model.Order">
		select
		<include refid="Base_Column_List" />
		from t_order
		where 1=1
		<if test="id != null and id != '' ">and id = #{id,jdbcType=INTEGER} </if>
		<if test="serialId != null and serialId != '' ">and serialId = #{serialId,jdbcType=VARCHAR} </if>
	</select>

```
如果不想写1=1，也可以直接使用where标签。
where标签知道只有在一个以上的if条件有值的情况下才去插入“WHERE”子句。而且，若最后的内容是“AND”或“OR”开头的，where 标签也知道如何将他们去除。
示例如下：
```
<select id="selectOrderList" resultMap="BaseResultMap"
		parameterType="com.model.Order">
		select
		<include refid="Base_Column_List" />
		from t_order
		<where>
		    <if test="id != null and id !='' "> id = #{id,jdbcType=INTEGER} </if>
		    <if test="serialId != null and serialId != '' ">and serialId = #{serialId,jdbcType=VARCHAR} </if>
      </where>
</select>
```
对应的Dao层如下：
此处直接将对象作为方法参数，假设参数为Order对象，传递到xml中的参数就包括了Order对象的属性变量，如上的id、serialId。
```
List<Order>  selectOrderList(Order order);
```
如果仅有一两个变量，也可以直接传递变量，如下：
```
List<Order>  selectOrderList( @Param("id ")Integer id  , @Param("serialId ")String serialId );
```

### Mybatis复用字段或条件：
通过 <sql> 标签将字段或查询条件包装起来，就可以用<include>引用，减少重复。

```
<sql id="Base_Column_List">
    id, user_name , code
</sql>

<sql id="Common_Condition_Sql">
	<where>
		<if test="userName != null">
			and t.user_name= #{userName}
		</if>
	</where>			
</sql>

<select id="selectByPrimaryKey" parameterType="java.lang.Long" resultMap="BaseResultMap">
    select 
    <include refid="Base_Column_List" />
    from tt_base_info t
    <include refid="Common_Condition_Sql" />
</select>
```

### Mybatis的 choose
choose 按顺序判断其内部 when 标签中的 test 条件出否成立。类似于 java的 switch()。

```
<choose>
	<when test="deptLevel == 1 "> and t.user_code = #{ code1 } </when>
	<when test="deptLevel !=0 and feedbackZone!=null "> and t.user_code = #{ code2 } </when>
	<otherwise>and t.user_code = #{ code3 } </otherwise>
</choose>
```


###Mybatis模糊查询：
模糊查询可以使用LIKE关键字和CONCAT()函数。
假设要查询的字段为product，从Dao层传递过来的参数为productName。
示例如下：
```
     WHERE 1=1  
       <if test="productName!=null and productName!='' "> AND product LIKE CONCAT('%',#{ productName , jdbcType=VARCHAR },'%') </if>                 
```

###Mybatis使用IN关键字指定条件范围：
IN关键字，需要通过foreach标签来实现。
其中，collection对应的是Dao层传递过来的参数(一般是集合或数组)，
如果懒得使用 @Param("")指定参数名称，可以直接用 collection="list"。
item是自己命名的，表示范围中的变量名称。
separator是指分隔符。index是指下标。
示例如下：
```
<select id="queryOrderUsedCount" resultType="java.lang.String">
	SELECT	count(*) 
        FROM	t_check_account 
        WHERE 	1=1
        <if test=" clientIdList!=null">    
           AND fclient_id IN  
           <foreach collection="clientIdList" item="clientId"  index="index" open="(" close=")" separator="," >
                #{clientId}
           </foreach>  
        </if>           
</select>
```
对应的Dao层为：
```
String  queryOrderListByCondition(@Param("clientIdList")List<String> clientIdList);
```

###Mybatis查询条件范围判断。
经常需要用Mysql查询在某个时间段的数据。
由于Mybatis中可能会将大于号>和小于号<视为标签，所以需要加上 <![CDATA[     ]]>字符。
示例如下：
```
 WHERE 1=1
   <if test="beginTime!=null and beginTime!='' "> <![CDATA[ AND t1.fcreate_time >= #{ beginTime , jdbcType=VARCHAR }  ]]>  </if>   
   <if test="endTime!=null and endTime!='' ">  <![CDATA[ AND t1.fcreate_time < #{ endTime, jdbcType=VARCHAR }  ]]>  </if>   
        
```
###Mybatis多表查询。
多表查询，分为一对一、一对多、多对多。
简单的Sql语句，一对一可以通过association标签实现，一对多和多对多通过collection标签实现。
详情见： https://www.cnblogs.com/expiator/p/9328338.html
复杂的Sql语句，可以直接设置返回的resultMap为Map，通过Map的键值对解析。
示例如下：
```
   <select id="queryOrderList" resultType="java.util.HashMap">
       SELECT
	      t2.allnum,
              t2.username 
       FROM	t_order t1 
       LEFT JOIN t_order_detail t2 ON t2.orderid = t1.id 
       WHERE 1=1 
          <if test="beginTime!=null and beginTime!='' "> <![CDATA[ AND t1.fcreate_time >= #{ beginTime , jdbcType=VARCHAR }    ]]>  </if>   
          <if test="endTime!=null and endTime!='' ">  <![CDATA[ AND t1.fcreate_time < #{ endTime, jdbcType=VARCHAR }  ]]>   </if>   
          <if test="productName!=null and productName!='' ">  AND t1.productname LIKE CONCAT('%',#{ productName , jdbcType=VARCHAR },'%')</if>                 
   </select>
```
对应的Dao层如下：
```
List<Map<String,Object>> queryOrderList( @Param("productName")String  productName , 
		     @Param("beginDate") String beginDate, @Param("endDate") String endDate );
```
返回类型为List<Map<String,Object>>，遍历List，获取Map中键对应的值即可。
Controller层如下所示：
```
       //....
       //忽略其他无关逻辑     
       List<Map<String,Object>> orderList=orderService.queryOrderList( productName, beginDate , endDate);
	    if( orderList.size()>0 ) {
	    	//取第一行的订货信息
	         Map<String, Object> orderMap=orderList.get(0);
                //Object转换为Integer类型
	         Integer allNum =   (Integer) orderMap.get("allnum")   ;
                //Object转换为String类型
	         String userName = String.valueOf( orderMap.get("username ")  )   ;
         }
	   
```
### Mybatis插入数据
对应sql语句：
```
insert into  表名 (字段1，字段2，字段3) values (字段1的值，字段2的值，字段3的值);
```
xml如下所示：
其中的useGeneratedKeys="true" keyProperty="id"表示主键自动产生。
而clientId 、clientSecret 、clientName 、eCheck 都是属于Developer类的属性。在Dao层传递对象过来后，可以使用该对象的属性。
```
<insert id="insert" parameterType="com.model.Developer" useGeneratedKeys="true" keyProperty="id">
		insert into t_bd_developer
		<trim prefix="(" suffix=")" suffixOverrides=",">
			<if test="clientId != null">client_id,</if>
			<if test="clientSecret != null">client_secret, </if>
			<if test="clientName != null">client_name,</if>
			<if test="eCheck != null">isCheck,</if>
		</trim>
		<trim prefix="values (" suffix=")" suffixOverrides=",">
			<if test="clientId != null">#{clientId,jdbcType=VARCHAR},</if>
			<if test="clientSecret != null">#{clientSecret,jdbcType=VARCHAR}, </if>
			<if test="clientName != null">#{clientName,jdbcType=VARCHAR},</if>
			<if test="eCheck != null">#{eCheck,jdbcType=INTEGER},</if>
		</trim>
</insert>
```
其中的  useGeneratedKeys="true" keyProperty="id" 表示自动产生主键，并将自动生成的主键通过keyProperty这个属性返回。

Dao层则如下所示：
```
int insert(Developer developer);

```
### Mybatis处理日期
当jdbcType="DATE"类型时，返回的时间只有年月日（yyyy-MM-dd）的，当jdbcType=“TIMESTAMP”的时候，返回的时间是年月日和时分秒（yyyy-MM-dd HH:mm:ss）
比如createDate为 2019-04-17，createTime为 2019-04-17 22:25:28，处理如下：
```
<resultMap  id="BaseResultMap"   type="com.model.Bill">
   <result property="createDate"   column="fcreate_date"  javaType="Date"  jdbcType="DATE"/> 
   <result property="createTime"   column="fcreate_time"  javaType="Date"  jdbcType="TIMESTAMP" />
</resultMap>
```
### Mybatis返回Map，并指定key
可以使用 @Mapkey。
xml跟平常的一样：
```
<select id="getUserMapper" resultMap="BaseResultMap" parameterType="string">
	
</select>
```
Mapper类，在方法上面添加一个注解 @MapKey("userName")，括号里的值可以写User对象的一个字段 ，作为 map的 key，方法的返回类型为 Map<String, User>.
```
public interface UserMapper {

    @MapKey("userName")
    Map<String, User> getUserMapper(String ids);
}

```

### 参考资料：
https://www.cnblogs.com/expiator/p/9328338.html
https://blog.csdn.net/u011781521/article/details/79669180
https://www.cnblogs.com/cyttina/p/3894428.html
