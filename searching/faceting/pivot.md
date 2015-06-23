# 枢轴(决策树) Faceting

枢轴是一个让你可以自动排序、计数、总计或平均存储在表中的数据的摘要工具。
其结果通常显示在第二个表格中用以显示摘要数据。
枢轴 faceting 让你可以根据多个字段从一个 faceting 文档中创建一个结果的摘要表。

另一种看待它的方式是查询会产生一个决策树，这时候 Solr 会告诉你
“对 facet A，其约束/计数为 X/N，Y/M等。若你约束 A 和 X，则对 B 的约束计数将是 S/P，T/Q等..”。
或者说，它会进一步告诉你当你从当前 facet 结果应用约束时，对某个字段 “下”一个 facet 结果将会是什么。

## facet.pivot

`facet.pivot` 参数定义了用作枢轴的字段。
多个 `facet.pivot` 值将在响应中创建多个 "facet_pivot" 部分。
每个字段列表之间以逗号分隔。

## facet.pivot.mincount

`facet.pivot.mincount` 参数定义了要被包含在 facet 中需要的最小的匹配文档的数量。默认为 1。

使用 `bin/solr -e techproducts` 示例，一个如下的查询 URL 将返回以下数据，
其中枢轴 faceting 结果可在 "facet_pivot" 部分找到：

```
http://localhost:8983/solr/techproducts/select?q=*:*&facet.pivot=cat,popularity,inStock
&facet.pivot=popularity,cat&facet=true&facet.field=cat&facet.limit=5
&rows=0&wt=json&indent=true&facet.pivot.mincount=2
```

```javascript
"facet_counts":{
  "facet_queries":{},
  "facet_fields":{
    "cat":[
      "electronics",14,
      "currency",4,
      "memory",3,
      "connector",2,
      "graphics card",2]},
    "facet_dates":{},
    "facet_ranges":{},
    "facet_pivot":{
      "cat,popularity,inStock":[{
        "field":"cat",
        "value":"electronics",
        "count":14,
        "pivot":[{
          "field":"popularity",
          "value":6,
          "count":5,
          "pivot":[{
            "field":"inStock",
            "value":true,
            "count":5}]},
...
```

## 联合 Stats 组件和 Pivots

除了某些由其它类型的 faceting 支持的 [通用本地参数](./local_params.md)外，
一个 `stats` 本地参数可用在 `facet.pivot` 上以给你希望计算的每个 Pivot 约束
(根据标签) 引用 [Stats组件](../stats_component.md)。

下例中，两个不同(重叠)集的统计对每个 `facet.pivot` 结果层级进行计算。

```
stats=true
stats.field={!tag=piv1,piv2 min=true max=true}price
stats.field={!tag=piv2 mean=true}popularity
facet=true
facet.pivot={!stats=piv1}cat,inStock
facet.pivot={!stats=piv2}manu,inStock
```

结果：

```javascript
"facet_pivot":{
  "cat,inStock":[{
    "field":"cat",
    "value":"electronics",
    "count":12,
    "pivot":[{
      "field":"inStock",
      "value":true,
      "count":8,
      "stats":{
        "stats_fields":{
          "price":{
            "min":74.98999786376953,
            "max":399.0}}}},
      {
      "field":"inStock",
      "value":false,
      "count":4,
      "stats":{
        "stats_fields":{
          "price":{
            "min":11.5,
            "max":649.989990234375}}}}],
    "stats":{
      "stats_fields":{
        "price":{
          "min":11.5,
          "max":649.989990234375}}}},
  {
    "field":"cat",
    "value":"currency",
    "count":4,
    "pivot":[{
      "field":"inStock",
      "value":true,
      "count":4,
      "stats":{
        "stats_fields":{
          "price":{
            ...
  "manu,inStock":[{
    "field":"manu",
    "value":"inc",
    "count":8,
    "pivot":[{
      "field":"inStock",
      "value":true,
      "count":7,
      "stats":{
        "stats_fields":{
          "price":{
            "min":74.98999786376953,
            "max":2199.0},
          "popularity":{
            "mean":5.857142857142857}}}},
    {
    "field":"inStock",
    "value":false,
    "count":1,
    "stats":{
      "stats_fields":{
        "price":{
          "min":479.95001220703125,
          "max":479.95001220703125},
        "popularity":{
          "mean":7.0}}}}],
...
```

## 其它 Pivot 参数

尽管 `facet.pivot.mincount` 偏离了用在字段 faceting 中的 `facet.mincount` 参数，
很多上面描述的其他 faceting 参数可能在枢轴 faceting 中使用：

* [facet.limit](./field_value.md#facet-limit)
* [facet.offset](./field_value.md#facet-offset)
* [facet.sort](./field_value.md#facet-sort)
* [facet.overrequest.count](./field_value.md#facet-overrequest)
* [facet.overrequest.ratio](./field_value.md#facet-overrequest)
