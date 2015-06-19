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

对标准查询解析器的查询会被分为词项和操作符。存在两种类型的词项：单词项和短语

* 单词项是单个的词，如 "test" 或 "hello"
* 短语是一组由双引号括起的单词，如 "hello dolly"

多个词项可以由布尔操作符组合到一起形成更复杂的查询(如下所述)。

> 确保针对查询的分析器解析词项和短语的方式和针对索引的分析器解析词项和短语的方式保持一致很重要；
> 否则，搜索可能会产生意料之外的结果。

### 词项修饰符

Solr 支持多种词项修饰符来根据具体搜索需求来添加灵活性或精度。
这些修饰符包括通配符、用于进行“模糊”搜索或更通用的字符，等等。
本节一下部分详细说明这些操作符。

#### 通配符搜索

Solr 的标准查询解析器支持单个词项中通配单个或多个字符的搜索。
通配符可应用于单词项，而不能应用于搜索短语。

|通配符搜索类型  |特殊字符| 示例                           |
|单个字符(匹配单个字符)|? |搜索字符串 `te?t` 能够匹配 `test` 和 `text`  |
|多个字符(匹配零或多个顺序字符)|* |通配符搜索 `tes*` 能够匹配 `test`、`testing`和 `tester`。<br>你也可以在词项中间使用通配符，如 `te*t`，它能匹配`test`和 `text`。<br>`*est`能匹配 `pest`和 `test`。 |

#### 模糊搜索

Solr 的标准查询解析器支持基于 Levenshtein 距离和编辑距离的模糊搜索。
模糊搜索能发现类似于特定项但不需要精确匹配的词项。
要执行模糊搜索，在单词项的末尾使用波浪号 `~`。

如，要搜索类似于 `roam` 的词项，使用模糊搜索：`roam~`。
这个搜索可以匹配像 `roams`，`foam`和 `foams` 等词项。它也能匹配 `roam` 本身。

默认的距离参数指定了允许的最大的编辑距离，从 0 到 2,默认为 2。 
如： `roam~1`。它会匹配 `roams` 和 `foam` - 但 `foams` 因为编辑距离为 2 所以不匹配。

> 很多情况下，词干化(将词项缩减为公共的词干)可以产生和模糊搜索和通配符搜索类似的效果。

#### 邻近搜索

A proximity search looks for terms that are within a specific distance from one another.

To perform a proximity search, add the tilde character ~ and a numeric value to the end of a search phrase. For
example, to search for a "apache" and "jakarta" within 10 words of each other in a document, use the search:

```
"jakarta apache"~10
```

The distance referred to here is the number of term movements needed to match the specified phrase. In the
example above, if "apache" and "jakarta" were 10 spaces apart in a field, but "apache" appeared before "jakarta",
more than 10 term movements would be required to move the terms together and position "apache" to the right
of "jakarta" with a space in between.

#### 范围搜索

范围搜索指定了某个字段的值的范围(一个范围具有上限和下限)。
该查询将匹配那些字段的值处于给定范围中的文档。
范围查询可以包含或排除上下限。
排序是按照字典序的，除了数值字段。
例如，下面的范围查询匹配所有 `mod_date` 字段的值在 `20020101` 和 `20030101` 之间的文档，它是闭区间。

```
mod_date:[20020101 TO 20030101]
```

范围查询不限于日期字段或数值字段。你也可以在非日期字段上进行范围查询：

```
title:{Aida TO Carmen}
```

它会找到所有标题在 `Aida` 和 `Carmen` 之间的文档，但不包括 Aida` 和 `Carmen`。

查询中的括号决定了其开闭性。

* 方括号 `[]` 表示包含上下限的值的闭区间范围查询
* 花括号 `{}` 表示匹配上下限之间的值，但不包括上下限的开区间范围查询
* 你可以混合这两种类型让范围的一端为开，一端为闭。如： `count:{1 TO 10]`

#### 用 `^` 给词项加权

Lucene/Solr 提供了基于匹配的文档中所找到的词项的相关度水平。
可以在你搜索的某个词项的后面添加脱字符 `^` 后加一个加权因子(一个数字) 来提升该词项的权重。
权重因子越高，该词项的相关度也越高。

加权可以让你通过给词项加权来控制文档的相关度。如，你搜索 "jakarta apache" 时希望
词项 "jakarta" 更相关一些，你可以通过在该词项后添加 `^` 及加权因子来提升权重。
如你可以写：

```
jakarta^4 apache
```

这会让词项 "jakarta" 更相关一些。你也可以提升短语的权重，如：

```
"jakarta apache"^4 "Apache Lucene"
```

默认权重因子为 1。尽管权重因子必须为正，但它可以小于 1 (如，它可以是 0.2)。

#### 带 `^=` 的常量分数

可以使用 `<query_clause>^=<score>` 来创建常量查询，
它会将任何匹配该子句的文档设置整个子句的分数为给定的分数。
这在你仅关心是否匹配某个子句，但不关心像词频(词项在文档中出现的次数)
或反向文档频率(测量词项在该字段的整个索引中的稀有程度)等其他因子时很有用。

如：

```
(description:blue OR color:blue)^=1.0 text:shoes
```

## <a name="specify-fields"></a>给标准查询解析器的查询指定字段

Solr 中索引的数据以字段进行组织，字段由 Solr 中的 `schema.xml` 文件所定义。
搜索可以利用字段来给查询添加精度。例如，你可以搜索仅在特定字段上的词项，如 `title` 字段。

`schema.xml` 文件定义了一个字段作为默认字段。
若你在查询中没有指定字段， Solr 只会搜索默认字段。
或者你可以在查询中指定一个不同的字段或字段的组合。

要指定一个字段，书写字段名，后跟一个冒号 ":"，然后跟你要在该字段搜索的词项。

如，假设一个索引包含两个字段 `title` 和 `text`，且 `text` 是默认字段。
若你希望查找名为 "The Right Way" 的文档，其中包含文本 "don't go this way"，
你可以在搜索查询中保护以下词项：

```
title:"The Right Way" AND text:go

title:"Do it right" AND go
```

因为 `text` 是默认字段，该字段的指示器不是必须的；因此第二个查询忽略了它。

字段名仅对后面直接跟着的词项有效，因此查询 `title:Do it right` 只会在 `title` 字段查找 "Do"，
而在默认字段(这里是 `text`) 中查找 "it" 和 "right"。

## <a name="boolean-operators"></a>标准查询解析器支持的布尔操作符
## <a name="grouping"></a>分组词项以形成子查询
## <a name="difference"></a>Lucene 查询解析其和 Solr 标准查询解析器的不同点
## <a name="related"></a>相关内容

