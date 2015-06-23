# 范围 Faceting

你可以在任何支持范围查询的日期字段或数值字段上使用范围 Faceting。
这在缝合像价格这样一系列范围查询(作为查询的 facet)时非常有用。
在 Solr 3.1 中，偏好范围 Faceting 而不是 [日期 Faceting](./date.md) 。

|参数       |描述                      |
|----------|-------------------------|
|[facet.range](#facet-range) |指定根据范围进行 facet 的字段 |
|[facet.range.start](#facet-range-start) |指定 facet 范围的起始 |
|[facet.range.end](#facet-range-end) |指定 facet 范围的结束 |
|[facet.range.gap](#facet-range-gap) |指定范围的跨度作为每次被添加到下限的值 |
|[facet.range.hardend](#facet-range-hardend) |一个布尔参数用于指定 Solr 在范围宽带不能被起止值平均分割时该如何处理。若为 true，最后的范围约束将具有 `facet.range.end` 作为其上限。若为 false，最后一个范围将具有比 `facet.range.end` 大的最小可能的上限，这样该范围的实际宽度为指定的 `facet.range.gap`。该参数默认为 false。 |
|[facet.range.include](#facet-range-include) |指定对范围的上下限的开闭偏好。 更多细节查看 `facet.range.include` 部分。 |
|[facet.range.other](#facet-range-other) |指定在每个 facet 范围约束之外进行计数 |
|[facet.range.method](#facet-range-method) |指定用于计算 facets 的算法或方法 |

## <a name="facet-range"></a> 参数 facet.range

The `facet.range` parameter defines the field for which Solr should create range facets. For example:

`facet.range=price&facet.range=age`

## <a name="facet-range-start"></a> 参数 facet.range.start

The `facet.range.start` parameter specifies the lower bound of the ranges. You can specify this parameter
on a per field basis with the syntax of `f.<fieldname>.facet.range.start` . For example:

`f.price.facet.range.start=0.0&f.age.facet.range.start=10`

## <a name="facet-range-end"></a> 参数 facet.range.end

The `facet.range.end` specifies the upper bound of the ranges. You can specify this parameter on a per field basis
with the syntax of `f.<fieldname>.facet.range.end` . For example:

`f.price.facet.range.end=1000.0&f.age.facet.range.start=99`

## <a name="facet-range-gap"></a> 参数 facet.range.gap

The span of each range expressed as a value to be added to the lower bound. For date fields, this should be
expressed using the 
[DateMathParser 语法](http://lucene.apache.org/solr/5_2_0/solr-core/org/apache/solr/util/DateMathParser.html) 
(such as, `facet.range.gap=%2B1DAY ... '+1DAY'` ). You
can specify this parameter on a per-field basis with the syntax of `f.<fieldname>.facet.range.gap` . For
example:

`f.price.facet.range.gap=100&f.age.facet.range.gap=10`

## <a name="facet-range-hardend"></a> 参数 facet.range.hardend

The `facet.range.hardend` parameter is a Boolean parameter that specifies how Solr should handle cases
where the `facet.range.gap` does not divide evenly between `facet.range.start` and `facet.range.end` .
If true , the last range constraint will have the `facet.range.end` value as an upper bound. If false , the last
range will have the smallest possible upper bound greater then `facet.range.end` such that the range is the
exact width of the specified range gap. The default value for this parameter is false.

This parameter can be specified on a per field basis with the syntax `f.<fieldname>.facet.range.hardend`

## <a name="facet-range-include"></a> 参数 facet.range.include

By default, the ranges used to compute range faceting between `facet.range.start` and `facet.range.end`
are inclusive of their lower bounds and exclusive of the upper bounds. 
The "before" range defined with the `facet.range.other` parameter is exclusive 
and the "after" range is inclusive. This default, equivalent to "lower"
below, will not result in double counting at the boundaries. 
You can use the `facet.range.include` parameter
to modify this behavior using the following options:

|选项    |描述                          |
|-------|-----------------------------|
|lower  |All gap-based ranges include their lower bound. |
|upper  |All gap-based ranges include their upper bound. |
|edge   |The first and last gap ranges include their edge bounds (lower for the first one, upper for the last one) even if the corresponding upper/lower option is not specified.|
|outer  |The "before" and "after" ranges will be inclusive of their bounds, even if the first or last ranges already include those boundaries.|
|all    |Includes all options: lower, upper, edge, outer. |

You can specify this parameter on a per field basis with the syntax 
of `f.<fieldname>.facet.range.include` , 
and you can specify it multiple times to indicate multiple choices.

> To ensure you avoid double-counting, do not choose both `lower` and `upper` , do not choose outer , and
> do not choose `all` .

## <a name="facet-range-other"></a> 参数 facet.range.other

The `facet.range.other` parameter specifies that in addition to the counts for each range constraint between
`facet.range.start` and `facet.range.end` , counts should also be computed for these options:

|选项    |描述                          |
|-------|-----------------------------|
|before |All records with field values lower then lower bound of the first range. |
|after  |All records with field values greater then the upper bound of the last range. |
|between|All records with field values between the start and end bounds of all ranges. |
|none   |Do not compute any counts. |
|all    |Compute counts for before, between, and after. |

This parameter can be specified on a per field basis with the syntax of `f.<fieldname>.facet.range.other`
. In addition to the `all` option, this parameter can be specified multiple times to indicate multiple choices, 
but `none` will override all other options.

## <a name="facet-range-method"></a> 参数 facet.range.method

The `facet.range.method` parameter selects the type of algorithm or method Solr should use for range
faceting. Both methods produce the same results, but performance may vary.

|选项    |描述                          |
|-------|-----------------------------|
|filter |This method generates the ranges based on other facet.range parameters, and for each of them executes a filter that later intersects with the main query resultset to get the count. It will make use of the filterCache, so it will benefit of a cache large enough to contain all ranges.|
|dv     |This method iterates the documents that match the main query, and for each of them finds the correct range for the value. This method will make use of docValues (if enabled for the field) or fieldCache. "dv" method is not supported for field type DateRangeField or when using group.facets .|

Default value for this parameter is "filter".

## 范围 Faceting 中的 facet.mincount 参数

The `facet.mincount` parameter, the same one as used in field faceting is also applied to range faceting.
When used, no ranges with a count below the minimum will be included in the response.

> #### Date Ranges & Time Zones
>
> Range faceting on date fields is a common situation where the 
> [TZ](https://cwiki.apache.org/confluence/display/solr/Working+with+Dates#WorkingwithDates-TZ) 
> parameter can be useful to ensure
> that the "facet counts per day" or "facet counts per month" are based on a meaningful definition of when
> a given day/month "starts" relative to a particular TimeZone.
> 
> For more information, see the examples in the [处理日期](../../schema/field_type/date.md) section.

