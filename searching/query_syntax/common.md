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



### <a name="rows"></a>rows 参数
### <a name="fq"></a>fq 参数
### <a name="fl"></a>fl 参数
### <a name="debug"></a>debug 参数
### <a name="explainOther"></a>explainOther 参数
### <a name="timeAllowed"></a>timeAllowed 参数
### <a name="omitHeader"></a>omitHeader 参数
### <a name="wt"></a>wt 参数
### <a name="logParamsList"></a>logParamsList 参数
