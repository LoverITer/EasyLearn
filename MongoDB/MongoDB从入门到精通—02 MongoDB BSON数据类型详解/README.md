## MongoDB从入门到精通—02 MongoDB BSON数据类型详解

MongoDB的文档相似于 JSON，JSON 是一种简单的表示数据的方式，仅包含6种数据类型，分别是：null、布尔、数字、字符串、数组和对象。

虽然这些类型的表现已经足够强大，可是对于绝大多数应用来讲还须要另一些不可或缺的类型。例如，日期类型、数字类型（只有一种，无法区分整型和浮点）、正则表达式等。正则表达式

MongoDB 在保留 JSON 基本的键值对特性的基础上，添加了一些其他的数据类型。在不同的编程语言下这些类型的表示有些差别。 下面列出MongoDB一般支持的一些类型，同时说明了在shell中这些类型的表示方法。每种BSON类型都具备整数和字符串标识符，以下表所示：

| 类型                     | 整数 | 别名                | 备注                             | 说明                         |
| :----------------------- | :--- | :------------------ | :------------------------------- | :--------------------------- |
| Double                   | 1    | double              | shell中的数字类型                | 64位浮点数                   |
| String                   | 2    | string              | {"x" : "foobar"}                 | 字符串类型                   |
| Object                   | 3    | object              |                                  | 对象类型                     |
| Array                    | 4    | array               | {"x" ： ["a", "b", "c"]}         | 数组类型                     |
| Binary data              | 5    | binData             | shell中不可用                    | 二进制数据类型               |
| Undefined                | 6    | undefined           | 已过期                           | 未定义类型                   |
| ObjectId                 | 7    | objectId            | {"X" :ObjectId() }               | 对象id类型                   |
| Boolean                  | 8    | bool                | {"x":true}+                      | 布尔类型                     |
| Date                     | 9    | date                |                                  | 日期类型                     |
| Null                     | 10   | null                |                                  | 用于表示空值或者不存在的字段 |
| Regular Expression       | 11   | regex               |                                  | 正则表达式类型               |
| DBPointer                | 12   | dbPointer           | 已过期                           |                              |
| JavaScript               | 13   | javascript          | {"x" ： function() { /* …… */ }} | JavaScript代码               |
| Symbol                   | 14   | symbol              | shell中不可用，已过期            |                              |
| JavaScript（with scope） | 15   | javascriptWithScope |                                  | 带做用域的JavaScript代码     |
| 32-bit integer           | 16   | int                 | shell中不可用                    | 32位整数                     |
| Timestamp                | 17   | timestamp           |                                  | 时间戳类型                   |
| 64-bit integer           | 18   | long                | shell中不可用                    | 64位整数                     |
| Decimal128               | 19   | decimal             | 3.4版本新增                      |                              |
| Min key                  | -1   | minKey              | shell中无此类型                  | 最小键                       |
| Max key                  | 127  | maxKey              | shell中无此类型                  | 最大键                       |



## 一、类型详解

### 1、数字

#### 32-bit integer（32位整数）

shell中这个类型不可用，由于JavaScript仅支持64位浮点数，因此32位整数会被自动转换为为64位浮点数。数组

#### 64-bit integer（64位整数）

shell中也不支持这个类型，shell中会使用一个特殊的内嵌文档来显示64位整数。服务器

#### Double（64位浮点数）

JavaScript中只有一种数字类型。MongoDB中有3种数字类型，shell必须绕过JavaScript的限制。默认状况下，shell中的数字都被MongoDB看成是双精度数。这就意味着若是从数据库张总得到一个32位整数，修改文档后，将文档存回数据库的时候，这个整数也被转换成了浮点数，即使是保持这个整数原封不动存回去，也是这样的。因此尽可能不要在shell下覆盖整个文档。编程语言

数字只能表示为双精度数，有些64位的整数并不能精确地表示为64位浮点数。因此要是存入一个64位整数，而后在shell中查看，它会显示一个内嵌文档，表示可能不许确。

例如，在集合中存入一个文档（不是在shell模式下存入的），其中myInterger键的值设为一个64位整数3，而后在shell中查看，如下：

```json
>doc = db.nums.findOn();
{
	“_id” : ObjectId(“4c0beecfd096a2580fe6fa08”),
	“myInteger” : {
	  “floatApprox” : 3
    }
}
```

内嵌文档只表示shell显示的是一个64位浮点数近似表示的64位整数，若内嵌文档只有一个键的话，实际上这个值是准确的。

要是插入的64位整数不能精确地做为双精度数显示，shell会添加两个键，分别是“top”（表示高32位）和“bottom”（表示低32位）。

例如，插入9223372036854775807，shell显示以下：

