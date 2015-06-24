# SolrConfig 中的 lib 指令

Solr 允许通过在 `solrconfig.xml` 中定义 `<lib>` 来加载插件。

这些插件以其出现在 `solrconfig.xml` 中的顺序来加载。
若它们存在依赖，将最低层级的依赖 jar 包列在前面。

正则表达式可用于提供对带有对相同目录下其它 jar 包有以依赖的 jar 包的加载控制。
所有目录都被解析为想对于 Solr 的 `instanceDir`。

```xml
<lib dir="../../../contrib/extraction/lib" regex=".*\.jar" />
<lib dir="../../../dist/" regex="solr-cell-\d.*\.jar" />

<lib dir="../../../contrib/clustering/lib/" regex=".*\.jar" />
<lib dir="../../../dist/" regex="solr-clustering-\d.*\.jar" />

<lib dir="../../../contrib/langid/lib/" regex=".*\.jar" />
<lib dir="../../../dist/" regex="solr-langid-\d.*\.jar" />

<lib dir="../../../contrib/velocity/lib" regex=".*\.jar" />
<lib dir="../../../dist/" regex="solr-velocity-\d.*\.jar" />
```
