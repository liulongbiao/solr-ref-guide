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
|idf    |Inverse document frequency; a measure of whether the term is common or rare across all documents. Obtained by dividing the total number of documents by the number of documents containing the term, and then taking the logarithm of that quotient. See also tf . |`idf(fieldName,'solr')`: measures the inverse of the frequency of the occurrence of the term 'solr' in fieldName .|
|if     |<p>启用条件式函数查询。在`if(test,value1,value2)` 中：</p><ul><li>test is or refers to a logical value or expression that returns a logical value (TRUE or FALSE).</li><li>value1 is the value that is returned by the function if test yields TRUE.</li><li>value2 is the value that is returned by the function if test yields FALSE.</li></ul><p>An expression can be any function which outputs boolean values, or even functions returning numeric values, in which case value 0 will be interpreted as false, or strings, in which case empty string is interpreted as false.</p> |`if(termfreq(cat,'electronics'),popularity,42)` : This function checks each document for the to see if it contains the term "electronics " in the cat field. If it does, then the value of the popularity field is returned, otherwise the value of 42 is returned.|
|linear |Implements m*x+c where m and c are constants and x is an arbitrary function. This s equivalent to sum(product(m,x),c) , but slightly more efficient as it is implemented as a single function. |`linear(x,m,c)` <br/> `linear(x,2,4)` returns `2*x+4` |
|log    |Returns the log base 10 of the specified function. |`log(x)` <nr/>`log(sum(x,100))` |
|map    |Maps any values of an input function x that fall within min and max inclusive to the specified target. The arguments min and max must be constants. The arguments `target` and `default` can be constants or functions. If the value of x does not fall between min and max, then either the value of x is returned, or a default value is returned if specified as a 5th argument. |`map(x,min,max,target)` <br/>  map(x,0,0,1) - changes any values of 0 to 1. This can be useful in handling default 0 values.<br/> map(x,min,max,target,default) map(x,0,100,1,-1) - changes any values between 0 and 100 to 1 , and all other values to -1 .<br/> map(x,0,100,sum(x,599),docfreq(text,solr)) - changes any values between 0 and 100 to x+599, and all other values to frequency of the term 'solr' in the field text.|
|max    |Returns the max of another function and a constant, which are specified as arguments: `max(x,c)` . The max function is useful for "bottoming out" another function at some constant. |`max(myfield,0)`  |
|maxdoc |Returns the number of documents in the index, including those that are marked as deleted but have not yet been purged. This is a constant (the same value for all documents in the index). |`maxdoc()` |
|ms     |||
|norm(field)|||
|not    |||
|numdocs|||
|or     |||
|ord    |||
|pow    |||
|product|||
|query  |||
|recip  |||
|rord   |||
|scale  |||
|sqedist|||
|sqrt   |||
|strdist|||
|sub    |||
|sum    |||
|sumtotaltermfreq|||
|termfreq|||
|tf     |||
|top    |||
|totaltermfreq|||
|xor()  |||

## <a name="example"></a>函数查询示例

要更好地理解如何在 Solr 中使用函数查询，假设有一个存储了某些假想的盒子的 x,y,z 的米制的维度
及其在字段 `boxname` 中存储的任意名称。
假设我们希望搜索匹配名称 `findbox` 的盒子，但排序基于的是盒子的体积。
其查询参数将是：

`q=boxname:findbox _val_:"product(x,y,z)"`

这个查询将基于体积进行排序。为了获得计算的体积值，你需要请求 `score` ，它将包含结果的体积：

`&fl=*, score`

假设你也具有字段存储了盒子的重量 `weight`。
要根据盒子的密度排序，并且以分数来返回其密度，你可以提交以下查询：

```
http://localhost:8983/solr/collection_name/select?q=boxname:findbox
_val_:"div(weight,product(x,y,z))"&fl=boxname x y z weight score
```

## <a name="sort"></a>根据函数排序

你可以根据某个函数的输出来排序查询结果。如，要根据距离排序结果，你可以输入：

```
http://localhost:8983/solr/collection_name/select?q=*:*&sort=dist(2, point1, point2) desc
```

根据函数排序也支持伪字段：字段可以动态生成且就好像它是索引中常规字段那样返回结果。如：

`&fl=id,sum(x, y),score`

将返回：

```xml
<str name="id">foo</str>
<float name="sum(x,y)">40</float>
<float name="score">0.343</float>
```

## <a name="related"></a>相关内容

* [FunctionQuery](https://wiki.apache.org/solr/FunctionQuery)
