# 间隔 Faceting

Another supported form of faceting is interval faceting. This sounds similar to range faceting, but the functionality
is really closer to doing facet queries with range queries. Interval faceting allows you to set variable intervals and
count the number of documents that have values within those intervals in the specified field.

Even though the same functionality can be achieved by using a facet query with range queries, the
implementation of these two methods is very different and will provide different performance depending on the
context. If you are concerned about the performance of your searches you should test with both options. Interval
faceting tends to be better with multiple intervals for the same fields, while facet query tend to be better in
environments where filter cache is more effective (static indexes for example). This method will use docValues if
they are enabled for the field, will use fieldCache otherwise.

|名称         |作用                   |
|------------|----------------------|
|facet.interval |Specifies the field to facet by interval. |
|facet.interval.set |Sets the intervals for the field. |

## 参数 facet.interval

This parameter Indicates the field where interval faceting must be applied. It can be used multiple times in the
same request to indicate multiple fields.

```
facet.interval=price&facet.interval=size
```

## 参数 facet.interval.set

This parameter is used to set the intervals for the field, it can be specified multiple times to indicate multiple
intervals. This parameter is global, which means that it will be used for all fields indicated with facet.interval
unless there is an override for a specific field. To override this parameter on a specific field you can use: 
`f.<fieldname>.facet.interval.set` , for example:

```
f.price.facet.interval.set=[0,10]&f.price.facet.interval.set=(10,100]
```

## 间隔语法

Intervals must begin with either '(' or '[', be followed by the start value, then a comma (','), the end value, and
finally a closing ')' or ']’.

For example:
* `(1,10)` -> will include values greater than 1 and lower than 10
* `[1,10)` -> will include values greater or equal to 1 and lower than 10
* `[1,10]` -> will include values greater or equal to 1 and lower or equal to 10

The initial and end values cannot be empty. If the interval needs to be unbounded, the special character '*' can
be used for both, start and end limit. When using '*', '(' and '[', and ')' and ']' will be treated equal. [*,*] will include
all documents with a value in the field. The interval limits may be strings but there is no need to add quotes. All
the text until the comma will be treated as the start limit, and the text after that will be the end limit. For example:
[Buenos Aires,New York]. Keep in mind that a string-like comparison will be done to match documents in string
intervals (case-sensitive). The comparator can't be changed.

Commas, brackets and square brackets can be escaped by using '\' in front of them. Whitespaces before and
after the values will be omitted. The start limit can't be grater than the end limit. Equal limits are allowed, this
allows you to indicate the specific values that you want to count, like [A,A], [B,B] and [C,Z].

Interval faceting supports output key replacement described below. 
Output keys can be replaced in both the `facet.interval` parameter 
and in the `facet.interval.set` parameter . For example:

```
&facet.interval={!key=popularity}some_field
&facet.interval.set={!key=bad}[0,5]
&facet.interval.set={!key=good}[5,*]
&facet=true
```

