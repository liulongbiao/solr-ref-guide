# 配置良好的 Solr 实例

本节告诉你如何微调你的额 Solr 实例以优化性能。
本节包含以下内容:

* [配置solrconfig.xml](./solrconfig/readme.md)：描述了如何处理 Solr 的主配置文件， `solrconfig.xml`，
包括了该文件的主要部分。
* [Solr Cores和solr.xml](./core/readme.md)：描述了如何处理 `solr.xml` 和 `core.properties`
以配置你的 Solr core 或在单个实例中的多个 Solr cores。
* [Solr插件](./plugins/readme.md)：介绍了 Solr 插件以及更多信息的链接
* [JVM设置](./jvm.md)：给出使用 JVM 时的一些最佳实践

> 本节通常关注于配置单个 Solr 实例，对在集群环境中伸缩 Solr 实现感兴趣的可以查看
> 小节 [SolrCloud](../solrcloud/readme.md)。
> 在小节 [遗留的伸缩性和分布式](../legacy_distribution/readme.md) 中也有对分片或副本伸缩的选项。
