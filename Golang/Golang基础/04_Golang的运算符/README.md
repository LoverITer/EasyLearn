# Golang的运算符



运算符是一种特殊的符号，用以表示数据的运算、赋值和比较等。Go的运算符分为以下6大类：

1. 算术运算符：`+、-、*、/，%，++，--`
2. 赋值运算符：`=、+=、-=、*=、/=、%=`
3. 关系运算符：`==、!=、>、<、>=、<=`
4. 逻辑运算符：`&&，||，!`
5. 位运算符：`&，|，^，<<，>>，&^`
6. 其他运算符：`&，*`



## 算术运算符

算术运算符是对数值类型的变量进行数学运算，比如加减乘除。

```go
package main

import "fmt"

func main() {
   var a = 10
   var b = 20
   // +: 1.加号  2. 字符串拼接
   // 加号，执行加法运算
   fmt.Printf("a+b=%v\n", a+b)

   var s = "Hello"
   //拼接符，拼接字符串
   fmt.Printf("%v\n", s+" World!")

   //-: 减号，执行减法运算
   fmt.Printf("a-b=%v\n",a-b)

   //*: 乘号，执行乘法运算
   fmt.Printf("a*b=%v\n",a*b)


   // /：除号，执行除法运算：
   //两个int类型数据进行运算，结果一定为整数类型
   fmt.Printf("a/b=%v\n",a/b)
   //浮点类型参与运算，结果为浮点类型
   var c float32=3.5
   fmt.Printf("float32(a)/c=%v\n",float32(a)/c)


   // %：取模，执行取模运算
   fmt.Printf("a mod b=%v\n",a%b)

   // 自增操作 ++ --
   // 在golang中， ++ 和 -- 只能单独使用,并且只有类似a++,a--的操作，没有++a,--a的操作
   a++
   fmt.Printf("a++=%v\n",a)
   b--
   fmt.Printf("b--=%v\n",b)
}
```



## 赋值运算符

赋值运算符就是将某个运算后的值，赋给指定的变量，Golang支持的赋值运算符：`=、+=、-=、*=、/=、%=`

```go
package main

import "fmt"

func main() {
	// 将等号右侧的值赋给左侧变量
	var a=10
	fmt.Println(a)

	// += 等价于：a=a+b == a+=b
  // -=、*=、/=、%= 类推
	a+=20
	fmt.Println(a)
	a=a+20
	fmt.Println(a)
}

```



## 关系运算符

关系运算符的结果都是 `bool` 类型，要么是 `true` 要么是 `false`，因此关系表达式经常用在流程控制中，Golang支持的关系运算符：`==、!=、>、<、>=、<=`

```go
package main

import "fmt"

func main() {
   // 关系运算符
   fmt.Println(3 == 8) // == 判断左右两侧的值是否相等
   fmt.Println(3 != 8) // 判断不等于
   fmt.Println(3 > 8)  // 大于
   fmt.Println(3 < 8)  // 小于
   fmt.Println(3 >= 8) // 大于等于
   fmt.Println(3 <= 8) // 小于等于
}
```

## 逻辑运算符

逻辑运算符：`&&(逻辑与)，||(逻辑或)，!(逻辑非)`

```go
package main

import "fmt"

func main() {
   // 与逻辑：&&，两个数值/表达式只要一侧为false结果一定为false
   fmt.Println(true && true)
   fmt.Println(true && false)
   fmt.Println(false && true)
   fmt.Println(false && false)
   fmt.Println()
   // 或逻辑：||，两个数值/表达式只要一侧为true结果一定为true
   fmt.Println(true || true)
   fmt.Println(true || false)
   fmt.Println(false || true)
   fmt.Println(false || false)
   // 非逻辑：!，取反
   fmt.Println()
   fmt.Println(!true)
   fmt.Println(!false)
}
```





## 位运算符

位运算符是对数据转换为二进制后某个位上的数进行的运算，Golang的位运算符有以下几种：

* `&`：与（AND）运算符是二元运算符，当两个操作位上值均为1时，结果为1，其余结果一概为0。
* `|`：或（OR）运算符是二元运算符，当两个操作位上值均为0时，结果为0，其余结果一概为1。
* `^`：异或（XOR）运算符既可以当做一元运算符也可以当做二元运算符，当为一元运算符时，表示取反操作，也就是该位原来值为0，取反后值为1；当为二元运算符时，当两个操作位上值相同时结果为0，不相同(相异)时结果为1。
* `&^`：按位清除运算符，该运算符的实际操作为&(^)操作。
* `<<`：左移运算符，相当于*2扩大两倍。
* `>>`：右移运算符相当于/2缩小两倍。

```go
package main

import "fmt"

func main() {
	var a=100
	var b=33
	// 输出 1100100
	fmt.Printf("%b\n",a)
	// 输出 100001
	fmt.Printf("%b\n",b)

	// &，两个操作位上值均为1时，结果为1，其余结果一概为0
	// a&b= 1100100 & 100001 = 100000
	fmt.Printf("%b\n",a&b)


	// |，两个操作位上值均为0时，结果为0，其余结果一概为1
	// a&b= 1100100 | 100001 = 1000101
	fmt.Printf("%b\n",a|b)

	// ^，两个操作位上值相同时结果为0，不相同(相异)时结果为1
	fmt.Printf("%b\n",a^b)

	// <<，左移运算符，相当于*2扩大两倍
	fmt.Printf("%b\n",1<<2)

	// >>，右移运算符相当于/2缩小两倍
	fmt.Printf("%b\n",15>>2)

	// &^, 按位清零
	// 将 15（1111） 的第二位置为0，输出 1101
	fmt.Printf("%b\n",15&^2)
}
```

