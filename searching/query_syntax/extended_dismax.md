# 扩展的 DisMax 查询解析器

扩展的 DisMax (eDisMax) 查询解析器是 [DisMax 查询解析器](./dismax.md) 的改进版。
除了支持所有 DisMax 查询解析器参数外， eDisMax 还具有：

* 支持完整的 Lucene 查询解析器语法
* 支持像 AND/OR/NOT/-/+ 这样的查询
* 将 "and" 或 "or" 处理为 Lucene 语法模式中的 "AND" 和 "OR"
* 支持名为 `_val_` 和 `_query_` 的 “魔法字段”。它们不是 `schema.xml` 中的真实字段，
但可用于完成特殊的事情(如使用 `_val_` 作函数查询或者用 `_query_` 做内嵌查询)。
若词项或者短语查询中使用了 `_val_`，其值将作为函数被解析。
* 包含在语法错误时改进的智能部分转码；字段查询、+/-、和短语查询在这种模式下依旧支持。
* 通过使用单词 shingles 改进近邻加权；不需要在近邻加权应用之前使用查询来匹配文档中的所有单词。
* 包含高级的停用词处理：停用词在查询中的主体部分是不需要的，但在近邻加权部分依旧使用。
若一个查询包含所有的停用词，如 "to be or not to be"，则所有的单词都是必需的。
* 包含改进的加权函数：在 eDisMax 中， `boost` 函数是一个乘法器而不是加法器，这改进了加权结果;
并且还支持额外的 DisMax 加权函数(`bf` 和 `bq`)。
* 支持纯负内嵌查询：像 `+foo (-foo)` 这样的查询将匹配所有文档
* 让你可以指定哪些字段允许用户进行查询，以及不允许直接字段搜索

## 扩展的 DisMax 参数

