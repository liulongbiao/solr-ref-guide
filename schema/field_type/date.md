# 处理日期

## 日期格式化

Solr 的 `TrieDateField`(及废弃的 `DateField`) 表示了毫秒精度的事件点。
其使用的格式为 [XML 模式规范](http://www.w3.org/TR/xmlschema-2/#dateTime) 中权威表示的严格形式。

`YYYY-MM-DDThh:mm:ssZ`

* `YYYY` 年
* `MM` 月
* `DD` 月中的日
* `hh` 24小时时钟里的小时
* `mm` 分钟
* `ss` 秒
* `Z` 字面量 `Z` 字符表示该字符串为 UTC 日期表示

注意不能指定时区；日期的字符串表示总是表示为 UTC 时间。如下例：

`1972-05-20T17:33:18Z`

你也可以选择包含分数型的秒，尽管任何超过毫秒的精度将被忽略。下面是亚秒级的示例：

* `1972-05-20T17:33:18.772Z`
* `1972-05-20T17:33:18.77Z`
* `1972-05-20T17:33:18.7Z`

## 日期范围格式化

Solr 的 `DateRangeField` 支持上述相同的时间点语法，并且扩展以表示日期范围。
一种示例是截断的日期，它表示了给定精度所表示的整个时间段。
另一种使用了范围语法 (`[ TO ]`)。下面有一些例子：

* `2000-11` - 2000 年 11 月一整个月
* `2000-11T13` - 同上，但针对该天的第13个小时(下午1-2点)
* `-0009` - 公元前 10 年。公元 0 年也被看做公元前 1 年。
* `[2000-11-01 TO 2014-12-01]` - 在天精度上指定的日期范围
* `[2014 TO 2014-12-01]` - 2014 年开始知道其 12 月第一天的结束
* `[* TO 2014-12-01]` - 从最早的日期直到 2014-12-01 的结束

限制：范围语法不支持内嵌的日期数学。
若你指定一个由 TrieDateField 字段支持的日期实例，并使用日期数学截取它，如 `NOW/DAY`，
你还是得到该天的第一个毫秒，而不是整个的日期范围。
排除范围语法 (使用 `{ & }`) 可在查询中使用，但在索引范围时不能使用。

## 日期数学

Solr 的日期字段类型也支持 *日期数学* 表达式，它使得创建相对于某个固定时刻的时间变得容易，
包括当前的时间可使用特殊值 `NOW` 来获取。

### 日期数学语法

日期数学表达式或者是添加以某个单位的某个量的时间，或者是根据某个单位对当前时间取整。
表达式可以串联并且从左到右求值。

如，表示当前开始的两个月后：

`NOW_2MONTHS`

一天以前：

`NOW-1DAY`

一个斜杠表示取整。获取当前小时的开始：

`NOW/HOUR`

下面 计算(毫秒精度)六个月零三天之后的时间点并将其取整到天的开始：

`NOW+6MONTHS+3DAYS/DAY`

虽然日期数学常用在相对于当前时间 `NOW` 上，但它其实还能应用在固定的时刻上：

`1972-05-20T17:33:18.772Z+6MONTHS+3DAYS/DAY`

### 影响日期数学的请求参数

#### NOW

`NOW` 参数用在 Solr 内部确保在分布式请求的多个节点间一致性地表达日期数学。
但它可以被指定以指示 Solr 使用时间(过去或未来)上的任意时刻来覆盖所有特殊值 `NOW`
将影响日期数学表达式的所有地方。

它必需被指定为标准时间点以来的毫秒数，如：

`q=solr&fq=start_date:[* TO NOW]&NOW=1384387200000`

#### TZ

默认所有的日期数学表达式都相对于 UTC 时区进行求值，但 `TZ` 参数可指定以覆盖默认行为，
强制所有基于日期的添加和取整都相对于给定的 [时区](http://docs.oracle.com/javase/7/docs/api/java/util/TimeZone.html)。

如以下请求将使用范围 faceting 来对当前月进行 facet，相对于 UTC 的每一天：

```
http://localhost:8983/solr/my_collection/select?q=*:*&facet.range=my_date_field&face
t=true&facet.range.start=NOW/MONTH&facet.range.end=NOW/MONTH%2B1MONTH&facet.range.ga
p=%2B1DAY
```

```xml
<int name="2013-11-01T00:00:00Z">0</int>
<int name="2013-11-02T00:00:00Z">0</int>
<int name="2013-11-03T00:00:00Z">0</int>
<int name="2013-11-04T00:00:00Z">0</int>
<int name="2013-11-05T00:00:00Z">0</int>
<int name="2013-11-06T00:00:00Z">0</int>
<int name="2013-11-07T00:00:00Z">0</int>
...
```

而下面的例子，“日期”的计算将相对于给定时区 - 包含任何可应用的夏令时调整：

```
http://localhost:8983/solr/my_collection/select?q=*:*&facet.range=my_date_field&face
t=true&facet.range.start=NOW/MONTH&facet.range.end=NOW/MONTH%2B1MONTH&facet.range.ga
p=%2B1DAY&TZ=America/Los_Angeles
```

```xml
<int name="2013-11-01T07:00:00Z">0</int>
<int name="2013-11-02T07:00:00Z">0</int>
<int name="2013-11-03T07:00:00Z">0</int>
<int name="2013-11-04T08:00:00Z">0</int>
<int name="2013-11-05T08:00:00Z">0</int>
<int name="2013-11-06T08:00:00Z">0</int>
<int name="2013-11-07T08:00:00Z">0</int>
...
```

## 更多 DateRangeField 细节

`DateRangeField` 几乎是对 `TrieDateField` 可使用的地方的替代品。
仅有的差别是 Solr 的 XML 或 SolrJ 响应格式将会其存储的日期暴露为一个字符串而不是日期。
该字段的底层索引数据会更大一些。对对齐单位在秒级以上的查询其速度比 `TrieDateField` 会更快一些，
特别是当其为 UTC 时。但 `DateRangeField` 的关键是其名称暗示了它允许所以日期范围。
要这样做，只需要简单地提供上述格式的字符串即可。
它也支持在索引数据和查询范围之间指定三种不同的关系谓语：`Intersects`(默认)、`Contains`、`Within`。
你可以通过使用 `op` 本地参数来指定谓语，如：

```
fq={!field f=dateRange op=Contains}[2013 TO 2018]
```

该例中，它将会查找索引范围 *包含*(或等于)范围  2013 到 2018 的数据。
文档中的多值覆盖的索引范围将会被高效地合并。