```json
>doc = db.nums.findOn();
{
	“_id” : ObjectId(“4c0beecfd096a2580fe6fa10”),
	“myInteger” : {
	  “floatApprox” : 9223372036854776000,
	  “top” :2147483647,
	  “bottom” : 4294967295
     }
}
```

floatApprox 是一种特殊的内嵌文档，能够做为值和文档来操做。

```shell
>doc.myInteger.floatApprox
3
> doc.myInteger + 1
4
```

### 2、String（字符串）

BSON字符串是UTF-8。一般，在序列化和反序列化BSON时，每种编程语言的驱动程序都会从语言的字符串格式转换为UTF-8。能够轻松地将大多数国际字符存储在BSON字符串中。此外，MongoDB的$regex查询在正则表达式字符串中支持UTF-8。

字符串类型可使用sort()方法进行排序，可是sort()是由C++的strcmpAPI实现的，排序可能会错误的处理某些字符。

### 3、Array（数组）

数组是一组值，既能够既能够偶组为有序对象来操做，也能够做为无序对象操做。

数组能够包含不一样数据类型的元素，实际上，常规键值对支持的值均可以做为数组的元素，甚至是套嵌数组。

文档中的数组有个特性，就是MongoDB能理解其结构，并指导如何深刻数组内部对其内容进行操做。这样就能用内容对数组进行查询和构建索引了。

MongoDB可使用原子更新修改数组中的内容。

值的集合或者列表能够表示成数组。

```json
{ “x” : [“a”, “b”, “c”]}
```

### 4、Binary data（二进制数据）

二进制数据能够由任意字节的串组成。不过shell中没法使用。

### 5、Undefined（未定义）

文档中也可使用未定义类型undefined。4.2版本中已经显示过期。

```json
{ “x” : unddefined }
```

### 6、ObjectId

ObjectId使用12字节的存储空间，每一个字节两位十六进制数字，是一个24位的字符串。

在早期版本中，这些字节是有特定的结构的：开头的4个字节是标准的Unix时间戳，编码了重新纪元开始的秒数；接下来的3个字节存储了机器ID；随后则是2个字节的进程ID；最后3个字节存储了进程局部的计数器，每次生成对象ID计数器都会加1。时间戳、机器ID和进程ID组合起来，提供了秒级别的惟一性。时间戳在前，意味着ObjectId大体会按照插入的顺序排序。能够将其做为索引提升效率，但不是绝对的，只是大体。这4个字节也隐含了文档建立的时间，绝大多数驱动都会公开一个方法从ObjectId获取这个信息。前9个字节保证了同一秒不一样机器不一样进程产生的ObjectId是惟一的，后3字节就是一个自动增长的计数器，保证了相同进程同一秒产生的ObjectId也是不同的。同一秒钟容许每一个进程拥有2563（16777216）个不一样的ObjectId。

当前4.2版本中是这样介绍的，ObjectId各个段含义以下：

前4个字节的值，表示自Unix纪元以来的秒数。中间5个字节是随机值。最后3个字节是计数器，以随机值开始。

```json
{“x” : objectId() }
```

使用 ObjectId 有如下两个优势：

- 1.在MongoDB shell中可使用该ObjectId.getTimestamp()方法访问建立时间。
- 2._id存储的ObjectId值的排序大体是按建立时间排序的。

```shell
> ObjectId("5b4c65a07a88f6e8893b70ef").getTimestamp()
ISODate("2018-07-16T09:30:08Z")
```

MongoDB中存储的文档必须有一个“\_id”键，这个键能够是任何类型的，默认是ObjectId对象。在一个集合中，每一个文档都有惟一的“_id”值，来确保集合里面每一个文档都能被惟一标识。此惟一是在一个集合中保证全局惟一的。

ObjectId是“\_id”的默认类型。它设计成轻量型，不一样的机器都能用全局惟一的同种方法方便地生成它。这是MongoDB采用这种类型的主要缘由。

若是插入文档的时候没有“\_id”键，系统会自动建立一个。这件事能够由MongoDB服务器来作，也能够在客户端由驱动程序完成。

一般会将自动生成_id放在客户端让驱动程序来完成，理由以下：

- 1.ObjectId的生成是有开销的，在客户端生成能够减小数据库扩展的负担。
- 2.在客户端生成ObjectId，驱动程序可以提供更加丰富的API。

### 7、Boolean（布尔）

布尔类型有两个值true和false。

```json
{ “x” : true }
```

### 8、Date（日期）

日期类型存储的是从标准纪元开始的毫秒数，不存储时区。

```json
{“x” : new Date() }
```

日期类型存储的日期大概为2.9亿年。毫秒数为负值，表示1970年以前的日期。

在JavaScript中，Date对象用作MongoDB的日期类型，建立一个新的Date对象时，调用new Date()而不是Date()。调用Date()实际上会返回对日期的字符串表示，而不是真正的Date对象。这不是MongoDB的特性，而是JavaScript自己的特性。