除了所有 [DisMax 参数](https://cwiki.apache.org/confluence/display/solr/The+DisMax+Query+Parser#TheDisMaxQueryParser-DisMaxParameters)
参数外，扩展的 DisMax 包含以下参数：

### 参数 boost

A multivalued list of strings parsed as queries with scores multiplied by the score from the main query for all
matching documents. This parameter is shorthand for wrapping the query produced by eDisMax using the `BoostQParserPlugin`

### 参数 lowercaseOperators

A Boolean parameter indicating if lowercase "and" and "or" should be treated the same as operators "AND" and
"OR".

### 参数 ps

Default amount of slop on phrase queries built with `pf` , `pf2` and/or `pf3` fields (affects boosting).

### 参数 pf2

A multivalued list of fields with optional weights, based on pairs of word shingles.

### 参数 ps2

This is similar to `ps` but overrides the slop factor used for `pf2` . If not specified, `ps` is used.

### 参数 pf3

A multivalued list of fields with optional weights, based on triplets of word shingles. Similar to `pf` , except that
instead of building a phrase per field out of all the words in the input, it builds a set of phrases for each field out
of each triplet of word shingles.

### 参数 ps3

This is similar to `ps` but overrides the slop factor used for `pf3` . If not specified, `ps` is used.

### 参数 stopwords

A Boolean parameter indicating if the `StopFilterFactory` configured in the query analyzer should be
respected when parsing the query: if it is false, then the `StopFilterFactory` in the query analyzer is ignored.

### 参数 uf

Specifies which schema fields the end user is allowed to explicitly query. This parameter supports wildcards. The
default is to allow all fields, equivalent to `uf=*` . To allow only title field, use `uf=title` . To allow title and all
fields ending with `_s`, use `uf=title,*_s` . To allow all fields except title, use `uf=*-title` . To disallow all
fielded searches, use `uf=-*` .

### 字段别名，使用 qf 针对字段进行覆盖

Per-field overrides of the qf parameter may be specified to provide 1-to-many aliasing from field names specified
in the query string, to field names used in the underlying query. By default, no aliasing is used and field names
specified in the query string are treated as literal field names in the index.

## 提交给 eDisMax 查询解析器的查询示例

所有本节的示例 URL 假设你运行的是 Solr 的 `techproducts` 示例：

```bash
bin/solr -e techproducts
```

基于文档的流行度加权查询词项 "hello" 的结果：

```
http://localhost:8983/solr/techproducts/select?defType=edismax&q=hello&pf=text&qf=text&boost=popularity
```

搜索 iPods 或 video：

```
http://localhost:8983/solr/techproducts/select?defType=edismax&q=ipod+OR+video
```

跨多个字段搜索，指定(通过加权)每个字段相对的重要程度：

```
http://localhost:8983/solr/techproducts/select?q=video&defType=edismax&qf=features^20.0+text^0.3
```

可以在某个字段匹配特定值时进行加权：

```
http://localhost:8983/solr/techproducts/select?q=video&defType=edismax&qf=features^20.0+text^0.3&bq=cat:electronics^5.0
```

使用 "mm" 参数，1 和 2 个单词查询需要所有可选子句都匹配，但对有 3个或以上的子句的查询，某个子句缺少是允许的：

```
http://localhost:8983/solr/techproducts/select?q=belkin+ipod&defType=edismax&mm=2
http://localhost:8983/solr/techproducts/select?q=belkin+ipod+gibberish&defType=edismax&mm=2
http://localhost:8983/solr/techproducts/select?q=belkin+ipod+apple&defType=edismax&mm=2
```

下例中，我们可以看到针对字段的 `qf` 参数重载以将 "name" 在查询中用作 last_name " 及 " first_name " 字段的别名：

```
defType=edismax
q=sysadmin name:Mike
qf=title text last_name first_name
f.name.qf=last_name first_name
```

### 使用负加权

负向查询加权在 "Query" 对象层已经支持很长时间了(导致对匹配文档的负分数)。
现在 QueryParsers 被更新也能够处理这个。

### 使用 'slop'

`DisMax` 和 `eDisMax` 可以运行在所有字段上的查询，以短语形式对短语字段运行查询。
(这仅针对文档加权，而不是具体的匹配)。然而，短语查询也可以包含一个 'slop'，
它是查询的词项间的距离依旧被考虑为匹配短语的最大距离。如：

```
q=foo bar
qf=field1^5 field2^10
pf=field1^50 field2^20
defType=dismax
```

对这些参数， DisMax 查询解析器会生成像下面这样的查询：

```
(+(field1:foo^5 OR field2:bar^10) AND (field1:bar^5 OR field2:bar^10))
```

但它也会生成另一个仅用作加权结果的查询：

```
field1:"foo bar"^50 OR field2:"foo bar"^20
```

这样，任何包含词项 "foo" 和 "bar" 的文档都会匹配，但若某些文档以短语包含这两个词项，
它的分数会更高，因为它的相关度更高。

若你添加参数 `ps`(短语 slop)，将第二个查询替换为：

```
ps=10 field1:"foo bar"~10^50 OR field2:"foo bar"~10^20
```

这意味着若词项 "foo" 和 "bar" 在文档中的距离少于 10 个词项，短语被认为是匹配的。
如，以下文档：

```
*Foo* term1 term2 term3 *bar*
```

将匹配短语查询。

如何使用短语 slop 呢？通常它被配置在请求处理器中(在 `solrconfig.xml` 中)。

对查询 slop (qs) 其概念是类似的，但它应用于来自用户的明确的短语查询。
如你希望搜索某个名字，你可以输入：

```
q="Hans Anderson"
```

包含 "Hans Anderson" 的文档将会匹配，但包含了中间名 "Christian" 
或姓放前面的写法 ("Anderson, Hans") 将不会匹配。
对这些场景，可以配置查询字段 `qs`，这样即使用户搜索明确的短语查询，slop 依旧会被应用。

最后，eDisMax 不仅包含了短语字段 (`pf`) 参数，也包含短语和查询字段 2 和 3.
你可以用这些字段来设置不同的字段或加权。其中每个都可以使用不同的短语 slop。

### 使用‘魔法字段’ `_val_` 和 `_query_`

若词项或短语查询中使用了名为 `_val_` 的魔法字段，其值被解析为一个函数。

Solr 查询解析器对 `_val_` 和 `_query_` 的使用和 Lucene 查询解析器有以下不同：

* 若词项或短语查询中使用了名为 `_val_` 的魔法字段，其值被解析为一个函数。
* 它提供了到 [FunctionQuery](http://wiki.apache.org/solr/FunctionQuery) 语法的一个 hook。
若包含括号时需使用引号将其封装，如：<br/><pre><code>_val_:myfield
_val_:"recip(rord(myfield),1,2,3)"</code></pre>
* Solr 查询解析器提供了对任何类型查询解析器的内嵌查询(通过 QParserPlugin)。
如果内嵌的查询中包含保留字符，引号可以提供必要的封装。如：<br/><pre><code>_query_:"{!dismax qf=myfield}how now brown cow"</code></pre>

尽管技术上没有语法差别，但需注意当你使用 Solr `TrieDateField` 类型(或废弃的 DateField 类型)时，
对这些字段的任何查询(通常是范围查询)应该使用该字段所支持的完整的 ISO 8601 日期格式，
或者 [DateMath 语法](http://lucene.apache.org/solr/5_2_0/solr-core/org/apache/solr/util/DateMathParser.html)
来获取相对日期。如：

```
timestamp:[* TO NOW]
createdate:[1976-03-06T23:59:59.999Z TO *]
createdate:[1995-12-31T23:59:59.999Z TO 2007-03-06T00:00:00Z]
pubdate:[NOW-1YEAR/DAY TO NOW/DAY+1DAY]
createdate:[1976-03-06T23:59:59.999Z TO 1976-03-06T23:59:59.999Z+1YEAR]
createdate:[1976-03-06T23:59:59.999Z/YEAR TO 1976-03-06T23:59:59.999Z]
```

> 注：`TO` 必须是大写的，否则 Solr 将报告一个 'Range Group' 错误。
