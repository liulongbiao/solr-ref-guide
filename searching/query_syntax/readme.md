# 查询语法和解析

Solr 支持多种查询解析器，提供给搜索应用的设计人员极大的控制如何解析查询的灵活性。

本节解释了如何指定要使用的查询解析器。
它也描述了 Solr 内置的主要查询解析器所支持的语法和特性
并描述了一些可能在特定场景下有用的其他解析器。
有一些查询参数通用于所有的 Solr 解析器；它们会在小节 [通用查询参数](./common.md) 中讨论。

本指南讨论的解析器有：

* [标准查询解析器](./standard.md)
* [DisMax查询解析器](./dismax.md)
* [扩展的DisMax查询解析器](./extended_dismax.md)
* [其他解析器](./other_parsers.md)

查询解析器插件都是 [QParserPlugin](http://lucene.apache.org/solr/5_2_0/solr-core/org/apache/solr/search/QParserPlugin.html)
的子类。如果你有定制解析需求，可以扩展该类并创建自己的查询解析器。

更多 Solr 中可用的查询解析器的信息，查看 https://wiki.apache.org/solr/SolrQuerySyntax 。
