# 函数查询

函数查询让你可以使用一或多个数值字段的实际值来生成一个相对分数。
函数查询可由 [DisMax](./dismax.md)、[dDisMax](./extended_dismax.md) 和
[标准](./standard.md) 查询解析器所支持。

函数查询使用 *函数*。这些函数可以是一个常量(数值或字符串字面量)、一个字段、另一个函数或一个待替换参数。
你可以使用这些函数来修改给用户的结果的排序。
它们可基于用户的地址或某些其它计算来改变结果排序。

本节包含的函数查询内容包括：

* [使用函数查询](#using)
* [可用函数](#available)
* [函数查询示例](#example)
* [根据函数排序](#sort)
* [相关内容](#related)

## <a name="using"></a>使用函数查询

函数必须表示为函数调用(如 `sum(a, b)` 而不是简单的 `a+b`)

在 Solr 查询中存在多种使用函数查询的方式

* 通过显式接收函数参数的 QParser，如 `func` 或 `frange`。如：

```
q={!func}div(popularity,price)&fq={!frange l=1000}customer_ratings
```

* 在 Solr 表达式中，如：

```
sort=div(popularity,price) desc, score desc
```

* 添加函数结果作为查询结果的文档中的伪字段。如，对： `&fl=sum(x, y),id,a,b,c,score` 其输出可以是：

```xml
...
<str name="id">foo</str>
<float name="sum(x,y)">40</float>
<float name="score">0.343</float>
...
```

* 用在显式接收函数的特定参数中，如 eDisMax 查询解析器的 `boost` 参数或 DisMax 的 
[`bf` 参数](https://cwiki.apache.org/confluence/display/solr/The+DisMax+Query+Parser#TheDisMaxQueryParser-Thebf(BoostFunctions)Parameter)。
(注意 bf 参数实际上接收一系列由空白分割的函数列表，其中每个具有一个可选的加权。
确保在单个函数查询中使用 `bf` 时消除掉任何内部的空白。如：

```
q=dismax&bf="ord(popularity)^0.5 recip(rord(price),1,1000,1000)^0.3"
```

* 在Lucene QParser 中通过 `_val_` 关键字引入行内函数查询。如：

```
q=_val_:mynumericfield _val_:"recip(rord(myfield),1,2,3)"
```

只有快速随机访问的函数是推荐的。

## <a name="available"></a>可用函数

下表列出了函数查询中可用的函数：

|函数    |描述                   |语法示例                 |
|-------|----------------------|------------------------|
|abs    |Returns the absolute value of the specified value or function.|`abs(x) abs(-5)`|
|and    |Returns a value of true if and only if all of its operands evaluate to true.|`and(not(exists(popularity)),exists(price))`: returns `true` for any document which has a value in the `price` field, but does not have a value in the `popularity` field |
|"常量"  |Specifies a floating point constant. |1.5 |
|def    |def is short for default. Returns the value of ield "field", or if the field does not exist, returns the default value specified. and yields the first value where `exists()==true`) |`def(rating,5)`: This `def()` function returns the rating, or if no rating specified in the doc, returns 5 `def(myfield, 1.0)`: equivalent to `if(exists(myfield),myfield,1.0)` |
|div    |Divides one value or function by another. div(x,y) divides x by y.|`div(1,y)` `div(sum(x,100),max(y,1))` |
|dist   |Return the distance between two vectors (points) in an n-dimensional space. Takes inthe power, plus two or more ValueSource instances and calculates the distances between the two vectors. Each ValueSource must be a number. There must be an even number of ValueSource instances passed in and the method assumes that the first half represent the first vector and the second half represent the second vector. |`dist(2, x, y, 0, 0)`: calculates the Euclidean distance between (0,0) and (x,y) for each document<br/>  `dist(1, x, y, 0, 0)` : calculates the Manhattan (taxicab) distance between (0,0) and (x,y) for each document<br/> `dist(2, x,y,z,0,0,0)`: Euclidean distance between (0,0,0) and (x,y,z) for each document.<br/> `dist(1,x,y,z,e,f,g)` : Manhattan distance between (x,y,z) and (e,f,g) where each letter is a field name|
|docfreq(field,val) |Returns the number of documents that contain the term in the field. This is a constant (the same value for all documents in the index).<br/>You can quote the term if it's more complex, or do parameter substitution for the term value.|`docfreq(text,'solr') ...&defType=func&q=docfreq(text,$myterm)&myterm=solr` |
|exists |Returns TRUE if any member of the field exists.|exists(author) returns TRUE for any document has a value in the "author" field.<br/> exists(query(price:5.00)) returns TRUE if "price" matches "5.00".|
|field  |Returns the numeric field value of an indexed (not multi-valued) field with a maximum of one value per document. The `field()` function can be called using the name of the field as a string, or for most conventional field names simply use the field name by itself.<br/> 0 is returned for documents without a value in the field.|`myFloatFieldName` `field("my complex float fieldName")` |
|hsin   |The Haversine distance calculates the distance between two points on a sphere when traveling along the sphere. The values must be in radians. hsin also take a Boolean argument to specify whether the function should convert its output to radians.|`hsin(2, true, x, y, 0, 0)`  |
|idf    |||
|if     |||
||||
||||
||||
||||
||||
||||
||||
||||
||||
||||
||||
||||
||||
||||
||||
||||
||||
||||
||||
||||
||||
||||
||||
||||
||||
||||
||||
||||

## <a name="example"></a>函数查询示例
## <a name="sort"></a>根据函数排序
## <a name="related"></a>相关内容

