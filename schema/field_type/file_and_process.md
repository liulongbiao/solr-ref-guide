# 处理外部文件和进程

## ExternalFileField 类型

`ExternalFileField` 类型让给字段指定一个处于 Solr 索引之外的文件成为可能。
对这种字段，其文件包含了从键字段到字段值的映射。另一种思考方式是，
取之像其被索引的那样指定文档中的字段， Solr 会在外部文件中找到该字段的值。

> 外部字段是不可搜索的，它们仅用做函数查询或显示。更多函数查询的信息，查看小节
> [函数查询](../../searching/query_syntax/function.md)

`ExternalFileField` 类型对你想比文档中其他部分更频繁地更新多个文档中的特定字段的场景特别有用。
如，假设你基于点击数量实现了文档的排序。你可能希望每天或没小时更新所有文档的排序，
但文档的其他部分的更新频率就没这么高。若没有 `ExternalFileField`，只是为了更新排序你就需要更新每个文档。
使用 `ExternalFileField` 就更高效，因为对某个特定字段所有文档值都存储在外部文件中，
你可以随心所欲地更新。

在 `schema.xml` 中，该字段类型的定义可能如下：

```xml
<fieldType name="entryRankFile" keyField="pkId" defVal="0" stored="false"
  indexed="false" class="solr.ExternalFileField" valType="pfloat"/>
```

`keyField` 属性定义了将定义在外部文件中的键值。它通常是索引的唯一键，但只要 `keyField` 可用作
标识索引中文档即可。`defVal` 定义了当外部文件对特定文档没有项时该使用的值。

`valType` 属性指定了将被在文件中找到的值的实际类型。
该类型必需属于某个浮点字段类型，因此该字段的有效值有 `pfloat`、`float`、`tfloat`。
该属性可被忽略。

## 外部文件格式

文件本身处于 Solr 的索引目录中，默认为 `$SOLR_HOME/data`。
其文件名称应该为 `external_fieldname ` 或 ` external_fieldname.*`。
对上面的示例，该文件应该为 `external_entryRankFile` 或 `external_entryRankFile.txt`。

> 若出现了任何使用了命名模式 `.*`(如 `.txt`)的文件，最后一个 (根据名称排序以后) 将被使用，
> 前面的版本将被删除。这个行为支持那些不能覆盖文件时的实现(例如 Windows 中该文件被使用时)

该文件包含的项将等号左边的键映射到右边的值上。如下：

```
doc33=1.414
doc34=3.14159
doc40=42
```

文件中列出的键不需要时唯一的。该文件不需要是有序的，但如果有序的话  Solr 可以进行快速的查找。

## 重新加载外部文件

可以定义一个事件监听器在搜索器被重新加载或开启了一个新的搜索器时重新加载外部文件。
查看 [查询相关的监听器](https://cwiki.apache.org/confluence/display/solr/Query+Settings+in+SolrConfig#QuerySettingsinSolrConfig-Query-RelatedListeners)
中更多的信息。一个示例定义在 `solrconfig.xml` 中可能看起来如下：

```xml
<listener event="newSearcher"
  class="org.apache.solr.schema.ExternalFileFieldReloader"/>
<listener event="firstSearcher"
  class="org.apache.solr.schema.ExternalFileFieldReloader"/>
```

## 预分析一个字段类型

`PreAnalyzedField` 类型提供了给 Solr 发送序列化分词流的方式，
可选地带有某个字段独立地存储值，并且可以让这个信息是存储的和索引的而不需要额外的在 Solr 中应用文本处理。
这在用户希望提交已经由某些外部已有的文本处理管道(如被分词、注解、提干、同义插入等)处理过的字段内容，
这些管道已经使用了 Lucene 的 TokenStream 所提供的丰富的属性(针对分词的属性)。

这种序列化格式通过使用 `PreAnalyzedParser` 接口的实现可以插入。
存在两种开箱即用的实现：

* [JsonPreAnalyzedParser](http://wiki.apache.org/solr/JsonPreAnalyzedParser) :
如其名所示，它用来解析使用 JSON 来表示字段内容的内容。它是字段类型上没有配置其他解析器时的默认解析器。
* [SimplePreAnalyzedParser](http://wiki.apache.org/solr/SimplePreAnalyzedParser) :
使用简单严格的纯文本格式，它在某些情况下可能比 JSON 容易创建一些。

它仅有一个配置参数 `parserImpl`。该参数的值应该是实现了  `PreAnalyzedParser` 接口的全限定类名。
该参数地默认值为 `org.apache.solr.schema.JsonPreAnalyzedParser`。
