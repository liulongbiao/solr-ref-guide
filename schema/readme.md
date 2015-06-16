# 文档、字段和模式设计

本节将会讨论 Solr 是如何将其数据组织为文档和字段，
以及如何使用 Solr 的模式文件，`schema.xml`。
它包含以下主题：

* [文档、字段和模式设计概述](./overview.md) : 介绍本节将涉及的一些概念
* [Solr字段类型](./field_type/readme.md) : 关于 Solr 中字段类型的详细信息，包括默认 Solr 模式中的字段类型
* [定义字段](./defining_fields.md) : 描述如何定义字段
* [拷贝字段](./copying_fields.md) : 描述如何用从另一个字段复制的数据来填充字段
* [动态字段](./dynamic_fields.md) : 关于使用动态字段来捕获和索引那些不能完全满足你的模式中其它字段定义的字段的信息
* [模式API](./schema_api.md) : 使用 curl 命令读取某个模式的多个部分或创建新字段和 copyField 规则
* [其他模式元素](./other_schema_elements.md) : 描述Solr模式中其它重要的元素
* [放到一起](./putting_together.md) : 一个关于 Solr 模式及其元素如何一起运行的更高的视角
* [DocValues](./docvalues.md) : 描述如何创建一个 DocValues 索引以更快地查找
* [无模式形式](./schemaless_mode.md) : 使用基于值的字段类型猜测来自动添加以前未知的模式字段
