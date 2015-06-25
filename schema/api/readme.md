# 模式API

Schema API 对 Solr 中每个集合(或使用独立 Solr 模式时的 core) 的模式提供了读写访问。
对所有模式元素的读访问都被支持。
字段、动态字段、字段类型和 `copyField` 规则可以被添加、移除或替换。
以后的 Solr 发布版本中可能会扩展更多写访问并运行更多模式元素的修改。

> 注：**在模式修改后需重新索引！**
> 
> 若你修改了你的模式，你很可能需要重新索引所有的文档。如果不这么做，你可能会失去对文档的访问，
> 或者不能正确地解释它们，如在替换了一个字段类型以后。
>
> 修改模式不会修改任何已经被索引的文档。
> 因此你必须重新索引文档以对它们应用模式的变更。

要用该 API 启用模式的修改，则该模式需要是受管理的且可变的。
查看 [受管理的模式定义](../config/solrconfig/managed_schema_definition.md) 中更多信息。

该 API 允许所有调用都有两种格式的输出: JSON 或 XML。
当请求完整的模式时，存在另一个输出形式，它是根据 `schema.xml` 本身建模的 XML。

当用该 API 修改模式时，会自动触发一次 core 的重加载，以使得后续被索引的文档能够立即可用这些变更。
以前索引的文档将 **不会** 被自动处理 - 如果它们使用了你所变更的模式元素 **必须** 被重新索引。

该 API 的基本地址是 `http://<host>:<port>/solr/<collection_name>`。
如你运行 Solr 的 `cloud` 示例(通过以下 `bin/solr` 命令)，
它创建了一个 `gettingstarted` 集合，
则其基本地址(作为本小节的所有示例URL)将会是 `http://localhost:8983/solr/gettingstarted`。

```bash
bin/solr -e cloud -noprompt
```

* [API入口](./entry.md)
* [修改模式](./modify.md)
* [检索模式信息](./retrieve.md)
* [管理资源数据](./resource.md)
