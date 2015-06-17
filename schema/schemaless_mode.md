# 无模式形式

无模式形式是一系列 Solr 特性，它们用在一起时，允许用户快速地
通过简单地索引样本数据构建一个高效的模式，而不用手动编辑该模式。
这些 Solr 特性，都由 `solrconfig.xml` 中指定，有：

1. 受管模式：Schema 修改通过 Solr API 而不是手动编辑来完成 - 
见[受管理的模式定义](../config/solrconfig/managed_schema_definition.md)
2. 字段值类猜测：以前没见过的字段会运行通过一个层叠的基于值的解析器的集合，
它们会猜测字段值的 Java 类 - 当前可用的有 Boolean、Integer、Long、Float、Double 和 Date。
3. 基于字段值的类自动进行模式字段的添加：前面没见过的字段会根据字段值的 Java 类型
映射到对应的模式字段类型并添加到模式中 - 见 [Solr字段类型](./field_type/readme.md)

