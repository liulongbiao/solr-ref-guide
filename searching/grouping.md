# 结果分组

结果分组会将具有某个相同字段值的文档分为一个组，并且对每个分组返回顶部的文档。
如你在某个电子零售商的电商网站搜索 "DVD" 时，你可能希望返回像 "TV and Video" "Movies" 和 "Computers"
这样三个分类，其中每个分类有三个结果。
这里，查询词项 "DVD" 出现在所有三个分类里，因此 Solr 将它们分组到一起以给用户提高相关度。

结果分组和 [Faceting](./faceting/readme.md) 是分离的。
尽管概念上类似，但 faceting 返回所有相关的结果，且允许用户基于 facet 分类进一步调优搜索结果。
比如，你在某个鞋子零售商的电商网站搜索 "shoes"，你将得到所有该查询词项的结果，
并且还具有 "size" 、"color"、"brand" 等可选的 facet。

然而你可以整合分组和 faceting：被分组的 faceting 支持 `facet.field` 和 `facet.range` 
但当前不支持日期和枢轴 faceting。
facet 计数基于地一个 `group.field` 参数进行计算，其他的 `group.field` 参数会被忽略。

分组 faceting 和非分组 facets 的区别在于 (所有 facets 的和) == (该属性产品总量)，
如下例所示：

Object 1

* name: Phaser 4620a
* ppm: 62
* product_range: 6

Object 2

* name: Phaser 4620i
* ppm: 65
* product_range: 6

Object 3

* name: ML6512
* ppm: 62
* product_range: 7

若你让 Solr 根据 "product_range" 分组这些文档，则总分组数量为 2,但对 ppm 的 facet 
对 62 是 2， 而对 65 是 1。

## 请求参数

结果分组接收以下请求参数。任意数量的这些请求参数可包含在单个请求中：

