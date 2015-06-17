# 其他模式元素

本节描述 `schema.xml` 中一些其他重要的元素。

## 唯一键

`uniqueKey` 元素指定了哪个字段是文档的唯一标识。
尽管 `uniqueKey` 不是必需的，但它在你的应用设计时几乎总是保证的。
例如，在你想更新索引中的文档时就需要使用 `uniqueKey`。

你可以通过名称来定义唯一键字段。

```xml
<uniqueKey>id</uniqueKey>
```

模式的默认字段和 `copyField` 字段不可用作填充 `uniqueKey` 字段。
你也不能使用 `UUIDUpdateProcessorFactory` 来自动生成 `uniqueKey` 的值。

更进一步，当 `uniqueKey` 字段被使用，但字段本身是多值的(或从 `fieldType` 继承了多值性)时，
操作会失败。
然而，`uniqueKey` 还能继续工作，只要字段是被适当地使用的。

## 默认搜索字段

若你在使用 Lucene 查询解析器，没有指定字段名称的查询将使用默认的搜索字段。
DisMax 和扩展的 DisMax 查询解析器在 `qf` 没有指定时，也会退化到默认搜索字段。

> 警告：`defaultSearchField` 元素的使用在 Solr 3.6+ 版本被废弃。
> 相反你应该使用 `df` 请求参数。某些程度上， `defaultSearchField` 元素应该被移除。

更多查询解析器的信息，查看小节 [查询语法和解析](../searching/query_syntax/readme.md)

## 查询解析器默认操作符

在有多个项的查询中， Solr 可以返回所有条件都匹配的结果或者某个或多个条件匹配的结果。
`operator` 控制了这个行为。
一个 `AND` operator 意味着所有的条件都必须满足，而一个 `OR` operator 意味着一或多个条件需为真。

在 `schema.xml` 中，`solrQueryParser` 元素控制了如果查询中没有制定操作符时应该使用哪个操作符。
默认的操作符设置仅对 Lucene 查询解析器有效，
而 DisMax 和扩展的 DisMax 查询解析器在其内部硬编码了它们的操作符为 `OR`。

> 警告：查询解析器默认操作符参数在 Solr 3.6+ 版本被废弃。
> 相反推荐你应该在你的请求处理器中指定查询解析器 `q.op` 参数。

## 相似度

Similarity 是用于在搜索中给文档评分的 Lucene 类。

你可以声明一个全局的 `<similarity>` 用来指定一个你希望 Solr 在处理你的索引时使用的自定义的相似度实现。
一个 `similarity` 可以直接通过引用不带构造器参数的类的名称来指定：

```xml
<similarity class="solr.DefaultSimilarityFactory"/>
```

也可以引用一个带有可选初始化参数的 `SimilarityFactory` 实现：

```xml
<similarity class="solr.DFRSimilarityFactory">
  <str name="basicModel">P</str>
  <str name="afterEffect">L</str>
  <str name="normalization">H2</str>
  <float name="c">7</float>
</similarity>
```

还有一个特殊的 `SchemaSimilarityFactory` 实现，它让独立的字段类型可以配置一个覆盖了默认行为的相似度：

```xml
<similarity class="solr.SchemaSimilarityFactory"/>
<fieldType name="text_ib">
  <analyzer/>
  <similarity class="solr.IBSimilarityFactory">
    <str name="distribution">SPL</str>
    <str name="lambda">DF</str>
    <str name="normalization">H2</str>
  </similarity>
</fieldType>
```

若字段类型没有明确配置相似度工厂，则会使用一个 `DefaultSimilarity` 实例。

上例显示了 `DFRSimilarityFactory` (随机分歧) 和 `IBSimilarityFactory` (使用基于信息的模型)，
但还有很多中相似度实现可用，诸如 `SweetSpotSimilarityFactory`、`BM25SimilarityFactory` 等。
详见 Solr javadoc 中 [相似度工厂](http://lucene.apache.org/solr/5_2_0/solr-core/org/apache/solr/search/similarities/package-summary.html)。

## 相关内容

* [SchemaXML-Miscellaneous Settings](http://wiki.apache.org/solr/SchemaXml#Miscellaneous_Settings)
* [UniqueKey](http://wiki.apache.org/solr/UniqueKey)
