# 检索模式信息

以下终端允许你读取模式如何定义的信息。你可以 GET 整个模式或需要的部分。

要修改模式，查看前一小节 [修改模式](./modify.md)。

## 检索整个模式

```
GET /collection/schema
```

### 输入

路径变量

|键         |描述                     |
|-----------|------------------------|
|collection |集合(或 core)的名称       |

查询参数

查询参数应该在 API 请求的 '?' 后添加

|键  |类型   |必需 |默认  |描述                       |
|----|------|----|-----|---------------------------|
|wt  |string|No  |json |定义响应格式。可选项有 `json`、`xml` 或 `schema.xml`。若没有指定，将默认返回 JSON。|

### 输出

输出内容

输出中将根据请求格式(JSON 或 xml)，包含所有字段、字段类型、动态规则和拷贝字段规则。
模式名称和版本也会被包含。

### 示例

以 JSON 获取整个模式

```bash
curl http://localhost:8983/solr/gettingstarted/schema?wt=json
```

```javascript
{
"responseHeader":{
  "status":0,
  "QTime":5},
"schema":{
  "name":"example",
  "version":1.5,
  "uniqueKey":"id",
  "fieldTypes":[{
    "name":"alphaOnlySort",
    "class":"solr.TextField",
    "sortMissingLast":true,
    "omitNorms":true,
    "analyzer":{
      "tokenizer":{
        "class":"solr.KeywordTokenizerFactory"},
      "filters":[{
        "class":"solr.LowerCaseFilterFactory"},
        {
        "class":"solr.TrimFilterFactory"},
        {
        "class":"solr.PatternReplaceFilterFactory",
        "replace":"all",
        "replacement":"",
        "pattern":"([^a-z])"}]}},
...
  "fields":[{
    "name":"_version_",
    "type":"long",
    "indexed":true,
    "stored":true},
    {
    "name":"author",
    "type":"text_general",
    "indexed":true,
    "stored":true},
    {
    "name":"cat",
    "type":"string",
    "multiValued":true,
    "indexed":true,
    "stored":true},
...
  "copyFields":[{
    "source":"author",
    "dest":"text"},
    {
    "source":"cat",
    "dest":"text"},
    {
    "source":"content",
    "dest":"text"},
...
    {
    "source":"author",
    "dest":"author_s"}]}}
```

以 XML 格式获取整个模式

```bash
curl http://localhost:8983/solr/gettingstarted/schema?wt=xml
```

```xml
<response>
<lst name="responseHeader">
  <int name="status">0</int>
  <int name="QTime">5</int>
</lst>
<lst name="schema">
  <str name="name">example</str>
  <float name="version">1.5</float>
  <str name="uniqueKey">id</str>
  <arr name="fieldTypes">
    <lst>
      <str name="name">alphaOnlySort</str>
      <str name="class">solr.TextField</str>
      <bool name="sortMissingLast">true</bool>
      <bool name="omitNorms">true</bool>
      <lst name="analyzer">
        <lst name="tokenizer">
          <str name="class">solr.KeywordTokenizerFactory</str>
        </lst>
        <arr name="filters">
          <lst>
            <str name="class">solr.LowerCaseFilterFactory</str>
          </lst>
          <lst>
            <str name="class">solr.TrimFilterFactory</str>
          </lst>
          <lst>
            <str name="class">solr.PatternReplaceFilterFactory</str>
            <str name="replace">all</str>
            <str name="replacement"/>
            <str name="pattern">([^a-z])</str>
          </lst>
        </arr>
      </lst>
    </lst>
...
    <lst>
      <str name="source">author</str>
      <str name="dest">author_s</str>
    </lst>
  </arr>
</lst>
</response>
```

以 "schema.xml" 格式获取整个模式

```bash
curl http://localhost:8983/solr/gettingstarted/schema?wt=schema.xml
```

