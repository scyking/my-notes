# MongoDB 中关联查询

标签：mongodb 

---

## 与MySQL对比

|mysql|mongodb|说明|
|---|---|---|
|database|database|数据库|
|tables|collections|表|
|column|field|列|
|row|document|数据|

## 数据类型

- ObjectId：document自生成_id
- String：字符串，必须是utf-8
- Boolean：布尔值，true或者false（首字母小写）
- Integer：整数
- Arrays：数组或者列表
- Object：字典
- Null：空数据类型（None Null）
- Timestamp：时间戳
- Date：日期（优先使用时间戳，更强大）
- Double：浮点数（没有float）

## 关联查询

### 订单orders集合
|_id|good_id|price|num|amount|
|---
|1|1|10|1|10|
|2|2|5|2|10|

### 商品goods集合
|_id|name|
|---
|1|钢笔|
|2|铅笔|

### 关联查询

```
db.orders.aggregate([   
    {
        $lookup: 
        {
            from: "goods",   //从表名
            localField: "good_id",  //主表关联字段
            foreignField: "_id",   //从表关联字段
            as: "goods"   //查询结果名
        }
    }
])
```

### 查询结果

|_id|good_id|price|num|amount|goods._id|goods.name|
|---
|1|1|10|1|10|1|铅笔|
|2|2|5|2|10|2|钢笔|

### java中实现

```
        LookupOperation lookupOperation = LookupOperation.newLookup().
                from("goods").
                localField("good_id").
                foreignField("_id").
                as("goods");

        Criteria criteria = Criteria.where("goods").not().size(0);
        List<Map> results =  mongoTemplate.aggregate(
                Aggregation.newAggregation(lookupOperation,Aggregation.match(criteria)),
                "orders",Orders.class).getMappedResults();
```

### 注意事项

关联查询字段类型需要一致，比如ObjectId类型不能与String类型直接关联。






