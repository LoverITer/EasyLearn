# Golang的数组





##  Array数组介绍

数组是指一系列同一类型数据的集合。数组中包含的每个数据被称为数组元素（element），这种类型可以是意的原始类型，比如int、string等，也可以是用户自定义的类型。一个数组包含的元素个数被称为数组的长度。在Golang中数组是一个长度固定的数据类型，数组的长度是类型的一部分，也就是说[5]int和[10]int是两个不同的类型。Golang中数组的另一个特点是占用内存的连续性，也就是说数组中的元素是被分配到连续的内存地址中的，因而索引数组元素的速度非常快。

和数组对应的类型是Slice（切片），Slice是可以增长和收缩的动态序列，功能也更灵活，但是想要理解slice工作原理的话需要先理解数组，所以本节主要为大家讲解数组的使用。





##  数组定义

基本语法

```go
var 数组变量名 [元素数量] T
```

例如：

```go
package main

import "fmt"

func main() {

	//定义数组
	var arr1 [3]int32
	var arr2 [4]string
	var arr3 [10]float32

	fmt.Println(arr1)
	fmt.Println(arr2)
	fmt.Println(arr3)
}
```





## 数组初始化

### 方式1：下标赋值

```go
package main

import "fmt"

func main() {
	// 数组初始化
	var arr4 [10]string
	arr4[0]="java"
	arr4[1]="c"
	arr4[2]="golang"
	fmt.Println(arr4)
}
```

### 方式2：字面量初始化

我们也可以通过字面量在声明数组的同时快速初始化数组：

```go
package main

import "fmt"

func main() {
	//字面量初始化
	var arr5 = [10]string{"java","c","golang","php"}
	fmt.Println(arr5)
}
```

### 方式3：自动推断数组长度

如果数组长度不确定，可以使用 `...` 代替数组的长度，编译器会根据元素个数自行推断数组的长度：

```go
package main

import "fmt"

func main() {
	//自动推断数组长度
	var arr6 = [...]string{"java", "c", "golang", "php", "c++"}
	fmt.Println(arr6)
	fmt.Println(len(arr6))
}
```

### 方式4：指定下标初始化

如果设置了数组的长度，我们还可以通过指定下标来初始化元素：

```go
package main

import "fmt"

func main() {
	//指定下标初始化
	var arr7 = [...]string{0: "java", 3: "c", 4: "golang", 9: "php", 10: "c++"}
	fmt.Println(arr7)
	fmt.Println(len(arr7))
}
```





## 数组遍历

### 方式1：通过数组下标遍历

```go
package main

import "fmt"

func main() {
	//自动推断数组长度
	var arr = [...]string{"java", "c", "golang", "php", "c++"}
	// for+下标遍历
	for i := 0; i < len(arr); i++ {
		fmt.Println(arr[i])
	}
}
```



### 方式2：通过for-range遍历

range 关键字可以对 array 进行迭代，每次返回一个 index 和对应的元素值。可以将 range 的迭代结合 for 循环对 array 进行遍历

```go
package main

import "fmt"

func main() {
	//自动推断数组长度
	var arr = [...]string{"java", "c", "golang", "php", "c++"}
	// for-range 遍历数组
	for index, val := range arr {
		fmt.Println("index:",index," value:",val)
	}
}
```



## 指针数组

可以声明一个指针类型的数组，这样数组中就可以存放指针。注意，指针的默认初始化值为 nil。

```go
package main

import "fmt"

func main() {
   //指针数组
   var arr8 = [10]*int32{0: new(int32), 1: new(int32), 9: new(int32)}
   fmt.Println(arr8)
   *arr8[0] = 1
   *arr8[9] = 10
	
	 fmt.Println(*arr8[0])
	 fmt.Println(*arr8[9])
}
```

上面的 *int32 表示数组只能存储 *int32 类型的数据，也就是指向 int32 的指针类型。new(TYPE) 函数会为一个 TYPE 类型的数据结构划分内存并做默认初始化操作，并返回这个数据对象的指针，所以 new(int32) 表示创建一个 int32 类型的数据对象，同时返回指向这个对象的指针。



##  数组的值类型

Go 中的传值方式是按值传递，这意味着给变量赋值、给函数传参时，都是直接拷贝一个副本然后将副本赋值给对方的。这样的拷贝方式意味着：

* **如果数据结构体积庞大，则要完整拷贝一个数据结构副本时效率会很低**
* **函数内部修改数据结构时，只能在函数内部生效，函数一退出就失效了，因为它修改的是副本对象**

```go
package main

import "fmt"

func main() {
	var arr = [...]int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
	fmt.Println(arr)

	//翻转数组
	reverseArr(arr)
	
	// 原始数组没有被改变
	fmt.Println(arr)
}

func reverseArr(arr [10]int) {
	i := 0
	j := len(arr) - 1
	for {
		if i > j {
			break
		}
		var tmp = arr[i]
		arr[i] = arr[j]
		arr[j] = tmp
		i++
		j--
	}

	// 在这里数组被改变的只是副本，原始数组没有被改变
	fmt.Println(arr)
}
```



##  多维数组

可以通过组合两个一维数组的方式构成二维数组。一般在处理具有父、子关系或者有坐标关系的数据时，二维数组比较有用。**多维数组声明方式：**

```go
var 数组名 [元素数据量][元素数据量][...] T
```



### 二维数组

二维数组是最简单的多维数组，二维数组本质上是由一维数组组成的。二维数组定义方式如下：

```go
var 数组变量名 [元素数量][元素数量] T
```

示例

```go
// 二维数组
var array = [2][2]int{{1,2},{2,3}}
fmt.Println(array)
```



### 初始化二维数组

多维数组可通过大括号来初始值。以下实例为一个 3 行 4 列的二维数组：

```go
var arr = [3][4]int{  
	{0, 1, 2, 3},     /* 第一行索引为 0 */
	{4, 5, 6, 7},     /* 第二行索引为 1 */
	{8, 9, 10, 11},   /* 第三行索引为 2 */
}

// 注意，上面的代码中倒数第二行的 } 必须有逗号，因为最后一行的 } 不能单独一行，也可以写成下面这样
var arr = [3][4]int {
	{0,1,2,3},			//第1行索引为0
	{4,5,6,7},			//第2行索引为1
	{8,9,10,11}}		//第3行索引为2

```



通过下标的方法也可以初始化多为数组，例如下面初始化一个 2 行 2 列的二维数组

```go
package main

import "fmt"

func main() {
  // 创建二维数组
	var arr2 [2][2]string
  
  
 // 按照下标方式初始化赋值
	arr2[0][0]="java"
	arr2[0][1]="c"
	arr2[1][0]="php"
	arr2[1][1]="golang"

  //打印数组
	fmt.Println(arr2)
}
```



### 访问二维数组

#### 使用下标遍历

二维数组可以通过指定下标来访问。如数组中的行索引与列索引，例如：

```go
package main

import "fmt"

func main() {
	var arr [2][2]string
	arr[0][0] = "java"
	arr[0][1] = "c"
	arr[1][0] = "php"
	arr[1][1] = "golang"

	for i := 0; i < len(arr); i++ {
		for j := 0; j < len(arr[0]); j++ {
			fmt.Printf("%v\t",arr[i][j])
		}
		fmt.Println()
	}
}
```



#### 使用for-range遍历

```go
package main

import "fmt"

func main() {
	var arr = [3][4]int{
		{0, 1, 2, 3},
		{4, 5, 6, 7},
		{8, 9, 10, 11},
	}

	for _, item := range arr {
		for _, val := range item {
			fmt.Println(val)
		}
	}
}
```

