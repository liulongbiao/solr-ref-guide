# SolrConfig 中的 IndexConfig

`solrconfig.xml` 中的 `<indexConfig>` 部分定义了 Lucene index writers 的底层行为。
默认，该设置在 Solr 的示例 `solrconfig.xml` 配置中是注释掉的，
即会使用默认配置。大多数情况下，默认值就可以了。

```xml
<indexConfig>
  ...
</indexConfig>
```

本节包含的参数有：

* [调整索引大小片段](#sizing)
* [合并索引片段](#merging)
* [索引锁](#locks)
* [其它索引设置](#other)

## <a name="sizing"></a>调整索引大小片段

### ramBufferSizeMB

Once accumulated document updates exceed this much memory space (defined in megabytes), then the
pending updates are flushed. This can also create new segments or trigger a merge. Using this setting is
generally preferable to `maxBufferedDocs` . If both `maxBufferedDocs` and `ramBufferSizeMB` are set 
in `solrconfig.xml` , then a flush will occur when either limit is reached. The default is 100Mb.

```xml
<ramBufferSizeMB>100</ramBufferSizeMB>
```

### maxBufferedDocs

Sets the number of document updates to buffer in memory before they are flushed as a new segment. This may
also trigger a merge. The default Solr configuration sets to flush by RAM usage ( `ramBufferSizeMB` ).

```xml
<maxBufferedDocs>1000</maxBufferedDocs>
```

### maxIndexingThreads

The maximum number of simultaneous threads used to index documents. Once this threshold is reached,
additional threads will wait for the others to finish. The default is 8.

```xml
<maxIndexingThreads>8</maxIndexingThreads>
```

### UseCompoundFile

Setting `<useCompoundFile>` to true combines the various files of each index segment into a single file. On
systems where the number of open files allowed per process is limited, setting this to true may avoid hitting that
limit (the open files limit might also be tunable for your OS with the Linux/Unix `ulimit` command, or something
similar for other operating systems).

Updating a compound index may incur a minor performance hit for various reasons, depending on the runtime
environment. For example, filesystem buffers are typically associated with open file descriptors, which may limit
the total cache space available to each index.

This setting may also affect how much data needs to be transferred during index replication operations.

The default is false .

```xml
<useCompoundFile>false</useCompoundFile>
```

> #### useCompountFile and noCFSRatio
> 
> Users who set the `<useCompoundFile>` option to true should also carefully consider whether they want
> to set the noCFSRatio on their `<mergePolicy>` as well. Many `MergePolicy` implementations have
> default values for this setting that prevent compound files from being used for large segments.

## <a name="merging"></a>合并索引片段

### mergePolicy

Defines how merging segments is done. The default in Solr is `TieredMergePolicy` , which merges segments
of approximately equal size, subject to an allowed number of segments per tier. Other policies available are the 
`LogByteSizeMergePolicy` and `LogDocMergePolicy` . For more information on these policies, please see the
[MergePolicy javadocs](http://lucene.apache.org/core/5_2_0/core/org/apache/lucene/index/MergePolicy.html) .

```xml
<mergePolicy class="org.apache.lucene.index.TieredMergePolicy">
  <int name="maxMergeAtOnce">10</int>
  <int name="segmentsPerTier">10</int>
</mergePolicy>
```

### mergeFactor

For the LogByteSizeMergePolicy, or the (default) TieredMergePolicy, the `<mergeFactor>` configuration option
can be used as a convenience setting to indicate how many segments a Lucene index is allowed to merge at
one time. It is equivalent to setting either `<int name="mergeFactor">` on the LogByteSizeMergePolicy, or
setting both `<int name="maxMergeAtOnce">` and `<int name="segmentsPerTier">` (to the same value)
on the TieredMergePolicy.

To understand why a mergeFactor is important, consider what happens when an update is made to an index:
Documents are always added to the most recently opened segment. When a segment fills up, a new segment is
created and subsequent updates are placed there. If creating a new segment would cause the number of
lowest-level segments to exceed the `mergeFactor` value, then all those segments are merged together to form
a single large segment. Thus, if the merge factor is 10, each merge results in the creation of a single segment
that is roughly ten times larger than each of its ten constituents. When there are 10 of these larger segments,
then they in turn are merged into an even larger single segment. This process can continue indefinitely.

Choosing the best merge factor is generally a trade-off of indexing speed vs. searching speed. Having fewer
segments in the index generally accelerates searches, because there are fewer places to look. It also can also
result in fewer physical files on disk. But to keep the number of segments low, merges will occur more often,
which can add load to the system and slow down updates to the index.

Conversely, keeping more segments can accelerate indexing, because merges happen less often, making an
update is less likely to trigger a merge. But searches become more computationally expensive and will likely be
slower, because search terms must be looked up in more index segments. Faster index updates also means
shorter commit turnaround times, which means more timely search results.

The default behavior of both LogByteSizeMergePolicy and TieredMergePolicy is an effective `mergeFactor` of 10:

```xml
<mergeFactor>10</mergeFactor>
```

### mergeScheduler

The merge scheduler controls how merges are performed. The default `ConcurrentMergeScheduler` performs
merges in the background using separate threads. The alternative, `SerialMergeScheduler` , does not perform
merges with separate threads.

```xml
<mergeScheduler class="org.apache.lucene.index.ConcurrentMergeScheduler"/>
```

### mergedSegmentWarmer

When using Solr in for 
[Near Real Time Searching](../../searching/near_realtime_searching.md)
a merged segment warmer can be configured to warm the
reader on the newly merged segment, before the merge commits. This is not required for near real-time search,
but will reduce search latency on opening a new near real-time reader after a merge completes.

```xml
<mergedSegmentWarmer class="org.apache.lucene.index.SimpleMergedSegmentWarmer"/>
```

### checkIntegrityAtMerge

If set to true , any actions that result in merging segments will first trigger an integrity check using checksums
stored in the index segments (if available). If the checksums are not correct, the merge will fail and throw an
Exception. (defaults to " false " for backwards compatibility)

```xml
<checkIntegrityAtMerge>true</checkIntegrityAtMerge>
```

## <a name="locks"></a>索引锁

### lockType

The LockFactory options specify the locking implementation to use.

The set of valid lock type options depends on the 
[DirectoryFactory](./DataDir_DirectoryFactory.md)
you have configured. The values listed below
are are supported by StandardDirectoryFactory (the default):

* `native` (default) uses NativeFSLockFactory to specify native OS file locking. If a second Solr process
attempts to access the directory, it will fail. Do not use when multiple Solr web applications are attempting
to share a single index.
* `simple` uses SimpleFSLockFactory to specify a plain file for locking.
* `single` (expert) uses SingleInstanceLockFactory. Use for special situations of a read-only index
directory, or when there is no possibility of more than one process trying to modify the index (even
sequentially). This type will protect against multiple cores within the same JVM attempting to access the
same index. WARNING! If multiple Solr instances in different JVMs modify an index, this type will not prote
ct against index corruption.

For more information on the nuances of each LockFactory, see http://wiki.apache.org/lucene-java/AvailableLockFactories .

```xml
<lockType>native</lockType>
```

### unlockOnStartup

If `true` , any write or commit locks that have been held will be unlocked on system startup. This defeats the
locking mechanism that allows multiple processes to safely access a Lucene index. The default is `false` , and
changing this should only be done with care. This parameter is not used if the `lockType` is "none" or "single".

```xml
<unlockOnStartup>false</unlockOnStartup>
```

### writeLockTimeout

The maximum time to wait for a write lock on an IndexWriter. The default is 1000, expressed in milliseconds.

```xml
<writeLockTimeout>1000</writeLockTimeout>
```

## <a name="other"></a>其它索引设置

There are a few other parameters that may be important to configure for your implementation. These settings
affect how or when updates are made to an index.

|设置         |描述                         |
|------------|----------------------------|
|reopenReaders|Controls if IndexReaders will be re-opened, instead of closed and then opened, which is often less efficient. The default is true.|
|deletionPolicy|Controls how commits are retained in case of rollback. The default is `SolrDeletionPolicy` , which has sub-parameters for the maximum number of commits to keep ( `maxCommitsToKeep` ), the maximum number of optimized commits to keep ( `maxOptimizedCommitsToKeep` ), and the maximum age of any commit to keep ( `maxCommitAge` ), which supports `DateMathParser` syntax.|
|infoStream|The InfoStream setting instructs the underlying Lucene classes to write detailed debug information from the indexing process as Solr log messages.|

```xml
<reopenReaders>true</reopenReaders>
<deletionPolicy class="solr.SolrDeletionPolicy">
  <str name="maxCommitsToKeep">1</str>
  <str name="maxOptimizedCommitsToKeep">0</str>
  <str name="maxCommitAge">1DAY</str>
</deletionPolicy>
<infoStream>false</infoStream>
```
