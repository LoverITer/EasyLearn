Golang的流程控制





流程控制是每种编程语言控制逻辑走向和执行次序的重要部分，流程控制可以说是一门语言的“经脉"

Go 语言中最常用的流程控制有if和for，而switch和goto主要是为了简化代码、降低重复代码而生的结构，属于扩展类的流程控制。





## if语句

条件语句需要开发者通过指定一个或多个条件，并通过测试条件是否为true来决定是否执行指定语句，并在条件为false的情况在执行另外的语句。下图展示了程序条件语句的结构：

![image-20221112124207698](https://img2022.cnblogs.com/blog/3005399/202211/3005399-20221112155904832-476211662.png)

### 单分支语句

**基本语法**

```go
if 条件表达式{

  执行代码块

}
```

**Demo**

```go
package main

import "fmt"

func main() {
	var age int32
  fmt.Scanln(&age)
  // 如果年龄大于18，程序输出成年了~
	if age>18{
		fmt.Println("成年了~")
	}
}
```





### 双分支语句

**基本语法**

```go
if 条件表达式 {
  
  执行代码块 
  
} else {
  
  执行代码块
  
}
```



**Demo**

```go
package main

import "fmt"

func main() {
	var age int32
  fmt.Scanln(&age)
	if age>18{
		fmt.Println("成年了~")
	} else{
		fmt.Println("未成年~")
	}
}
```





### 多分支语句

**基本语法**

```go
if 条件表达式1 {
  
  执行代码块1
  
} else if 条件表达式2 {
  
  执行代码块2
  
}
...
else {
  
  执行代码块n
  
}
```



**Demo**

```go
package main

import "fmt"

func main() {
	var grade int32
	fmt.Scanln(&grade)
	if grade >= 90 {
		fmt.Println("成绩优秀")
	} else if grade >= 70 {
		fmt.Println("成绩良好")
	} else if grade >= 60 {
		fmt.Println("成绩及格")
	} else {
		fmt.Println("成绩不及格")
	}
}
```





## switch语句

switch语句基于不同条件执行不同动作，每一个case分支都是唯一的，从上到下逐一测试，直到匹配为止。

匹配后面也不需要加break

**基本语法**

```go
switch 表达式 {
  case 表达式1,表达式2...: 语句块1
  case 表达式3,表达式4...: 语句块2
  ...
  default: 语句块
}
```



**Demo**

```go
package main

import "fmt"

func main() {
	var date int32
	fmt.Scanln(&date)
	switch date {
	case 1:
		fmt.Println("星期天")
	case 2:
		fmt.Println("星期一")
	case 3:
		fmt.Println("星期二")
	case 4:
		fmt.Println("星期三")
	case 5:
		fmt.Println("星期四")
	case 6:
		fmt.Println("星期五")
	case 7:
		fmt.Println("星期六")
	default:
		fmt.Println("非法输入")
	}
}
```



Switch 语句细节：

1）case/switch后是一个表达式（即常量值，变量，一个返回值的函数就可以）

2）case后的数据类型必须和switch的表达式数据一致

3）case后可以带多个表达式，使用逗号间隔，比如：case 表达式1，表达式2...

4）case后如果是常量值（字面值）则要求不能重复

5）default语句不是必须的

6）switch后也可以不带表达式

7）switch后也可以值即声明一个变量，分号结束，不推荐

8）switch穿透-fallthrough，在case后加上这一个会继续执行下一个case

![](http://image.easyblog.top/1672471275442c1fd8933-6f4a-4810-a300-a842d60606dd.png)

9）type switch：switch语句还可以用于type-switch来判断某个interface，变量中实际指向的是变量类型

![](http://image.easyblog.top/167247149418628bc6df8-26b3-4841-9f61-c5cb646c32f9.png)



## for循环控制



### for语句

Go语言中的所有循环操作均可使用for关键字来完成

**基本语法**

```go
for 初始语句; 条件表达式; 结束语句 {
  
	循环体
  
}
```

条件表达式返回true时循环体不停地进行循环，直到条件表达式返回false时自动退出循环



实例：打印1~100

```go
package main

import "fmt"

func main() {

	for i:=1;i<=100;i++{
		fmt.Printf("%v\t",i)
	}

}
```



注意，**在Go语言中，没有while语句，我们可以通过for来代替**

```go
package main

import "fmt"

func main() {

	// while
	index := 1
	for {
		if index <=100 {
			fmt.Printf("%v\t", index)
			index++
			continue
		}
		break
	}
}
```

for循环可以通过break、goto、return、panic语句退出循环



### for-range 语句

Go提供for-range的方式，可以方便遍历字符串和数组。

（1）传统方式遍历字符串

```go
package main

import "fmt"

func main() {
	// for-range
	str :="hello world"
	for index:=0;index<len(str);index++{
		fmt.Printf("%c\t",str[index])
	}
}
```

使用这种方式遍历字符串，如果字符串中包含有在ASCII字符串之外的字符的话就会乱码，所以我们可以使用for-range来安全的遍历字符串。

（2）for-range方式遍历

* for-range遍历字符串

```go
package main

import "fmt"

func main() {
	str2 :="你好，这里是Go的世界！"
	for index,ch:=range str2{
		fmt.Printf("%v-%c\t",index,ch)
	}
}
```



* 使用for-range遍历数组

```go
package main

import "fmt"

func main() {
	// for-range遍历数组
	var arr = []string{"php", "java", "c++", "golang"}
	for index, val := range arr {
		fmt.Printf("%v,%v\n", index, val)
	}
}
```





## goto语句

goto 语句通过标签进行代码间的无条件跳转。goto 语句可以在快速跳出循环、避免重复退出上有一定的帮助。Go语言中使用goto语句能简化一些代码的实现过程。

基本语法

```go
goto lable
......
lable: statement
```



**Demo**

```go
package main

import "fmt"

func main() {
	var age int32
	fmt.Scanln(&age)

	if age>18{
		fmt.Println("成年人")
	}else {
		goto lable1
	}

	fmt.Println("aaa")
	fmt.Println("bbb")
lable1:
	fmt.Println("ccc")
	fmt.Println("ddd")
}
```



