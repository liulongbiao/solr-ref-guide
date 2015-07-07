# 通过索引处理器上传数据

索引处理器是设计用于给索引添加、删除、更新文档的请求处理器。除了通过
[使用 Tika](../solr_cell.md) 来导入富文档插件和
使用 [DataImportHandler](../DataImportHandler.md) 从结构化数据源中导入之外，
Solr 原生支持索引 XML、JSON、CSV 格式的文档。

推荐的配置和使用请求处理器的方式是通过基于路径的名称将路径映射为请求 URL。
然而，如果 [RequestDispatcher](../../config/solrconfig/RequestDispatcher.md) 被适当地配置的话，
请求处理器也可以通过 `qt` (请求类型) 参数来指定。
可以通过不止一个名字来访问相同的处理器，这在你希望指定不同集合的默认选项时很有用。

单个统一的更新请求处理器支持 XML、JSON、CSV 和 javabin 更新请求，
它会基于 [内容流](../content_streams.md) 的 `Content-Type` 来委托给合适的 `ContentStreamLoader`。

本节内容包括：

* [UpdateRequestHandler 配置](./config.md)
* [XML 格式索引更新](./xml.md)
* [JSON 格式索引更新](./json.md)
* [CSV 格式索引更新](./csv.md)
* [内嵌子文档](./nested.md)