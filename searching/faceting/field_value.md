# 字段-值 Faceting 参数

多个参数可用于触发基于某个字段中被索引词项的 faceting。

当使用该参数时，需记住的是 "term" 在 Lucene 中是一个非常特殊的概念：它关系到
在任何分析发生以后被索引的字面的 字段/值 对。
对经过了提干、小写或分词的文本字段，其结果词项可能不是你所希望的。
若你希望 Solr 既执行分析(用于搜索)，又针对完整的字面字符串进行 faceting，
可以在 `schema.xml` 中使用 `copyField` 指令来创建相同字段的多个版本：一个文本一个字符串。
确保它们都设置了 `indexed="true"`。
更多有关 `copyField` 指令的信息，查看 [文档、字段和模式设计](../../schema/readme.md)。

下表列出了字段值 faceting 参数。

|参数           |描述                  |
|--------------|----------------------|
|[facet.field](#facet-field) |指示某个字段应被处理为一个 facet。 |
|[facet.prefix](#facet-prefix) |限制用作 faceting 的词项为以特定前缀开头的词项。 |
|[facet.contains](#facet-contains) |限制用作 faceting 的词项为包含特定子字符串的词项。 |
|[facet.contains.ignoreCase](#facet-contains-ignoreCase) |若使用了 `facet.contains`，当搜索特定子字符串时忽略大小写。 |
|[facet.sort](#facet-sort) |控制 facet 的结果如何排序。 |
|[facet.limit](#facet-limit) |控制每个 facet 应该返回多少个约束。 |
|[facet.offset](#facet-offset) |指定对 facet 结果的偏移量以开始显示 facets。 |
|[facet.mincount](#facet-mincount) |指定应该被包含在响应中的 facet 字段的最小必需计数。 |
|[facet.missing](#facet-missing) |控制 Solr 是否应该在某个字段基于词项的约束之外，计算所有对该字段没有值的匹配结果进行计数。 |
|[facet.method](#facet-method) |选择当 faceting 一个字段时， Solr 应该使用的算法或方法。 |
|[facet.enum.cache.minDF](#facet-enum-cache-minDF) |(高级)指定在决定该项的约束数量时，`filterCache` 应该被使用的最小文档频度(文档匹配某个词项的数量) |
|[facet.overrequest.count](#facet-overrequest) |(高级)文档的数量，在高效的 `facet.limit`之外以从分布式搜索的每个分片中请求。 |
|[facet.overrequest.ratio](#facet-overrequest) |(高级)一个和高效的 `facet.limit` 的乘积以从一个分布式搜索的每个分片中请求。 |
|[facet.threads](#facet-threads) |(高级)控制字段 faceting 的并行执行。 |

这些参数的描述如下。

## <a name="facet-field"></a>参数 `facet.field`

The `facet.field` parameter identifies a field that should be treated as a facet. It iterates over each Term in the
field and generate a facet count using that Term as the constraint. This parameter can be specified multiple
times in a query to select multiple facet fields.

> If you do not set this parameter to at least one field in the schema, none of the other parameters
> described in this section will have any effect.

## <a name="facet-prefix"></a>参数 `facet.prefix`

The `facet.prefix` parameter limits the terms on which to facet to those starting with the given string prefix.
This does not limit the query in any way, only the facets that would be returned in response to the query.
This parameter can be specified on a per-field basis with the syntax of `f.<fieldname>.facet.prefix` .

## <a name="facet-contains"></a>参数 `facet.contains`

The `facet.contains` parameter limits the terms on which to facet to those containing the given substring. This
does not limit the query in any way, only the facets that would be returned in response to the query.
This parameter can be specified on a per-field basis with the syntax of `f.<fieldname>.facet.contains` .

## <a name="facet-contains-ignoreCase"></a>参数 `facet.contains.ignoreCase`

If `facet.contains` is used, the `facet.contains.ignoreCase` parameter causes case to be ignored when
matching the given substring against candidate facet terms.
This parameter can be specified on a per-field basis with the syntax of `f.<fieldname>.facet.contains.ignoreCase` .

## <a name="facet-sort"></a>参数 `facet.sort`

This parameter determines the ordering of the facet field constraints.

|facet.sort 设置 |结果                         |
|---------------|----------------------------|
|count          |根据计数进行排序(计数高的在前)    |
|index          |以其索引顺序排序约束(索引词项的字典序)。对在 ASCII 范围的词项，它是字母序的。 |

默认在 `facet.limit` 大于 0 时是 `count`。否则，默认为 `index`。
该参数可以以语法 `f.<fieldname>.facet.sort` 针对每个字段指定。

## <a name="facet-limit"></a>参数 `facet.limit`

This parameter specifies the maximum number of constraint counts (essentially, the number of facets for a field
that are returned) that should be returned for the facet fields. A negative value means that Solr will return
unlimited number of constraint counts.

The default value is 100.

This parameter can be specified on a per-field basis to apply a distinct limit to each field with the syntax of 
`f.<fieldname>.facet.limit` .

## <a name="facet-offset"></a>参数 `facet.offset`

The `facet.offset` parameter indicates an offset into the list of constraints to allow paging.

The default value is 0.

This parameter can be specified on a per-field basis with the syntax of `f.<fieldname>.facet.offset` .

## <a name="facet-mincount"></a>参数 `facet.mincount`

The `facet.mincount` parameter specifies the minimum counts required for a facet field to be included in the
response. If a field's counts are below the minimum, the field's facet is not returned.

The default value is 0.

This parameter can be specified on a per-field basis with the syntax of `f.<fieldname>.facet.mincount` .

## <a name="facet-missing"></a>参数 `facet.missing`

If set to true, this parameter indicates that, in addition to the Term-based constraints of a facet field, a count of all
results that match the query but which have no facet value for the field should be computed and returned in the
response.

The default value is false.

This parameter can be specified on a per-field basis with the syntax of `f.<fieldname>.facet.missing` .

## <a name="facet-method"></a>参数 `facet.method`

The facet.method parameter selects the type of algorithm or method Solr should use when faceting a field.

|设置    |结果                        |
|-------|---------------------------|
|enum   |Enumerates all terms in a field, calculating the set intersection of documents that match the term with documents that match the query. This method is recommended for faceting multi-valued fields that have only a few distinct values. The average number of values per document does not matter. For example, faceting on a field with U.S. States such as Alabama, Alaska, ... Wyoming would lead to fifty cached filters which would be used over and over again. The filterCache should be large enough to hold all the cached filters.|
|fc     |Calculates facet counts by iterating over documents that match the query and summing the terms that appear in each document. This is currently implemented using an UnInvertedField cache if the field either is multi-valued or is tokenized (according to FieldType.isTokened() ). Each document is looked up in the cache to see what terms/values it contains, and a tally is incremented for each value. This method is excellent for situations where the number of indexed values for the field is high, but the number of values per document is low. For multi-valued fields, a hybrid approach is used that uses term filters from the filterCache for terms that match many documents. The letters fc stand for field cache.|
|fcs    |Per-segment field faceting for single-valued string fields. Enable with facet.method=fcs and control the number of threads used with the threads local parameter. This parameter allows faceting to be faster in the presence of rapid index changes.|

The default value is fc (except for fields using the BoolField field type) since it tends to use less memory and
is faster when a field has many unique terms in the index.

This parameter can be specified on a per-field basis with the syntax of `f.<fieldname>.facet.method` .

## <a name="facet-enum-cache-minDF"></a>参数 `facet.enum.cache.minDF`

This parameter indicates the minimum document frequency (the number of documents matching a term) for
which the filterCache should be used when determining the constraint count for that term. This is only used with
the facet.method=enum method of faceting.

A value greater than zero decreases the filterCache's memory usage, but increases the time required for the
query to be processed. If you are faceting on a field with a very large number of terms, and you wish to decrease
memory usage, try setting this parameter to a value between 25 and 50, and run a few tests. Then, optimize the
parameter setting as necessary.

The default value is 0, causing the filterCache to be used for all terms in the field.

This parameter can be specified on a per-field basis with the syntax of `f.<fieldname>.facet.enum.cache.minDF` .

## <a name="facet-overrequest"></a>Over-Request 参数

In some situations, the accuracy in selecting the "top" constraints returned for a facet in a distributed Solr query
can be improved by "Over Requesting" the number of desired constraints (ie: facet.limit ) from each of the
individual Shards. In these situations, each shard is by default asked for the top "10 + (1.5 *
facet.limit) " constraints.

In some situations, depending on how your docs are partitioned across your shards, and what facet.limit val
ue you used, you may find it advantageous to increase or decrease the amount of over-requesting Solr does.
This can be achieved by setting the facet.overrequest.count (defaults to 10) and facet.overrequest.
ratio (defaults to 1.5) parameters.

## <a name="facet-threads"></a>参数 `facet.threads`

This param will cause loading the underlying fields used in faceting to be executed in parallel with the number of
threads specified. Specify as facet.threads=N where N is the maximum number of threads used. Omitting
this parameter or specifying the thread count as 0 will not spawn any threads, and only the main request thread
will be used. Specifying a negative number of threads will create up to Integer.MAX_VALUE threads.
