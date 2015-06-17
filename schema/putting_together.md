# 放到一起

在更高一层， `schema.xml` 有如下结构。
它不是真实的 XML，但让你对文件结构有一个印象。

```xml
<schema>
  <types>
  <fields>
  <uniqueKey>
  <copyField>
</schema>
```

显然，大多数信息在 `types` 和 `fields` 中，它们是字段类型和实际字段定义所处的地方。
`copyFields` 对它们做了补充。`uniqueKey` 总是会被定义。
在旧的 Solr 版本中，你也会找到 `defaultSearchField` 和 `solrQueryParser` 标签，
但尽管它们还能用，它们是被废弃且不推荐使用的，详见 [其他模式元素](./other_schema_elements.md)。

> 注： **`types` 和 `fields` 是可选标签**
>
> 注意 `types` 和 `fields` 是可选的，即你可以混合 `field`、`dynamiField`、`copyField`和`fieldType`
> 作为顶级元素。这让你可以在模式中将相关的标签定义为更有逻辑的分组。

## 选择合适的数值类型

对常规的数值需求，使用带 `precisionStep="0"` 的 
`TrieIntField`、`TrieLongField`、`TrieFloatField`和 `TrieDoubleField`。

如果你知道用户会频繁地在数值类型上进行范围查询，使用默认的 `precisionStep` (但不指定它)
或者指定 `precisionStep="8"`(它是默认的)。
这为范围查询提供更快的速度，但会增加索引大小。

## 处理文本

正确地处理文本，对文本搜素提供给你的用户更好的结果，会让他们更愉快。

一种技术是使用一个文本字段用作所有关键字搜索。
大多数用户对他们的搜索都不精明，且很多常见的搜索只是简单的关键字搜索。
你可以使用 `copyField` 接收多个字段并将它们汇总到单个文本字段用于搜索。
在 Solr 的 `techproducts` 示例的 schema.xml 中，多个 `copyField` 声明
将 `cat`、`name`、`manu`、`features`及`includes`的内容汇总到单个字段， `text`。
另外将 `ID` 也拷贝到 `text` 字段也很有用，比如说用户可能想搜索某个产品，
然后直接将产品标识作为关键字搜索传入。

另一个技术是使用 `copyField` 将相同的字段进行不同的处理。
假设你有一个字段为一个作者的列表，如：

```
Schildt, Herbert; Wolpert, Lewis; Davies, P.
```

对根据作者搜索，你可以将该字段分词，转换为小写并移除符号：

```
schildt / herbert / wolpert / lewis / davies / p
```

对于排序，就使用一个未分词的字段，转换为小写并移除符号：

```
schildt herbert wolpert lewis davies p
```

最后，对 faceting，仅通过一个 `StringField` 字段使用主作者：

```
Schildt, Herbert
```

## 相关内容

* [SchemaXML](http://wiki.apache.org/solr/SchemaXml)
