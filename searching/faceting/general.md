# 通用参数

下表列出了用于控制 faceting 的通用参数。

|参数      |描述                  |
|---------|---------------------|
|[facet](#facet)|若为 true，启用 faceting。|
|[facet.query](#facet-query)|指定一个 Lucene 查询来生成一个 facet 计数。 |

这些参数由以下小节描述：

## <a name="facet"><a>参数 `facet`

若设置为 `true`，该参数启用查询响应中的 facet 计数。
若设置为 `false` 或空白或缺失值，该参数会禁用 faceting。
除非该参数的值为 `true`，否则以下列出的所有参数都没有效果。
其默认值为空白。

## <a name="facet-query"><a>参数 `facet.query`

该参数让你可以指定一个 Lucene 默认语法格式的任意的查询来生成一个 facet 计数。
默认， Solr 的 faceting 特性会自动决定某个字段的唯一项并对其中的每个词项返回一个数值。
使用 `facet.query`，你可以覆盖这种默认行为并准确地选择你希望被计数的哪个词项或表达式。
在一个典型的 faceting 实现中，你将指定多个 `facet.query` 参数。
这个参数对基于数值范围或基于前缀的 facet 非常有用。

你可以多次设置 `facet.query` 参数以指示有多个查询应被用于独立的 facet 约束。

要以默认语法以外的语法来使用 facet 查询，需要给该 facet 查询前缀查询记法的名称。
如使用假想的 `myfunc` 查询解析器，你可以设置 `facet.query` 参数如下：

```
facet.query={!myfunc}name~fred
```
