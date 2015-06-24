# SolrConfig 中受管理的模式定义

[Schema API](../../schema/schema_api.md) 使得可以通过 REST 接口来进行 
[Schema](../../schema/readme.md) 的修改。
(也支持对所有模式元素的只读访问)

允许对配置文件的编程式访问存在挑战，它也开放了手动编辑：
系统生成的和手动编辑可能重叠且系统生成的编辑可能移除注释
或者其它对于理解为何字段和字段类型等以这种方式来定义等组织方式非常重要的定制。
你可能希望以源码控制来管理文件的版本，或者限制手动的编辑等。

`solrconfig.xml` 中的 `schemaFactory` 选项控制了 Schema 是否应该定义为“受管理的索引模式”：
模式的修改仅能通过 [Schema API](../../schema/schema_api.md) 进行。

默认，若没有指定 `schemaFactory`，则其默认行为是使用 `ClassicIndexSchemaFactory`，
就像 `sample_techproducts_configs` [Config Sets](../core/config_sets.md)
中的示例一样：

```xml
<schemaFactory class="ClassicIndexSchemaFactory"/>
```

`ClassicIndexSchemaFactory` 需要使用一个 `schema.xml` 文件，
它可以手动编辑且仅在集合被加载时加载。
这个设置不允许 Schema API 的方式来修改模式。

然而在 `data_driven_schema_configs` 设置中，我们可以看到 `ManagedIndexSchemaFactory` 的用法：

```xml
<!-- To disable dynamic schema REST APIs, use the following for <schemaFactory>:
  <schemaFactory class="ClassicIndexSchemaFactory"/>
  When ManagedIndexSchemaFactory is specified instead, Solr will load the schema from
  he resource named in 'managedSchemaResourceName', rather than from schema.xml.
  Note that the managed schema resource CANNOT be named schema.xml. If the managed
  schema does not exist, Solr will create it after reading schema.xml, then rename
  'schema.xml' to 'schema.xml.bak'.
  
  Do NOT hand edit the managed schema - external modifications will be ignored and
  overwritten as a result of schema modification REST API calls.
  When ManagedIndexSchemaFactory is specified with mutable = true, schema
  modification REST API calls will be allowed; otherwise, error responses will be
  sent back for these requests.
-->
  <schemaFactory class="ManagedIndexSchemaFactory">
    <bool name="mutable">true</bool>
    <str name="managedSchemaResourceName">managed-schema</str>
  </schemaFactory>
```

这里你可以看到对受管理模式的配置。为使得可以通过 [Schema API](../../schema/schema_api.md)
进行模式的修改，需要使用 `ManagedIndexSchemaFactory`。
参数 `mutable` 也必须设置为 `true`。其 `managedSchemaResourceName` 也可以指定，
其默认为 “managed-schema”，并且可以是任何 "schema.xml" 以外的值。

有了上述配置，我们可以使用 [Schema API](../../schema/schema_api.md) 来根据需要对模式进行修改，
且后续如果你希望“锁定”模式并避免后续变更时可以将 `mutable` 的值该为 `false`。

若你想将已有的 Solr 集合转换为使用受管理的模式，你可以简单地修改 `solrconfig.xml`
来指定使用 `ManagedIndexSchemaFactory`。
一旦 Solr 被重启且它侦测到 `schema.xml` 存在，但 `managedSchemaResourceName` 不存在时，
已有的 `schema.xml` 会被重命名为 `schema.xml.bak` 且其内容会被写到
`managedSchemaResourceName` 所定义的文件中。
若你查看结果文件，可以在头部看到：

```xml
<!-- Solr managed schema - automatically generated - DO NOT EDIT -->
```

你现在随意根据需要使用 [Schema API](../../schema/schema_api.md) 进行修改，
以及移除 `schema.xml.bak`。

