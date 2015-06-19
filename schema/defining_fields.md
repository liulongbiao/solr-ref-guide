# 定义字段

字段在 `schema.xml` 中以 `field` 元素来定义。
一旦你设置了字段类型，定义字段本身就很简单了。

## 示例

以下示例定义了一个名为 `price`的字段，其类型名为 `float` 且默认值为 `0.0`；
其 `indexed` 和 `stored` 属性都被明确设置为 `true`，而其它指定在 `float` 上的属性都被继承了。

```xml
<field name="price" type="float" default="0.0" indexed="true" stored="true"/>
```

## 字段属性

| 属性   | 描述    |
|-------|:--------|
|name   |字段的名称。字段名应仅由字母数字或下划线组成且不以数字开头。当前这不是严格强制的，但其它形式的名称将不被所有组建一级支持，且后向兼容性也不被保证。前后都有下划线的名称(如 `_version_`)是保留的。每个字段都必须有一个 `name` 属性。 |
|type   |该字段的 `fieldType` 名称。它将找到 `fieldType` 上对应的 `name` 属性。每个字段都必须有一个 `type` 属性。 |
|default|当索引时，该字段没有值的文档会被自动添加上的一个默认值。若该属性没有指定，就没有默认值。 |

## 可选字段类型覆盖属性

字段可以具有和字段类型相同的选项。
字段类型的选项用作具体每个字段定义时可覆盖的选项的默认值。
下表包含 [字段类型的定义和属性](./field_type/definition.md) 中字段类型属性列表：

|  属性  |   描述    |  值  |
|-------|:---------|------|
| indexed | 若为 true，该字段的值可用在查询中来检索匹配的文档 | `true/false` |
| stored | 若为 true，该字段的实际值可被查询检索出来 | `true/false` |
| docValues | 若为 true，该字段的值可被放入一个面向列的 [DocValues](./docvalues.md) 结构中 | `true/false` |
| sortMissingFirst <br/> sortMissingLast | 控制当排序字段不存在时文档的位置。在 Solr 3.5 中，它们对所有的数值字段都适用，包括 Trie 和日期字段 | `true/false` |
| multiValued | 若为 true，表示单个文档可能包含多个该字段类型的值 | `true/false` |
| omitNorms | 若为 true，忽略该字段关联的范数(它会对该字段禁用长度规范化和索引时加权，并节省部分内存)。默认对所有的原生(非解析)字段类型为true， 如 int、float、data、bool 和 string。仅有全文字段或需要索引时加权的字段需要范数。 | `true/false` |
| omitTermFreqAndPositions | 若为 true，在对该字段 posting时，忽略词频、位置和荷载。它对不需要这些信息的字段能起到性能加权的效果。它也能减少索引所需的存储空间。对该字段发出的依赖位置的查询将默默地查不到文档。该属性对所有非文本字段默认为 true | `true/false` |
| omitPositions | 类似于 `omitTermFreqAndPositions`，但保留了词频信息 | `true/false` |
| termVectors <br/> termPositions <br/> termOffsets <br/> termPayloads | 这些选项指示 Solr 对每个文档维护完整的词频向量，可选地对这些向量中出现的每个项包含位置、位移和荷载信息。这些可用于加速高亮及其他辅助功能，但需要在索引尺寸上引入一些成本。它们对典型的 Solr 使用是不必需的 | `true/false` |
| required | 指示 Solr 拒绝任何添加不包含该字段的值的请求。该属性默认为 false | `true/false` |

## 相关内容

* [SchemaXML-Fields](http://wiki.apache.org/solr/SchemaXml#Fields)
* [Field Options by Use Case](http://wiki.apache.org/solr/FieldOptionsByUseCase)
