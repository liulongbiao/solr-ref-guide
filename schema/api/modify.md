# 修改模式

`POST /collection/schema`

要添加、删除或替换字段、动态字段规则、拷贝字段规则或新字段类型，
你可以给 `/collection/schema` 终端发生一个带有一系列命令的 POST 请求来执行所请求动作。
它支持以下命令：

* [`add-field`](#add-field) ：以你提供的参数添加一个新的字段
* [`delete-field`](#delete-field) ：删除字段
* [`replace-field`](#replace-field) ：替换已有字段为一个不同配置的字段
* [`add-dynamic-field`](#add-dynamic-field) ：以你提供的参数添加一个新的动态字段规则
* [`delete-dynamic-field`](#delete-dynamic-field) ：删除动态字段规则
* [`replace-dynamic-field`](#replace-dynamic-field) ：替换已有动态字段为一个不同配置的动态字段
* [`add-field-type`](#add-field-type) ：以你提供的参数添加一个新的字段类型
* [`delete-field-type`](#delete-field-type) ：删除字段类型
* [`replace-field-type`](#replace-field-type) ：替换已有字段类型为一个不同配置的字段类型
* [`add-copy-field`](#add-copy-field) ：添加一个新的拷贝字段规则
* [`delete-copy-field`](#delete-copy-field) ：删除拷贝字段规则

这些命令可以在独立的 POST 请求中分别发送，也可以放在一个 POST 请求中。
命令以它们被指定的顺序进行执行。

每次使用时，响应中会包含请求处理的状态和时间，但不会包含整个的模式。

当以该 API 修改模式时，将自动触发 core 的重加载以使其变更对后续待索引文档立即可用。
以前索引的文档将 **不** 会被自动处理 - 它们 **必需** 在其使用的模式元素变更时被重新索引。

## <a name="add-field"></a>添加新字段

`add-field` 命令可给你的模式添加新的字段定义。若已存在同名字段将抛出一个错误。

所有在手动编辑 `schema.xml` 定义字段时可用的属性都可以传给该 API。
这些请求属性在 [定义字段](../defining_fields.md) 中有详细描述。

如，要定义一个名为 “sell-by” 的类型为 “tdate” 的存储字段，你可以 POST 以下请求：

```bash
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "add-field":{
    "name":"sell-by",
    "type":"tdate",
    "stored":true }
}' http://localhost:8983/solr/gettingstarted/schema
``` 

## <a name="delete-field"></a>删除字段

`delete-field` 命令可从模式中删除某个字段定义。
若模式中不存在该字段或该字段是某个拷贝字段规则的来源或目标时，将抛出一个错误。

如，要删除名为 “sell-by” 的字段，你可以 POST 以下请求：

```bash
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "delete-field" : { "name":"sell-by" }
}' http://localhost:8983/solr/gettingstarted/schema
```

## <a name="replace-field"></a>替换字段

`replace-field` 命令可替换某个字段定义。
注意你需要提供该字段的完整定义 - 该命令 **不会** 部分地修改字段定义。
若该字段在模式中不存在，将抛出一个错误。

所有在手动编辑 `schema.xml` 定义字段时可用的属性都可以传给该 API。
这些请求属性在 [定义字段](../defining_fields.md) 中有详细描述。

如，要替换某个已有的名为 "sell-by" 的字段定义，使其类型改为 "date" 并为非存储的，
你可以发送以下 POST 请求：

```bash
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "replace-field":{
    "name":"sell-by",
    "type":"date",
    "stored":false }
}' http://localhost:8983/solr/gettingstarted/schema
```

## <a name="add-dynamic-field"></a>添加动态字段规则

`add-dynamic-field` 命令可在模式中添加新的动态字段规则。

所有在手动编辑 `schema.xml` 定义字段时可用的属性都可以传给该 API。
这些可用于定义在动态字段规则上属性在 [动态字段](../dynamic_fields.md) 中有详细描述。

如，要创建一个新的动态字段规则使得所有输入的以 "_s" 结尾的字段都将是存储的且具有字段类型 "string"，
你可以发送如下 POST 请求：

```bash
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "add-dynamic-field":{
    "name":"*_s",
    "type":"string",
    "stored":true }
}' http://localhost:8983/solr/gettingstarted/schema
```

## <a name="delete-dynamic-field"></a>删除动态字段规则

`delete-dynamic-field` 命令可用于从模式中删除某个动态字段规则。
若该动态字段规则在模式中不存在，或者模式中包含了一条具有仅匹配该动态字段规则的目标的拷贝字段规则，
将抛出一个错误。

如，要删除匹配 "*_s" 的动态字段规则，你可以 POST 以下请求：

```bash
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "delete-dynamic-field":{ "name":"*_s" }
}' http://localhost:8983/solr/gettingstarted/schema
```

## <a name="replace-dynamic-field"></a>替换动态字段规则

`replace-dynamic-field` 命令可用于替换模式中的动态字段规则。
注意你需要提供该动态字段规则的完整定义 - 该命令 **不会** 部分地修改动态字段规则定义。
若该动态字段规则在模式中不存在，将抛出一个错误。

所有在手动编辑 `schema.xml` 定义字段时可用的属性都可以传给该 API。
这些可用于定义在动态字段规则上属性在 [动态字段](../dynamic_fields.md) 中有详细描述。

如，要将 "*_s" 动态字段规则替换为字段类型为 "text_general" 且非存储的规则，
你可以发送以下 POST 请求：

```bash
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "replace-dynamic-field":{
    "name":"*_s",
    "type":"text_general",
    "stored":false }
}' http://localhost:8983/solr/gettingstarted/schema
```

## <a name="add-field-type"></a>添加新字段类型

`add-field-type` 命令可给模式添加新的字段类型。

所有在手动编辑 `schema.xml` 定义字段类型时可用的属性都可以传给该 API。
该命令的结构为一个 标准字段类型定义的 json 映射，包括了 name、class、index 和 query analyzer 定义等。
这些选项在 [Solr 字段类型](../field_type/readme.md) 中有详细描述。

如，要创建一个名为 "myNewTxtField" 的字段类型，你可以 POST 如下请求：

```bash
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "add-field-type" : {
    "name":"myNewTxtField",
    "class":"solr.TextField",
    "positionIncrementGap":"100",
    "analyzer" : {
      "charFilters":[{
        "class":"solr.PatternReplaceCharFilterFactory",
        "replacement":"$1$1",
        "pattern":"([a-zA-Z])\\\\1+" }],
      "tokenizer":{
        "class":"solr.WhitespaceTokenizerFactory" },
      "filters":[{
        "class":"solr.WordDelimiterFilterFactory",
        "preserveOriginal":"0" }]}}
}' http://localhost:8983/solr/gettingstarted/schema
```

注意本例中我们仅定义了单个解析器部分，它将应用于索引分析和查询分析中。
若我们希望定义独立的分析，我们可以替换上述的 `analyzer` 部分
为独立的 `indexAnalyzer` 和 `queryAnalyzer` 两部分，如：

```bash
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "add-field-type":{
    "name":"myNewTextField",
    "class":"solr.TextField",
    "indexAnalyzer":{
      "tokenizer":{
        "class":"solr.PathHierarchyTokenizerFactory",
        "delimiter":"/" }},
    "queryAnalyzer":{
      "tokenizer":{
        "class":"solr.KeywordTokenizerFactory" }}}
}' http://localhost:8983/solr/gettingstarted/schema
```

## <a name="delete-field-type"></a>删除字段类型

`delete-field-type` 命令可从模式中移除某个字段类型。
若该字段类型在模式中不存在，或模式中任何字段或动态字段规则使用了该字段类型，将抛出一个错误。

如，要删除名为 "myNewTxtField" 的字段类型，你可以发送如下 POST 请求：

```bash
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "delete-field-type":{ "name":"myNewTxtField" }
}' http://localhost:8983/solr/gettingstarted/schema
```

## <a name="replace-field-type"></a>替换字段类型

`replace-field-type` 命令可用于替换模式中的字段类型。
注意你需要提供该字段类型的完整定义 - 该命令 **不会** 部分地修改字段类型定义。
若该字段类型在模式中不存在，将抛出一个错误。

所有在手动编辑 `schema.xml` 定义字段类型时可用的属性都可以传给该 API。
该命令的结构为一个 标准字段类型定义的 json 映射，包括了 name、class、index 和 query analyzer 定义等。
这些选项在 [Solr 字段类型](../field_type/readme.md) 中有详细描述。

如，要替换名为 "myNewTxtField" 的字段类型，你可以发送如下 POST 请求：

```bash
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "replace-field-type":{
    "name":"myNewTxtField",
    "class":"solr.TextField",
    "positionIncrementGap":"100",
    "analyzer":{
      "tokenizer":{
        "class":"solr.StandardTokenizerFactory" }}}
}' http://localhost:8983/solr/gettingstarted/schema
```

## <a name="add-copy-field"></a>添加新拷贝字段规则

`add-copy-field` 命令可给模式添加新的拷贝字段规则。

该命令支持的属性和在手动编辑 `schema.xml` 以创建拷贝字段类型规则相同，如下：

|名称     |必需|描述                    |
|source  |是  |源字段                  |
|dest    |是  |源字段将被拷贝至的某个字段或一个字段的数组 |
|maxChars|否  |被拷贝的字符数量上限。小节 [拷贝字段](../copying_fields.md) 中有更多信息 |

如，要定义将字段 "shelf" 拷贝至 "location" 和 "catchall" 字段，可以发送以下 POST 请求:

```bash
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "add-copy-field":{
    "source":"shelf",
    "dest":[ "location", "catchall" ]}
}' http://localhost:8983/solr/gettingstarted/schema
```

## <a name="delete-copy-field"></a>删除拷贝字段规则

`delete-copy-field` 命令可从模式中删除某个拷贝字段规则。
若该拷贝字段规则在模式中不存在，则抛出一个错误。

该命令中 `source` 和 `dest` 属性是必需的。

如，要删除拷贝字段 "shelf" 到 "location" 字段的规则，可以发送以下 POST 请求：

```bash
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "delete-copy-field":{ "source":"shelf", "dest":"location" }
}' http://localhost:8983/solr/gettingstarted/schema
```

## 单个 POST 中的多个命令

可以在单个请求中执行一或多个添加请求。
该 API 是事务性的，所有单次调用的命令要么同时成功或者失败。

这些命令以其被指定的顺序执行。这意味着如果你想创建一个新的字段类型且在相同的请求中
对某个新的字段使用该字段类型，则创建字段类型的请求部分将出现在创建字段的命令之前。
类似的，因为用在拷贝字段规则中的字段必须存在，添加该字段的命令必须在其用作拷贝源或目标
的拷贝字段规则之前。

发起多个命令的请求支持多种方式。
第一种，命令可以简单地是顺序的，如下请求创建了一个新的字段类型，随后的字段中使用了该类型：

```bash
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "add-field-type":{
    "name":"myNewTxtField",
    "class":"solr.TextField",
    "positionIncrementGap":"100",
    "analyzer":{
      "charFilters":[{
        "class":"solr.PatternReplaceCharFilterFactory",
        "replacement":"$1$1",
        "pattern":"([a-zA-Z])\\\\1+" }],
      "tokenizer":{
        "class":"solr.WhitespaceTokenizerFactory" },
      "filters":[{
        "class":"solr.WordDelimiterFilterFactory",
        "preserveOriginal":"0" }]}},
  "add-field" : {
    "name":"sell-by",
    "type":"myNewTxtField",
    "stored":true }
}' http://localhost:8983/solr/gettingstarted/schema
```

或者，相同的命令可以被重复，如下：

```bash
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "add-field":{
    "name":"shelf",
    "type":"myNewTxtField",
    "stored":true },
  "add-field":{
    "name":"location",
    "type":"myNewTxtField",
    "stored":true },
  "add-copy-field":{
    "source":"shelf",
    "dest":[ "location", "catchall" ]}
}' http://localhost:8983/solr/gettingstarted/schema
```

最后，重复的命令可以以数组发送：

```bash
curl -X POST -H 'Content-type:application/json' --data-binary '{
"add-field":[
{ "name":"shelf",
"type":"myNewTxtField",
"stored":true },
{ "name":"location",
"type":"myNewTxtField",
"stored":true }]
}' http://localhost:8983/solr/gettingstarted/schema
```

## 跨副本的模式变更

当运行在 SolrCloud 模式时，对一个节点上模式的变更将会传播给集合的所有副本。
你可以在请求中传递 `updateTimeoutSecs` 参数来设置在所有副本都确认它们应用了模式变更的等待的秒数。
这能帮助你的客户端应用更加健壮，你能够确保所有的副本在给定时间内都有给定的模式变更。
若在指定时间内没有达成所有副本的同意，则请求失败且错误消息中会包含那些副本存在问题的信息。
大多数时候，仅有的选择是在等待简短的时间后进行重试。
若问题还存在，你可能需要检查应用变更存在问题的副本上的服务器日志。
若你没有提供 `updateTimeoutSecs` 参数，默认行为是对接收节点在将变更持久化到 ZooKeeper 后立即返回。
所有其他副本将异步的应用更新。因此，没有提供超时时间时，你的客户端应用不能确保所有副本都应用了该变更。
