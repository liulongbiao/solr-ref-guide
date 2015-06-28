# 处理枚举字段

`EnumField` 类型允许定义值为封闭集合中的值的字段，且其排序顺序为预先确定的而不是字母序或数值。
这种例子有严重列表或风险定义。

## 在 schema.xml 中定义枚举字段

`EnumField` 类型定义非常简单，下面定义了 "priorityLevel" 和
"riskLevel" 的枚举字段类型。

```xml
<fieldType name="priorityLevel" class="solr.EnumField" enumsConfig="enumsConfig.xml"
  enumName="severity"/>
<fieldType name="riskLevel" class="solr.EnumField" enumsConfig="enumsConfig.xml"
  enumName="risk" />
```
除了通用于所有字段类型的 `name` 和 `class` 外，该类型还需要两个参数：

* `enumsConfig` ：包含 `<enum/>` 字段值的列表及其你希望该字段类型的顺序的配置文件的名称。
若到该文件的路径没有指定，则该文件应该处于该集合 `conf` 目录下。
* `enumName` ： `enumsConfig` 文件中用于该类型的特定枚举的名称

## 定义枚举配置文件

`enumsConfig` 参数对应的文件可以通过不同的名称包含多个枚举值列表，
如果你的 Solr 模式中存在多个枚举的使用的话。

下例定义了两个值的列表。每个列表包含在 `enum` 标签内：

```xml
<?xml version="1.0" ?>
<enumsConfig>
  <enum name="priority">
    <value>Not Available</value>
    <value>Low</value>
    <value>Medium</value>
    <value>High</value>
    <value>Urgent</value>
  </enum>
  <enum name="risk">
    <value>Unknown</value>
    <value>Very Low</value>
    <value>Low</value>
    <value>Medium</value>
    <value>High</value>
    <value>Critical</value>
  </enum>
</enumsConfig>
```

> #### 改变值
> 
> 你无法不重新索引就对已有值变更顺序或移除。
> 但是你可以再后面添加新的值。
