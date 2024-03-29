# Golang语言发展简史





##  开发文档

[https://studygolang.com/pkgdoc](https://gitee.com/link?target=https%3A%2F%2Fstudygolang.com%2Fpkgdoc)



## 一、前言

  Go语言和Golang其实就是同一回事，go语言的全称：“go programming language”，Go语言通常被叫做Golang的原因主要有两个：

1、go.org域名被注册了，所以Go只能用golang.org作为官网域名；

2、go太广泛了，搜索引擎不能很好的识别，搜索golang更能缩小范围精确的找到答案；



## 二、Go语言的核心开发团队

Ken Thompson（肯·汤普逊）:1983年图灵奖、1998年美国国家技术奖得主，他与Dennis Ritchie是Unix系统的原创者。Thompson也发明了C语言、B语言，同时也是C语言的主要发明人。
Rob Pike（罗布·派克）： 加拿大人，曾是贝尔实验室的Unix团队和Plan 9操作计划的成员。他与Thompson公事多年，并共创出广泛使用的UTF-8字元编码。（ps：Go语言的图标-gopher 囊地鼠，是Rob Pike老婆制作的）
Robert Griesemer：曾协助制作Java的HotSpot编译器，和Chrom浏览器的JavaScript引擎V8.

![](https://img-blog.csdnimg.cn/img_convert/e84dcb62e821a925096f631d239e7fd4.png)

##  三、Google为什么要创建Go

- 计算机硬件技术更新频繁，性能提高很快。目前主流的编程语言发展明显落后于硬件，不能合理利用多核多CPU的优势提升软件系统性能。
- 软件系统复杂度越来越高，维护成本越来越高，目前缺乏一个足够简洁高效的编程语言。
  - 现有编程语言存在：风格不统一、计算能力不够、处理大并发不够好
- 企业运行维护很多c/c++的项目，c/c++程序运行速度虽然很快，但是编译速度确很慢，同时还存在内存泄漏的一系列的困扰需要解决。



## 四、Go语言的里程碑

* 2007年，谷歌工程师Ken Thompson、Rob Pike、Robert Griesemer开始设计一门全新的语言，这是Go语言的最初原型。
* 2009.11.10 ，Google将Go语言以开放源代码的形式向全球发布。
* 2015年8月19日  ，Go1.5版本发布，本次更新中移除了“最后残余的C代码”，请内存管理方面权威专家Rick Hudson对GC进行重新设计（重要的修正）
* 2017年2月16日 ， Go1.8版本发布
* 2017年8月24日 ， Go1.9版本发布
* 2018年2月16日 ， Go1.10版本发布
* 2018年8月24日 ， Go1.11版本发布，开始不支持WinXP系统
* 2019年2月25日 ， Go1.12版本发布
* 2019年9月03日 ， Go1.13版本发布
* 2020年2月25日 ， Go1.14版本发布
* 2020年8月11日 ， Go1.15版本发布
* 2021年2月16日 ， Go1.16版本发布
* 2021年8月16日，  Go1.17版本发布
* 2022年3月15日，  Go1.18版本发布
* 2022年8月2日，    Go1.19版本发布



##  五、Go语言的特点

Go语言保证了既能到达静态编译语言的安全和性能，又达到了动态语言开发维护的高效率，使用一个表达式来形容Go语言：**Go=C+Python**，说明Go语言既有C静态语言程序的运行速度，又能达到Python动态语言的快速开发。

* 从c语言中继承了很多理念，包括表达式语法，控制结构，基础数据类型，调用参数传值，指针等等，也保留了和C语言一样的编译执行方式及弱化的指针。

  ```go
  func test(val *int){
    *val = 20
  }
  ```

* 引入包的概念，用于组织程序结构，Go语言的一个文件都要归属于一个包，而不能单独存在。

* 垃圾回收机制，内存自动回收，不需开发人员管理 【稍微不注意就会出现内存泄漏】

* **天然并发**【重要特点】

  - 从语言层面支持并发，实现简单
  - goroutine，轻量级线程，可实现大并发处理，高效利用多核。
  - 基于CPS并发模型（Communicating Sequential Processes）实现

* 吸收了管道通信机制，形成go语言特有的管道channel，通过管道channel，可以实现不同的goroute之间的相互通信

* 函数返回多个值（实例代码）

* 新的创新：比如切片slice，延时执行defer等





##  六、Go语言开发注意事项

- Go源文件以“go”为扩展名
- Go应用程序的执行入口是main()方法
- Go语言严格区分大小写。
- Go方法由一条条语句构成，每个语句后不需要分号（Go语言会在每行后自动加分号），这也体现出Golang的简洁性。
- Go编译器是一行行进行编译的，因此我们一行就写一条语句，不能把多条语句写在同一个，否则报错
- Go语言定义的变量或者import的包如果没有使用到，代码不能编译通过
- 大括号都是成对出现的，缺一不可。