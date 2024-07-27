### 使用场景
便用mybatis，可以用 pagehelper 分页 。


### maven依赖
```
<dependency>
	<groupId>com.github.pagehelper</groupId>
	<artifactId>pagehelper</artifactId>
	<version>4.1.6</version>
</dependency>
```
### PageHelper配置
在resources文件夹下面，新增mybatis-config.xml，配置如下：

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <!-- 配置分页插件 -->
  <plugins>
    <plugin interceptor="com.github.pagehelper.PageHelper">
      <!-- 设置数据库类型 可以选择Oracle,Mysql,MariaDB,SQLite,Hsqldb,PostgreSQL等数据库-->
      <property name="dialect" value="mysql"/>
      <property name="pageSizeZero" value="true"/>
    </plugin>
  </plugins>

</configuration>
```

### 主要类及方法：
* PageHelper类:
```
Page<E> startPage(int pageNum, int pageSize)： 开始分页

```

* Page类：
```
getResult(): 获取分页后的结果

getTotal()：获取总数
```

* PageInfo类:
```
PageInfo(List<T> list)： 构造PageInfo对象

getTotal()：获取总数

getList(): 获取结果集
```


### PageHelper 原理：

(1) 执行 PageHelper.startPage()，会初始化一个 ThreadLocal<Page> 属性变量 LOCAL_PAGE ，这个 ThreadLocal 属性变量会在后续设置线程副本变量 Page.

(2) PageHelper类实现了Interceptor接口。本质上是一个拦截器。
这个拦截器会在我们的sql查询语句之前，执行 SELECT count(0) 语句进行计数，
还会在startPage()之后的第一个select查询语句中加入 limit 进行分页。

```
select count(0) from tt_user t where t.city = 'sz';

select  * from tt_user t where t.city = 'sz' order by t.create_time desc limit 0,10 ;
```

(3)获取 ThreadLocal 中设置的 Page 信息，获得分页的总数和结果。

### PageHelper 使用：
(1)设置页数和每页数量，开始分页
PageHelper.startPage(pageNum, pageSize);

(2)查询并分页
将含有查询sql的查询方法放在startPage()后面执行即可。
注意：**只有紧跟在startPage后面的第一个select语句会被分页。
PageHelper分页失效的场景： 如果你想要分页的select语句前面还有其他的select语句，只有第一个select语句会分页。**

(3)获取分页的结果和总数
通过 Page或者PageInfo获取。
注意：**完成分页后，就不要再对集合做过滤了，否则返回的数据不足每页的数量。**

### PageHelper 示例一：
```
//设置页数和每页数量，开始分页
PageHelper.startPage(pageNum, pageSize);
//查询并分页
//userService.selectBy(userInfoDTO) 是查询方法，替换成自己的查询方法即可。
Page<UserInfo> page = (Page<UserInfo>) userService.selectBy(userInfoDTO);
//总数
long total = page.getTotal();
//分页后得到的结果
List<UserInfo> result = page.getResult();

```

### PageHelper 示例二：
如果分页之后，还需要进行数据转换，如下：

```
//设置页数和每页数量，开始分页
PageHelper.startPage(pageNum, pageSize);
//查询并分页
//userService.selectBy(userInfoDTO) 是查询的方法，替换成自己的查询方法即可。
List<UserInfo> resultList = userService.selectBy(userInfoDTO);
//可以在此进行数据转换
//...

PageInfo<UserInfo> pageInfo = new PageInfo<>(resultList);
//通过pageInfo得到总数，而不是每页的数量
long total = pageInfo.getTotal();
//获取结果集
List<UserInfo> list = pageInfo.getList();


```
