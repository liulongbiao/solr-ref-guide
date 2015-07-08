# JSON 格式索引更新

Solr 可以接收遵循定义结构的 JSON ，或者可以接收任意 JSON 格式的文档。
若要发送任意格式的 JSON，需要给更新请求发送一些额外的参数，如 [转换和索引任意 JSON](#custom) 所述。

## Solr 风格 JSON

JSON 格式更新请求可以使用 `Content-Type: application/json` 或 `Content-Type: text/json .` 发送给 `/update` 处理器。

JSON 格式更新可以有 3 中基本形式，如下：

* [添加单个文档](#single) ，表示为一个顶层 JSON 对象。为和命令的集合相区分，需要设置 `json.command=false` 请求参数。
* [添加文档列表](#list) ，表示为一个顶层 JSON 数组，其中每个文档对应一个 JSON 对象。
* [更新命令序列](#update-seq) ，表示为一个顶层 JSON 对象(即 Map)。

## <a name="single"></a> 添加单个文档

最简单的通过 JSON 添加文档的方式是将每个文档以一个 JSON 对象进行发送，使用 `/update/json/docs` 路径：

```bash
curl -X POST -H 'Content-Type: application/json'
'http://localhost:8983/solr/my_collection/update/json/docs' --data-binary '
{
    "id": "1",
    "title": "Doc 1"
}'
```

## <a name="list"></a> 添加文档列表

通过 JSON 一次添加多个文档可以通过 JSON 对象的 JSON 数组来完成，其中每个对象表示一个文档：

```bash
curl -X POST -H 'Content-Type: application/json'
'http://localhost:8983/solr/my_collection/update' --data-binary '
[
  {
    "id": "1",
    "title": "Doc 1"
  },
  {
    "id": "2",
    "title": "Doc 2"
  }
]'
```

一个示例 JSON 文件提供在 `example/exampledocs/books.json` 且包含了你可以添加到 Solr `techproducts` 示例的对象数组：

```bash
curl 'http://localhost:8983/solr/techproducts/update?commit=true' --data-binary
@example/exampledocs/books.json -H 'Content-type:application/json'
```

## <a name="update-seq"></a> 发送多个 JSON 更新命令

更通用的，JSON 更新语法可以通过直接的映射，来支持所有 XML 更新处理器支持的更新命令。
多个命令，如添加和删除文档，可以包含在单个消息中：

```bash
curl -X POST -H 'Content-Type: application/json'
'http://localhost:8983/solr/my_collection/update' --data-binary '
{
  "add": {
    "doc": {
      "id": "DOC1",
      "my_boosted_field": { /* use a map with boost/value for a boosted field */
        "boost": 2.3,
        "value": "test"
      },
      "my_multivalued_field": [ "aaa", "bbb" ] /* Can use an array for a multi-valued field */
    }
  },
  "add": {
    "commitWithin": 5000, /* commit this document within 5 seconds */
    "overwrite": false, /* don't check for existing documents with the same uniqueKey */
    "boost": 3.45, /* a document boost */
    "doc": {
      "f1": "v1", /* Can use repeated keys for a multi-valued field */
      "f1": "v2"
    }
  },
  "commit": {},
  "optimize": { "waitSearcher":false },
  "delete": { "id":"ID" }, /* delete by ID */
  "delete": { "query":"QUERY" } /* delete by query */
}'
```

> 注：JSON 中不允许注释，但允许重复的名称。
>
> 上述注释仅用于说明目的，在实际发送给 Solr 的命令中需移除。

和其它更新处理器一样，像 `commit`、`commitWithin`、`optimize` 和 `overwrite` 这样的参数必须在 URL 而不是消息体中指定。

JSON 更新格式运行简单的 delete-by-id 。 `delete` 的值可以是待删除的零或多个指定文档 ID (不是范围) 的数组。
如，单个文档如下：

```javascript
{ "delete":"myid" }
```

或者文档 ID 列表：

```javascript
{ "delete":["id1","id2"] }
```

`delete` 的值可以是待删除的零或多个指定文档 ID 的数组。它不是一个范围(起止)。

你也可以对每个  `delete` 指定 `_version_`：

```javascript
{
    "delete":"id":50,
    "_version_":12345
}
```

你也可以在更新请求体中指定删除的版本。

## JSON 更新便捷路径

除了 `/update` 处理器，Solr 中默认还有一些额外的 JSON 特有的请求处理器路径，它隐式地覆盖了某些请求参数的行为：

|路径                    |默认参数                            |
|-----------------------|-----------------------------------|
|/update/json           |stream.contentType=application/json |
|/update/json/docs      |stream.contentType=application/json <nr/>json.command=false|

`/update/json` 路径对难以设置 `Content-Type` 的应用以 JSON 格式发送更新请求非常有用，
而 `/update/json/docs` 对总是希望以文档格式(不管是独立文档还是文档列表)来发送非常方便，而不需要担心完整的 JSON 格式语法。

## <a name="custom"></a> 转换和索引自定义 JSON

若你有一些不想转换成 Solr 结构来进行索引的文档，你可以通过在更新请求中包含一些额外的参数来添加它们。
这些参数提供了如何将单个 JSON 文件分割为多个 Solr 文档以及如何将字段映射到 Solr 模式的信息。
可以用该配置参数来给 `/update/json/docs` 发送一或多个有效的 JSON 文档。

### 映射参数

这些参数让你可以定义 JSON 文件应该如何被读为多个 Solr 文档。

* `split` ：定义在哪个路径将输入 JSON 分割为多个 Solr 文档，且如果你在单个文件中有多个文档时是必需的。
若整个 JSON 对应单个 Solr 文档，其路径必须为 `/`。
* `f` : 它是一个多值映射参数。必须提供至少一个字段映射。其格式为 `target-field-name:json-path`。 其中 `json-path` 是必需的。
`target-field-name` 是 Solr 文档字段名，且是可选的。若没有指定，它自动派生自输入的 JSON。
这里可以使用通配符，可查看以下 [通配符](#wildcards) 相关信息。
* `mapUniqueKeyOnly` (boolean) :该参数在输入 JSON 中的字段在模式中不存在且 [无模式形式](../../schema/schemaless_mode.md)
没有启用时非常方便。它将索引所有的字段到默认搜索字段 (使用以下 `df` 参数) 中，且仅有 `uniqueKey` 字段被映射到模式中相应字段。
若输入 JSON 对 `uniqueKey` 字段不具有值，则该文档会生成一个 UUID 值。
* `df` : 若使用了 `mapUniqueKeyOnly` 参数，更新处理器需要一个数据应该被索引至的字段。
这也是其它处理器用作默认搜索字段的相同的字段。
* `srcField` : 这是 JSON source 将会被存储到的字段的名称。它只能使用在 `split=/` 的情况
(即你希望 JSON 输入文件被索引为单个 Solr 文档)。注意自动更新将导致该字段和该文档不再同步。
* `echo` : 它仅用于调试目的。设置为 true，这些文档将被返回为响应。没有东西会被索引。

如，若你有包含两个文档的 JSON 文件，我们可以定义一个更新请求如下。

```
curl 'http://localhost:8983/solr/my_collection/update/json/docs'\
'?split=/exams'\
'&f=first:/first'\
'&f=last:/last'\
'&f=grade:/grade'\
'&f=subject:/exams/subject'\
'&f=test:/exams/test'\
'&f=marks:/exams/marks'\
-H 'Content-type:application/json' -d '
{
    "first": "John",
    "last": "Doe",
    "grade": 8,
    "exams": [
        {
        "subject": "Maths",
        "test": "term1",
        "marks" : 90},
        {
        "subject": "Biology",
        "test": "term1",
        "marks" : 86}
    ]
}'
```

对这个请求，我们定义了包含多个文档的 "exams"。另外我们将多个字段从输入文档映射为 Solr 字段。

当索引请求完成时，以下两个文档会被添加到索引中：

```javascript
{
    "first":"John",
    "last":"Doe",
    "marks":90,
    "test":"term1",
    "subject":"Maths",
    "grade":8
}
{
    "first":"John",
    "last":"Doe",
    "marks":86,
    "test":"term1",
    "subject":"Biology",
    "grade":8
}
```

上面的示例中，所有字段我们都想用在 Solr 中，且具有和输入 JSON 相同的字段名。
这种情况下我们可以简写为：

```
curl 'http://localhost:8983/solr/my_collection/update/json/docs'\
'?split=/exams'\
'&f=/first'\
'&f=/last'\
'&f=/grade'\
'&f=/exams/subject'\
'&f=/exams/test'\
'&f=/exams/marks'\
-H 'Content-type:application/json' -d '
{
    "first": "John",
    "last": "Doe",
    "grade": 8,
    "exams": [
        {
        "subject": "Maths",
        "test"
        : "term1",
        "marks" : 90},
        {
        "subject": "Biology",
        "test"
        : "term1",
        "marks" : 86}
    ]
}'
```

这里我们简单地命名了字段路径(如 `/exams/test`) 。Solr 将自动尝试将 JSON 输入中该字段的内容添加到索引中具有相同名称的字段。

> 注：若你没有运行在 [无模式形式](../../schema/schemaless_mode.md) 下，其中不存在的字段会通过 Solr 对字段类型的猜测动态创建，
> 这时若索引前对应字段不存在的话文档可能会被拒绝。

### <a name="wildcards"></a> 通配符

取之明确地指定所有字段的名称，也可以指定通配符来自动映射字段。
它有两个限制：通配符只能用在 `json-path` 的末尾，且 `split` 路径不能使用通配符。
单个星号 `*` 仅映射直接子节点，而双个星号 `**` 可递归地映射到所有后代节点。
以下是通配符路径映射的示例：

* `f=/docs/*` : 映射所有 `docs` 下面的字段到 JSON 中相应的名称
* `f=/docs/**` : 映射所有 `docs` 下面的字段及其子字段到 JSON 中相应的名称
* `f=searchField:/docs/*` : 映射所有 `docs` 下面的字段到名为 `searchField` 的单个字段
* `f=searchField:/docs/**` : 映射所有 `docs` 下面的字段及其子字段到名为 `searchField` 的单个字段
* `f=$FQN:/**` : 映射所有字段到其 JSON 字段的全限定名 (`$FQN`) 。全限定名由层级中的所有键以点号 (`.`) 连接而成。

> 注：从 Solr 5.0 开始，`f` 的默认值为 `$FQN:/**`。它在 4.10.x 中曾经为 `/**`。这是后向不兼容的。
> 如果你希望具有旧版的行为，可以明确地指定 `f=/**`。

用通配符我们可以进一步简化上述示例：

```
curl 'http://localhost:8983/solr/my_collection/update/json/docs'\
'?split=/exams'\
'&f=/**'\
-H 'Content-type:application/json' -d '
{
    "first": "John",
    "last": "Doe",
    "grade": 8,
    "exams": [
        {
        "subject": "Maths",
        "test"
        : "term1",
        "marks" : 90},
        {
        "subject": "Biology",
        "test"
        : "term1",
        "marks" : 86}
    ]
}'
```

因为我们希望字段被索引为和 JSON 输入中相同名称的字段，`f=/**` 中的双通配符将映射所有字段及其后代字段为 Solr 中同名字段。

它也可以发送所有值到单个字段并在其上做全文搜索。
这是一个盲目索引和查询 JSON 文档的良好选项，而不用担心字段和模式。

```
curl 'http://localhost:8983/solr/my_collection/update/json/docs'\
'?split=/'\
'&f=txt:/**'\
-H 'Content-type:application/json' -d '
{
    "first": "John",
    "last": "Doe",
    "grade": 8,
    "exams": [
        {
        "subject": "Maths",
        "test"
        : "term1",
        "marks" : 90},
        {
        "subject": "Biology",
        "test"
        : "term1",
        "marks" : 86}
    ]
}'
```

上例中我们表示所有字段应该被添加到 Solr 中名为 `txt` 的字段。
这会将多个字段添加到单个字段中，因此你所选择的字段应该是多值的。

默认行为是使用节点的全限定名(FQN)。因此若我们没有定义任何字段映射，如：

```
curl 'http://localhost:8983/solr/my_collection/update/json/docs?split=/exams'\
-H 'Content-type:application/json' -d '
{
    "first": "John",
    "last": "Doe",
    "grade": 8,
    "exams": [
        {
        "subject": "Maths",
        "test"
        : "term1",
        "marks" : 90},
        {
        "subject": "Biology",
        "test"
        : "term1",
        "marks" : 86}
    ]
}'
```

被添加到索引中的文档如下：

```javascript
{
"first":"John",
"last":"Doe",
"grade":8,
"exams.subject":"Maths",
"exams.test":"term1",
"exams.marks":90},
{
"first":"John",
"last":"Doe",
"grade":8,
"exams.subject":"Biology",
"exams.test":"term1",
"exams.marks":86}
```

### 设置 JSON 默认值

可以给 `/update/json/docs` 终端发送任何 json，且该组件默认配置如下：

```xml
<initParams path="/update/json/docs">
  <lst name="defaults">
    <!-- this ensures that the entire json doc will be stored verbatim into one field -->
    <str name="srcField">_src_</str>
    <!-- This means a the uniqueKeyField will be extracted from the fields and
    all fields go into the 'df' field. In this config df is already configured to
    be 'text'
    -->
    <str name="mapUniqueKeyOnly">true</str>
    <!-- The default search field where all the values are indexed to
    -->
    <str name="df">text</str>
  </lst>
</initParams>
```

因此若没有传入参数，整个 JSON 文件会被索引到 `_src_` 字段中，且输入 JSON 中的所有值会映射到名为 `text` 的字段。
若存在存储的唯一键且从输入 JSON 中获取不到对应的值，则会生成一个 UUID 作为对应唯一键的字段值。
