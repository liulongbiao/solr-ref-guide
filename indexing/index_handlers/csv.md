# CSV 更新便捷路径

除了 `/update` 处理器，Solr 中默认还有一个 CSV 特有的请求处理器，
它隐式地覆盖了某些请求参数：

|路径            |默认参数                 |
|---------------|-------------------------|
|`/update/csv`  |`stream.contentType=application/csv`|

`/update/csv` 路径对难以设置 `Content-Type` 的应用以 CSV 格式发送更新请求非常有用，

更多有关 CSV 更新请求处理器的信息，查看 https://wiki.apache.org/solr/UpdateCSV
