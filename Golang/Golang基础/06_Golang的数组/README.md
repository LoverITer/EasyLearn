Golang的数组





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

```go
package main

import "fmt"

func main() {
	//自动推断数组长度
	var arr = [...]string{"java", "c", "golang", "php", "c++"}
	// for-range 遍历数组
	for _, val := range arr {
		fmt.Println(val)
	}
}
```

