### mongodb 可视化工具：

Robo3T。复制数据很方便。

DataGrip，对mongodb的日期格式不友好。

### MongoDB常用的语句：
注意，**凡是涉及到日期类型的，最好都用这种格式 ISODate("2023-03-27T16:00:00.000Z")**
如果 是 ObjectId 类型的，可以用  ObjectId("124f2598e4b0654e360e8430")。

### MongoDb建表：

MongoDB不需要建表，直接插入数据就会建表。

```
db.getCollection("mongoDbTest").insert({userId:"dxcefg", status:1,  price:1.23, updateTime : ISODate("2022-02-13T07:06:25.371Z")})
```

### MongoDB条件查询
```
db.getCollection("mongoDbTest").find({userId:"abc", status: 1})
```
如果 是 ObjectId 类型的， 可以查询如下：
```
db.getCollection('mongoDbTest').find({'_id': ObjectId("124f2598e4b0654e360e8430")})
```

### MongoDB模糊查询
```
//以abc开头
db.getCollection("mongoDbTest").find({userId: /^abc/})
//以abc结尾
db.getCollection("mongoDbTest").find({userId: /efg$/})

```

### MongoDB倒序，查询：
```
db.getCollection('abcde').find().sort({ 'createTime': -1 })
```

### MongoDB查询，限制个数：
```
db.getCollection('abcdef').find().sort({ 'createTime': -1 }).limit(10)
```

### MongoDB插入数据
MongoDB不需要建表，直接插入数据就会建表。
日期用 ISODate() 转换。
```
db.getCollection("mongoDbTest").insert({userId:"dxcefg", status:1,  price:1.23, updateTime : ISODate("2022-02-13T07:06:25.371Z")})
```

### MongoDB查询全部
MongoDB的 Collection(集合)，类似于 数据库的表。
```
db.getCollection("mongoDbTest").find()
```




### MongoDB查询数量
```
db.getCollection("mongoDbTest").find({userId: /^abc/}).count()
```

### mongoDB查询日期：
lte 小于等于，gte 大于等于
```
db.getCollection('mongoDbTest').find({
   "startTime" : { "$lte" :  ISODate("2023-03-27T16:00:00.000Z")  },
  "endTime" : { "$gte" : ISODate("2023-03-27T16:00:00.000Z")  }
})
```





### MongoDB修改字段值
update第一个参数是 条件，而 set部分就是修改内容
以下类似于： update mongoDbTest set status= 4 where status= 1;

```
db.getCollection("mongoDbTest").update(
{"status": 1},
{$set:    { "status" : 4 } })
```

* 只更新符合条件的一条记录
  db.getCollection("mongoDbTest").update({userId:"abc"}, {$set:{ status: 100}});

* 更新符合条件的所有记录， 使用  updateMany
  db.getCollection("mongoDbTest").updateMany({userId:"abc"}, {$set:{ status: 333}});
### MongoDB删除：
```
db.getCollection("mongoDbTest").deleteMany({ id:1 })
```

### MongoDB删除字段
比如字段名为 fieldTest，如下删除该字段：
```
db.getCollection("mongoDbTest").update({
    "fieldTest": {
        "$exists": true
    }
}, {
    "$unset": {
        "fieldTest":null
    }
})
```
### MongoDB新增字段
比如新增一个字段叫 fieldTest，如下 ：
```
db.getCollection("mongoDbTest").update({},{$set:{ fieldTest:""}})
```


### MongoDB修改字段名
```
//参数提示：
//第一个false：可选，这个参数的意思是，如果不存在update的记录，true为插入新的记录，默认是false，不插入。
//第二个true：可选，mongodb默认是false，只更新找到的第一条记录，如果这个参数为true，就把按条件查出来多条记录全部更新。
db.getCollection("mongoDbTest").update({}, {$rename : {"orderId" : "status"}}, false, true)

```


### MongoDb 判断数组非空

```
db.getCollection('MongoDbTest').find({ "$or": [
    {
        "数组字段名称": {
            "$eq": []
        }
    },
    {
        "数组字段名称": {
            "$eq": null
        }
    }
 ]})
```


###  MongoDB添加索引/查询索引：

```
//查索引
db.getCollection('表名').getIndexes();

//加索引
db.getCollection('表名').createIndex({"personList":1})

//给Json的某个字段加索引
db.getCollection('表名').createIndex({"personList.personId":1})

```



### MongoDB查看执行计划

```
//查看执行计划
db.getCollection('表名').find({'id':'abcd'}).explain()
//explain()得到的执行计划字段：
//stage	查询方式，常见的有COLLSCAN/全表扫描、IXSCAN/索引扫描、FETCH/根据索引去检索文档、SHARD_MERGE/合并分片结果、IDHACK/针对_id进行查询
//mongoDB的执行计划字段，详情见： https://www.uoften.com/article/221616.html

```



### 打印mongoDB语句的日志关键词：
使用 mongoTemplate 查询。

搜索关键词： find using query

SpringBoot配置mongodb打印日志。详情见：https://www.cnblogs.com/expiator/p/17375443.html


### 参考资料：

https://www.uoften.com/article/221616.html

