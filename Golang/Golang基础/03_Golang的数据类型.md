# Golang的数据类型





## 概述

Go语言中的数据类型分为**基本数据类型**和**复合数据类型**

* 基本数据类型：整型、浮点型、布尔型、字符串
* 复合数据类型：数组、切片、结构体、函数、map、通道（channel）、接口等



## 整型

整型的类型有很多中，包括 int8，int16，int32，int64。我们可以根据具体的情况来进行定义

如果我们直接写 int也是可以的，它在不同的操作系统中，int的大小是不一样的

- 32位操作系统：int -> int32
- 64位操作系统：int -> int64

![](https://img2022.cnblogs.com/blog/1725357/202207/1725357-20220702204142527-1486114269.png)

我们可以通过`unsafe.Sizeof() `查看不同长度的整型在内存里面的存储空间：

```go
package main

import (
	"fmt"
	"unsafe"
)

func main() {
	var num1=999
	// 64位系统下输出8，表示num1占用8字节
	fmt.Println(unsafe.Sizeof(num1))

	var num2 int16=999
	// 输出2，表示占用2字节
	fmt.Println(unsafe.Sizeof(num2))
}
```



###  类型转换

通过在变量前面添加指定类型，就可以进行强制类型转换

```go
var a1 int16 = 10
var a2 int32 = 12
// 强制转换a1的类型为int32
var a3 = int32(a1) + a2
fmt.Println(a3)
```

注意，高位转低位的时候，需要注意，会存在精度丢失，比如上述16转8位的时候，就丢失了

```go
var n1 int16 = 130
fmt.Println(int8(n1)) // 变成 -126
```



###  数字字面量语法

Go1.13版本之后，引入了数字字面量语法，这样便于开发者以二进制、八进制或十六进制浮点数的格式定义数字，例如：

```go
val:=0b100100 //0b表示二进制数字
val2:=0o31521551 //0o表示8进制数字
val3:=0xffff   //0x表示16进制数字
```



###  进制转换

```go
var number = 17
// 原样输出
fmt.Printf("%v\n", number)
// 十进制输出
fmt.Printf("%d\n", number)
// 以八进制输出
fmt.Printf("%o\n", number)
// 以二进制输出
fmt.Printf("%b\n", number)
// 以十六进制输出
fmt.Printf("%x\n", number)
```





## 浮点型

Go语言支持两种浮点型数：**float32和float64**。这两种浮点型数据格式遵循IEEE754标准：

float32的浮点数的最大范围约为3.4e38，可以使用常量定义：`math.MaxFloat32`。float64的浮点数的最大范围约为1.8e308，可以使用一个常量定义：`math.MaxFloat64`

打印浮点数时，可以使用%f格式化输出，代码如下：

```go
import (
	"fmt"
	"math"
)

func main() {
	var pi=math.Pi
	// 打印浮点类型，默认小数点6位
	fmt.Printf("%f\n",pi)

	// 格式化打印小数，精确到小数后两位
	fmt.Printf("%.2f\n",pi)
}
```

###  Golang中精度丢失的问题

几乎所有的编程语言都有精度丢失的问题，这是典型的二进制浮点数精度损失问题，在定长条件下，二进制小数和十进制小数互转可能有精度丢失

```go
l :=1.1111112
//会输出111.11112000000001
fmt.Println(l *100)
```

解决方法，使用第三方包来解决精度损失的问题：https://pkg.go.dev/github.com/shopspring/decimal#section-readme

```go
package main

import (
	"fmt"
	"github.com/shopspring/decimal"
)

func main() {
	//使用第三方包：decimal 解决go的精度丢失问题
	price, err := decimal.NewFromString("1.1111112")
	if err != nil {
		fmt.Printf(err.Error())
		return
	}

	//1.1111112*100
	price.Mul(decimal.NewFromInt(100))
	// 打印1.1111112
	fmt.Printf(price.String())
}
```



##  布尔类型

Go 保留了true和false两个布尔关键字

```go
package main

import "fmt"

func main() {
	var b=false
	if b{
		fmt.Println("true")
	}else{
		fmt.Println("false")
	}
}
```

##  字符串类型

Go 语言中的字符串以原生数据类型出现，使用字符串就像使用其他原生数据类型（int、bool、float32、float64等）一样。Go语言里的字符串的内部实现使用UTF-8编码。字符串的值为双引号（"）中的内容，可以在Go语言的源码中直接添加非ASCll码字符，例如：

```go
package main

import "fmt"

func main() {
   str1 := "This is go string type"
   fmt.Println(str1)
}
```

如果想要定义多行字符串，可以使用反引号

```go
package main

import "fmt"

func main() {
   var str2 = `
     这是多行文本
     这是多行文本
     这是多行文本
     这是多行文本
     这是多行文本
     这是多行文本
     这是多行文本
     这是多行文本
   `
   fmt.Println(str2)
}
```

###  字符串常见操作

- len(str)：求长度
- +或fmt.Sprintf：拼接字符串
- strings.Split：分割
- strings.contains：判断是否包含
- strings.HasPrefix，strings.HasSuffix：前缀/后缀判断
- strings.Index()，strings.LastIndex()：子串出现的位置
- strings.Join()：join操作
- strings.Index()：判断在字符串中的位置



##  byte 和 rune类型

组成每个字符串的元素叫做 “字符”，可以通过遍历字符串元素获得字符。字符用单引号 '' 包裹起来

Go语言中的字符有以下两种类型：

- **byte型**：或者叫uint8类型，代表了ACII码的一个字符
- **rune类型**：代表一个UTF-8字符

当需要处理中文，日文或者其他复合字符时，则需要用到rune类型，rune类型实际上是一个int32

Go使用了特殊的rune类型来处理Unicode，让基于Unicode的文本处理更为方便，也可以使用byte型进行默认字符串处理，性能和扩展性都有照顾。

需要注意的是，在go语言中，一个汉字占用3个字节（utf-8），一个字母占用1个字节

```go
package main

import "fmt"

func main() {
	var ch byte='a'
	//输出的是ASCII码值，也就是说当我们直接输出byte（字符）的时候，输出的是这个字符对应的码值
	fmt.Println(ch)
	//输出的是字符
	fmt.Printf("%c",ch)

	s:="Hello World!你好 Go!"
	for i:=0;i<len(s);i++{
		// 超出ASCII码值的字符会乱码
		fmt.Printf("%v(%c)\t", s[i], s[i])
	}

	for index,v:=range s{
		// 正常打印
		fmt.Println(index,v)
	}

}
```

###  修改字符串

要修改字符串，需要先将其转换成[]rune 或 []byte类型，完成后在转换成string，无论哪种转换都会重新分配内存，并复制字节数组。

* 转换为 []byte 类型

```go
package main

import "fmt"

func main() {
	s:="Hello World!"
	s2 :=[]byte(s)
    s2[0]='h'
    fmt.Println(string(s2))
}
```



* 转换为[]rune类型

```go
package main

import "fmt"

func main() {
	s:="Hello World!你好 Go!"
	s2 :=[]rune(s)
    s2[0]='h'
    fmt.Println(string(s2))
}
```

