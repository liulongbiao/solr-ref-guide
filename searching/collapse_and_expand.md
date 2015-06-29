# 折叠和展开结果

折叠查询解析器和展开组件联合在一起形成了对搜索结果字段折叠以进行文档分组的方式。

## 折叠查询解析器

`CollapsingQParser` 实际上是一个 *post filter* ，
它在结果集中不同的分组数量很高时，提供了比 Solr 的标准方式性能更好的字段折叠方式。
该解析器在其将结果集传递给后续搜索组件之前，将每个分组折叠为单个文档。
这样所有下游的组件(faceting、高亮等)将作用在一个折叠的结果集上。

`CollapsingQParser` 接收以下本地参数：

|参数    |描述                          |默认值  |
|-------|-----------------------------|-------|
|field  |待折叠的字段。该字段必须是单值的 String、Int 或 Float |none |
|min/max|根据某个数值字段或函数查询的最小或最大值来选择分组头部。若没有指定，分组头部将基于结果集中分数最高的文档来选择。 |none |
|nullPolicy|存在 3 种 null 策略： <ul><li>**ignore**: 移除所有在折叠字段中 null 值文档。它是默认的。</li><li>**expand**: 将所有折叠字段中 null 值文档处理为一个独立的分组。</li><li>**collapse**: 将所有折叠字段中 null 值文档折叠到最高分或者最大/最小值中。</li></ul> |ignore |
|hint   |Currently there is only one hint available "top_fc", which stands for top level FieldCache. The top_fc hint is only available when collapsing on String fields. top_fc provides the best query time speed but takes the longest to warm on startup or following a commit. top_fc also will result in having the collapsed field cached in memory twice if the it's used for faceting or sorting.|none |
|size   |Sets the initial size of the collapse data structures when collapsing on a **numeric field only** . The data structures used for collapsing grow dynamically when collapsing on numeric fields. Setting the size above the number of results expected in the result set will eliminate the resizing cost. |100,000 |

语法示例：

基于最高分文档折叠：

`fq={!collapse field=field_name}`

基于某个数值字段的最小值折叠：

`fq={!collapse field=field_name min=field_name}`

基于某个数值字段的最大值折叠：

`fq={!collapse field=field_name max=field_name}`

基于某个函数的最小/最大值折叠。其 `cscore()` 函数可用于 `CollapsingQParserPlugin` 中，
来返回当前被折叠文档的分数。

`fq={!collapse field=field_name max=sum(cscore(),field(A))}`

使用 null 策略折叠：

`fq={!collapse field=field_name nullPolicy=nullPolicy}`

使用 hint 折叠：

`fq={!collapse field=field_name hint=top_fc}`

`CollapsingQParserPlugin` 完全支持 `QueryElevationComponent`。

## 展开组件

`ExpandComponent` 可用于展开被
[CollapsingQParserPlugin](http://heliosearch.org/the-collapsingqparserplugin-solrs-new-high-performance-field-collapsing-postfilter/)
所折叠的分组。

`CollapsingQParserPlugin` 使用示例：

`q=foo&fq={!collapse field=ISBN}`

上述查询中，`CollapsingQParserPlugin` 会在 ISBN 字段上折叠搜索结果。
主搜索结果将对每本书包含排序最高的文档。

`ExpandComponent` 现在可用于展开该结果，这样你可以看到由 ISBN 所分组的文档。如：

`q=foo&fq={!collapse field=ISBN}&expand=true`

其 `expand=true` 启用了 `ExpandComponent`。
`ExpandComponent` 给搜索输出中添加了额外的 "expanded" 部分。

在 "expanded" 部分中存在每个分组头指向包含在分组中的被展开文档的 *映射*。
当应用在主折叠结果集上迭代时，它们可以访问 *expanded* 映射以获取被展开分组。

`ExpandComponent` 具有以下参数：

|参数         |描述                          |默认值  |
|------------|-----------------------------|-------|
|expand.sort |被展开分组中文档的排序            |`score desc`|
|expand.rows |每个分组中显示的行数              |5|
|expand.q    |覆盖主 `q` 参数，决定哪些文档被包含在主分组中。|main q|
|expand.fq   |覆盖主 `fq` 参数，决定哪些文档被包含在主分组中。|main fq|
