# 查询中的本地参数

本地参数是 Solr 请求中给查询参数所指定的参数。
本地参数提供了一种给像查询字符串这样的特定参数类型添加元数据的方式。
(在 Solr 文档中，本地参数有时被引用为 `LocalParams`)

本地参数以参数的前缀形式来指定。对以下查询参数：

`q=solr rocks`

我们可以以本地参数来前缀这个查询字符串，以给标准查询解析器提供更多信息。
如，我们可以将默认操作符改变为 "AND" 并且使默认字段变为 "title"：

`q={!q.op=AND df=title}solr rocks`

这些本地参数将查询改变为需要同时匹配 "solr" 和 "rocks"，并且默认搜索字段为 "title"。

## 本地参数基本语法

要指定本地参数，需在待修改参数的前面插入以下内容：

* 以 `{!` 开头
* 插入任意数量的由空格分隔的 `key=value` 对
* 以 `}` 结尾，且后面立即紧跟原查询参数

每个参数前只能指定一个本地参数。在键值对中的值可以通过单引号或双引号括起，
并且在被括起的字符串内反斜杠转码可用。

## 查询类型简写

若本地参数值出现时不带名称，其隐式的名称为 "type"。
这使得可以简写用于解析查询字符串的查询解析器的类型。因此：

`q={!dismax qf=myfield}solr rocks`

等价于：

`q={!type=dismax qf=myfield}solr rocks`

若没指定 "type"(不管是隐式还是显式地)，则默认使用 [Lucene 解析器](./standard.md)。因此：

`fq={!df=summary}solr rocks`

等价于：

`fq={!type=lucene df=summary}solr rocks`

## 用 `v` 键指定参数值

特殊的键 `v` 在本地参数中是指定该参数的值的一种替代方式。

`q={!dismax qf=myfield}solr rocks`

等价于：

`q={!type=dismax qf=myfield v='solr rocks'}`

## 参数解引用

参数解引用或间接引用让你可以使用另一个参数的值，而不是直接指定它。
这可用于简化查询，解耦了用户输入和查询参数，或者解耦了前端 GUI 参数和在 `sorlconfig.xml` 中的默认设置。

`q={!dismax qf=myfield}solr rocks`

等价于：

`q={!type=dismax qf=myfield v=$qq}&qq=solr rocks`
