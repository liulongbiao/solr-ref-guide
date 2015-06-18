# 更新部分文档

一旦你根据需要将你的内容索引到了 Solr 索引中，你将需要思考如何处理对这些文档的更新策略。
Solr 支持两种仅改变文档某一部分的更新文档的方式。

第一种为 **原子更新**。这种方式允许只改变某个文档的一或多个字段而不需要重新索引整个文档。

第二种方式称为 **乐观并发** 或 **乐观锁**。
它是很多 NoSQL 数据库的特性，且允许基于其版本来条件性地更新文档。
这种方式包含了如何处理版本匹配或不匹配的语义和规则。

原子更新和乐观并发可以用作管理文档变更的独立的策略，
也可以联合使用：你可以使用乐观并发来条件性地应用一个原子更新。

## 原子更新

Solr 支持多种操作符来原子更新一个文档中的值。
它运行仅更新某些字段，这在索引添加速度对应用非常重要的环境中能帮助加快索引进程。

要使用原子更新，可以给需要被更新的字段添加一个修饰符。
其内容可以被更新、添加、或数值增量。

|修饰符      |用法                      |
|-----------|-------------------------|
|set        |Set or replace the field value(s) with the specified value(s), or remove the values if 'null' or empty list is specified as the new value.<br/>May be specified as a single value, or as a list for multivalued fields|
|add        |Adds the specified values to a multivalued field. May be specified as a single value, or as a list.|
|remove     |Removes (all occurrences of) the specified values from a multivalued field. May be specified as a single value, or as a list.|
|removeregex|Removes all occurrences of the specified regex from a multiValued field. May be specified as a single value, or as a list.|
|inc        |Increments a numeric value by a specific amount. Must be specified as a single numeric value.|

> 为字段修饰符能正确运行，所有原始的源字段都必须是 `stored` 的，这也是 Solr 的默认值。

例如，若集合中存在以下文档：

```javascript
{
  "id":"mydoc",
  "price":10,
  "popularity":42,
  "categories":["kids"],
  "promo_ids":["a123x"],
  "tags":["free_to_try","buy_now","clearance","on_sale"]
}
```

我们应用一下更新命令:

```javascript
{
  "id":"mydoc",
  "price":{"set":99},
  "popularity":{"inc":20},
  "categories":{"add":["toys","games"]},
  "promo_ids":{"remove":"a123x"},
  "tags":{"remove":["free_to_try","on_sale"]}
}
```

其结果文档将是：

```javascript
{
  "id":"mydoc",
  "price":99,
  "popularity":62,
  "categories":["kids","toys","games"],
  "tags":["buy_now","clearance"]
}
```

## 乐观并发

## 文档为中心的版本约束