# 通用查询参数

下表汇总了 Solr 通用的查询参数，它们由 [standard](./standard.md)、
[DisMax](./dismax.md) 和 [eDisMax](./extended_dismax.md) 请求处理器所支持。

|参数      |描述                       |
|---------|--------------------------|
|[defType](#defType) |Selects the query parser to be used to process the query.|
|[sort](#sort) |Sorts the response to a query in either ascending or descending order based on the response's score or another specified characteristic.|
|[start](#start) |Specifies an offset (by default, 0) into the responses at which Solr should begin displaying content.|
|[rows](#rows) |Controls how many rows of responses are displayed at a time (default value: 10)|
|[fq](#fq) |Applies a filter query to the search results.|
|[fl](#fl) |Limits the information included in a query response to a specified list of fields. The fields need to have been indexed as stored for this parameter to work correctly.|
|[debug](#debug) |Request additional debugging information in the response. Specifying the `debug=timing` parameter returns just the timing information; specifying the `debug=results` parameter returns "explain" information for each of the documents returned; specifying the `debug=query` parameter returns all of the debug information.|
|[explainOther](#explainOther) |Allows clients to specify a Lucene query to identify a set of documents. If non-blank, the explain info of each document which matches this query, relative to the main query (specified by the q parameter) will be returned along with the rest of the debugging information.|
|[timeAllowed](#timeAllowed) |Defines the time allowed for the query to be processed. If the time elapses before the query response is complete, partial information may be returned.|
|[omitHeader](#omitHeader) |Excludes the header from the returned results, if set to true. The header contains information about the request, such as the time the request took to complete. The default is false.|
|[wt](#wt) |Specifies the Response Writer to be used to format the query response.|
|[logParamsList](#logParamsList) |By default, Solr logs all parameters. Set this parameter to restrict which parameters are logged. Valid entries are the parameters to be logged, separated by commas (i.e., `logParamsList=param1,param2` ). An empty list will log no parameters, so if logging all parameters is desired, do not define this additional parameter at all.|

以下是这些参数的详细信息

### <a name="defType"></a>defType 参数

defType 参数用来选择 Solr 应用于处理请求中的主查询参数 (`q`) 的查询解析器。如：

```
defType=dismax
``` 

若没有指定 defType，则默认使用 [标准查询解析器](./standard.md)。(即 `defType=lucene`)

### <a name="sort"></a>sort 参数

`sort` 参数会以升序(`asc`)或降序(`desc`) 来排列搜索结果。
该参数用于数值或字母序内容上。其排序方向可以全小写或全大写来输入(即 `asc` 或 `ASC`)。

Solr 可以根据文档的分数或某个字段的单个的
indexed 或使用了 [DocValues](../../schema/docvalues.md) 的字段的值
(即任何 `schema.xml` 中属性包括 `multiValued="false"` 并且 `docValues="true"` 或 `indexed="true"` 之一为真
 - 若字段没有启用 DocValues，将使用被索引的项在运行时来构建它们)
来排序。
给定：

* 字段是非分词的(即字段没有分析器且其内容已被解析为 tokens ，这样会使得排序不一致)，或者
* 字段使用了仅产生单个项的分析器(如 KeywordTokenizer)

如果你想能够在字段上排序又希望将其分词，请在 schema.xml 中使用 `<copyField>` 指令来克隆字段。
然后在该字段上搜索而在其克隆字段上排序。

下表解释了 Solr 如何在多种设置下如何响应 `sort` 参数

|示例        |结果                        |
|-----------|---------------------------|
|           |If the sort parameter is omitted, sorting is performed as though the parameter were set to score desc .|
|score desc |Sorts in descending order from the highest score to the lowest score.|
|price asc  |Sorts in ascending order of the price field|
|inStock desc, price asc|Sorts by the contents of the inStock field in descending order, then within those results sorts in ascending order by the contents of the price field.|

根据 sort 参数的参数：

* 一个 sort 排序必须包含一个字段名称(或 `score` 作为伪字段)，
后跟一个空格(URL 字符串中转码为 `+` 或 `%20`)，
再后面跟一个排序方向(`asc` 或 `desc`)。
* 多个 sort 排序间可用逗号分割，使用语法 `sort=<field name>+<direction>,<field name>+<direction>],...`
    * 当提供了超过一个排序条件时，第二个项仅会在第一个项相同时起作用。第三项会在前两项都相同时起作用。等等。

### <a name="start"></a>start 参数

当指定时， `start` 参数指定了查询结果集的偏移量并指示 Solr 从该偏移量开始显示结果。

默认值为 '0'。或者说默认 Solr 返回的结果没有偏移量，就从结果集本身的起始处开始显示。

设置 `start` 参数为其它值，如 3,会导致 Solr 跳过前面的记录并从该偏移量标识的文档处开始。

你可以将 `start` 参数用作分页。例如， 当 `row` 参数设置为 10 时，你可以显示三个连续的页面结果
通过将 `start` 设置为 0 发送一个查询, 然后设置为 10 再发送相同的查询，在设置为 20 再发送查询。

### <a name="rows"></a>rows 参数

你可以使用 `row` 参数对查询结果分页。
该参数指定了 Solr 应该在一次请求中返回给客户端的完整数据集中最大的文档数量。

其默认值为 10。即默认 Solr 每次查询的响应中返回 10 个文档。

### <a name="fq"></a>fq(Filter Query) 参数

`fq` 参数定义了可用于限制可被返回的文档的超集的查询，而不会影响分数。
它对于加速复杂的查询时很有用，因为以 `fq` 指定的查询会在主查询缓存之外被独立地进行缓存。
当后续的查询使用相同的过滤器时，缓存命中，这样过滤的结果可以从缓存中很快返回。

当使用 `fq` 参数时，记住以下几点：

* `fq` 参数可以在一个查询中多次指定。只有每个过滤参数实例的文档集的交集中的文档才会被返回。
下例中，仅有流行度大于10 且 section 为 0 的才会匹配。

```
fq=popularity:[10 TO *]&fq=section:0
```

* 过滤器查询可以涉及复杂的布尔查询。上例也可以写做单个的具有两个强制子句的查询，如：

```
fq=+popularity:[10 TO *] +section:0
```

* 每个过滤器查询的文档集都被独立地缓存。
因此考虑上面的例子：如果两个条件常常一起出现时使用单个具有两个强制子句的 `fq` 查询，
而如果两个条件相对独立时使用两个独立的 `fq` 参数。
(学习调整缓存大小并确保过滤器缓存确实存在，可查看 [配置Solr实例](../../config/readme.md) )
* 对所有的参数，URL 中的特殊字符需要进行合适的转码并编码为Hex 值。
有很多在线工具可帮助你完成 URL 编码。如 http://meyerweb.com/eric/tools/dencoder/ 。

### <a name="fl"></a>fl(Field List) 参数

`fl` 参数限制查询响应中只包含指定字段列表中的信息。
该参数的这些字段必须被所因为 stored 以使它能正确工作。

字段列表可以以空格分隔或逗号分隔的字段名称的列表。
字符串 `score` 可用于表示每个文档在特定查询中应该作为一个字段返回。
通配符 `*` 选择了文档中所有 stored 的字段。
你也可以给字段列表请求添加伪字段、函数和转换器。

下表展示了一些使用 `fl` 的基本示例：

|字段列表        |结果                    |
|--------------|------------------------|
|id name price |Return only the id, name, and price fields. |
|id,name,price |Return only the id, name, and price fields. |
|id name,price |Return only the id, name, and price fields. |
|id score      |Return the id field and the score.          |
|*             |Return all the fields in each document. This is the default value of the fl parameter. |
|* score       |Return all the fields in each document, along with each field's score. |

#### 函数值

[函数](./function.md) 可针对结果中的每个文档进行计算，并作为伪字段返回：

```
fl=id,title,product(price,popularity)
```

#### 文档转换器

[文档转换器](searching/transform.md) 可用于修改查询结果中每个文档返回的信息：

```
fl=id,title,[explain]
```

#### 字段别名

你可以通过前缀一个 `<displayName>:` 来修改用在相应中某字段、函数或转换器的键的名称。如：

```
fl=id,sales_price:price,secret_sauce:prod(price,popularity),why_score:[explainstyle=nl]
```

响应：

```javascript
"response":{"numFound":2,"start":0,"docs":[
  {
    "id":"6H500F0",
    "secret_sauce":2100.0,
    "sales_price":350.0,
    "why_score":{
      "match":true,
      "value":1.052226,
      "description":"weight(features:cache in 2) [DefaultSimilarity], result of:",
      "details":[{
...
```

### <a name="debug"></a>debug 参数

`debug` 参数可被指定多次且支持以下参数：

* `debug=query`： 仅返回关于查询的调试信息
* `debug=timing`： 返回关于查询耗时的调试信息
* `debug=reaults`： 返回分数结果(也称 "explain") 的调试信息
* `debug=all`：返回关于请求的所有可用的调试信息。(或者使用: `debug=true`)

为后向兼容， `debugQuery=true` 可作为 `debug=all` 的一种替代方式来指定

默认行为是不包含调试信息。

### <a name="explainOther"></a>explainOther 参数

`explainOther` 参数指定了一个 Lucene 查询来标识一个文档集。
若该参数被包含并设置为非空白值，该查询将返回调试信息，以及每个文档匹配该 Lucene 查询的 “解释信息”，
相对于主查询(它由 `q` 参数指定)。如：

```
q=supervillians&debugQuery=on&explainOther=id:juggernaut
```

上面的查询让你可以检查顶部的匹配文档的分数解释，
将其和对匹配 `id:juggernaut` 的文档做比较，并决定为何该排序不是你想要的。

该参数的默认值为空，这样不会返回任何额外的 “解释信息”。

### <a name="timeAllowed"></a>timeAllowed 参数

该参数指定了对搜索完成所允许的毫秒值的时间量。
若该时间在搜索完成前到期，则任何已有的部分的结果将被返回。

### <a name="omitHeader"></a>omitHeader 参数

该参数应被设为 true 或 false。

若为 true，该参数从返回结果中排除头部。
头部包含了关于请求的信息，如花费了多长时间完成。
该参数的默认值为 false。

### <a name="wt"></a>wt 参数

`wt` 参数选择了 Solr 应该用作查询响应的 Response Writer。
更多信息，查看 [ResponseWriters](../response_writers.md)

### `cache=false` 参数

Solr 默认会缓存所有查询以及过滤器查询的结果。
要禁用缓存，可设置 `cache=false` 参数。

你也可以使用 `cost` 选项来控制非缓存的过滤器查询被求值的顺序。
这让你可以将便宜的非缓存过滤器放在昂贵的非缓存过滤器之前。

对非常高成本的过滤器，若 `cache=false` 和 `cost>=100` 且该查询实现了 `PostFilter` 接口，
一个 Collector 将会被查询所请求并用于在它们匹配主查询及其它过滤器查询之后进行文档的过滤。
存在多种后处理过滤器；且它们也可以根据成本排序。

例如：

```
// normal function range query used as a filter, all matching documents
// generated up front and cached
fq={!frange l=10 u=100}mul(popularity,price)

// function range query run in parallel with the main query like a traditional
// lucene filter
fq={!frange l=10 u=100 cache=false}mul(popularity,price)

// function range query checked after each document that already matches the query
// and all other filters. Good for really expensive function queries.
fq={!frange l=10 u=100 cache=false cost=100}mul(popularity,price)
```

### <a name="logParamsList"></a>logParamsList 参数

默认 Solr 给所有请求的参数记录日志。从 4.7 开始，设置这个属性可用于限制请求的哪些参数被记入日志。
这可以帮助控制仅被认为对你的组织有意义的参数。

如，你可以定义：

```
logParamsList=q,fq
```

这样就只记录 `q` 和 `fq` 参数。

若不想记录任何字段，你可以设置 `logParamsList` 为空 (即， `logParamsList=`)。

> 该参数不仅适用于查询请求，也适用于任何对 Solr 的请求。
