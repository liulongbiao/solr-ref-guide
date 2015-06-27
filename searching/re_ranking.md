# 查询重排序

查询重排序让你可以运行简单的查询(A)来匹配文档，然后从一个更复杂的查询 (B) 得到的分数对最高的 N 个
文档进行重排序。因为来自查询 B 的成本更高的排序仅应用在最高的 N 个文档上，
它将比纯使用复杂查询 B 性能影响更小 - 其代价是对使用简单查询 A 分数较低的文档可能在重排序阶段不会被考虑，
即使它们在使用查询 B 时可能分数更高。

### 指定排序查询

排序查询可以通过 `rq` 请求参数指定。`rq` 参数必须指定为一个查询字符串，该字符串解析后能产生一个 RankQuery。
这可以使用你自己写的自定义的 QParserPlugin 插件来完成，
但大多数用户可以仅使用由 Solr 提供的 `rerank` 解析器。

`rerank` 解析器包装了由一个本地参数所指定的查询，以及额外的指示应该有多少文档应该被重排序，
以及最终分数应该如何被计算的参数：

|参数         |默认 |描述                          |
|------------|----|------------------------------|
|reRankQuery |必需 |复杂排序查询的查询字符串 - 大多数时候 [一个变量](./query_syntax/local_params.md) 将被用于引用其它请求参数 |
|reRankDocs  |200 |原查询中分数最高的 N 个应该被重排序的文档。这个数量将被当作最小值，且为了能排序足够的满足查询的文档将会内部自动增长(即 start+rows )  |
|reRankWeight|2.0 |将被应用到每个最高匹配的文档的和来自 `reRandQuery` 的分数的相乘因子，然后才会被添加到原始分数上 |

下例中，匹配查询 "greeting" 的最高 1000 个文档将会使用查询 "(hi hello hey hiya)" 进行重排序。
这 1000 个文档的结果分数将是它们来自 "(hi hello hey hiya)" 的分数乘以 3，
然后加上原始的 "greeting" 查询的分数：

```
q=greetings&rq={!rerank reRankQuery=$rqq reRankDocs=1000
reRankWeight=3}&rqq=(hi+hello+hey+hiya)
```

若文档匹配原始查询，但不批评重排序查询，文档的原始分数将保留。

## 组合排序查询和其他 Solr 特性

`rq` 参数和重排序特性通常能和其它 Solr 特性一起运行。
如，它可以连同 [收缩解析器](./collapse_and_expand.md) 一起以在它们被收缩时
重排序分组头。
它也能保留由 [提升组件](./query_elevation_component.md) 提升的文档的顺序。
且它甚至包含自己的自定义解释，这样你可以通过查看 
[调试信息](https://cwiki.apache.org/confluence/display/solr/Common+Query+Parameters#CommonQueryParameters-ThedebugParameter)
得到重排序分数是如何得出的。
