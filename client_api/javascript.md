# 使用 JavaScript

从 JavaScript 客户端使用 Solr 是如此直接因此它需要特别提一下。
实际上，它如此直接因此没有客户端 API。
你不需要安装任何包或者配置任何东西。

HTTP 请求可以通过标准的 `XMLHttpRequest` 机制发送给 Solr。

默认，Solr 可以发送 [JSON 响应](https://cwiki.apache.org/confluence/display/solr/Response+Writers#ResponseWriters-JSONResponseWriter),
JavaScript 中可以很容易地解析它。
只要在请求 URL 中添加 `wt=json` 就可以让响应以 JSON 发送。

更多信息和优秀的示例，可以在 Solr Wiki 上读 SolJSON 页：

http://wiki.apache.org/solr/SolJSON
