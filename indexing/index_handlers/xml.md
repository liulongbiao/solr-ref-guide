# XML 格式索引更新

索引更新命令可以通过使用 `Content-type: application/xml` 或 `Content-type: text/xml`
以 XML 消息的形式发送给更新处理器。

## 添加文档

由更新处理器所识别的用于添加文档的 XML 模式非常直接

* `<add>` 元素引入了一或多个待添加的文档
* `<doc>` 元素引入了组成文档的字段
* `<field>` 元素表示特定字段的内容

如：

```xml
<add>
    <doc>
        <field name="authors">Patrick Eagar</field>
        <field name="subject">Sports</field>
        <field name="dd">796.35</field>
        <field name="numpages">128</field>
        <field name="desc"></field>
        <field name="price">12.40</field>
        <field name="title" boost="2.0">Summer of the all-rounder: Test and championship cricket in England 1982</field>
        <field name="isbn">0002166313</field>
        <field name="yearpub">1982</field>
        <field name="publisher">Collins</field>
    </doc>
    <doc boost="2.5">
    ...
    </doc>
</add>
```

每个元素都具有可以指定的特定的可选属性。

|命令     |可选参数    |参数描述                      |
|--------|-----------|-----------------------------|
|`<add>` |commitWithin=*number* |在指定毫秒时间内添加文档 |
|`<add>` |overwrite=*boolean* |默认为 true。表示是否需要检查唯一键约束来覆盖相同文档的以前版本（如下）|
|`<doc>` |boost=*float* |默认为 1.0。设置文档的加权值。要学习关于加权的信息，查看 [搜索](../../searching/readme.md) |
|`<field>` |boost=*float* |默认为 1.0。设置字段的加权值。 |

若文档模式定义了一个唯一键，则默认一个添加文档的 `/update` 操作将覆盖(替换)索引中
任何具有相同唯一键的文档。若没有定义唯一键，索引的性能会快一些，
因为不需要检查是否需要替换已有文档。

若存在唯一键字段，但你非常自信可以安全地通过唯一性检查(如在你批量构建索引时，或者你的索引代码确保不会多次添加相同文档)，
你可以在添加文档中指定 `overwrite="false"` 选项。

## XML 更新命令

### Commit 和 Optimize 操作

`<commit>` 操作将上次提交以来的所有加载的文档写入到硬盘上的一或多个片段文件中。
在 commit 被触发之前，新索引的内容对搜索来讲是不可见的。
commit 操作开启了一个新的搜索器，并且会触发已配置的任何事件监听器。

Commits 可以通过 `<commit>` 消息明确地触发，也可以被 `solrconfig.xml` 文件中的 `<autocommit>` 参数自动触发。

`<optimize>` 操作请求 Solr 合并内部数据结构以提升搜索性能。
对一个大的索引，优化会需要一定的时间，但通过将多个小片段文件合并成一个大文件，搜索性能将会提高。
若你正在使用 Solr 的复制机制来在多个系统间分发搜索，需注意在一次优化后，一个完整的索引将需要被传输。
通常，commit 后再传输通常更小一些。

`<commit>` 和 `<optimize>` 元素支持以下可选属性:

|可选属性        |描述                         |
|---------------|-----------------------------|
|waitSearcher   |默认为 true。阻塞直到一个新的搜索器被打开并注册为主查询搜索器，以使得变更可见。 |
|expungeDeletes |(commit only) 默认为 false。合并具有超过 10% 已删除文档的片段，并在此过程中清除它们。 |
|maxSegments    |(optimize only) 默认为 1 。合并片段直到片段数量不超过该数量。 |

以下是使用了可选属性的 `<commit>` 和 `<optimize>` 示例：

```xml
<commit waitSearcher="false"/>
<commit waitSearcher="false" expungeDeletes="true"/>
<optimize waitSearcher="false"/>
```

### Delete 操作

文档可以有两种方式从索引中移除。"Delete by ID" 根据指定的 ID 删除文档，但只能用在模式中定义了唯一键的情况。
"Delete by Query" 删除所有匹配指定查询的文档，但对根据查询删除而言 `commitWithin` 是被忽略的。
单个删除消息可以包含多个删除操作。

```xml
<delete>
    <id>0002166313</id>
    <id>0031745983</id>
    <query>subject:sport</query>
    <query>publisher:penguin</query>
</delete>
```

### Rollback 操作

