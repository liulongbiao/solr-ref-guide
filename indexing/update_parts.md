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

乐观并发是 Solr 的一个特性可用在要更新/替换文档的客户端应用来确保他们正在更新/替换的文档
没有被其它客户端应用所修改。
这个特性的运作需要索引中的所有文档都有一个 `_version_` 字段，
并且在更新时需和更新命令中制定的 `_version_` 进行比较。
默认的 `schema.xml` 包含了一个 `_version_` 字段，且该字段会自动添加到每个新文档中。

通常，使用乐观并发涉及以下工作流程：

1. 客户端读取一个文档。Solr 中，客户端可以使用 `/get` 处理器确保检索到文档的最新版本。
2. 客户端在本地变更该文档
3. 客户端重新提交变更后的文档给 Solr，比如说可能是 `/update` 处理器
4. 若存在版本冲突(HTTP 错误号为 409)，客户端需要重新开始这个过程

当客户端重新提交变更后的文档给 Solr 时，`_version_` 字段可包含在更新中以调用乐观并发控制。
这里使用了特定的语义来定义何时文档应被更新或何时应报告冲突。

* 若内容中的 `_version_` 字段比 '1'(如 '12345') 大，则文档中的 `_version_` 必须匹配索引中的 `_version_`
* 若内容中的 `_version_` 字段等于 '1'，则文档必须简单的存在。这时不会进行版本匹配；但若文档不存在，更新将被驳回
* 若内容中的 `_version_` 字段小于 '0'(如 '-1')，则文档必须 **不** 存在。这时不会进行版本匹配；但若文档存在，更新将被驳回
* 若内容中的 `_version_` 字段等于 '0'，则不关心文档版本是否匹配还是存在与否。若存在，它会被覆盖；若不存在，则会被添加

若被更新的文档中不包含 `_version_` 字段，且没有使用原子更新，
该文档将被由常规的 Solr 规则处理，通常是丢弃前面的版本。

当使用乐观并发时，客户端可以包含一个可选的 `version=true` 请求参数以指示
文档被添加后响应中应包含一个 **新** 版本号。
这允许客户端能立即知道每个添加的文档的 `_version_` 是什么，
而不需要发送一个冗余的 [`/get` 请求](../searching/realtime_get.md)。

例如：

```bash
$ curl -X POST -H 'Content-Type: application/json'
'http://localhost:8983/solr/techproducts/update?versions=true' --data-binary '
[ { "id" : "aaa" },
  { "id" : "bbb" } ]'
{"responseHeader":{"status":0,"QTime":6},
 "adds":["aaa",1498562471222312960,
         "bbb",1498562471225458688]}
$ curl -X POST -H 'Content-Type: application/json'
'http://localhost:8983/solr/techproducts/update?_version_=999999&versions=true'
--data-binary '
[{ "id" : "aaa",
   "foo_s" : "update attempt with wrong existing version" }]'
{"responseHeader":{"status":409,"QTime":3},
 "error":{"msg":"version conflict for aaa expected=999999
actual=1498562471222312960",
          "code":409}}
$ curl -X POST -H 'Content-Type: application/json'
'http://localhost:8983/solr/techproducts/update?_version_=1498562471222312960&versio
ns=true&commit=true' --data-binary '
[{ "id" : "aaa",
   "foo_s" : "update attempt with correct existing version" }]'
{"responseHeader":{"status":0,"QTime":5},
 "adds":["aaa",1498562624496861184]}
$ curl 'http://localhost:8983/solr/techproducts/query?q=*:*&fl=id,_version_'
{
  "responseHeader":{
    "status":0,
    "QTime":5,
    "params":{
      "fl":"id,_version_",
      "q":"*:*"}},
  "response":{"numFound":2,"start":0,"docs":[
    {
      "id":"bbb",
      "_version_":1498562471225458688},
    {
      "id":"aaa",
      "_version_":1498562624496861184}]
  }}
```

更多信息可以查看 Apache Lucene EuroCon 2012 的
[Yonik Seeley's presentation on NoSQL features in Solr 4](https://www.youtube.com/watch?v=WYVM6Wz-XTw)

> 提示:
>
> `_version_` 字段默认存储在反向索引中(`indexed="true"`)。
> 然而对某些有大量文档的系统，FieldCache 内存增量可能成本太高。
> 一种解决方案可能是将 `_version_` 字段声明为 [DocValues](../schema/docvalues.md)：

示例字段定义：

```xml
<field name="_version_" type="long" indexed="false" stored="true"
  required="true" docValues="true"/>
```

## 文档为中心的版本约束