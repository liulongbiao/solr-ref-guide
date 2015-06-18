# Solr 索引简介

本节描述建索引的过程：添加内容到 Solr 索引以及，如果需要的话，修改该内容或者删除它。
通过给一个索引添加内容，我们让它可以被 Solr 搜索到。

Solr 索引可以接收很多不同来源的数据，包括 XML 文件、CSV 文件、从数据库的表中提取的数据
以及像 Microsoft Word 或 PDF 这样的常见的文件格式的文件。

以下是三种最常见的加载数据到 Solr 索引中的方式：

* 使用构建在 Apache Tika 上的 [Solr Cell](./solr_cell.md) 框架
来摄取二进制文件或像 Office、Word、PDF 和其它合适格式的结构化文档。
* 从任何可以生成请求的环境中，通过发送 HTTP 请求上传 XML 文件给 Solr 服务器
* 写一个自定义的 Java 应用通过 Solr 的 Java 客户端 API (详见 [使用SolrJ](../client_api/solrj.md))
来摄取数据。使用 Java API 可能是你使用一个提供了 Java API 的应用(如 CMS) 时更好的选择。

不管使用何种方式摄取数据，给 Solr 索引的的数据有一个基本的数据结构：
一个 **文档** 包含多个 **字段**，每个字段有一个 **名称** 并包含 **内容**， 内容可以为空。
这些字段中的某个字段常设计用作唯一的 ID 字段(可看作数据库中的主键)，
尽管唯一键字段在 Solr 中不是必需的。

若字段名称在关联到该索引的 `schema.xml` 文件中有定义，
则该字段关联的分析步骤将会在内容被分词时应用到其内容上。
模式中没有明确定义的字段将被忽略
或者在存在一个匹配的字段名称时，会映射到一个动态字段定义
(见 [动态字段](../schema/dynamic_fields.md)).

更多 Solr 中建索引的信息，查看 [Solr Wiki](https://wiki.apache.org/solr/FrontPage)

## Solr 示例目录

当以 `-e` 选项启动 Solr 时，其 `example/` 目录将被用作所创建的示例 Solr 实例的基目录。
该目录也包含一个 `example/exampledocs/` 子目录，包含了多种格式的示例文档可用于
索引到多个示例中。

## 用于传输文件的 `curl` 工具

本节的很多指令和示例都使用了 curl 工具来通过 URL 传输内容。
curl 可以在 HTTP、FTP 和很多其它协议上发布和检索数据。
大多数 Linux 系统都包含了 curl 拷贝。
你可以在 http://curl.haxx.se/download.html 上找到 Linux、Windows 的 curl 下载。
curl 的文档可在 http://curl.haxx.se/docs/manpage.html 查看。

> 注：使用 curl 和其他命令行工具来发布数据只是为了测试或示例，而不是产品环境中以最佳性能地更新
> 推荐的方式。使用 Solr Cell 或本节描述的其它方式能得到更好的性能。
>
> 除了 curl， 你也可以使用像 GNU [wget](http://www.gnu.org/software/wget/) 
> 或用 Perl 来管理 GETs 和 POSTs，尽管命令行选项可能不一样。




