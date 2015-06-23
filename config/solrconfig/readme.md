# 配置solrconfig.xml

`solrconfig.xml` 文件是具有大多数影响 Solr 本身的参数的配置文件。
在配置 Solr 时，你会经常和 `solrconfig.xml` 打交道。
该文件包含一系列 XML 语句来对一个独立的集合设置配置值。

在 `solrconfig.xml` 中，你可以配置诸如以下重要特性：

* 请求处理器，它会处理到 Solr 的请求，如添加文档到索引或给某个查询返回响应的请求
* 监听器，处理对特定查询相关的事件的“监听”；监听器可用于触发特殊代码的执行，如调用某些常见查询以热生缓存
* 请求分发器，用于管理 HTTP 通信
* Admin Web 接口
* 和副本和重复相关的参数(这些参数在 [遗留的伸缩性和分布式](../../legacy_distribution/readme.md) 中有详述)

`solrconfig.xml` 文件放置在每个集合的 `conf/` 目录下。
在 `server/solr/configsets/` 目录下可找到多个具有良好注释的示例文件，
它们演示了很多不同类型的安装的最佳实践。

在以下小节中我们会包含以下内容：

* [DataDir和DirectoryFactory](./DataDir_DirectoryFactory.md)
* [Lib指令](./lib_directives.md)
* [受管理的模式定义](./managed_schema_definition.md)
* [IndexConfig](./IndexConfig.md)
* [RequestHandler和SearchComponents](./RequestHandler_SearchComponents.md)
* [InitParams](./InitParams.md)
* [UpdateHandlers](./UpdateHandlers.md)
* [Query设置](./query_settings.md)
* [RequestDispatcher](./RequestDispatcher.md)
* [Update Request Processors](./Update_Request_Processors.md)

## 替换 SolrConfig 文件中的属性

Solr 支持在配置文件中属性值的变量替换，这允许在 `solrconfig.xml` 中多个配置选项的运行时指定。
其语法为 `${propertyname[:option default value] }`。
这允许定义一个可在 Solr 被启动时覆盖的默认值。
若没有指定默认值，则属性 **必须** 在运行时指定，否则配置文件在解析时将生成一个错误。

存在多种方法用于在配置文件中指定属性。

### JVM 系统参数

任何 JVM 系统参数，通常在启动 JVM 时使用 `-D` 标记来指定，可用作 Solr 中任何配置文件的变量。

例如，在示例 `solrconfig.xml` 文件中，你可以看到以下定义了使用的锁类型的值：

```
<lockType>${solr.lock.type:native}</lockType>
```

这意味着该锁类型默认为 “native”, 但当启动 Solr 时，你可以在启动 Solr 时使用系统参数来覆盖它，如：

```bash
bin/solr start -Dsolr.lock.type=none
```

通常，任何你想设置的 Java 系统属性可通过使用标准的 `-Dproperty=value` 语法传递给 `bin/solr` 脚本。
或者你可以给 `SOLR_OPTS` 环境变量添加常见的系统属性，它定义在 Solr 包含的文件 (`bin/solr.in.sh`) 中。
对更多有关 Solr 包含文件如何运行的信息，查看 [在产品环境使用Solr](../../manage/production.md)

### solrcore.properties

若 Solr core 的配置目录下包含一个名为 `solrcore.properties` 的文件，
该文件可以使用 Java 标准的 [属性文件格式](https://en.wikipedia.org/wiki/.properties) 
来包含任意的用户定义的属性名和值，且这些属性在该 Solr core 的 XML 配置文件中可用作变量。

例如，一下 `solrcore.properties` 文件可以创建在某个集合的 `conf/` 目录下，
使用某个示例配置，来覆盖所使用的 lockType。

```
#conf/solrcore.properties
solr.lock.type=none
```

> `solrcore.properites` 文件的路径和名称可在 
> [core.properties](../core/defining_core_properties.md) 中的 `properties` 属性项进行覆盖。

### 来自 core.properties 的用户定义属性

若你连同 [solr.xml](../core/readme.md) 一起使用一个 `core.properties` 文件，
则可以在这里定义任何用户自定义的属性，且这些属性在解析该 Solr core 的配置文件时都可用于替换。

例如，考虑以下 `core.properties` 文件：

```
#core.properties
name=collection2
my.custom.prop=edismax
```

其中的 `my.custom.prop` 可像这样在 `solrconfig.xml` 文件中用作一个变量：

```xml
<requestHandler name="/select">
  <lst name="defaults">
    <str name="defType">${my.custom.prop}</str>
  </lst>
</requestHandler>
```

### 隐式 core 属性

Solr core 存在多个属性作为 “隐式” 属性，它们可用于变量替换，独立于其底层的值是在哪里和如何被初始化的。
例如，不管对某个特定 Solr core 的名称是在 `core.properties` 中被明确地配置，
还是从实例的目录中推断出来的，隐式属性 `solr.core.name` 总是可用作这个 core 的配置文件的变量：

```xml
<requestHandler name="/select">
  <lst name="defaults">
    <str name="collection_name">${solr.core.name}</str>
  </lst>
</requestHandler>
```

所有的隐式属性都使用 `solr.core.` 命名前缀，且反射到等价的
[core.properties](../core/defining_core_properties.md)
中的属性的运行时值：

* solr.core.name
* solr.core.config
* solr.core.schema
* solr.core.dataDir
* solr.core.transient
* solr.core.loadOnStartup
