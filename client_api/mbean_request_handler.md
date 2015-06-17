# MBean 请求处理器

MBean 请求处理器提供了和管理 UI 的 [插件&统计视图](../adminui/core_tools/plugins.md) 页面
提供的信息的编程式的访问。

MBean 请求处理器接收以下参数：

|参数  |类型        |默认值 |描述                   |
|-----|------------|-----|----------------------|
|key  |multivalued |all  | 通过对象的键来限制结果 |
|cat  |multivalued |all  | 通过分类名称来限制结果 |
|stats|boolean     |false| 指定统计是否和结果一起返回。你可以针对每个字段覆盖 `stats` 参数。 |
|wt   |multivalued |xml  | 输出格式。该操作和 [查询中的 `wt` 参数](../searching/response_writers.md) 一样的效果 |

## 示例

下例假设你运行的是 Solr 的 `techproducts` 示例配置：

```bash
bin/solr start -e techproducts
```

要仅返回 `CACHE` 分类的信息：

```
http://localhost:8983/solr/techproducts/admin/mbeans?cat=CACHE
```

要仅返回 `CACHE` 分类的信息和统计，并以 JSON 格式输出：

```
http://localhost:8983/solr/techproducts/admin/mbeans?stats=true&cat=CACHE&indent=true&wt=json
```

要返回所有东西的信息以及除了 `fieldCache` 外的所有统计：

```
http://localhost:8983/solr/techproducts/admin/mbeans?stats=true&f.fieldCache.stats=false
```

要仅返回 `fieldCache` 的信息和统计：

```
http://localhost:8983/solr/techproducts/admin/mbeans?key=fieldCache&stats=true
```
