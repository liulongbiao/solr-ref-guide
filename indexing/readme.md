# 建索引和基本数据操作

本节描述 Solr 如何添加数据到索引中。它包含以下内容：

* [简介](./intro.md) : Solr 索引过程概览
* [Post工具](./post.md) : 关于使用 `post.jar` 来快速上传某些内容到你的系统的信息
* [用 Index Handlers 上传数据](./index_handlers.md) : 关于使用 Solr 的 Index Handlers 来上传
XML/XSLT、JSON 和 CSV 数据。
* [用带 Apache Tika 的 SolrCell 上传数据](./solr_cell.md) : 关于使用 Solr Cell 框架来上传
用于索引的数据的信息。
* [用 DataImportHandler上传结构型数据存储的数据](./DataImportHandler.md) : 关于
从结构化数据存储上传和索引数据的信息。
* [上传文档的部分](./update_parts.md) : 关于如何在 Solr 中使用原子更新和乐观并发的信息
* [去重](./de_duplication.md) : 关于配置 Solr 在文档被索引时标记重复文档的信息
* [建索引时侦测语言](./detect_lang.md) : 关于在建索引过程中使用语言识别的信息
* [内容流](./content_streams.md) : 关于提供给 Solr Request Handlers 的流内容
* [UIMA集成](./UIMA.md) : 关于集成 Solr 和 UIMA 的信息。UIMA 让你定义自定义的分析引擎的管道
以增量地将元数据作为注解添加到你的文档上。

## 使用客户端 APIs 建索引

使用像 [SolrJ](../client_api/solrj.md) 这样的客户端来从你的应用中更新 Solr 索引也是一个重要选项。
查看 [客户端APIs](../client_api/readme.md) 更多信息。
