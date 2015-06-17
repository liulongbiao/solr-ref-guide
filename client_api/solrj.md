# 使用 SolrJ

SolrJ 是一个让 Java 应用更容易和 Solr 交互的 API。
SolrJ 隐藏了大量连接 Solr 的细节并让你的应用可以用简单高层的方法来和 Solr 交互。

SolrJ 的中心是 `org.apache.solr.client.solrj` 包，里面仅包含了五个主要的类。
首先我们需要创建一个 [`SolrClient`](http://lucene.apache.org/solr/5_2_0/solr-solrj/org/apache/solr/client/solrj/SolrClient.html)
，它表示了你想使用的 Solr 实例。
然后你可以发送 `SolrRequest`s 或 `SolrQuery`s 并获取 `SolrResponse`s。

`SolrClient` 是一个抽象类，因此要连接到一个远程 Solr 实例，你需要创建
一个 `HttpSolrClient` 或 `CloudSolrClient` 的实例。
它们都通过 HTTP 来和 Solr 交流，不同的是 `HttpSolrClient` 需使用明确的 Solr URL 来配置，
而 `CloudSolrClient` 则使用针对一个 [SolrCloud](../solrcloud/readme.md) 集群
的 `zkHost` 字符串来配置。

```
### 单节点 Solr 客户端 ###
String urlString = "http://localhost:8983/solr/techproducts";
SolrClient solr = new HttpSolrClient(urlString);
```

```
### SolrCloud 客户端 ###
String zkHostString = "zkServerA:2181,zkServerB:2181,zkServerC:2181/solr";
SolrClient solr = new CloudSolrClient(zkHostString);
```

一旦有了一个 `SolrClient`，你可以通过调用像 `query()`、`add()` 和 `commit()` 这样的方法来使用。

## 构建和运行 SolrJ 应用

Solr 中自带了 SolrJ API，因此你不需要下载或安装任何其他东西。
然而为了构建和运行使用 SolrJ 的应用，你需要添加一些类库到 classpath 里。

构建时，本节展示的示例需要 classpath 中有 `solr-solrj-x.y.z.jar`。

运行时，本节的示例需要在 `dist/solrj-lib` 目录可找到的类库。

本节捆绑的 Ant 脚本包含了构建和运行时所需的合适的类库。

你也可以跳过所有这些东西，直接使用 Maven 替代 Ant。
你所需要做的只是在项目的 `pom.xml` 文件中引入以下依赖：

```xml
<dependency>
  <groupId>org.apache.solr</groupId>
  <artifactId>solr-solrj</artifactId>
  <version>x.y.z</version>
</dependency>
```

如果你担心 SolrJ 类库会增加你的客户端应用尺寸，你可以使用像
[ProGuard](http://proguard.sourceforge.net/) 这样的代码混淆器移除不用的 API。

## 设置 XMLResponseParser

SolrJ 使用二进制格式而不是 XML 来作为其默认格式。
以前版本的 Solr 用户希望继续使用 XML 的话可以明确地将 `parser` 设置为 `XMLResponseParser`，如：

```java
server.setParser(new XMLResponseParser());
```

## 执行查询

使用 `query()` 来让 Solr 搜索所需的结果。
你需要传入一个描述查询的 `SolrQuery` 对象，然后你将得到一个 `QueryResponse`
(来自 `org.apache.solr.client.solrj.response` 包)。

`SolrQuery` 具有方法来让它可以很容易地添加参数来选择一个请求处理器及给它发送参数。
下面是一个非常简单的示例，它使用了默认的请求处理器并设置了一个 `q` 参数：

```java
SolrQuery parameters = new SolrQuery();
parameters.set("q", mQueryString);
```

要选择一个不同的请求处理器，只需要设置 `qt` 参数，如：

```java
parameters.set("qt", "/spellCheckCompRH");
```

一旦你设置好了 `SolrQuery`，你可以将它提交给 `query()`：

```java
QueryResponse response = solr.query(parameters);
```

客户端会发起网络链接并发送该查询。Solr 会处理该查询，并发送响应，响应会被解析为一个
`QueryResponse`。

该 `QueryResponse` 是一个满足查询参数的文档的集合。
你可以直接用 `getResults()` 检索文档，并且可以调用其它方法来找出关于高亮或 facets 等信息。

```java
SolrDocumentList list = response.getResults();
```

## 索引文档

其他的操作也很简单。要索引(添加)一个文档，你所需做的就是创建一个 `SolrInputDocument`
并将它传给 `SolrClient` 的 `add()` 方法。
下例假设名为 `solr` 的 `SolrClient` 对象已经按上面的例子创建。

```java
SolrInputDocument document = new SolrInputDocument();
document.addField("id", "552199");
document.addField("name", "Gouda cheese wheel");
document.addField("price", "49.99");
UpdateResponse response = solr.add(document);

// Remember to commit your changes!
solr.commit();
```

## 上传 XML 或二进制格式的内容

SolrJ 允许你上传 XML 格式的内容以及替代默认 XML 格式的二进制格式的内容。
使用以下代码来设置使用二进制格式上传，它和设置 SolrJ 获取结果非常相像。

```java
server.setRequestWriter(new BinaryRequestWriter());
```

## 使用 ConcurrentUpdateSolrClient

当实现一个将会一次性批量加载大量文档的 java 应用时，应该考虑使用
[ConcurrentUpdateSolrClient](http://lucene.apache.org/solr/5_2_0/solr-solrj/org/apache/solr/client/solrj/impl/ConcurrentUpdateSolrClient.html)
来替代 `HttpSolrClient`。
`ConcurrentUpdateSolrClient` 会缓冲所有被添加的文档并将它们写到开启的 HTTP 连接中。
这个类是线程安全的。
尽管任何 `SolrClient` 请求都可以通过该实现完成，
我们通常推荐仅将 `ConcurrentUpdateSolrClient` 用在 `/update` 上。

## EmbeddedSolrServer

[EmbeddedSolrServer](http://lucene.apache.org/solr/5_2_0/solr-core/org/apache/solr/client/solrj/embedded/EmbeddedSolrServer.html)
类提供了一个 `SolrClient` 客户端 API 的实现，
它和一个直接运行在你的应用中的一个 Solr 微实例直接对话。
这种内嵌方式在大多数场景下都是不推荐的，它仅支持有限的特性 - 特别的它不能用于
[SolrCloud](../solrcloud/readme.md) 或
[索引复制](../solrcloud/works/replication.md)。
`EmbeddedSolrServer` 主要用于帮助测试。

更多如何使用 `EmbeddedSolrServer` 的信息可查看 SolrJ 源码中
`org.apache.solr.client.solrj.embedded` 包下的 JUnit 测试。

## 相关内容

* [SolrJ API 文档](http://lucene.apache.org/solr/5_2_0/solr-solrj/)
* [Solr Wiki 中关于 SolrJ 的页面](http://wiki.apache.org/solr/Solrj)
* [建索引和基本数据操作](../indexing/readme.md)
