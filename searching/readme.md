# 搜索

本节描述了 Solr 如何进行搜索请求。它包含一下内容：

* [Solr 搜索概览](searching/overview.md) : Solr 搜索简介
* [Velocity搜索UI](searching/ui.md) : 简单的使用 VelocityResponseWriter 的搜索 UI
* [相关性](searching/relevance.md) : 理解搜索结果相关性的概念信息
* [查询语法和解析](searching/query_syntax/readme.md) : 查询语法和解析简明概念。它包含以下子节：
    * [通用查询参数](searching/query_syntax/common.md) : 不管何种查询解析器都通用的一些参数
    * [标准查询解析器](searching/query_syntax/standard.md) : 标准Lucene 查询解析器的详细信息
    * [DisMax查询解析器](searching/query_syntax/dismax.md) : Solr 的 DisMax 查询解析器的详细信息
    * [扩展的DisMax查询解析器](searching/query_syntax/extended_dismax.md) : Solr 扩展的 DisMax(eDisMax) 查询解析器的详细信息
    * [函数查询](searching/query_syntax/function.md) : 使用来自一或多个数值字段来生成相关度的参数的信息
    * [查询中的本地变量](searching/query_syntax/local_params.md) : 如何给查询添加本地参数
    * [其他解析器](searching/query_syntax/other_parsers.md) : 更多针对特定场景的解析器
* [Faceting](searching/faceting.md) : 关于基于被索引的项的分类搜索结果的详细信息
* [高亮](searching/highlight/readme.md) : Solr 高亮工具的详细信息。子节包含了不同的高亮器
    * [StandardHighlighter](searching/highlight/standard.md) : 使用三种高亮器中最复杂和细粒度的查询表示
    * [FastVectorHighlighter](searching/highlight/fastvector.md) : 针对字段上词向量选项进行优化，适用于大文档和多种语言
    * [PostingsHighlighter](searching/highlight/postings.md) : 使用类似于 FastVectorHighlighter 的选项，但更精简和高效
* [语法检查](searching/spell_check.md) : 关于 Solr 语法检查的详细信息
* [查询重排序](searching/re_ranking.md) : 对简单查询使用更复杂的分数重排高分文档的详细信息
* [转换结果文档](searching/transform.md) : 使用 `DocTransformers` 给各个独立文档添加计算的值的详细信息
* [推荐器](searching/suggester.md) : Solr 强大的自动推荐组件的详细信息
* [MoreLikeThis](searching/morelikethis.md) : Solr 的相似结果组件的详细信息
* [结果分页](searching/pagination.md) : 为在 UI 中显示获取分页结果或为获取满足一个查询的所有文档的详细信息
* [结果分组](searching/grouping.md) : 基于相同的字段值对结果进行分组的详细信息
* [收缩和展开结果](searching/collapse_and_expand.md) : 
* [结果聚类](searching/clustering.md) : 基于对文本字段的集群分析做搜索结果分组的详细信息。有点像“无监督” faceting。
* [空间搜索](searching/spatial_search.md) : 如何使用 Solr 的空间搜索能力
* [Terms组件](searching/terms_component.md) : 访问被索引的词项和包含它们的文档的详细信息
* [TermVector组件](searching/termvector_component.md) : 如何获取有关特定文档词项信息
* [Stats组件](searching/stats_component.md) : 如何从一个文档集中的数值字段返回信息
* [QueryElevation组件](searching/query_elevation_component.md) : 如何强制文档对特定的查询作为结果顶部返回
* [ResponseWriters](searching/response_writers.md) : 配置和使用 Solr ResponseWriters 的详细信息
* [近实时搜索](searching/near_realtime_searching.md) : 如何在被索引后几乎即时包含文档到索引结果中
* [RealTime Get](searching/realtime_get.md) : 如何不开启搜索器就获取一个文档的最新版本
* [导出结果集](searching/exporting.md) & [流式表达式](searching/streaming_expressions.md) : 将大量数据以流的形式导出 Solr 的功能 
