# Post Tool

Solr 包含了一个简单的命令行工具来 POST 多种类型的内容到 Solr 服务器。
这个工具是 `bin/post`。`bin/post` 是一个 Unix shell 脚本;
对 Windows (非 Cygwin) 用户，可以查看下面 [Windows 部分](#windows)。

要运行它，可以在终端输入：

```bash
bin/post -c gettingstarted example/films/films.json
```

它会连接 `localhost:8983` 上的服务器。
指定 `collection/core` 的名称是 **必需** 的。
`-help`(或简写 `-h`) 选项可以输出它的使用方式。(即 `bin/post -help`)。

## 使用 `bin/post` 工具

使用 `bin/post` 指定 `collection/core` 的名称或者完整的更新 `url` 是 **必需** 的。
基本用法为：

```bash
$ bin/post -help

Usage: post -c <collection> [OPTIONS] <files|directories|urls|-d ["...",...]>
    or post -help
  collection name defaults to DEFAULT_SOLR_COLLECTION if not specified

OPTIONS
=======
  Solr options:
    -url <base Solr update URL> (overrides collection, host, and port)
    -host <host> (default: localhost)
    -port <port> (default: 8983)
    -commit yes|no (default: yes)
  Web crawl options:
    -recursive <depth> (default: 1)
    -delay <seconds> (default: 10)
  Directory crawl options:
    -delay <seconds> (default: 0)
  stdin/args options:
    -type <content/type> (default: application/xml)
  Other options:
    -filetypes <type>[,<type>,...] (default:
xml,json,csv,pdf,doc,docx,ppt,pptx,xls,xlsx,odt,odp,ods,ott,otp,ots,rtf,htm,html,txt
,log)
    -params "<key>=<value>[&<key>=<value>...]" (values must be URL-encoded; these
pass through to Solr update request)
    -out yes|no (default: no; yes outputs Solr response to console)
...
```

## 示例

以下是使用 `bin/post` 的多种方式。本节展示其中几个示例。

### 索引 XML

添加所有扩展名为 `.xml` 的文档到名为 `gettingstarted` 的集合或 core。

```bash
bin/post -c gettingstarted *.xml
```

添加所有扩展名为 `.xml` 的文档到端口 8984 的 Solr 上名为 `gettingstarted` 的集合/core。

```bash
bin/post -c gettingstarted -port 8984 *.xml
```

发送 XML 参数以从 `gettingstarted` 删除一个文档。

```bash
bin/post -c gettingstarted -d '<delete><id>42</id></delete>'
```

### 索引 CSV

索引所有 CSV 文件到 `gettingstarted`：

```bash
bin/post -c gettingstarted *.csv
```

索引一个 TAB 分割的文件到 `gettingstarted`：

```bash
bin/post -c signals -params "separator=%09" -type text/csv data.tsv
```

这里为了以合适的类型进行处理，内容类型(`-type`) 参数是必需的;否则它会被忽略并且
一个 WARNING 日志会打出因为它不知道 .tsv 文件的类型。
[CSV 处理器](https://cwiki.apache.org/confluence/display/solr/Uploading+Data+with+Index+Handlers#UploadingDatawithIndexHandlers-CSVFormattedIndexUpdates)
支持 `separator` 参数，并且它通过使用 `-params` 设置传入。

### 索引 JSON

索引所有 JSON 文件到 `gettingstarted`：

```bash
bin/post -c gettingstarted *.json
```

### 索引富文本(PDF、Word、HTML 等)

索引一个 PDF 文件到 `gettingstarted`：

```bash
bin/post -c gettingstarted a.pdf
```

自动检测目录下的内容类型，并递归扫描需索引到 `gettingstarted` 的文档。

```bash
bin/post -c gettingstarted afolder/
```

自动检测目录下的内容类型，但限制只有 PPT 和 HTML 文件，并索引到 `gettingstarted`。

```bash
bin/post -c gettingstarted -filetypes ppt,html afolder/
```

## <a name="windows"><a>Windows 支持

`bin/post` 当前只是 Unix shell 脚本，但它委托它的工作给了跨平台的 Java 程序。
`SimplePostTool` 可以直接在支持的环境中运行，包括 Windows。

## SimplePostTool

`bin/post` 脚本当前委托它的工作给了名为 `SimplePostTool` 的独立的 Java 程序。
这个工具打包为一个可执行的 jar 包，并且可以直接通过
`java -jar example/exampledocs/post.jar` 运行。
查看其帮助输出并且根据它来 POST 文件、递归网页或文件系统目录、或直接给 Solr 服务器发送命令。

```bash
$ java -jar example/exampledocs/post.jar -h
SimplePostTool version 5.0.0
Usage: java [SystemProperties] -jar post.jar [-h|-] [<file|folder|url|arg>
[<file|folder|url|arg>...]]
.
.
.s
```