若是使用错误，就会致使日期和字符串混淆，字符串和日期不能互相匹配，最终会给删除、更新、查询等不少操做带来问题。

shell中的日期显示时使用本地时区设置。日期在数据中是以标准纪元开水的毫秒数的形式存储的，没有与之相关的时区信息。

### 9、Null

null用于表示空值或者不存在的字段。

```json
{“x” : null }
```

### 10、Regular Expression（正则表达式）

文档中能够包含正则表达式，采用JavaScript的正则表达式语法。

```json
{ “x” : /foobar/i }
```

### 11、JavaScript代码

文档中还能够包含JavaScript代码。

```json
{“x”: function() { /*…*/} }
```

### 12、Symbol（符号）

shell不支持这种类型。shell将数据库里的符号类型转换成字符串。如今已通过时。

### 13、Timestamp（时间戳）

BSON有一个MongoDB内部使用的特殊的时间戳类型，和常的日期类型没有关系。

时间戳记值是64位值，其中：前32位是一个time_t值（自Unix时代以来的秒数），后32位是ordinal给定秒内操做的增量。

在单个mongod实例中，时间戳记值始终是惟一的。

在复制中，操做日志具备一个ts字段。该字段中的值反映了使用BSON时间戳值的操做时间。

注意时间戳类型只是在MongoDB内部使用。开发过程当中使用的是日期类型。

### 14、Max key（最大值）

BSON包括一个特殊类型，表示可能的最大值。shell中没有这个类型。

### 15、Min key（最小值）

BSON包括一个特殊类型，表示可能的最小值。shell中没有这个类型。



## 二、类型之间的比较和排序

比较不一样BSON类型的值时，MongoDB使用如下比较顺序，从最低到最高：

MinKey（内部类型）、Null、数字（整数，整数，双精度数，小数）、符号，字符串、Object、数组、BinData、ObjectId、布尔、日期、时间戳、正则表达式、MaxKey（内部类型）

### 1、数值类型

为了进行比较，MongoDB将这些类型视为等效的，在进行比较以前，先将数字类型进行转换。

### 2、字符串

#### 二进制比较法

默认状况下，MongoDB将字符串转换成二进制来进行比较。

#### Collation

Collation是3.4版本的新功能，Collation容许用户为字符串比较指定特定的语言规则。

Collation具备如下语法：

```json
{
   locale: <string>,
   caseLevel: <boolean>,
   caseFirst: <string>,
   strength: <int>,
   numericOrdering: <boolean>,
   alternate: <string>,
   maxVariable: <string>,
   backwards: <boolean>
}
```

指定排序规则时，该locale字段为必填字段；全部其余排序规则字段都是可选的。

##### locale

用来选择语言环境，官方提供了全球不少国家的语言，在其中能够看到中文的选项值为zh，英文的值为en。其余值的选项，以下：

| Locale | caseFirst | alternate     | normalization | backwards |
| :----- | :-------- | :------------ | :------------ | :-------- |
| zh     | off       | non-ignorable | FALSE         | FALSE     |
| en     | off       | non-ignorable | FALSE         | FALSE     |

默认排序规则参数值取决于语言环境。如下默认参数在全部语言环境中都是一致的：

```
caseLevel : false
strength : 3
numericOrdering : false
maxVariable : punct
```

若是没有为集合或操做指定排序规则，则MongoDB使用先前版本中使用的简单二进制比较进行字符串比较。

### 3、Arrays

对于数组，小于比较或升序排序比较的是数组中的最小元素，大于比较或降序排序比较的是数组中的最大元素。

当字段是单元素数组与非数组字段进行比较时，比较的是数组的元素和非数组字段的值。空数组参与比较的话，会将空数组视为小于null或缺乏此字段。

### 4、Objects

MongoDB对BSON对象的比较使用如下顺序：

- 1.按照键值对在BSON对象中出现的顺序递归比较它们。
- 2.比较关键字段名称。
- 3.若是关键字段名称相等，则比较字段值。
- 4.若是字段值相等，则比较下一个键/值对（返回步骤1）。没有下一个字段的对象小于有下一个字段的对象。

### 5、日期和时间戳

在3.0.0版本中进行了更改，将日期对象放在时间戳对象以前排序。

在早期的版本中是将两种对象放在一块儿进行比较的。

### 6、不存在的字段

MongoDB将不存在的字段视为是空的BSON对象。

例如：{}和｛a : null｝进行比较，那么在比较的时候，a字段和空文档将视为等价的。

### 7、BinData

MongoDB按BinData如下顺序排序：

- 首先，比较数据的长度或大小。
- 而后，按BSON的一字节子类型进行比较。
- 最后，根据数据执行逐字节比较。



### 参考

* [MongoDB4.2官方文档]()

