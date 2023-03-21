Golang的map



##  map的介绍

map是一种无序的基于key-value的数据结构，**Go语言中的map是引用类型，必须初始化才能使用**。

Go语言中map的定义语法如下：

```go
map[KeyType]ValueType
```

其中：

- KeyType：表示键的类型
- ValueType：表示键对应的值的类型



## map的声明和初始化

### 方法1：使用make函数

**map类型的变量默认初始值为nil，需要使用make()函数来分配内存**。语法为：

```go
package main

import "fmt"

func main() {
	// 声明一个key和value类型均为string的map
	var userIdMap=make(map[string]string)
	userIdMap["100100"]="张三"
	userIdMap["100101"]="李四"
	userIdMap["100102"]="王五"
	userIdMap["100103"]="富贵"
  // map[100100:张三 100101:李四 100102:王五 100103:富贵]
	fmt.Println(userIdMap)
  // 张三
	fmt.Println(userIdMap["100100"])
}
```



### 方法2：字面量初始化

```go
package main

import "fmt"

func main() {
  //字面量初始化map
	var orderIdMap = map[int]string{
		1: "6123000333113",
		2: "6123000333114",
		3: "6123000333115",
		4: "6123000333116",
		5: "6123000333117",
	}
  // map[1:6123000333113 2:6123000333114 3:6123000333115 4:6123000333116 5:6123000333117]
	fmt.Println(orderIdMap)
  // 6123000333113
	fmt.Println(orderIdMap[1])
}
```



##  遍历map

### 基本的遍历方式

Map可以使用for range遍历

```go
package main

import "fmt"

func main() {

  // 创建map
	var orderIdMap = map[int]string{
		1: "6123000333113",
		2: "6123000333114",
		3: "6123000333115",
		4: "6123000333116",
		5: "6123000333117",
	}

	// for-range 遍历map
	for key,value:=range orderIdMap{
		fmt.Printf("key:%v,value:%v\n",key,value)
	}
}
```

**注意：Map遍历的时候元素的顺序是随机的**



### 在遍历中增加元素

在遍历的同时增加元素，增加的元素是可以被遍历到的

```go
package main

import "fmt"

func main() {
	var orderIdMap = map[int]string{
		1: "6123000333113",
		2: "6123000333114",
		3: "6123000333115",
		4: "6123000333116",
		5: "6123000333117",
	}

	// for-range 遍历map
	for key,value:=range orderIdMap{
		orderIdMap[6]="6123000333118"
		fmt.Printf("key:%v,value:%v\n",key,value)
	}
}
```



##  判断map中某个键值是否存在

获取map中的元素时会返回两个，一个是值本身，另一个是值是否存在，如下示例：

```go
package main

import "fmt"

func main() {
	var orderIdMap = map[int]string{
		1: "6123000333113",
		2: "6123000333114",
		3: "6123000333115",
		4: "6123000333116",
		5: "6123000333117",
	}

	//判断key是否存在
	var value, exists = orderIdMap[5]
	fmt.Printf("存在：%v,值=%v", exists, value)
}
```



##  使用delete()函数移除键值对

使用delete()函数从map中移除指定键值对，函数定义如下：

```go
// The delete built-in function deletes the element with the specified key
// (m[key]) from the map. If m is nil or there is no such element, delete
// is a no-op.
func delete(m map[Type]Type1, key Type)
```

其中：

- map对象：表示要删除键值对的map对象
- key：表示要删除的键值对的键



示例代码如下

```go
package main

import "fmt"

func main() {
	var orderIdMap = map[int]string{
		1: "6123000333113",
		2: "6123000333114",
		3: "6123000333115",
		4: "6123000333116",
		5: "6123000333117",
	}

	//删除key=1的键
	delete(orderIdMap, 1)
	for key, value := range orderIdMap {
		fmt.Printf("%v,%v\n", key, value)
	}
}
```

