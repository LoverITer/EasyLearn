# Golang的切片



##  为什么要使用切片

切片（Slice）是一个拥有相同类型元素的**可变长度的序列**。它是基于数组类型做的一层封装。 它非常灵活，**支持自动扩容**。**切片是一个引用类型**，它的内部结构包含地址、长度和容量。



## 定义切片

声明切片类型的基本语法如下：

```go
var var_name [] T
```

其中：

- var_name：表示变量名
- T：表示切片中的元素类型

例如下面是定义了一个int类型切片：

```go
package main

import "fmt"

func main() {

	// 和定义一维数组的方式唯一区别是：切片不需要指定数组长度
	var list []int32
	fmt.Println(list)
}

```



##  基于数组定义切片

由于切片的底层就是一个数组，所以我们可以基于数组来定义切片

```go
package main

import "fmt"

func main() {
	// 定义一个数组
	var arr = [10]int32{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
	// 将数组全部转换为切片
	var list2 = arr[:]
	// 输出 [1 2 3 4 5 6 7 8 9 10]
	fmt.Println(list2)

	//截取数组下标0~5的元素组成切片
	var list3 = arr[0:5]
	// 输出 [1 2 3 4 5]
	fmt.Println(list3)

	//截取数组下标3之前的元素组成切片
	var list4 = arr[:3]
	// 输出 [1 2 3]
	fmt.Println(list4)

	//截取数组下标5之后的元素组成切片
	var list5 = arr[5:]
	// 输出 [6 7 8 9 10]
	fmt.Println(list5)

}
```

需要注意的是，我们不仅可以对数组进行切片，还可以切片再切片



##  切片的长度和容量

**切片拥有自己的长度和容量**，我们可以通过使用内置的`len()`函数求长度，使用内置的`cap()` 函数求切片的容量。

* 切片的长度就是它所包含的元素个数。

* 切片的容量是从它的第一个元素开始数，到其底层数组元素末尾的个数。切片s的长度和容量可通过表达式len（s）和cap（s）来获取。

```go
package main

import "fmt"

func main() {
	// 定义一个数组
	var arr = [10]int32{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
	// 将数组全部转换为切片
	var list2 = arr[:]
	fmt.Printf("切片长度：%v,切片容量：%v\n", len(list2), cap(list2))

	//截取数组下标0~5的元素组成切片
	var list3 = arr[0:5]
	fmt.Printf("切片长度：%v,切片容量：%v\n", len(list3), cap(list3))

	//截取数组下标3之前的元素组成切片
	var list4 = arr[:3]
	fmt.Printf("切片长度：%v,切片容量：%v\n", len(list4), cap(list4))

	//截取数组下标5之后的元素组成切片
	var list5 = arr[5:]
	fmt.Printf("切片长度：%v,切片容量：%v\n", len(list5), cap(list5))

}
```

运行结果：

```te
切片长度：10,切片容量：10
切片长度：5,切片容量：10
切片长度：3,切片容量：10
切片长度：5,切片容量：5
```





##  切片的遍历

切片的遍历和数组的遍历方式一样，可以通过下标遍历，也可以通过for-range遍历

### 下标遍历

```go
package main

import "fmt"

func main() {

	// 和定义一维数组的方式唯一区别是：切片不需要指定数组长度
	var list = []int32{1, 2, 3, 4, 5, 6, 7, 8}

	for i := 0; i < len(list); i++ {
		fmt.Printf("%v\t", list[i])
	}

}
```



### for-range遍历

```go
package main

import "fmt"

func main() {

	// 和定义一维数组的方式唯一区别是：切片不需要指定数组长度
	var list = []int32{1, 2, 3, 4, 5, 6, 7, 8}
	for _, val := range list {
		fmt.Printf("%v\t", val)
	}
}
```





##  切片的本质

切片的本质就是对底层数组的封装，它包含了三个信息

- 底层数组的指针
- 切片的长度(len)
- 切片的容量(cap)

举个例子，现在有一个数组 a := [8]int {0,1,2,3,4,5,6,7}，切片 s1 := a[:5]，相应示意图如下：

