# 用 DataImportHandler 上传结构型数据存储的数据

很多搜索应用都将其待索引内容存储在结构化数据存储中，如关系型数据库。
数据导入处理器(DIH) 提供了从一个数据存储中导入内容并索引它的机制。
除了关系型数据库， DIH 还可以索引来自像 RSS 和 ATOM 种子这样基于 HTTP 的数据源的内容、
e-mail 仓库以及有一个 XPath 处理其可用于生成字段的结构化的 XML。

`example/example-DIH` 目录包含了多种数据导入处理器特性的集合。
要运行 `dih` 示例：

```bash
bin/solr -e dih
```

更多数据导入处理器的信息，查看 https://wiki.apache.org/solr/DataImportHandler 。

本节包含的内容有：

* [概念和术语](#concepts)
* [配置](#configuration)
* [数据导入处理器命令](#commands)
* [属性 Writer](#prop-writer)
* [数据源](#data-sources)
* [实体处理器](#entity-processors)
* [转换器](#transformers)
* [数据导入处理器的特殊命令](#special-commands)

## <a name="concepts"></a>概念和术语

数据导入处理器的描述以特殊的方式使用了多个类似的词，如实体(entity)和处理器(processor)，
其解释如下表。

|词项          |定义                  |
|:------------|:--------------------|
|Datasource   |数据源定义了感兴趣的数据的位置。对于数据库，它是一个 DSN。对于 HTTP 数据源，它是基本 URL。 |
|Entity       |概念上，实体会被处理以生成包含多个字段的文档的集合，它们(可以选择以多种不同的方式进行转换)会被发送给 Solr 进行索引。对一个 RDBMS 数据源，一个实体是一个视图或者表，它可能会被一或多个 SQL 语句处理来生成一系列具有一或多个列(字段)的行(文档)。 |
|Processor    |一个实体处理器会完成整个从数据源抽取内容、转换并将其添加到索引的工作。可以用自定义实体处理器扩展或替换提供的处理器。 |
|Transformer  |从实体中取得的每个字段的集都可以选择进行转换。这个过程可以修改字段、创建新字段或者从单个行生成多个行/文档。DIH 中内置了多个转换器，它们可以执行像修改日期、剥离 HTML 等功能。可以使用公共可用的接口来编写自定义转换器。 |

## <a name="configuration"></a>配置

### 配置 solrconfig.xml

数据导入处理器需注册在 `solrconfig.xml` 中。如：

```xml
<requestHandler name="/dataimport"
  class="org.apache.solr.handler.dataimport.DataImportHandler">
  <lst name="defaults">
    <str name="config">/path/to/my/DIHconfigfile.xml</str>
  </lst>
</requestHandler>
```

其唯一必需的参数是 `config` 参数，它指定了 DIH 配置文件的位置。
DIH 配置文件中包含了数据源规格、如何获取数据、获取什么数据以及如何处理
以生成可被发送给索引的 Solr 文档。

你可以有多个 DIH 配置文件。
每个文件需要在 `solrconfig.xml` 中有独立的定义，指定其文件路径。

### 配置 DIH 配置文件

一个 `dih` 示例中基于 `db` 集合的带注释的配置文件如下
(`example/example-DIH/solr/db/conf/db-data-config.xml`)。
这个模式中，它从一个定义的简单的产品的数据库的四个表中提取字段。
这里更多参数和选项的信息会在后续小节中阐述。

```xml
<dataConfig>
<!-- The first element is the dataSource, in this case an HSQLDB database.
     The path to the JDBC driver and the JDBC URL and login credentials are all
specified here.
     Other permissible attributes include whether or not to autocommit to Solr,the
batchsize
     used in the JDBC connection, a 'readOnly' flag -->

  <dataSource driver="org.hsqldb.jdbcDriver"
    url="jdbc:hsqldb:./example-DIH/hsqldb/ex" user="sa" />
<!-- a 'document' element follows, containing multiple 'entity' elements.
     Note that 'entity' elements can be nested, and this allows the entity
     relationships in the sample database to be mirrored here, so that we can
     generate a denormalized Solr record which may include multiple features
     for one item, for instance -->
  <document>

<!-- The possible attributes for the entity element are described below.
     Entity elements may contain one or more 'field' elements, which map
     the data source field names to Solr fields, and optionally specify
     per-field transformations -->
<!-- this entity is the 'root' entity. -->
    <entity name="item" query="select * from item"
      deltaQuery="select id from item where last_modified > '${dataimporter.last_index_time}'">
      <field column="NAME" name="name" />

<!-- This entity is nested and reflects the one-to-many relationship between an item
and its multiple features.
     Note the use of variables; ${item.ID} is the value of the column 'ID' for the
current item
     ('item' referring to the entity name) -->
      <entity name="feature"
        query="select DESCRIPTION from FEATURE where ITEM_ID='${item.ID}'"
        deltaQuery="select ITEM_ID from FEATURE where last_modified > '${dataimporter.last_index_time}'"
        parentDeltaQuery="select ID from item where ID=${feature.ITEM_ID}">
        <field name="features" column="DESCRIPTION" />
      </entity>
      <entity name="item_category"
        query="select CATEGORY_ID from item_category where ITEM_ID='${item.ID}'"
        deltaQuery="select ITEM_ID, CATEGORY_ID from item_category where last_modified > '${dataimporter.last_index_time}'"
        parentDeltaQuery="select ID from item where ID=${item_category.ITEM_ID}">
        <entity name="category"
        query="select DESCRIPTION from category where ID = '${item_category.CATEGORY_ID}'"
        deltaQuery="select ID from category where last_modified > '${dataimporter.last_index_time}'"
        parentDeltaQuery="select ITEM_ID, CATEGORY_ID from item_category where CATEGORY_ID=${category.ID}">
          <field column="description" name="cat" />
        </entity>
      </entity>
    </entity>
  </document>
</dataConfig>
```

数据源也可以在 `solrconfig.xml` 中指定。
这些必须在 `solrconfig.xml` 中该处理器的默认部分中指定。
然而它们会在配置被加载时才会被解析。

整个配置本身可以在请求参数中一 `dataconfig` 参数传入，而不是使用一个文件。
当配置出现错误时，其错误消息会以 XML 格式返回。

还支持一个 `reload-config` 命令，它在验证新配置文件或者你希望指定文件并加载它而不希望它在导入
时重新加载等情况下会很有用。
若配置中存在 `xml` 错误，会返回一个友好的 XML 格式的消息。
然后你可以修复问题，重新 `reload-config`。

> 你也可以查看 Solr Admin UI 中的 DIH 配置，那里有一个导入内容的接口。

## <a name="commands"></a>数据导入处理器命令

DIH 命令通过 HTTP 请求发送给 Solr。下面是支持的操作：

|命令          |描述                           |
|-------------|------------------------------ |
|abort        |取消一个进行的操作。其 URL 是 `http://<host>:<port>/solr/<collection_name>/dataimport?command=abort`|
|delta-import |用于增量导入和变更侦测。其命令形式是 `http://<host>:<port>/solr/<collection_name>/dataimport?command=delta-import`。它支持和完全导入一样的 `clean/commit/optimize/debug`参数。 |
|full-import  |完全导入操作可以以 `http://<host>:<port>/solr/<collection_name>/dataimport?command=full-import` 格式的 URL开始。该命令会立即返回。其操作会启动一个新线程且其响应中的 *status* 属性应该显示为 *busy*。该操作可能会根据数据集大小花费一些时间。完全导入时对 Solr 的查询不会阻塞。当一个完全导入命令被执行时，它会在位于 `conf/dataimport.properties` 的文件中存储操作的起始时间。这个存储的时间戳在 `delta-import` 操作被执行时使用。该命令可接收的参数列表，请查看下文。 |
|reload-config|若配置有变更且你希望不用重启 Solr 就重加载它，可以运行命令 `http://<host>:<port>/solr/<collection_name>/dataimport?command=reload-config` |
|status       |其 URL 为 `http://<host>:<port>/solr/<collection_name>/dataimport?command=status`。它会返回创建、删除、查询运行、获取行数、状态等统计信息。 |

### full-import 命令的参数

`full-import` 命令接收以下参数：

|参数      |描述                                |
|---------|-----------------------------------|
|clean    |默认为 true。指示在索引开始时是否清理索引。|
|commit   |默认为 true。指示在操作后是否提交。      |
|debug    |默认为 false。以 debug 模式运行命令。用于交互式开发模式。注意在 debug 模式中，文档不会自动提交。若你希望运行 debug 模式，你也需要通过在请求中添加 `commit=ture` 参数来提交结果。|
|entity   |直接在配置文件的 `<document>` 标签的实体名称。使用它可以选择性地执行一或多个实体。可以一次性传入多个 `entity` 参数来执行多个实体。若没有该参数，所有的实体都会被执行。 |
|optimize |默认为 true。指示在操作后是否优化。      |

## <a name="prop-writer"></a>属性 Writer

`propertyWriter` 元素定义了在 delta 查询时使用何种日期格式和地区。
这是一个可选配置。直接在 DIH 配置文件的 `dataConfig` 元素下添加该元素。

```xml
<propertyWriter dateFormat="yyyy-MM-dd HH:mm:ss" type="SimplePropertiesWriter"
  directory="data" filename="my_dih.properties" locale="en_US" />
```

可用的参数有：

|参数       |描述                                |
|----------|-----------------------------------|
|dateFormat|用于将日期转换为文本的 `java.text.SimpleDateFormat`， 默认为 "yyyy-MM-dd HH:mm:ss"|
|type      |其实现类。在非 SolrCloud 环境使用 `SimplePropertiesWriter`，在 SolrCloud 环境使用 `ZKPropertiesWriter`。若没有指定，它会根据 SolrCloud 环境是否启用自动默认合适的类。 |
|directory |仅用于`SimplePropertiesWriter`。属性文件的目录。若没有指定，默认为 "conf" |
|filename  |仅用于`SimplePropertiesWriter`。属性文件的名称。若没有指定，默认为 `requestHandler` 的名称(根据 `solrconfig.xml` 中的定义，后缀 `.properties`, 即 `dataimport.properties`)。 |
|locale    |地区。若没有定义，则使用 `ROOT` 地区。它必须是 语言-国家 格式，如 `en-US` |

## <a name="data-sources"></a>数据源

数据源指定了数据的来源及其类型。
有时会使人迷惑的是，某些数据源可以在相关的实体处理器内部配置。
数据源也可以在 `solrconfig.xml` 中指定，它在你有多个环境(如开发、QA和产品环境)仅有数据源地址不同时有用。

你可以通过编写扩展了 `org.apache.solr.handler.dataimport.DataSource` 的类来创建自定义数据源。

数据源定义的必需属性为其名称和类型。名称标识了实体元素对应哪个数据源。

可用的数据源类型如下：

### ContentStreamDataSource

它接收 POST 数据作为数据源。它可以在任何使用了 `DataSource<Reader>` 的 `EntityProcessor` 上使用。

### <a name="FieldReaderDataSource"></a>FieldReaderDataSource

它可用于数据库字段包含了你希望使用 `XPathEntityProcessor` 来处理的 XML 内容时使用。
你可以在配置中同时设置 JDBC 和 FileReader 数据源，如下：

```xml
<dataSource name="a1" driver="org.hsqldb.jdbcDriver" ... />
<dataSource name="a2" type=FieldReaderDataSource" />
<document>

<!-- processor for database -->
  <entity name ="e1" dataSource="a1" processor="SqlEntityProcessor" pk="docid"
    query="select * from t1 ...">
    
    <!-- nested XpathEntity; the field in the parent which is to be used for
         Xpath is set in the "datafield" attribute in place of the "url" attribute
    -->
    
    <entity name="e2" dataSource="a2" processor="XPathEntityProcessor"
      dataField="e1.fieldToUseForXPath">
      <!-- Xpath configuration follows -->
      ...
    </entity>
  </entity>
```

`FieldReaderDataSource` 可以接收一个 `encoding` 参数，它默认为 "UTF-8"。
它必须是 语言-国家 格式，如 `en-US`。

### <a name="FileDataSource"></a>FileDataSource

它可以像 [URLDataSource](#URLDataSource) 一样使用，但它用于从硬盘中的文件获取内容。
它和 URLDataSource 的区别只在于在访问硬盘文件时路径名称是如何指定的。

该数据源接收以下可选属性。

|可选属性   |描述                                |
|---------|-----------------------------------|
|basePath |对其值求值时若不是绝对路径时的相对的基础路径|
|encoding |定义使用的字符编码。默认为 "UTF-8"      |

### JdbcDataSource

默认的数据源。连同 [SqlEntityProcessor](#SqlEntityProcessor) 使用。
查看 [FieldReaderDataSource](#FieldReaderDataSource) 小节的配置示例。

### <a name="URLDataSource"></a>URLDataSource

该数据源常用于 `XPathEntityProcessor` 来从一个底层的 `file://` 或 `http://` 地址获取内容。
如下例：

```xml
<dataSource name="a"
  type="URLDataSource"
  baseUrl="http://host:port/"
  encoding="UTF-8"
  connectionTimeout="5000"
  readTimeout="10000"/>
```

URLDataSource 类型接收以下可选参数：

|可选属性   |描述                                |
|---------|-----------------------------------|
|baseURL  |指定路径名称的新的 baseURL。你可以用它来在 Dev/QA/Prod 环境间指定 host/port 变更。使用这个属性使得不需要改变 `solrconfig.xml` 文件。 |
|connectionTimeout |指定连接超时时间的毫秒值。默认为 5000ms。|
|encoding |响应头中使用的默认编码。可以用它来覆盖默认编码。  |
|readTimeout |指定读操作的超时时间毫秒值。默认为 10000ms。 |

## <a name="entity-processors"></a>实体处理器

实体处理器提取、转换数据并将其添加到 Solr 索引中。
实体的例子包括数据存储中的视图或表。

每个处理器有自己的属性集合，它们会在其自身的小节中描述。
除此之外还有一些对任何实体都通用的属性：

|属性     |用法                |
|--------|--------------------|
|dataSource|数据源名称。若定义有多个数据源，使用该属性来给实体配置数据源名称 |
|name    |必需。用于标识实体的唯一名称。             |
|pk      |实体主键。可选，仅在需要使用 delta-import 时需要。它和 schema.xml 中定义的 uniqueKey 没有关系，但可以是相同的。如果你要做 delta-import 时是必需的，且可以以 `${dataimporter.delta.<column-name>}` 引用字段名以用作主键。|
|processor|Default is SqlEntityProcessor. Required only if the datasource is not RDBMS.|
|onError |Permissible values are (`abort/skip/continue`) . The default value is 'abort'. 'Skip' skips the current document. 'Continue' ignores the error and processing continues.|
|preImportDeleteQuery|Before a full-import command, use this query this to cleanup the index instead of using '*:*'. This is honored only on an entity that is an immediate sub-child of `<document>` .|
|postImportDeleteQuery|Similar to the above, but executed after the import has completed.|
|rootEntity|By default the entities immediately under the `<document>` are root entities. If this attribute is set to false, the entity directly falling under that entity will be treated as the root entity (and so on). For every row returned by the root entity, a document is created in Solr.|
|transformer|Optional. One or more transformers to be applied on this entity.|
|cacheImpl|Optional. A class (which must implement DIHCache ) to use for caching this entity when doing lookups from an entity which wraps it. Provided implementation is "SortedMapBackedCache ".|
|cacheKey|The name of a property of this entity to use as a cache key if cacheImpl is specified.|
|cacheLookup|An entity + property name that will be used to lookup cached instances of this entity if cacheImpl is specified.|

DIH 提供实体的缓存以避免对相同名称实体的重复查看。
默认的 `SortedMapBackedCache` 是一个 `HashMap` 其键是行中的一个字段而其值是具有相同键的一系列行。

下例中，每个 `manufacturer` 实体都使用 `id` 属性作为缓存键进行了缓存。
对每个 `product` 实体的基于产品的 `manu` 属性的查询将查看缓存。
当缓存在特定键上没有数据时，查询会运行并填充缓存。

```xml
<entity name="product" query="select description,sku, manu from product" >
  <entity name="manufacturer" query="select id, name from manufacturer"
    cacheKey="id" cacheLookup="product.manu" cacheImpl="SortedMapBackedCache"/>
</entity>
```

### <a name="SqlEntityProcessor"></a>SqlEntityProcessor

SqlEntityProcessor 是默认的处理器。其关联的 [数据源](#data-sources) 应该为一个 JDBC URL。

该处理器特有的实体属性有：

|属性     |用法                |
|--------|--------------------|
|query   |Required. The SQL query used to select rows.|
|deltaQuery|SQL query used if the operation is delta-import. This query selects the primary keys of the rows which will be parts of the delta-update. The pks will be available to the deltaImportQuery through the variable `${dataimporter.delta.<column-name> }`.|
|parentDeltaQuery|SQL query used if the operation is delta-import.|
|deletedPkQuery|SQL query used if the operation is delta-import.|
|deltaImportQuery|SQL query used if the operation is delta-import. If this is not present, DIH tries to construct the import query by(after identifying the delta) modifying the 'query' (this is error prone).There is a namespace `${dataimporter.delta.<column-name> }` which can be used in this query. For example, `select * from tbl where id=${dataimporter.delta.id }`.|

### <a name="XPathEntityProcessor"></a>XPathEntityProcessor

该处理器用于索引 XML 格式的数据。其数据源通常是 [URLDataSource](#URLDataSource)
或 [FileDataSource](#FileDataSource)。
Xpath 也可用于下面的 [FileListEntityProcessor](#FileListEntityProcessor),
来从每个文件中那个生成文档。

该处理器特有的实体属性有：

|属性     |用法                |
|--------|--------------------|
|Processor|Required. Must be set to "XpathEntityProcessor".|
|url     |Required. HTTP URL or file location.|
|stream  |Optional: Set to true for a large file or download.|
|forEach |Required unless you define useSolrAddSchema . The Xpath expression which demarcates each record. This will be used to set up the processing loop.|
|xsl     |Optional: Its value (a URL or filesystem path) is the name of a resource used as a preprocessor for applying the XSL transformation.|
|useSolrAddSchema|Set this to true if the content is in the form of the standard Solr update XML schema.|
|flatten |Optional: If set true, then text from under all the tags is extracted into one field. |

实体中的每个字段元素都可以有以下属性以及默认属性。

|属性     |用法                |
|--------|--------------------|
|xpath   |Required. The XPath expression which will extract the content from the record for this field.Only a subset of Xpath syntax is supported.|
|commonField|Optional. If true, then when this field is encountered in a record it will be copied to future records when creating a Solr document.|

下面是 `dih` 示例中 `rss` 集合的配置(`example/example-DIH/solr/rss/conf/rss-data-config.xml`):

```xml
<!-- slashdot RSS Feed --->
<dataConfig>
  <dataSource type="HttpDataSource" />
  <document>
    <entity name="slashdot"
      pk="link"
      url="http://rss.slashdot.org/Slashdot/slashdot"
      processor="XPathEntityProcessor"

<!-- forEach sets up a processing loop ; here there are two expressions-->
      forEach="/RDF/channel | /RDF/item"
      transformer="DateFormatTransformer">
      <field column="source" xpath="/RDF/channel/title" commonField="true" />
      <field column="source-link" xpath="/RDF/channel/link" commonField="true"/>
      <field column="subject" xpath="/RDF/channel/subject" commonField="true" />
      <field column="title" xpath="/RDF/item/title" />
      <field column="link" xpath="/RDF/item/link" />
      <field column="description" xpath="/RDF/item/description" />
      <field column="creator" xpath="/RDF/item/creator" />
      <field column="item-subject" xpath="/RDF/item/subject" />
      <field column="date" xpath="/RDF/item/date"
        dateTimeFormat="yyyy-MM-dd'T'hh:mm:ss" />
      <field column="slash-department" xpath="/RDF/item/department" />
      <field column="slash-section" xpath="/RDF/item/section" />
      <field column="slash-comments" xpath="/RDF/item/comments" />
    </entity>
  </document>
</dataConfig>
```

# <a name="MailEntityProcessor"></a>MailEntityProcessor

MailEntityProcessor 使用 Java Mail API 来使用 IMAP 协议索引 email 消息。
MailEntityProcessor 通过使用用户名密码来连接到特定的邮箱，
获取每个消息的 email 头部，然后获取完整的 email 内容来构造一个文档(每个邮件一个文档)。

下面是 `dih` 示例中 `mail` 集合的配置(`example/example-DIH/solr/mail/conf/mail-data-config.xml`):

```xml
<dataConfig>
  <document>
    <entity processor="MailEntityProcessor"
      user="email@gmail.com"
      password="password"
      host="imap.gmail.com"
      protocol="imaps"
      fetchMailsSince="2009-09-20 00:00:00"
      batchSize="20"
      folders="inbox"
      processAttachement="false"
      name="sample_entity"/>
  </document>
</dataConfig>
```

MailEntityProcessor 独有的实体属性有：

|属性      |用法                |
|---------|--------------------|
|processor|Required. Must be set to "MailEntityProcessor".|
|user     |Required. Username for authenticating to the IMAP server; this is typically the email address of the mailbox owner|
|password |Required. Password for authenticating to the IMAP server|
|host     |Required. The IMAP server to connect to|
|protocol |Required. The IMAP protocol to use, valid values are: imap, imaps, gimap, and gimaps|
|fetchMailsSince|Optional. Date/time used to set a filter to import messages that occur after the specified date; expected format is: yyyy-MM-dd HH:mm:ss|
|folders  |Required. Comma-delimited list of folder names to pull messages from, such as "inbox"|
|recurse  |Optional (default is true). Flag to indicate if the processor should recurse all child folders when looking for messages to import|
|include  |Optional. Comma-delimited list of folder patterns to include when processing folders (can be a literal value or regular expression)|
|exclude  |Optional. Comma-delimited list of folder patterns to exclude when processing folders (can be a literal value or regular expression); excluded folder patterns take precedent over include folder patterns.|
|processAttachement/processAttachements|Optional (default is true). Use Tika to process message attachments.|
|includeContent|Optional (default is true). Include the message body when constructing Solr documents for indexing|

#### 仅导入新邮件

After running a full import, the MailEntityProcessor keeps track of the timestamp of the previous import so that
subsequent imports can use the fetchMailsSince filter to only pull new messages from the mail server. This
occurs automatically using the Data Import Handler dataimport.properties file (stored in conf). For instance, if you
set fetchMailsSince=2014-08-22 00:00:00 in your mail-data-config.xml, then all mail messages that occur after
this date will be imported on the first run of the importer. Subsequent imports will use the date of the previous
import as the fetchMailsSince filter, so that only new emails since the last import are indexed each time.

#### GMail 扩展

When connecting to a GMail account, you can improve the efficiency of the MailEntityProcessor by setting the
protocol to **gimap** or **gimaps** . This allows the processor to send the fetchMailsSince filter to the GMail server to
have the date filter applied on the server, which means the processor only receives new messages from the
server. However, GMail only supports date granularity, so the server-side filter may return previously seen
messages if run more than once a day.

### <a name="TikaEntityProcessor"></a>TikaEntityProcessor

TikaEntityProcessor 使用 Apache Tika 来处理传入的文档。
它和 [用带 Apache Tika 的 SolrCell 上传数据](./solr_cell.md) 类似，
但使用的是 DataImportHandler 选项。

下面是 `dih` 示例中 `tika` 集合的配置(`example/example-DIH/solr/tika/conf/tika-data-config.xml`):

```xml
<dataConfig>
  <dataSource type="BinFileDataSource" />
  <document>
    <entity name="tika-test" processor="TikaEntityProcessor"
      url="../contrib/extraction/src/test-files/extraction/solr-word.pdf"
      format="text">
      <field column="Author" name="author" meta="true"/>
      <field column="title" name="title" meta="true"/>
      <field column="text" name="text"/>
    </entity>
  </document>
</dataConfig>
```

该处理器特有的实体属性有：

|属性      |用法                |
|---------|--------------------|
|dataSource|<p>This parameter defines the data source and an optional name which can be referred to in later parts of the configuration if needed. This is the same dataSource explained in the description of general entity processor attributes above.</p><p>The available data source types for this processor are:</p><ul><li>BinURLDataSource: used for HTTP resources, but can also be used for files.</li><li>BinContentStreamDataSource: used for uploading content as a stream.</li><li>BinFileDataSource: used for content on the local filesystem.</li></ul>|
|url      |The path to the source file(s), as a file path or a traditional internet URL. This parameter is required.|
|htmlMapper|Allows control of how Tika parses HTML. The "default" mapper strips much of the HTML from documents while the "identity" mapper passes all HTML as-is with no modifications. If this parameter is defined, it must be either default or identity ; if it is absent, "default" is assumed.|
|format   |The output format. The options are text , xml , html or none . The default is "text" if not defined. The format "none" can be used if metadata only should be indexed and not the body of the documents.|
|parser   |The default parser is `org.apache.tika.parser.AutoDetectParser` . If a custom or other parser should be used, it should be entered as a fully-qualified name of the class and path.|
|fields   |The list of fields from the input documents and how they should be mapped to Solr fields. If the attribute meta is defined as "true", the field will be obtained from the metadata of the document and not parsed from the body of the main text.|
|extractEmbedded|Instructs the TikaEntityProcessor to extract embedded documents or attachments when true . If false, embedded documents and attachments will be ignored.|
|onError  |By default, the TikaEntityProcessor will stop processing documents if it finds one that generates an error. If you define onError to "skip", the TikaEntityProcessor will instead skip documents that fail processing and log a message that the document was skipped.|

### <a name="FileListEntityProcessor"><a>FileListEntityProcessor

该处理器基本就是一个封装器，它设计用于生成一系列满足由属性指定的条件的文件，
然后这些文件可以被传递给另一个处理器，比如 [XPathEntityProcessor](#XPathEntityProcessor)。
该处理器的实体信息将内嵌在 `FileListEnitity` 实体中。
它会生成四个隐式的字段：`fileAbsolutePath`、`fileSize`、`fileLastModified`、`fileName`,
它们可用在内嵌的处理器中。该处理器没有使用数据源。

该处理器特有的属性有：

|属性      |用法                |
|---------|--------------------|
|fileName |Required. A regular expression pattern to identify files to be included.|
|basedir  |Required. The base directory (absolute path).|
|recursive|Whether to search directories recursively. Default is 'false'.|
|excludes |A regular expression pattern to identify files which will be excluded.|
|newerThan|A date in the format `yyyy-MM-ddHH:mm:ss` or a date math expression ( `NOW - 2YEARS` ).|
|olderThan|A date, using the same formats as newerThan.|
|rootEntity|This should be set to false. This ensures that each row (filepath) emitted by this processor is considered to be a document.|
|dataSource|Must be set to null. |

下面的示例组合了 FileListEntityProcessor 和另一个会针对每个找到的文件生成一系列字段的处理器。

```xml
<dataConfig>
  <dataSource type="FileDataSource"/>
  <document>
    <!-- this outer processor generates a list of files satisfying the conditions
         specified in the attributes -->
    <entity name="f" processor="FileListEntityProcessor"
      fileName=".*xml"
      newerThan="'NOW-30DAYS'"
      recursive="true"
      rootEntity="false"
      dataSource="null"
      baseDir="/my/document/directory">
      
      <!-- this processor extracts content using Xpath from each file found -->
      <entity name="nested" processor="XPathEntityProcessor"
        forEach="/rootelement" url="${f.fileAbsolutePath}" >
        <field column="name" xpath="/rootelement/name"/>
        <field column="number" xpath="/rootelement/number"/>
      </entity>
    </entity>
  </document>
</dataConfig>
```

### <a name="LineEntityProcessor"></a>LineEntityProcessor

LineEntityProcessor 会一行一行地读取数据源中的内容，并对每个读取的行返回名为 `rawLine` 的字段。
其内容不会被解析；然而你可以添加转换器来操作 `rawLine` 字段的数据，或创建其它额外的字段。

读取的行可由两个属性 `acceptLineRegex` 和 `omitLineRegex` 上指定的正则表达式进行过滤。
下表描述了 LineEntityProcessor 的属性：

|属性          |描述                  |
|-------------|---------------------|
|url          |A required attribute that specifies the location of the input file in a way that is compatible with the configured data source. If this value is relative and you are using FileDataSource or URLDataSource, it assumed to be relative to baseLoc.|
|acceptLineRegex|An optional attribute that if present discards any line which does not match the regExp.|
|omitLineRegex|An optional attribute that is applied after any acceptLineRegex and that discards any line which matches this regExp.|

例如：

```xml
<entity name="jc"
  processor="LineEntityProcessor"
  acceptLineRegex="^.*\.xml$"
  omitLineRegex="/obsolete"
  url="file:///Volumes/ts/files.lis"
  rootEntity="false"
  dataSource="myURIreader1"
  transformer="RegexTransformer,DateFormatTransformer">
  ...
```

While there are use cases where you might need to create a Solr document for each line read from a file, it is
expected that in most cases that the lines read by this processor will consist of a pathname, which in turn will be
consumed by another EntityProcessor, such as XPathEntityProcessor.

### <a name="PlainTextEntityProcessor"></a>PlainTextEntityProcessor

PlainTextEntityProcessor 读取数据源中的所有内容为单个隐式字段 `plainText`。
其内容不会被解析；然而你可以添加转换器来操作 `plainText` 字段的数据，或创建其它额外的字段。

例如：

```xml
<entity processor="PlainTextEntityProcessor" name="x" url="http://abc.com/a.txt"
  dataSource="data-source-name">
  <!-- copies the text to a field called 'text' in Solr-->
  <field column="plainText" name="text"/>
</entity>
```

确保数据源是 `DataSource<Reader>`(`FileDataSource`、`URLDataSource`) 类型的。

## <a name="transformers"></a>转换器

转换器可以操作有实体返回的文档中字段。一个转换器可以创建新的字段或者修改已有的字段。
你需要通过在 `<entity>` 元素上添加一个包含逗号分割的属性来告诉实体你的导入操作将使用哪些转换器。

```xml
<entity name="abcde" transformer="org.apache.solr....,my.own.transformer,..." />
```

特定的转换规则随后被添加作为 `<field>` 元素的属性，如下所示。
多个转换器以其在 `transformer` 属性所指定的顺序被应用。

数据导入处理器包含多个内置的转换器。
你也可以按 [相关 Solr Wiki](http://wiki.apache.org/solr/DIHCustomTransformer) 所述
编写自定义的转换器。`ScriptTransformer` (下文会描述) 也提供了编写你自己的转换器的替代方法。

Solr 包含以下内置转换器：

|转换器名称     |用法                  |
|-------------|---------------------|
|[ClobTransformer](#ClobTransformer)|Used to create a String out of a Clob type in database.|
|[DateFormatTransformer](#DateFormatTransformer)|Parse date/time instances.|
|[HTMLStripTransformer](#HTMLStripTransformer)|Strip HTML from a field.|
|[LogTransformer](#LogTransformer)|Used to log data to log files or a console.|
|[NumberFormatTransformer](#NumberFormatTransformer)|Uses the NumberFormat class in java to parse a string into a number.|
|[RegexTransformer](#RegexTransformer)|Use regular expressions to manipulate fields.|
|[ScriptTransformer](#ScriptTransformer)|Write transformers in Javascript or any other scripting language supported by Java.|
|[TemplateTransformer](#TemplateTransformer)|Transform a field using a template.|

这些转换器会在下文描述。

### <a name="ClobTransformer"></a>ClobTransformer

你可以使用 ClobTransformer 从数据库中的 CLOB 创建一个字符串。
CLOB 是字符大对象：字符数据的集合，通常存储在一个由数据库指向的独立的位置。
详见 http://en.wikipedia.org/wiki/Character_large_object。
下面是调用 ClobTransformer 的例子：

```xml
<entity name="e" transformer="ClobTransformer" ...>
  <field column="hugeTextField" clob="true" />
  ...
</entity>
```

ClobTransformer 接收这些参数：

|属性          |描述                  |
|-------------|---------------------|
|clob         |Boolean value to signal if ClobTransformer should process this field or not. If this attribute is omitted, then the corresponding field is not transformed.|
|sourceColName|The source column to be used as input. If this is absent source and target are same|

### <a name="DateFormatTransformer"></a>DateFormatTransformer

该转换器将日期从一种格式转换为另一种格式。这在像你需要将一个字段从完整的日期/时间格式
转换成低精度的日期格式时很有用，可用于 faceting。

DateFormatTransformer 仅应用于带有 `dateTimeFormat` 属性的字段。
其他字段不会修改。

该转换器能识别以下属性：

|属性          |描述                  |
|-------------|---------------------|
|dateTimeFormat|The format used for parsing this field. This must comply with the syntax of the Java [SimpleDateFormat](http://docs.oracle.com/javase/7/docs/api/java/text/SimpleDateFormat.html) class.|
|sourceColName|The column on which the dateFormat is to be applied. If this is absent source and target are same.|
|locale       |The locale to use for date transformations. If not specified, the ROOT locale will be used. It must be specified as language-country. For example, en-US .|

下面的代码会将日期截取为月份 "2007-JUL":

```xml
<entity name="en" pk="id" transformer="DateFormatTransformer" ... >
  ...
  <field column="date" sourceColName="fulldate" dateTimeFormat="yyyy-MMM"/>
</entity>
```

### <a name="HTMLStripTransformer"></a>HTMLStripTransformer

你可以使用该转换器来剥离一个字段的 HTML。如：

```xml
<entity name="e" transformer="HTMLStripTransformer" ... >
  <field column="htmlText" stripHTML="true" />
  ...
</entity>
```

该转换器只有一个属性，`stripHTML`,它是一个布尔值，表示 HTMLStripTransformer 是否应该处理该字段。

### <a name="LogTransformer"></a>LogTransformer

你可以使用该转换器将数据记录日志到控制台或者日志文件。如：

```xml
<entity ...
  transformer="LogTransformer"
  logTemplate="The name is ${e.name}" logLevel="debug">
   ....
</entity>
```

不像其它转换器，LogTransformer 不是应用于任何字段上的，因此这些属性应用在实体本身。

### <a name="NumberFormatTransformer"></a>NumberFormatTransformer

该转换器从一个字符串中解析成一个数字，将其转换成特殊格式，并可选使用一个不同的地区。

NumberFormatTransformer 只会应用到具有 `formatStyle` 属性的字段。

该转换器识别以下属性：

|属性          |描述                  |
|-------------|---------------------|
|formatStyle  |The format used for parsing this field. The value of the attribute must be one of (`number|percent|integer|currency` ). This uses the semantics of the Java NumberFormat class.|
|sourceColName|The column on which the NumberFormat is to be applied. This is attribute is absent. The source column and the target column are the same.|
|locale       |The locale to be used for parsing the strings. If this is absent, the ROOT locale is used. It must be specified as language-country. For example, en-US .|

如：

```xml
<entity name="en" pk="id" transformer="NumberFormatTransformer" ...>
  ...
  
  <!-- treat this field as UK pounds -->
  
  <field name="price_uk" column="price" formatStyle="currency" locale="en-UK"/>
</entity>
```

### <a name="RegexTransformer"></a>RegexTransformer

RegexTransformer 使用正则表达式来帮助提取或操作字段(来源)中的值。
其实际类名为 `org.apache.solr.handler.dataimport.RegexTransformer`，
但因为它属于默认包，因此包名可以忽略。

下表描述了正则转换器所识别的属性。

|属性          |描述                  |
|-------------|---------------------|
|regex        |The regular expression that is used to match against the column or sourceColName's value(s). If replaceWith is absent, each regex group is taken as a value and a list of values is returned.|
|sourceColName|The column on which the regex is to be applied. If not present, then the source and target are identical.|
|splitBy      |Used to split a string. It returns a list of values.|
|groupNames   |A comma separated list of field column names, used where the regex contains groups and each group is to be saved to a different field. If some groups are not to be named leave a space between commas.|
|replaceWith  |Used along with regex . It is equivalent to the method `new String(<sourceColVal>).replaceAll(<regex>, <replaceWith>)` .|

下面是配置正则转换器的示例：

```xml
<entity name="foo" transformer="RegexTransformer"
  query="select full_name, emailids from foo">
  <field column="full_name"/>
  <field column="firstName" regex="Mr(\w*)\b.*" sourceColName="full_name"/>
  <field column="lastName" regex="Mr.*?\b(\w*)" sourceColName="full_name"/>
  
  <!-- another way of doing the same -->
  <field column="fullName" regex="Mr(\w*)\b(.*)" groupNames="firstName,lastName"/>
  <field column="mailId" splitBy="," sourceColName="emailids"/>
</entity>
```

本例中，`regex` 和 `sourceColName` 是由转换器所使用的特定属性。
该转换器从结果集中读取 `full_name` 字段并将其转换为两个新的目标字段 `firstName` 和 `lastName`。
尽管查询仅返回一个列 `full_name`，在结果集中 Solr 文档得到了两个额外的"派生"字段 `firstName` 和 `lastName`。
这些新字段仅在正则表达式匹配时创建。

表中的 `emailids` 字段可能是一个逗号分割的值。
它最终会生成一或多个 email IDs，且我们假设 `mailId` 在 Solr 中是多值字段。

注意这个转换器既可以将一个字符串根据 splitBy 模式分割为多个分词，
也可以用 replaceWith 做字符串替换，还可以给一组具有某个模式的分组赋予一系列 `groupNames`。
它根据上面属性中的 `splitBy`、`replaceWith`以及`groupNames` 来决定具体应该怎么做，
这三个属性是按照顺序查询的。
第一个找到的会被响应，其他非相关的属性会被忽略。

### <a name="ScriptTransformer"></a>ScriptTransformer

脚本转换器允许以任何 Java 支持的脚本语言来书写任意的转换函数，
如 Javascript、JRuby、Jython、Groovy 或 BeanShell。
Javascript 已经集成进了 Java 7 中，其他语言你需要自己集成。

每个函数接收一个行变量(对应于 Java 的 `Map<String,Object>`，因此允许 get、put、remove 操作)。
这样你可以修改已有字段的值或者添加新字段。函数的返回值及被返回的对象。

脚本需要作为 DIH 配置文件的顶级元素插入，且会对每一个行调用一次。

下面是一个简单示例：

```xml
<dataconfig>

  <!-- simple script to generate a new row, converting a temperature from Fahrenheit
       to Centigrade -->
  <script><![CDATA[
    function f2c(row) {
      var tempf, tempc;
      tempf = row.get('temp_f');
      if (tempf != null) {
        tempc = (tempf - 32.0)*5.0/9.0;
        row.put('temp_c', temp_c);
      }
      return row;
    }
  ]]></script>
  
  <document>
    <!-- the function is specified as an entity attribute -->
    
    <entity name="e1" pk="id" transformer="script:f2c" query="select * from X">
      ....
    </entity>
  </document>
</dataConfig>
```

### <a name="TemplateTransformer"></a>TemplateTransformer

你可以使用模板转换器来构造或修改一个字段值，或许是使用其它字段的值。
你可以在模板中插入额外的文本。

```xml
<entity name="en" pk="id" transformer="TemplateTransformer" ...>
  ...
  <!-- generate a full address from fields containing the component parts -->
  <field column="full_address" template="${en.street},${en.city},${en.zip}" />
</entity>
```

## <a name="special-commands"></a>数据导入处理器的特殊命令

你可以通过添加下面列出的任何变量给由任何组件所返回的行上来给 DIH 传递特殊命令：

|变量          |描述                       |
|-------------|--------------------------|
|$skipDoc     |Skip the current document; that is, do not add it to Solr. The value can be the string `true|false` .|
|$skipRow     |Skip the current row. The document will be added with rows from other entities. The value can be the string `true|false`|
|$docBoost    |Boost the current document. The boost value can be a number or the toString conversion of a number.|
|$deleteDocById|Delete a document from Solr with this ID. The value has to be the uniqueKey value of the document.|
|$deleteDocByQuery|Delete documents from Solr using this query. The value must be a Solr Query. |