|参数         |类型      |描述                          |
|------------|---------|------------------------------|
|group       |Boolean  |If true, query results will be grouped.|
|group.field |string   |The name of the field by which to group results. The field must be single-valued, and either be indexed or a field type that has a value source and works in a function query, such as `ExternalFileField` . It must also be a string-based field, such as `StrField` or `TextField` |
|group.func  |query    |Group based on the unique values of a function query. <br/> NOTE: 该选项不能用在 [分布式搜索](#distributed) 中。|
|group.query |query    |Return a single group of documents that match the given query. |
|rows        |integer  |The number of groups to return. The default value is 10. |
|start       |integer  |Specifies an initial offset for the list of groups. |
|group.limit |integer  |Specifies the number of results to return for each group. The default value is 1。 |
|group.offset|integer  |Specifies an initial offset for the document list of each group. |
|sort        |sortspec |Specifies how Solr sorts the groups relative to each other. For example, `sort=popularity desc` will cause the groups to be sorted according to the highest popularity document in each group. The default value is `score desc` .|
|group.sort  |sortspec |Specifies how Solr sorts documents within a single group. The default value is `score desc` . |
|group.format|grouped/simple|If this parameter is set to simple , the grouped documents are presented in a single flat list, and the `start` and `rows` parameters affect the numbers of documents instead of groups.|
|group.main  |Boolean  |If true, the result of the first field grouping command is used as the main result list in the response, using `group.format=simple` .|
|group.ngroups |Boolean|If true, Solr includes the number of groups that have matched the query in the results. The default value is false. <br/>当使用分片索引时，请查看下面 [分布式结果分组警告](#distributed)。 |
|group.truncate|Boolean|If true, facet counts are based on the most relevant document of each group matching the query. The default value is false.|
|group.facet |Boolean  |Determines whether to compute grouped facets for the field facets specified in facet.field parameters. Grouped facets are computed based on the first specified group. As with normal field faceting, fields shouldn't be tokenized (otherwise counts are computed for each token). Grouped faceting supports single and multivalued fields. Default is false.<br/>当使用分片索引时，请查看下面 [分布式结果分组警告](#distributed)。 |
|group.cache.percent |0-100 内的整数|Setting this parameter to a number greater than 0 enables caching for result grouping. Result Grouping executes two searches; this option caches the second search. The default value is 0. Testing has shown that group caching only improves search time with Boolean, wildcard, and fuzzy queries. For simple queries like term or "match all" queries, group caching degrades performance.|

任意数量的分组命令(`group.field`、`group.func`、`group.query`)可在单个请求中指定：


## 示例

以下搜索示例假设使用的是 " bin/solr -e techproducts " 示例。

### 根据字段分组结果

本例中我们将根据 `manu_exact` 字段分组结果，它指定了数据集中该词项的制造商。

```
http://localhost:8983/solr/techproducts/select?wt=json&indent=true&fl=id,name&q=so
lr+memory&group=true&group.field=manu_exact
```

```javascript
{
...
"grouped":{
  "manu_exact":{
    "matches":6,
    "groups":[{
      "groupValue":"Apache Software Foundation",
      "doclist":{"numFound":1,"start":0,"docs":[
      {
        "id":"SOLR1000",
        "name":"Solr, the Enterprise Search Server"}]
      }},
    {
      "groupValue":"Corsair Microsystems Inc.",
      "doclist":{"numFound":2,"start":0,"docs":[
        {
        "id":"VS1GB400C3",
        "name":"CORSAIR ValueSelect 1GB 184-Pin DDR SDRAM Unbuffered DDR 400 (PC 3200) System Memory - Retail"}]
      }},
    {
      "groupValue":"A-DATA Technology Inc.",
      "doclist":{"numFound":1,"start":0,"docs":[
        {
        "id":"VDBDB1A16",
        "name":"A-DATA V-Series 1GB 184-Pin DDR SDRAM Unbuffered DDR 400 (PC 3200) System Memory - OEM"}]
      }},
    {
      "groupValue":"Canon Inc.",
      "doclist":{"numFound":1,"start":0,"docs":[
        {
        "id":"0579B002",
        "name":"Canon PIXMA MP500 All-In-One Photo Printer"}]
        }},
    {
      "groupValue":"ASUS Computer Inc.",
      "doclist":{"numFound":1,"start":0,"docs":[
        {
        "id":"EN7800GTX/2DHTV/256M",
        "name":"ASUS Extreme N7800GTX/2DHTV (256 MB)"}]
      }
    }
  ]
}
}
```

响应中表示对我们的查询总共有 6 条数据。对每个 `group.field` 的唯一值，
SOlr 返回一个具有最高分数文档的 `docList`。
`docList` 中也包含分组中的匹配数量，作为其 `numFound` 值。
这些分组根据每个分组中顶部文档的分数进行排序。

我们可以使用请求参数 `group.main=true` 来运行相同的查询。
这将会把结果格式化为单个平的文档列表。
这种平的格式没有像常规的结果分组查询结果那样多的信息，但对已有的 Solr 客户端更容易解析一些。

```
http://localhost:8983/solr/techproducts/select?wt=json&indent=true&fl=id,name,manu
facturer&q=solr+memory&group=true&group.field=manu_exact&group.main=true
```

```javascript
{
"responseHeader":{
  "status":0,
  "QTime":1,
  "params":{
    "fl":"id,name,manufacturer",
    "indent":"true",
    "q":"solr memory",
    "group.field":"manu_exact",
    "group.main":"true",
    "group":"true",
    "wt":"json"}},
"grouped":{},
"response":{"numFound":6,"start":0,"docs":[
  {
    "id":"SOLR1000",
    "name":"Solr, the Enterprise Search Server"},
  {
    "id":"VS1GB400C3",
    "name":"CORSAIR ValueSelect 1GB 184-Pin DDR SDRAM Unbuffered DDR 400 (PC 3200) System Memory - Retail"},
  {
    "id":"VDBDB1A16",
    "name":"A-DATA V-Series 1GB 184-Pin DDR SDRAM Unbuffered DDR 400 (PC 3200) System Memory - OEM"},
  {
    "id":"0579B002",
    "name":"Canon PIXMA MP500 All-In-One Photo Printer"},
  {
    "id":"EN7800GTX/2DHTV/256M",
    "name":"ASUS Extreme N7800GTX/2DHTV (256 MB)"}]
  }
}
```

### 根据查询分组

下例中我们使用 `group.query` 参数找到匹配 "memory" 的两个不同的价格段， 0.00-99.99 和 超过 100，三个最高的结果。

```
http://localhost:8983/solr/techproducts/select?wt=json&indent=true&fl=name,price&q
=memory&group=true&group.query=price:[0+TO+99.99]&group.query=price:[100+TO+*]&gro
up.limit=3
``` 

```javascript
{
"responseHeader":{
  "status":0,
  "QTime":42,
  "params":{
    "fl":"name,price",
    "indent":"true",
    "q":"memory",
    "group.limit":"3",
    "group.query":["price:[0 TO 99.99]",
    "price:[100 TO *]"],
    "group":"true",
    "wt":"json"}},
"grouped":{
  "price:[0 TO 99.99]":{
    "matches":5,
    "doclist":{"numFound":1,"start":0,"docs":[
      {
        "name":"CORSAIR ValueSelect 1GB 184-Pin DDR SDRAM Unbuffered DDR 400 (PC 3200) System Memory - Retail",
        "price":74.99}]
      }},
  "price:[100 TO *]":{
    "matches":5,
    "doclist":{"numFound":3,"start":0,"docs":[
      {
        "name":"CORSAIR XMS 2GB (2 x 1GB) 184-Pin DDR SDRAM Unbuffered DDR 400(PC 3200) Dual Channel Kit System Memory - Retail",
        "price":185.0},
      {
        "name":"Canon PIXMA MP500 All-In-One Photo Printer",
        "price":179.99},
      {
        "name":"ASUS Extreme N7800GTX/2DHTV (256 MB)",
        "price":479.95}]
      }
    }
  }
}
```

这里，Solr 找到了五个匹配 "memory" 的文档，但仅返回根据价格分组后的四个结果。
这是因为其中某个 "memory" 的结果没有赋予价格。

## <a name="distributed"></a> 分布式结果分组警告

分组支持 [分布式搜索](../solrcloud/readme.md)，但有一些警告：

* 当前 `group.func` 不能在任何分布式搜索中支持
* `group.ngroups` 和 `group.facet` 需要每个分组中的所有文档必须存在于同一个分片中，
以返回准确的数量。
[通过组合键路由文档](../solrcloud/works/shards.md) 在很多情况下可能会是有用的解决方案。