```xml
<schema name="example" version="1.5">
  <uniqueKey>id</uniqueKey>
  <types>
    <fieldType name="alphaOnlySort" class="solr.TextField" sortMissingLast="true"
      omitNorms="true">
      <analyzer>
        <tokenizer class="solr.KeywordTokenizerFactory"/>
        <filter class="solr.LowerCaseFilterFactory"/>
        <filter class="solr.TrimFilterFactory"/>
        <filter class="solr.PatternReplaceFilterFactory" replace="all"
          replacement="" pattern="([^a-z])"/>
      </analyzer>
    </fieldType>
...
  <copyField source="url" dest="text"/>
  <copyField source="price" dest="price_c"/>
  <copyField source="author" dest="author_s"/>
</schema>
```

## 列出字段

```
GET /collection/schema/fields

GET /collection/schema/fields/fieldname
```

### 输入

路径变量

|键         |描述                     |
|-----------|------------------------|
|collection |集合(或 core)的名称       |
|fieldname  |特定字段名称(若限制仅请求单个字段)|

查询参数

查询参数应该在 API 请求的 '?' 后添加

|键  |类型   |必需 |默认  |描述                       |
|----|------|----|-----|---------------------------|
|wt  |string|No  |json |定义响应格式。可选项有 `json`、`xml` 或 `schema.xml`。若没有指定，将默认返回 JSON。|

### 输出

输出内容

输出中将包含每个字段和针对每个字段的任何定义的配置。
对每个字段的定义的配置可能有差别，但至少会包含字段 `name` 及其 `type`、其是否 `indexed` 及是否 `stored`。
若 `multiValued` 被定义为 true 或 false (通常为 true)，它也将显示。
查看 [定义字段](../defining_fields.md) 中每个参数的更多信息。

### 示例

获取所有字段的列表

```bash
curl http://localhost:8983/solr/gettingstarted/schema/fields?wt=json
```

以下示例输出被截取仅显示其中一部分

```javascript
{
  "fields": [
    {
    "indexed": true,
    "name": "_version_",
    "stored": true,
    "type": "long"
    },
    {
    "indexed": true,
    "name": "author",
    "stored": true,
    "type": "text_general"
    },
    {
    "indexed": true,
    "multiValued": true,
    "name": "cat",
    "stored": true,
    "type": "string"
    },
...
  ],
  "responseHeader": {
    "QTime": 1,
    "status": 0
  }
}
```

## 列出动态字段

```
GET /collection/schema/dynamicfields

GET /collection/schema/dynamicfields/name
```

### 输入

路径变量

|键         |描述                     |
|-----------|------------------------|
|collection |集合(或 core)的名称       |
|name       |动态字段规则名称(若限制仅请求单个动态字段规则)|

查询参数

查询参数应该在 API 请求的 '?' 后添加

|键  |类型   |必需 |默认  |描述                       |
|----|------|----|-----|---------------------------|
|wt  |string|No  |json |定义响应格式。可选项有 `json`、`xml` 或 `schema.xml`。若没有指定，将默认返回 JSON。|

### 输出

输出内容

输出中将包含每个动态字段规则和针对每个动态字段规则的任何定义的配置。
对每个动态字段规则的定义的配置可能有差别，但至少会包含字段 `name` 及其 `type`、其是否 `indexed` 及是否 `stored`。
查看 [动态字段](../dynamic_fields.md) 中每个参数的更多信息。

### 示例

获取所有动态字段声明的列表：

```bash
curl http://localhost:8983/solr/gettingstarted/schema/dynamicfields?wt=json
```

以下是被截取的示例输出：

```javascript
{
  "dynamicFields": [
    {
    "indexed": true,
    "name": "*_coordinate",
    "stored": false,
    "type": "tdouble"
    },
    {
    "multiValued": true,
    "name": "ignored_*",
    "type": "ignored"
    },
    {
    "name": "random_*",
    "type": "random"
    },
    {
    "indexed": true,
    "multiValued": true,
    "name": "attr_*",
    "stored": true,
    "type": "text_general"
    },
    {
    "indexed": true,
    "multiValued": true,
    "name": "*_txt",
    "stored": true,
    "type": "text_general"
    }
...
  ],
  "responseHeader": {
    "QTime": 1,
    "status": 0
  }
}
```

