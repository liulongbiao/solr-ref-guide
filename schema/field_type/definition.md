# 字段类型的定义和属性

一个字段类型定义可以包含一下四种信息：

* 字段类型的名称(必需)
* 一个实现类的名称(必需)
* 若字段类型为 `TextField`，针对该字段类型可以有一个字段分析的描述
* 字段类型属性 - 依赖于具体的实现类，某些属性可能是必需的

## `schema.xml` 中的字段类型定义

字段类型定义在 `schema.xml` 中。
每个字段类型在 `fieldTYpe` 元素之间定义。
它们可以选择分组在一个 `types` 元素中。
下面是一个称为 `text_general` 的类型的字段类型定义的示例：

```xml
<fieldType name="text_general" class="solr.TextField" positionIncrementGap="100">
  <analyzer type="index">
    <tokenizer class="solr.StandardTokenizerFactory"/>
    <filter class="solr.StopFilterFactory" ignoreCase="true" words="stopwords.txt"/>
    <!-- in this example, we will only use synonyms at query time
    <filter class="solr.SynonymFilterFactory" synonyms="index_synonyms.txt"
      ignoreCase="true" expand="false"/>
    -->
    <filter class="solr.LowerCaseFilterFactory"/>
  </analyzer>
  <analyzer type="query">
    <tokenizer class="solr.StandardTokenizerFactory"/>
    <filter class="solr.StopFilterFactory" ignoreCase="true" words="stopwords.txt"/>
    <filter class="solr.SynonymFilterFactory" synonyms="synonyms.txt"
      ignoreCase="true" expand="true"/>
    <filter class="solr.LowerCaseFilterFactory"/>
  </analyzer>
</fieldType>
```

上例中的第一行包含了字段类型的名称：`text_general`，以及实现类的名称：`solr.TextField`。
定义的其它部分为字段分析，在 [理解分析器、分词器和过滤器](../../analyzer/readme.md) 中有详述。

实现类负责确保字段被正确处理。
在 `schema.xml` 中的类名中，
字符串 `solr` 是 `org.apache.solr.schema` 或 `org.apache.solr.analysis` 的简写。
因此 `solr.TextField` 实际上是 `org.apache.solr.schema.TextField`。

## 字段类型属性

字段类型的 `class` 决定了字段类型的大部分行为，但还可以定义可选的属性。
例如，以下一个日期字段类型的定义中定义了两个属性 `sortMissingLast` 和 `omitNorms`。

```xml
<fieldType name="date" class="solr.TrieDateField"
  sortMissingLast="true" omitNorms="true"/>
```

可对给定字段类型指定的属性主要分成三类：

* 特定于字段类型类的属性
* Solr 支持的针对任何字段类型的 [通用属性](#general-props)
* 可以在字段类型上指定的 [字段默认属性](#field-default-props)，它们将取代默认行为，
被所有使用该类型的字段所继承

## <a name="general-props"></a>通用属性

|  属性  |   描述    |  值  |
|-------|:---------|------|
| name  | fieldType 的名称。它在字段定义中以 `type`属性使用。强烈推荐名称一致地仅使用字母数字和下划线且不以数字开头。这不是强制的。||
| class | 用于存储和索引该类型的类的名称。注意你可以用 `solr.` 前缀包含的类名且 Solr 会自动找到应该到哪个包中搜索类，因此 `solr.TextField` 能运行。如果你在使用一个第三方类，你可能需要全限定类名。`solr.TextField` 对应的全限定类名为 `org.apache.solr.schema.TextField` ||
| positionIncrementGap | 对多值字段，指定多个值之间的距离，它能避免虚假的短语匹配 | integer |
| autoGeneratePhraseQueries | 针对文本字段。若为 `true`，Solr 会自动针对邻接项生成短语查询。若为 `false`，项必须以双引号括起才能被视作短语 | `true/false` |
| docValuesFormat | 给该类型的字段定义一个自定义 `DocValuesFormat`。这需要一个了解模式的编码器，如在 `solrconfig.xml` 中已配置的 `SchemaCodecFactory` | n/a |
| postingsFormat | 给该类型的字段定义一个自定义 `PostingsFormat`。这需要一个了解模式的编码器，如在 `solrconfig.xml` 中已配置的 `SchemaCodecFactory` | n/a |

> Lucene 索引的后向兼容仅支持默认的编码器。
> 若你选择在 `schema.xml` 中定制 `DocValuesFormat` 或 `PostingsFormat`，
> 升级到未来版本可能需要你或者切换回默认的编码器，或者在升级前优化你的索引并重写为默认编码器，
或者在升级以后从头重新构建整个索引。

## <a name="field-default-props"></a>字段默认属性

|  属性  |   描述    |  值  |
|-------|:---------|------|
| indexed | 若为 true，该字段的值可用在查询中来检索匹配的文档 | `true/false` |
| stored | 若为 true，该字段的实际值可被查询检索出来 | `true/false` |
| docValues | 若为 true，该字段的值可被放入一个面向列的 [DocValues](../docvalues.md) 结构中 | `true/false` |
| sortMissingFirst <br/> sortMissingLast | 控制当排序字段不存在时文档的位置。在 Solr 3.5 中，它们对所有的数值字段都适用，包括 Trie 和日期字段 | `true/false` |
| multiValued | 若为 true，表示单个文档可能包含多个该字段类型的值 | `true/false` |
| omitNorms | 若为 true，忽略该字段关联的范数(它会对该字段禁用长度规范化和索引时加权，并节省部分内存)。默认对所有的原生(非解析)字段类型为true， 如 int、float、data、bool 和 string。仅有全文字段或需要索引时加权的字段需要范数。 | `true/false` |
| omitTermFreqAndPositions | 若为 true，在对该字段 posting时，忽略词频、位置和荷载。它对不需要这些信息的字段能起到性能加权的效果。它也能减少索引所需的存储空间。对该字段发出的依赖位置的查询将默默地查不到文档。该属性对所有非文本字段默认为 true | `true/false` |
| omitPositions | 类似于 `omitTermFreqAndPositions`，但保留了词频信息 | `true/false` |
| termVectors <br/> termPositions <br/> termOffsets <br/> termPayloads | 这些选项指示 Solr 对每个文档维护完整的词频向量，可选地对这些向量中出现的每个项包含位置、位移和荷载信息。这些可用于加速高亮及其他辅助功能，但需要在索引尺寸上引入一些成本。它们对典型的 Solr 使用是不必需的 | `true/false` |
| required | 指示 Solr 拒绝任何添加不包含该字段的值的请求。该属性默认为 false | `true/false` |
