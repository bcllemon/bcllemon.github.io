layout: post
title: "Mongodb和Python进行数据统计"
date: "2015-10-18 09:46"
categories: 技术
---
# Group
## Mongodb语法
`db.collection.group({ key, reduce, initial [, keyf] [, cond] [, finalize] })`

keyf:用来对要 group 的字段进行处理，类似 sql 中通过 date 函数按日期分组

详细文档：http://docs.mongodb.org/manual/reference/method/db.collection.group/
##### 例子：
```
db.orders.group(
   {
     key: { ord_dt: 1, 'item.sku': 1 },
     cond: { ord_dt: { $gt: new Date( '01/01/2012' ) } },
     reduce: function( curr, result ) {
                 result.total += curr.item.qty;
                 result.count++;
             },
     initial: { total : 0,count:0 }
   }
)
```
等价于
```
SELECT ord_dt, item_sku, SUM(item_qty) as total,count(0) as count
FROM orders
WHERE ord_dt > '01/01/2012'
GROUP BY ord_dt, item_sku
```
## PyMongo语法
```
from bson.code import Code
reducer = Code("""
            function( curr, result ) {
                        result.total += curr.item.qty;
                        result.count++;
                    }
""")
results = db.orders.group(key={"ord_dt": 1, "item.sku": 1}, condition={"ord_dt": { "$gt": "new Date( '01/01/2012' )" }}, initial={"count": 0,"total:0"}, reduce=reducer)
```
<font color="red">PS:results 的结果是一个数组，这就受到 mongo 单个数据集大小的限制，最大为20000。</font>

# Aggregate
## Mongodb语法
![图片引自 mongodb.org](http://docs.mongodb.org/manual/_images/aggregation-pipeline.png)
上图写明了 mongo 处理聚集的方式，和 linux 中管道的理念一致，每一级的处理结果作为下一级的输入，最终完成数据的计算。
其中$match、$group及更多参见:
[Aggregation Pipeline Operators](http://docs.mongodb.org/manual/reference/operator/aggregation/)
##### 例子：
```
db.orders.aggregate([
    {$match:{ord_dt: { $gt: new Date( '01/01/2012' ) }}},
    {$group:{_id:{ ord_dt: "$ord_dt", sku: "$item.sku" },total:{$sum:"$item.qty"},count:{$sum:1}}},
    {$project:{_id:0,ord_dt:"$_id.ord_dt",sku:"$_id.sku",total:1,count:1}}
    ]
    )
```
## PyMongo语法
```
pipeline = [
    {"$match":{"ord_dt": { "$gt": "new Date( '01/01/2012' )" }}},
    {"$group": {"_id": {"ord_dt": "$ord_dt", "sku": "$item.sku"}, "total":{"$sum":"$item.qty"},"count": {"$sum": 1}}},
    {"$project":{"_id":0,"ord_dt":"$_id.ord_dt","sku":"$_id.sku","total":1,"count":1}}
]
result = db.requestLog.aggregate(pipeline)
```

PS:aggregate返回的是 cursor,需要进行迭代获取数据，并且可用进行分页查询
# 杂
## Flask中ObjectId 处理
```
class ObjectIDConverter(BaseConverter):
    def to_python(self, value):
        try:
            return ObjectId(base64_decode(value))
        except (InvalidId, ValueError, TypeError):
            raise ValidationError()

    def to_url(self, value):
        return base64_encode(value.binary)


app.url_map.converters['objectid'] = ObjectIDConverter
```