![](https://gitee.com/moxi159753/LearningNotes/raw/master/Golang/Golang%E5%9F%BA%E7%A1%80/7_Go%E7%9A%84%E5%88%87%E7%89%87/images/image-20200720094247624.png)

切片 s2 := a[3:6]，相应示意图如下：

![](https://gitee.com/moxi159753/LearningNotes/raw/master/Golang/Golang%E5%9F%BA%E7%A1%80/7_Go%E7%9A%84%E5%88%87%E7%89%87/images/image-20200720094336749.png)



##  使用make函数构造切片

我们上面都是基于数组来创建切片的，如果需要动态的创建一个切片，我们就需要使用内置的make函数，格式如下：

```go
make ([]T, size, cap)
```

其中：

- T：切片的元素类型
- size：切片中元素的数量
- cap：切片的容量



举例：定义一个int类型的，len=5，cap=10的切片

```go
var list=make([]int,5,10)
// [0,0,0,0,0]
fmt.Println(list)
// 长度：5, 容量10
fmt.Printf("长度：%d, 容量%d", len(list), cap(list))
```

需要注意的是，Golang中没办法通过下标来给切片扩容，如果需要扩容，需要用到append

```go
var list=[]int{1,2,3,4,5}
// [1,2,3,4,5]
fmt.Println(list)

// 拼接6
list=append(list,6)
// [1,2,3,4,5,6]
fmt.Println(list)
```

同时切片还可以将两个切片进行合并：

```go
var list=[]int{1,2,3,4,5}
// [1,2,3,4,5]
fmt.Println(list)

var list1=[]int{6,7,8}
//注意这里list1后面的... 这是Golang的语法，不是省略号！
list=append(list,list1...)
// [1,2,3,4,5,6,7,8]
fmt.Println(list)
```

需要注意的是，切片会有一个扩容操作，当元素存放不下的时候，**会将原来的容量扩大两倍**

```go
package main

import "fmt"

func main() {
	var list = make([]int, 5, 10)
	// [0,0,0,0,0]
	fmt.Println(list)
	// 长度：5, 容量10
	fmt.Printf("长度：%d, 容量%d\n", len(list), cap(list))

	for i := 0; i < len(list); i++ {
		list[i] = i
	}
  
	var list1 = []int{8, 9, 10, 11, 12, 13}
	list = append(list, list1...)
	fmt.Println(list)
  //长度：11, 容量20
	fmt.Printf("长度：%d, 容量%d\n", len(list), cap(list))
}
```

##  使用copy()函数复制切片

前面我们知道，**数组是值类型，切片是引用数据类型**

- 值类型：改变变量副本的时候，不会改变变量本身
- 引用类型：改变变量副本值的时候，会改变变量本身的值

如果我们需要改变切片的值，同时又不想影响到原来的切片，那么就需要用到copy函数

```go
package main

import "fmt"

func main() {
	var list = []int{1, 2, 3, 4, 5}
  // 输出 [1,2,3,4,5]
	fmt.Println(list)

	var list1 = make([]int, len(list), cap(list))
	copy(list1,list)
	list1[0]=100
  // 输出 [100,2,3,4,5]
	fmt.Println(list1)
}
```

##  切片的排序算法以及sort包

使用切片编写一个简单的冒泡排序算法

```go
package main

import "fmt"

func main() {
   var list = []int{5, 9, 3, 1, 0, 12, 11, 4, 5, 8, 2, -3}
   for i := 0; i < len(list); i++ {
      flag := false
      for j := 0; j < len(list)-i-1; j++ {
         if list[j] > list[j+1] {
            var tmp = list[j]
            list[j] = list[j+1]
            list[j+1] = tmp
            flag = true
         }
      }
      if !flag {
         break
      }
   }

  
   // 输出 [-3 0 1 2 3 4 5 5 8 9 11 12]
   fmt.Println(list)
}
```

对于int、float64 和 string数组或是切片的排序，go分别提供了sort.Ints()、sort.Float64s() 和 sort.Strings()函数，默认都是从小到大进行排序

```go
package main

import (
	"fmt"
	"sort"
)

func main() {
	var list = []int{5, 9, 3, 1, 0, 12, 11, 4, 5, 8, 2, -3}
	sort.Ints(list)
	// 输出 [-3 0 1 2 3 4 5 5 8 9 11 12]
	fmt.Println(list)


	var strs=[]string{"banana","apple","orange","cherry"}
	// 按照自然顺序排列，输出 [apple banana cherry orange]
	sort.Strings(strs)
	fmt.Println(strs)
}
```