## 列出字段类型

```
GET /collection/schema/fieldtypes

GET /collection/schema/fieldtypes/name
```

### 输入

路径变量

|键         |描述                     |
|-----------|------------------------|
|collection |集合(或 core)的名称       |
|name       |字段类型名称(若限制仅请求单个字段类型)|

查询参数

查询参数应该在 API 请求的 '?' 后添加

|键  |类型   |必需 |默认  |描述                       |
|----|------|----|-----|---------------------------|
|wt  |string|No  |json |定义响应格式。可选项有 `json`、`xml` 或 `schema.xml`。若没有指定，将默认返回 JSON。|

### 输出

输出内容

输出中将包含每个字段类型和针对每个字段类型的任何定义的配置。
对每个字段类型的定义的配置可能有差别，但至少会包含字段 `name` 及其 `class`。
若定义了查询或索引分析器、分词器或过滤器，它们将和其它定义的参数一起显示。
查看 [Solr 字段类型](../field_type/readme.md) 中关于如何配置多种字段类型的更多信息。

### 示例

获取所有字段类型的列表

```bash
curl http://localhost:8983/solr/gettingstarted/schema/fieldtypes?wt=json
```

以下被截取的示例输出显示了列表中不同部分的一些不同的字段类型。

```javascript
{
  "fieldTypes": [
    {
    "analyzer": {
      "class": "solr.TokenizerChain",
      "filters": [
        {
        "class": "solr.LowerCaseFilterFactory"
        },
        {
        "class": "solr.TrimFilterFactory"
        },
        {
        "class": "solr.PatternReplaceFilterFactory",
        "pattern": "([^a-z])",
        "replace": "all",
        "replacement": ""
        }
      ],
      "tokenizer": {
        "class": "solr.KeywordTokenizerFactory"
      }
    },
    "class": "solr.TextField",
    "dynamicFields": [],
    "fields": [],
    "name": "alphaOnlySort",
    "omitNorms": true,
    "sortMissingLast": true
  },
...
  {
    "class": "solr.TrieFloatField",
    "dynamicFields": [
      "*_fs",
      "*_f"
    ],
    "fields": [
      "price",
      "weight"
    ],
    "name": "float",
    "positionIncrementGap": "0",
    "precisionStep": "0"
  },
...
}
```

## 列出拷贝字段

```
GET /collection/schema/copyfields
```

### 输入

路径变量

|键         |描述                     |
|-----------|------------------------|
|collection |集合(或 core)的名称       |

查询参数

查询参数应该在 API 请求的 '?' 后添加

|键  |类型   |必需 |默认  |描述                       |
|----|------|----|-----|---------------------------|
|wt  |string|No  |json |定义响应格式。可选项有 `json`、`xml` 或 `schema.xml`。若没有指定，将默认返回 JSON。|

### 输出

输出内容

输出中将包含定义在 `schema.xml` 中每个拷贝字段规则的 `source` 和 `dest`。
更多有关拷贝字段的信息，查看 [拷贝字段](../copying_fields.md)

### 示例

获取所有拷贝字段的列表

```bash
curl http://localhost:8983/solr/gettingstarted/schema/copyfields?wt=json
```

以下示例输出被截取仅显示部分拷贝定义

```javascript
{
  "copyFields": [
    {
    "dest": "text",
    "source": "author"
    },
    {
    "dest": "text",
    "source": "cat"
    },
    {
    "dest": "text",
    "source": "content"
    },
    {
    "dest": "text",
    "source": "content_type"
    },
...
  ],
  "responseHeader": {
    "QTime": 3,
    "status": 0
  }
}
```

## 显示模式名称

```
GET /collection/schema/name
```

### 输入

路径变量

|键         |描述                     |
|-----------|------------------------|
|collection |集合(或 core)的名称       |

查询参数

查询参数应该在 API 请求的 '?' 后添加

