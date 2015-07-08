# 结果分页

## 基本分页

大多数搜索应用中，会找到"顶部"的匹配文档(根据分数或其它条件)并显示给人类用户。
很多应用中给用户显示的这些排序的结果的 UI 以包含固定匹配结果的 "分页" 来显示，
且用户常常只会查看最前面的几页。

在 Solr 中，基本的分页搜索可通过使用 `start` 和 `rows` 参数来支持，
且这种常见行为的性能可根据你希望的分页大小来通过利用 
[queryResultCache](https://cwiki.apache.org/confluence/display/solr/Query+Settings+in+SolrConfig#QuerySettingsinSolrConfig-queryResultCache)
和调整 [queryResultWindowSize](https://cwiki.apache.org/confluence/display/solr/Query+Settings+in+SolrConfig#QuerySettingsinSolrConfig-queryResultWindowSize)
进行调优。

## 基本分页示例

最简单地看待简单分页的方式是，简单地将页数(第一页处理为 0)乘以每页行数；如以下伪代码：

```javascript
function fetch_solr_page($page_number, $rows_per_page) {
  $start = $page_number * $rows_per_page
  $params = [ q = $some_query, rows = $rows_per_page, start = $start ]
  return fetch_solr($params)
}
```

## 基本分页如何被索引更新所影响

Solr 请求中的 `start` 参数表示的是对匹配客户端希望 Solr 用作完整地有序文档列表开始到
当前"页" 的 **绝对** "偏移量"。若某个索引的修改(如添加或移除文档) 会影响两次顺序分页查询请求间
分页结果中有序文档的序号，则可能导致相同的文档在多个页面中出现，或者某些文档在结果集收缩或增长时被 "跳过"了。

如，考虑包含以下26个文档的索引：

|id  |name  |
|----|------|
|1   |A     |
|2   |B     |
|...||
|26  |Z     |

假设以下请求和索引修改交替进行：

* 客户端请求 `q=*:*&rows=5&start=0&sort=name asc`
    * 具有id 1-5 的文档将被返回给客户端
* 文档id 3 被删除
* 客户端请求"第二页" `q=*:*&rows=5&start=5&sort=name asc`
    * 文档 7-11 将被返回
    * 文档 6 被跳过，因为现在它是所有匹配文档集合中的第五条 - 它将在新的请求的第一页中返回
* 3 个新的文档 90、91、92 被加入；所有三个文档具有名称 A
* 客户端请求"第三页" `q=*:*&rows=5&start=10&sort=name asc`
    * 文档 9-13 将被返回
    * 文档 9、10、11 现在都会在第二第三页返回，因为它们在有序结果中往回移了
    
典型的场合中，索引的变更对分页结果的影响不会显著影响用户体验 - 不管是它们在静态集合中很少出现，
还是因为用户会察觉到数据集合持续地演进且能接受看到文档在结果集中上移或下移。

