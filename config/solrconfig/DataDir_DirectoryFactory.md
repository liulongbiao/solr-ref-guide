# SolrConfig 中的 DataDir 和 DirectoryFactory

## 用 dataDir 参数指定索引数据的地址

默认， Solr 会将其索引数据存储在 Solr home 下名为 `/data` 的目录下。
若你想为存储索引数据指定一个不同的目录，可以在 `solrconfig.xml` 文件中使用 `<dataDir>` 参数。
你可以以全路径或者相对于 Solr Core 实例目录的路径来指定另一个目录。如：

```xml
<dataDir>/var/data/solr/</dataDir>
```

若你在使用副本来复制 Solr 索引(如 [遗留的伸缩性和分布式](../legacy_distribution/readme.md) 中所述)，
则 `<dataDir>` 目录应该对应于用在复制配置中的索引目录。

## 给索引指定 DirectoryFactory

默认的 `solr.StandardDirectoryFactory` 是基于文件系统的，且会试图选取当前 JVM 和平台上的最佳实现。
你可以通过指定 `solr.MMapDirectoryFactory`、`solr.NIOFSDirectoryFactory` 
或 `solr.SimpleFSDirectoryFactory` 来强制某个实现。

```xml
<directoryFactory name="DirectoryFactory"
  class="${solr.directoryFactory:solr.StandardDirectoryFactory}"/>
```

其中 `solr.RAMDirectoryFactory` 是基于内存的，非持久的，且不能用在复制中。
要使用这个 `DirectoryFactory` 在 RAM 中存储索引：

```xml
<directoryFactory class="org.apache.solr.core.RAMDirectoryFactory"/>
```
