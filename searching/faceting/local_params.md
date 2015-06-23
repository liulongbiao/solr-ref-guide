# Faceting 本地变量

[本地变量语法](../query_syntax/local_params.md) 允许覆盖全局设置。
它也提供了给其它参数值添加元数据的方式，很像 XML 属性。

## 标记和排除过滤器

你可以标记特定的过滤器并在 faceting 时排除这些过滤器。
这在做多选 faceting 时很有用。

考虑以下查询 faceting 示例：

```
q=mainquery&fq=status:public&fq=doctype:pdf&facet=true&facet.field=doctype
```

因为所有东西都由过滤器 `doctype:pdf` 进行约束，则当前的 `facet.field=doctype` facet 命令就是冗余的
且将会对所有除了 `doctype:pdf` 之外的东西返回计数 0。

要实现对 doctype 的多选， GUI 可能希望依旧显示其他 doctype 的值及其关联的计数，就好象 `doctype:pdf`
约束没有应用一样。如：

```
=== Document Type ===
  [ ] Word  (42)
  [x] PDF   (96)
  [ ] Excel (11)
  [ ] HTML  (63)
```

要返回当前没有被选中的 doctype 值的计数，标记直接约束 doctype 的过滤器，并在对 doctype 进行 faceting
时排除这些过滤器。

```
q=mainquery&fq=status:public&fq={!tag=dt}doctype:pdf&facet=true&facet.field
={!ex=dt}doctype
```

过滤器排除被所有类型的 facet 所支持。
`tag` 和 `ex` 本地变量都可以通过逗号分隔来指定多个值。

## 改变输出键

要改变某个 faceting 命令的输出键，以 `key` 本地参数来指定一个新的名称。如：

```
facet.field={!ex=dt key=mylabel}doctype
```

上述参数设置使得对 doctype 字段的字段 facet 结果使用键 "mylabel" 而不是 "doctype" 来返回。
这在对相同的字段多次以不同的过滤器排除方式来 faceting 时非常有用。

