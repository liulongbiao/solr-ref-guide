# DocValues

DocValues 是一种针对某些目的，如排序和 faceting，比传统索引更高效的记录字段值的内部方式。

## 为何使用 DocValues？

Solr 构建索引的标准方式是一个 **反向索引**。
这种方式在索引中构建了一个找到的所有词项的列表，紧跟着每个项的是一个该词项所出现的文档的列表
(以及该词项在对应文档中出现的次数)。这使得搜索很快 - 因为用户是根据词项搜索的，
有一个词项到文档的值对列表让查询过程更加快。

对其它和搜索相关的特性，如排序、faceting 和高亮，这种方式就不是很有效。
例如 faceting 引擎必须查看每个文档中的每个词项以设置结果集并拉取文档 ID 以构建 facet 列表。
在 Solr 中，它是维护在内存中的，因此可能会加载很慢(根据文档、词项等的数量)。

在 Lucene 4.0 中，引入了一种新的方式。
DocValues 字段现在是面向列的字段，它有一个索引时构建的文档到值的映射。
这种方式承诺减少了一些对 fieldCache 的内存需求并使得针对 faceting、排序和分组的查找更快。

## 如何使用 DocValues

要使用 DocValues，你只需要对某个想使用它的字段启用即可。
像所有的模式设计一样，你需要定义一个字段类型然后定义该类型的字段启用 docValues。
所有这些操作都在 `schema.xml` 中欧给你完成。

对某个字段启用 docValues 只需要在字段(或字段类型)定义上添加 `docValues="true"`，
如下例采自 Solr `sample_techproducts_configs` 示例 `schema.xml` 中的
[Config Sets](../config/core/config_sets.md)：

```xml
<field name="manu_exact" type="string" indexed="false" stored="false"
  docValues="true" />
```

> 注：若你已经在 Solr 索引中有了索引数据，你需要在改变 `schema.xml` 中的字段定义后
> 完全重新索引你的内容，以成功地使用 docValues。

DocValues 仅针对特定的字段类型有效。具体的类型决定了底层将使用哪种 Lucene docValues 类型。
可用的 Solr 字段类型有：

* `StrField` 和 `UUIDField`
    * 若字段是单值的(即 `multiValued` 是 `false`)，Lucene 将使用 `SORTED` 类型
    * 若字段是多值的，Lucene 将使用 `SORTED_SET` 类型
* 任何 `Trie*` 数值字段和 `EnumField`
    * 若字段是单值的(即 `multiValued` 是 `false`)，Lucene 将使用 `NUMERIC` 类型
    * 若字段是多值的，Lucene 将使用 `SORTED_SET` 类型

这些 Lucene 类型关系到值是如何被排序和存储的。

还有一个额外的配置选项，可用于修改 
[由字段类型所使用的](https://cwiki.apache.org/confluence/display/solr/Field+Type+Definitions+and+Properties#FieldTypeDefinitionsandProperties-docValuesFormat)
`docValuesFormat`。
默认的实现采用了加载某些东西到内存中和保存某些东西在硬盘上的混合。
然而，某些情况下你可以选择指定一个替代的 
[DocValuesFormat 实现](http://lucene.apache.org/core/5_2_0/core/org/apache/lucene/codecs/DocValuesFormat.html)。
例如，你可以通过在字段类型上指定 `docValuesFormat="Memory"` 来将所有东西都保存在内存中：

```xml
<fieldType name="string_in_mem_dv" class="solr.StrField" docValues="true"
  docValuesFormat="Memory" />
```

请注意 `docValuesFormat` 选项在未来版本中可能改变。

> 注：Lucene 索引的后向兼容性仅支持默认的编码器。
> 如果你选择在 schema.xml 中自定义 `docValuesFormat`，
> 升级到 Solr 的未来版本可能需要你切换回默认编码器并优化你的索引以在升级前重写到默认编码器，
> 或者你需要在升级后重新构建整个索引。

