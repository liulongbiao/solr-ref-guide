# 转换结果文档

文档转换器可用于修改搜索结果中返回的每个文档的信息。

## 使用文档转换器

当执行请求时，可以通过使用方括号在 `fl` 参数中包含它来使用文档转换器。如：

```
fl=id,name,score,[shard]
```

某些转换器允许(或必须要)在方括号中以键值对的形式指定本地变量：

```
fl=id,name,score,[explain style=nl]
```

对常规的字段，当给某个字段添加转换器时，你可以通过前缀改变所使用的键：

```
fl=id,name,score,my_val_a:[value v=42 t=int],my_val_b:[value v=7 t=float]
```

以下小节讨论了多种转换器实际能做的事情。

## 可用的转换器

### `[value]` - ValueAugmenterFactory

修改每个文档以包含相同的值，就好像它在每个文档中是存储字段一样：

```
q=*:*&fl=id,greeting:[value v='hello']
```

上述查询将生成以下结果：

```xml
<result name="response" numFound="32" start="0">
  <doc>
    <str name="id">1</str>
    <str name="greeting">hello</str></doc>
  </doc>
...
```

默认，值以字符串返回，但一个 `t` 参数可用于指定使用的是 `int、float、double 或 date` 类型强制作为返回类型：

```
q=*:*&fl=id,my_number:[value v=42 t=int],my_string:[value v=42]
```

除了这些请求参数外，你可以配置额外的 ValueAugmenterFactory 具名实例，
或者在 solrconfig.xml 文件中覆盖已有 `[value]` 转换器的默认行为：

```xml
<transformer name="mytrans2"
  class="org.apache.solr.response.transform.ValueAugmenterFactory" >
  <int name="value">5</int>
</transformer>
<transformer name="value"
  class="org.apache.solr.response.transform.ValueAugmenterFactory" >
  <double name="defaultValue">5</double>
</transformer>
```

其 "value" 选项强制总是使用一个显式的值，而 "defaultValue" 选项提供了
一个可使用 "v" 和 "t" 本地参数来覆盖的默认值。

### `[explain]` - ExplainAugmenterFactory

将像调试小节所描述的关于分数的可用信息作为每个文档的行内解释。

```
q=features:cache&wt=json&fl=id,[explain style=nl]
```

“style” 支持的值有 "text"、"html" 以及 "nl"，后者以结构化数据返回信息：

```json
"response":{"numFound":2,"start":0,"docs":[
{
  "id":"6H500F0",
  "[explain]":{
    "match":true,
    "value":1.052226,
    "description":"weight(features:cache in 2) [DefaultSimilarity], result of:",
    "details":[{
...
``` 

默认的 style 可以在配置文件中指定 "args" 参数来配置：

```xml
<transformer name="explain"
  class="org.apache.solr.response.transform.ExplainAugmenterFactory" >
  <str name="args">nl</str>
</transformer>
```

### `[child]` - ChildDocTransformerFactory

该转换器返回每个匹配查询的父文档的所有 
[后代文档](https://cwiki.apache.org/confluence/display/solr/Uploading+Data+with+Index+Handlers#UploadingDatawithIndexHandlers-NestedChildDocuments)
并将这些后代文档作为平的列表内嵌到匹配的父文档中。
这在你具有索引的内嵌文档且针对任何类型的搜索查询希望对相关的父文档检索子文档。

```
fl=id,[child parentFilter=doc_type:book childFilter=doc_type:chapter limit=100]
```

注意这个转换器即使在查询本身不是一个 
[BlockJoinQuery](https://cwiki.apache.org/confluence/display/solr/Other+Parsers#OtherParsers-BlockJoinQueryParsers)
时依旧可用。

当使用这个转换器时，其 `parentFilter` 参数必须被指定，且和所有 BlockJoinQuery 中一样运行。
额外的参数有：

* `childFilter` - 过滤哪些子文档应该被包含的查询，这在你具有多个层级的层级文档时非常有用(默认：所有后代)
* `limit` - 每个父文档应返回的最大的子文档数量(默认： 10)

###  `[shard]` - ShardAugmenterFactory

这个转换器添加关于每个独立文档都是来自分布式请求中的哪个分片的信息

ShardAugmenterFactory 不支持任何请求参数或配置选项。

### `[docid]` - DocIdAugmenterFactory

这个转换器添加内部 Lucene 文档 ID 到每个文档上 - 这常仅对调试有用。

DocIdAugmenterFactory 不支持任何请求参数或配置选项。

### `[elevated]` 和 `[excluded]`

这些参数仅在使用 [查询提升组件](./query_elevation_component.md) 时有效。

* `[elevated]` 标注每个文档指示它是否被提升
* `[excluded]` 标注每个文档指示它是否被排除 - 它仅在你同时使用了 `markExcludes` 参数时被支持。

```
fl=id,[elevated],[excluded]&excludeIds=GB18030TEST&elevateIds=6H500F0&markExcludes=true
```

```javascript
"response":{"numFound":32,"start":0,"docs":[
{
  "id":"6H500F0",
  "[elevated]":true,
  "[excluded]":false},
{
  "id":"GB18030TEST",
  "[elevated]":false,
  "[excluded]":true},
{
  "id":"SP2514N",
  "[elevated]":false,
  "[excluded]":false},
...
```