|键  |类型   |必需 |默认  |描述                       |
|----|------|----|-----|---------------------------|
|wt  |string|No  |json |定义响应格式。可选项有 `json`、`xml` 或 `schema.xml`。若没有指定，将默认返回 JSON。|

### 输出

输出内容

输出中将简单地显示模式的名称

### 示例

获取模式名称

```bash
curl http://localhost:8983/solr/gettingstarted/schema/name?wt=json
```

```javascript
{
  "responseHeader":{
    "status":0,
    "QTime":1},
  "name":"example"}
```

## 显示模式版本

```
GET /collection/schema/version
```

### 输入

路径变量

|键         |描述                     |
|-----------|------------------------|
|collection |集合(或 core)的名称       |

查询参数

查询参数应该在 API 请求的 '?' 后添加

|键  |类型   |必需 |默认  |描述                       |
|----|------|----|-----|---------------------------|
|wt  |string|No  |json |定义响应格式。可选项有 `json`、`xml` 或 `schema.xml`。若没有指定，将默认返回 JSON。|

### 输出

输出内容

输出中将简单地显示使用的模式的版本号

### 示例

获取模式版本

```bash
curl http://localhost:8983/solr/gettingstarted/schema/version?wt=json
```

```javascript
{
  "responseHeader":{
    "status":0,
    "QTime":2},
  "version":1.5}
```

## 列出唯一键

```
GET /collection/schema/uniquekey
```

### 输入

路径变量

|键         |描述                     |
|-----------|------------------------|
|collection |集合(或 core)的名称       |

查询参数

查询参数应该在 API 请求的 '?' 后添加

|键  |类型   |必需 |默认  |描述                       |
|----|------|----|-----|---------------------------|
|wt  |string|No  |json |定义响应格式。可选项有 `json`、`xml` 或 `schema.xml`。若没有指定，将默认返回 JSON。|

### 输出

输出内容

输出中仅简单包含用作索引唯一键的字段的名称

### 示例

列出唯一键

```bash
curl http://localhost:8983/solr/gettingstarted/schema/uniquekey?wt=json
```

```javascript
{
"responseHeader":{
  "status":0,
  "QTime":2},
"uniqueKey":"id"}
```

## 显示全局相似度

```
GET /collection/schema/similarity
```

### 输入

路径变量

|键         |描述                     |
|-----------|------------------------|
|collection |集合(或 core)的名称       |

查询参数

查询参数应该在 API 请求的 '?' 后添加

|键  |类型   |必需 |默认  |描述                       |
|----|------|----|-----|---------------------------|
|wt  |string|No  |json |定义响应格式。可选项有 `json`、`xml` 或 `schema.xml`。若没有指定，将默认返回 JSON。|

### 输出

输出内容

输出中将包含全局定义的相似度(如果存在的话)的类的名称

### 示例

获取相似度实现

```bash
curl http://localhost:8983/solr/gettingstarted/schema/similarity?wt=json
```

```javascript
{
"responseHeader":{
  "status":0,
  "QTime":1},
"similarity":{
  "class":"org.apache.solr.search.similarities.DefaultSimilarityFactory"}}
```

## 获取默认查询操作符

```
GET /collection/schema/solrqueryparser/defaultoperator
```

### 输入

路径变量

|键         |描述                     |
|-----------|------------------------|
|collection |集合(或 core)的名称       |

查询参数

查询参数应该在 API 请求的 '?' 后添加

|键  |类型   |必需 |默认  |描述                       |
|----|------|----|-----|---------------------------|
|wt  |string|No  |json |定义响应格式。可选项有 `json`、`xml` 或 `schema.xml`。若没有指定，将默认返回 JSON。|

### 输出

输出内容

输出中将包含用户没有定义时的默认操作符。

### 示例

获取默认操作符

```bash
curl http://localhost:8983/solr/gettingstarted/schema/solrqueryparser/defaultoperator?wt=json
```

```javascript
{
"responseHeader":{
  "status":0,
  "QTime":2},
"defaultOperator":"OR"}
```
