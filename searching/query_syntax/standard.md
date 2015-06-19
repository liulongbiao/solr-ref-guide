# 标准查询解析器

Solr 默认的查询解析器为 "lucene" 解析器。

这个标准查询解析器的优点是它支持一个健壮的非常直觉的语法来允许你创建多种结构化的查询。
其最大的缺点是相较 [DisMax](./dismax.md) 这样设计用于尽可能少抛出错误的查询解析器而言，
它对语法错误缺少容忍力。

本节包含的内容有：

* [标准查询解析器的参数](#parameters)
* [标准查询解析器的响应](#response)
* [给标准查询解析器指定词项](#specify-terms)
* [给标准查询解析器的查询指定字段](#specify-fields)
* [标准查询解析器支持的布尔操作符](#boolean-operators)
* [分组词项以形成子查询](#grouping)
* [Lucene 查询解析其和 Solr 标准查询解析器的不同点](#difference)
* [相关内容](#related)

## <a name="parameters"></a>标准查询解析器的参数

除了 [通用查询参数](./common.md)、[Faceting 参数](../faceting.md)、
[高亮参数](../highlight/readme.md) 和 [MoreLikeThis 参数](../morelikethis.md) 外，
标准查询解析器还支持以下参数：

|参数  |描述                             |
|-----|---------------------------------|
|q    |使用标准查询语法定义一个查询。该参数是必需的。|
|q.op |指定查询表达式的默认操作符，覆盖 `schema.xml` 中定义的默认操作符。可选值为 `AND` 或 `OR`。 |
|df   |指定默认字段，覆盖 `schema.xml` 中定义的默认字段。|

这些参数的默认值在 `solrconfig.xml` 中指定，或者在请求时被查询时的值所覆盖。

## <a name="response"></a>标准查询解析器的响应

默认，标准查询解析器的响应只包含一个 `<result>` 块，它是非命名的。
若使用了 [debug](https://cwiki.apache.org/confluence/display/solr/Common+Query+Parameters#CommonQueryParameters-ThedebugParameter) 参数，
则额外的 `<lst>` 块会被返回，使用了名称 `debug`。
它会包含由于的调试信息，包括了原始查询字符串、解析后的查询字符串
以及对 `<result>` 块中每个文档的解释信息。
若还存在 [explainOther](https://cwiki.apache.org/confluence/display/solr/Common+Query+Parameters#CommonQueryParameters-TheexplainOtherParameter)
参数，则将会给匹配该查询的所有文档提供额外的解释信息。

#### 示例响应

本节展示了标准查询解析器的响应示例。

以下 URL 提交了一个简单的查询并请求 XML Response Writer 使用缩进来使得 XML 响应更加易读。

```
http://localhost:8983/solr/techproducts/select?q=id:SP2514N
```

结果

```xml
<?xml version="1.0" encoding="UTF-8"?>
<response>
<responseHeader><status>0</status><QTime>1</QTime></responseHeader>
<result numFound="1" start="0">
  <doc>
    <arr name="cat"><str>electronics</str><str>hard drive</str></arr>
    <arr name="features"><str>7200RPM, 8MB cache, IDE Ultra ATA-133</str>
      <str>NoiseGuard, SilentSeek technology, Fluid Dynamic Bearing (FDB) motor</str></arr>
    <str name="id">SP2514N</str>
    <bool name="inStock">true</bool>
    <str name="manu">Samsung Electronics Co. Ltd.</str>
    <str name="name">Samsung SpinPoint P120 SP2514N - hard drive - 250 GB - ATA-133</str>
    <int name="popularity">6</int>
    <float name="price">92.0</float>
    <str name="sku">SP2514N</str>
  </doc>
</result>
</response>
```

下面是带有限制的字段列表的查询：

```
http://localhost:8983/solr/techproducts/select?q=id:SP2514N&fl=id+name
```

结果：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<response>
<responseHeader><status>0</status><QTime>2</QTime></responseHeader>
<result numFound="1" start="0">
  <doc>
    <str name="id">SP2514N</str>
    <str name="name">Samsung SpinPoint P120 SP2514N - hard drive - 250 GB - ATA-133</str>
  </doc>
</result>
</response>
```

## <a name="specify-terms"></a>给标准查询解析器指定词项
## <a name="specify-fields"></a>给标准查询解析器的查询指定字段
## <a name="boolean-operators"></a>标准查询解析器支持的布尔操作符
## <a name="grouping"></a>分组词项以形成子查询
## <a name="difference"></a>Lucene 查询解析其和 Solr 标准查询解析器的不同点
## <a name="related"></a>相关内容

