# Solr内置的字段类型

以下表格列出了 Solr 中可用的字段类型。
`org.apache.solr.schema` 包中包含了所有一下列出的类。

|   类   |       描述       |
|:-------|:----------------|
|BinaryField| 二进制数据     |
|BoolField| 包含true或false。第一个字符为`1`、`t`或`T`的值被解释为true，其他首字符的值被解释为false|
|CollationField| 支持 Unicode 校点来排序和范围查询。如果你可以使用 ICU4J, `ICUCollationField`是一个更好的选择。见 [Unicode Collation](https://cwiki.apache.org/confluence/display/solr/Language+Analysis#LanguageAnalysis-UnicodeCollation) |
|CurrencyField| 支持货币和汇率。见小节 [处理货币和汇率](./currency.md) |
|DateRangeField| 支持索引日期范围，也包括时间日期实例的点(单个毫秒时间段)。见小节 [处理日期](./date.md)。即使对日期实例也推荐使用这个，特别是当查询通常退化为 UTC 年/月/日/时 等范围时 |
|ExternalFileField| 从硬盘上的文件中拉取值。见小节 [处理外部文件和进程](./file_and_process.md) |
|EnumField| 允许定义一个值的枚举集，它们可能不能很容易的通过字母序或者数值序进行排序(如严重程度的列表)。该字段类型接收一个配置文件，其中列出了其字段值的合适的顺序。见小节 [处理枚举字段](./enum.md) 中更多信息 |
|ICUCollationField| 支持 Unicode 校点来排序和范围查询。见 [Unicode Collation](https://cwiki.apache.org/confluence/display/solr/Language+Analysis#LanguageAnalysis-UnicodeCollation) |
|LatLonType| [空间搜索](../../searching/spatial_search.md)：一个纬度/经度坐标对。纬度在前。 |
|PointType| [空间搜索](../../searching/spatial_search.md)：一个任意的 n 维点，适用于诸如蓝图或 CAD 图这样的搜索源 |
|PreAnalyzedField| Provides a way to send to Solr serialized token streams, optionally with independent stored values of a field, and have this information stored and indexed without any additional text processing. Useful if you want to submit field content that was already processed by some existing external text processing pipeline (e.g. tokenized, annotated, stemmed, inserted synonyms, etc.), while using all the rich attributes that Lucene's TokenStream provides via token attributes. |
|RandomSortField| 不包含值。在该字段类型上排序的查询将以随机顺序返回结果。需在动态字段上使用该特性 |
|SpatialRecursivePrefixTreeFieldType| (简写为RPT)[空间搜索](../../searching/spatial_search.md)：接收纬度逗号经度字符串或其他 WKT 格式的形状 |
|StrField| 字符串(UTF-8编码的字符串或 Unicode) |
|TextField| 文本，通常有多个单词或符号 |
|TrieDateField| 日期字段。表示毫秒精度的时间中的一个点。见小节 [处理日期](./date.md)。`precisionStep="0"` 使得能够高效的日期排序且最小化索引大小;`precisionStep="8"`(默认值)使得能够进行高效的范围查询。 |
|TrieDoubleField| Double字段(64位IEEE浮点值)。`precisionStep="0"` 使得能够高效的数值排序且最小化索引大小;`precisionStep="8"`(默认值)使得能够进行高效的范围查询。 |
|TrieField| 若该字段类型被使用，一个 `type` 属性也必须被指定，有效值为 `integer , long , float , double , date`。使用该字段和使用任何对应的 Trie 字段一样。`precisionStep="0"` 使得能够高效的数值排序且最小化索引大小;`precisionStep="8"`(默认值)使得能够进行高效的范围查询。 |
|TrieFloatField| Float字段(32位IEEE浮点值)。`precisionStep="0"` 使得能够高效的数值排序且最小化索引大小;`precisionStep="8"`(默认值)使得能够进行高效的范围查询。 |
|TrieIntField| Integer字段(32位有符号证书)。`precisionStep="0"` 使得能够高效的数值排序且最小化索引大小;`precisionStep="8"`(默认值)使得能够进行高效的范围查询。 |
|TrieLongField| Long字段(64位有符号整数)。`precisionStep="0"` 使得能够高效的数值排序且最小化索引大小;`precisionStep="8"`(默认值)使得能够进行高效的范围查询。 |
|UUIDField| 宇宙唯一标识(UUID)。传入一个值为 `NEW` 且 Solr 将创建一个新的 UUID。**注**：在使用 SolrCloud 时，配置一个默认值为 `NEW` 的 UUIDField 实例对大多数用户都是不推荐的(且当该 UUID 值被配置为唯一键字段时也是不可能的)，因为其结果将是文档的每个副本各自都有一个唯一的 UUID 值。相反，推荐在文档被添加时使用 `UUIDUpdateProcessorFactory` 来生成 UUID 值。 |

`MultiTermAwareComponent` 被添加到了 `schema.xml` 中相关的 `solr.TextField` 项中
(如 wildcards、 regex、 prefix、 range 等)以允许对多词项查询自动小写化。

更进一步，你可以选择在你的模式中指定一个多词项分析其 `analyzer type="multiterm">`，
若你没有这么做， `analyzer` 将根据它们特定的属性处理字段。