回滚操作会回滚所有在上次 commit 之后的所有对索引的添加和删除操作。
它既不会调用任何事件监听器也不会创建新的搜索器。其语法非常简单： `<rollback/>`。

## 使用 curl 来执行更新

你可以使用 `curl` 工具来执行任何上述命令，使用其 `--data-binary` 选项来追加 XML 消息给 `curl` 命令，
然后生成一个 HTTP POST 请求。如：

```
curl http://localhost:8983/solr/my_collection/update -H "Content-Type: text/xml"
--data-binary '
<add>
  <doc>
    <field name="authors">Patrick Eagar</field>
    <field name="subject">Sports</field>
    <field name="dd">796.35</field>
    <field name="isbn">0002166313</field>
    <field name="yearpub">1982</field>
    <field name="publisher">Collins</field>
  </doc>
</add>'
```

对提交包含在文件中的 XML 消息，你可以使用替代的形式：

```bash
curl http://localhost:8983/solr/my_collection/update -H "Content-Type: text/xml"
--data-binary @myfile.xml
```

短请求也可以使用 HTTP GET 命令来发送，通过 URL 编码来编码请求，如下。注意 "<" 和 ">" 的转码：

```bash
curl http://localhost:8983/solr/my_collection/update?stream.body=%3Ccommit/%3E
```

来自 Solr 的响应具有以下格式：

```xml
<response>
  <lst name="responseHeader">
    <int name="status">0</int>
    <int name="QTime">127</int>
  </lst>
</response>
```

状态字段在失败情况下将不是零。

## 使用 XSLT 来转换 XML 索引更新

`UpdateRequestHandler` 让你可以通过使用 `<tr>` 参数来应用一个
[XSL 转换](https://en.wikipedia.org/wiki/XSLT) 来索引任意的 XML。
在 [Config Sets](../../config/core/config_sets.md) 的 `conf/xslt` 目录下必须具有一个 XSLT 样式表
可以将输入的数据转换成期待的  `<add><doc/></add>` 格式，
并且你需要使用 `tr` 参数来指定对应样式表的名称。

以下是 XSLT 样式表的示例：

```xml
<xsl:stylesheet version='1.0' xmlns:xsl='http://www.w3.org/1999/XSL/Transform'>
  <xsl:output media-type="text/xml" method="xml" indent="yes"/>
  <xsl:template match='/'>
    <add>
      <xsl:apply-templates select="response/result/doc"/>
    </add>
  </xsl:template>
  <!-- Ignore score (makes no sense to index) -->
  <xsl:template match="doc/*[@name='score']" priority="100">
  </xsl:template>
  <xsl:template match="doc">
    <xsl:variable name="pos" select="position()"/>
    <doc>
      <xsl:apply-templates>
        <xsl:with-param name="pos"><xsl:value-of select="$pos"/></xsl:with-param>
      </xsl:apply-templates>
    </doc>
  </xsl:template>
  <!-- Flatten arrays to duplicate field lines -->
  <xsl:template match="doc/arr" priority="100">
    <xsl:variable name="fn" select="@name"/>
    <xsl:for-each select="*">
      <xsl:element name="field">
        <xsl:attribute name="name"><xsl:value-of select="$fn"/></xsl:attribute>
        <xsl:value-of select="."/>
      </xsl:element>
    </xsl:for-each>
  </xsl:template>
  <xsl:template match="doc/*">
    <xsl:variable name="fn" select="@name"/>
    <xsl:element name="field">
      <xsl:attribute name="name"><xsl:value-of select="$fn"/></xsl:attribute>
      <xsl:value-of select="."/>
    </xsl:element>
  </xsl:template>
  <xsl:template match="*"/>
</xsl:stylesheet>
```

该样式表将 Solr 的 XML 搜索结果格式转换成 Solr 的更新 XML 语法。
一个用例可能是将 Solr 1.3 索引(它不具有 CSV 响应输出) 拷贝为可被索引到另一个 Solr 文件的格式 (提供了所有被存储的字段)：

```
http://localhost:8983/solr/my_collection/select?q=*:*&wt=xslt&tr=updateXml.xsl&rows=1000
```

你也可以在 `XsltUpdateRequestHandler` 中使用样式表来在索引时转换一个索引：

```
curl "http://localhost:8983/solr/my_collection/update?commit=true&tr=updateXml.xsl"
-H "Content-Type: text/xml" --data-binary @myexporteddata.xml
```

更多关于 XML 更新请求处理器的信息，查看 https://wiki.apache.org/solr/UpdateXmlMessages 。