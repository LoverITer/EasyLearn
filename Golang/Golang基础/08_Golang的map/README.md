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

