# 拷贝字段

你可能希望以多种不同的方式来解释某些文档字段。
Solr 具有一种机制来拷贝字段，这样你可以对传入的信息的某个单个片段应用多种不同的字段类型。

你希望进行拷贝的字段的名称成为 `source`，而拷贝的名称是 `destination`。
在 `schema.xml` 中，拷贝字段非常简单：

```xml
<copyField source="cat" dest="text" maxChars="30000" />
```

若 `text` 目标字段在输入文档中具有自己的数据，
则 `cat` 字段的内容将被添加为额外的值 - 就好像被客户端原始地指定一样。
当它们最终可能会得到多个值时(不管是从一个多值源，还是多个 `copyField` 指令等等)，
你需要将该字段设置为 `multivalued="true"`。

`maxChars` 参数是一个 `int` 型参数，建立了当从源值拷贝的以构造被添加到目标字段的值的最大字符数量。
这个限制在你希望从源字段拷贝数据，但也需要控制索引文件大小时非常有用。

`copyField` 元素的源和目标都可以包含前缀或后缀的星号，它可以匹配任何东西。
例如，以下代码将拷贝所有传入字段名匹配通配符模式 `*_t` 的字段的内容到该文本字段：

```xml
<copyField source="*_t" dest="text" maxChars="25000" />
```

> `copyField` 命令的 `dest` 参数也可以使用一个通配符(*)，当且仅当 `source` 参数也包含一个通配符。
> `copyField` 使用 `source` 字段中的匹配块来作为 `dest` 匹配标识名来拷贝内容。

## 相关内容

* [SchemaXML-Copy Fields](http://wiki.apache.org/solr/SchemaXml#Copy_Fields)
