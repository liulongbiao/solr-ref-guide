# 字段属性用例

以下是一些常见的用例摘要，以及为支持该用例字段或字段类型所需的属性。
表格中的一个项为 true 或 false，表示为用例功能正常对应选项必须被设置为给定的值。
若没有提供项，则对应属性的设置不影响用例功能。

|    用例    |indexed|stored|multiValued|omitNorms|termVectors|termPositions|docValues|
|:----------|-------|------|-----------|---------|-----------|-------------|---------|
|字段内搜索   |true   |||||||
|检索内容     ||       ||||||
|用作唯一键   |true   ||false  |||||
|字段上排序   |true[^7]||false |true[^1] |||true[^7]|
|使用字段助推[^5]||||false ||||
|文档助推影响字段内搜索||||false ||||
|高亮        |true[^4] |true |||true[^2] |true[^3] ||
|faceting[^5] |true[^7] ||||||true[^7] |
|添加多个值，维护顺序 |||true |||||
|字段长度影响文档分数 ||||false ||||
|MoreLikeThis[^5] |||||true[6] |||

注：

[^1]: 推荐但非必需
[^2]: 如果存在将被使用，但非必需
[^3]: (若 termVectors=true)
[^4]: 该字段必须定义一个分词器，但它不需要是 indexed
[^5]: 在 [理解分析器、分词器和过滤器](../../analyzer/readme.md) 中有详述
[^6]: 这里词向量不是强制的。若为真，则一个 stored 字段会被分析。因此词向量是推荐的，但仅当 `stored=false` 时是必需的 
[^7]: `indexed` 或 `docValues` 之一必须为 true，但两者都不是必需的。[DocValues](../docvalues.md) 在某些场景下更高效一些。
