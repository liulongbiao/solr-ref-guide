# 内嵌子文档

Solr 内嵌子文档在索引时使用 "Block Join" 来作为一种建模包含其它文档的文档的方式。
如博客内容作为父文档而其评论作为子文档，或者产品作为父文档而尺寸、颜色或其它变量作为子文档。
在查询时， [BlockJoinQueryParsers](https://cwiki.apache.org/confluence/display/solr/Other+Parsers#OtherParsers-BlockJoinQueryParsers)
可用于在这些关系上进行搜索。
在性能上，索引文档之间的关系可能比试图在查询时进行 join 更加高效，
因为关系已经存储于索引中且不需要重新计算。

内嵌文档可以通过 XML 或 JSON 数据语法(或使用 [SolrJ](../client_api/solrj.md) )来索引。
但不管语法如何，你可以包含一个字段来表示父文档为父;
它可以是任何适合该目的的字段，且它将用作
[BlockJoinQueryParsers](https://cwiki.apache.org/confluence/display/solr/Other+Parsers#OtherParsers-BlockJoinQueryParsers)
的输入。

## XML 示例

如，下面有两个文档及其子文档：

```xml
<add>
    <doc>
    <field name="id">1</field>
    <field name="title">Solr adds block join support</field>
    <field name="content_type">parentDocument</field>
        <doc>
        <field name="id">2</field>
        <field name="comments">SolrCloud supports it too!</field>
        </doc>
    </doc>
    <doc>
    <field name="id">3</field>
    <field name="title">New Lucene and Solr release is out</field>
    <field name="content_type">parentDocument</field>
        <doc>
        <field name="id">4</field>
        <field name="comments">Lots of new features</field>
        </doc>
    </doc>
</add>
```

这里，我们有被索引的父文档，其具有字段 `content_type`，其值为 "parentDocument"。
我们也可以使用一个布尔字段，如 `isParent`，值为 `true`，或其它类似的方式。

## JSON 示例

本例等价于上面的 XML 示例，注意需要特殊的 `_childDocuments_` 键来指示 JSON 中的内嵌文档。

```javascript
[
{
    "id": "1",
    "title": "Solr adds block join support",
    "content_type": "parentDocument",
    "_childDocuments_": [
        {
        "id": "2",
        "comments": "SolrCloud supports it too!"
        }
    ]
},
{
    "id": "3",
    "title": "New Lucene and Solr release is out",
    "content_type": "parentDocument",
    "_childDocuments_": [
        {
        "id": "4",
        "comments": "Lots of new features"
        }
    ]
}
]
```

> 注：一个索引内嵌文档的限制是父子文档的整个块在任何需要变更的时候需要被一起更新。
> 或者说，即使只有单个子文档或单个父文档有变更，整个父子文档块需要一起更新。
