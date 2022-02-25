## Scala从入门到精通—概述



Scala是一门多范式的编程语言，一种类似Java的编程语言，设计初衷是实现可伸缩的语言、并集成面向对象编程和函数式编程的各种特性。

Java语言在不停地变化之中，为了优化自身代码冗长和数据处理方面的不足，向Lisp，Clojure、Erlang、Haskell等函数式编程语言借鉴了经验，Java8的出现才使得Java具备了[函数式编程](https://www.zhihu.com/search?q=函数式编程&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A1301190035})的能力，某些场景下开发效率大幅提高，让我对Java有了更大的信心。

Scala的语法跟Java8比较接近，比如Scala特质Trait相当于Java的接口Interface，继承都是extends，

**只要真正理解了面向对象OOP和函数式FP的思想，就不会觉得学习Scala困难。**

**未来，Java开发者将会更多地使用Scala和Java混合编程。**

随着Java的版本不断升级，Java的特性是越来越靠近Scala，Java8引入函数式编程，Java11才有了类型推断，但是仍然没有Scala那么简洁和强大：

**1、Scala可以用一个简单的下划线“_”去表示一系列参数和参数列表。**

**2、Scala可以实现多重赋值和返回多个值。**

**3、Java的泛型存在类型检测的缺陷，尤其是Map数据结构，即使Map的key是String类型，我们仍然可以向get方法传入Integar类型的值，运行也不会报错，但是却获取不了对应的value（null），导致粗心的程序员以为是逻辑的错误，白白浪费了调试和检查逻辑的时间！**

**4、Java对字符串的处理是日常开发中最经常做的事，用“+”拼接、StringBuffer、StringBuilder、String.formmat等等方式都很繁琐，而且容易出错，而Scala字符串插值真的很便捷。**

**5、当你在使用Java，你可能会为double[]和Double[]之间不能直接赋值而烦恼，你可能会为list列表转Array数组、Array数组转list列表专门写了一些Util工具类，你可能为了将单线程改成多线程，要把以前写死的ArrayList类型一个一个地改写成Vector向量类型或者其他线程安全的List类型，你可能检查很久才发现Map的get方法的参数类型传错而程序不报错。**

**6、使用Scala编程，就已经具备了使用Lombok的@val @var @Getter @Setter @Accessors @Value @Data @Builder的便捷性。**

**7、Scala内置了协变的特性，可以显著提高使用集合类开发的效率，这一特性是Java8所不具备，Java8的协变要编写泛型方法才能实现。**

**8、Java 8的惰性求值是通过实现Supplier接口，每一次都要提供初始化的方法，而Scala直接使用关键字lazy或者view，无需显式调用get方法。**

**9、Scala没有static，使用伴生对象实现单例，是真正意义上的面向对象OOP。**

**Java项目中通常会有很多Util工具类，而这些工具类的方法都是被写成static静态方法（类方法），是面向过程的设计而不是面向对象OOP的设计，导致封装性和重用性极差（无状态，无法链式调用等等）。**

**10、Scala拥有强大的模式匹配。Scala的模式变量是专门用于匹配模式的，可以被后续代码引用。**

**11、Java不支持运算符重载，语法冗长，Scala支持运算符重载，灵活性更强，可以显著地减少代码量。**

**注意！不热爱学习的人，任何编程语言都难。不遵循代码规范，不会总结编程方法论的人，任何语言都会写出很烂的代码，跟编程语言没有太大关系，即使是使用Python也不例外。**

**Scala比其他函数式编程语言更进一步的是Actor模型，即基于共享数据的隔离对多线程代码进行的封装。使用Akka可以轻松实现健壮的高并发分布式系统，Flink的一些版本集成了Akka作为基础组件。**

Scala的生态完善，就应用来说，有大数据流计算框架spark、分布式发布订阅系统Kafka、Akka、Play和Lift等Web框架、Rpc框架Finagle、测试框架Specs等等。

编程语言很多，Go、Python、Scala、Kotlin、Rust、Ruby、Dart、Julia，讨论比较多的是Scala、Go、Python，就目前的应用而言：

Go适合服务端、桌面应用程序开发。

Scala适合服务端、大数据、数据挖掘、NLP、图像识别、机器学习、[深度学习](https://www.zhihu.com/search?q=深度学习&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A1301190035})…等等开发。

Python适合做网络爬虫、自动化运维、快速地实现算法的原型。

但是Python仍有一些不足之处。

Python性能是个问题，而且多线程并发是劣势。

Python大型项目，架构和重构是灾难。

Python的代码缩进是个坑，当你在使用Python，一小部分代码的修改可能导致你要重新调整整个文件的缩进。

Python是动态语言，一些本应该报错的地方没有报错，比如你直接把Json字符串拷贝到.py文件，想要把Json字符串设置为post网络请求的参数，编译器不会报错，运行程序执行post网络请求也不会报错，你以为是程序逻辑的错误，但其实仅仅是因为Json字符串没有包括在两个双引号里面。还有一些本应该马上报错的地方只能在调试的时候才报错，结果你可能要修改很多地方。

未来，随着Scala生态进一步完善，Scala在数据科学和人工智能领域将会大有建树，越来越多Java开发者会拥抱Scala，Java和Scala混合编程开发会是Java开发者的最佳选择。





### 参考

* [为什么要使用 Scala 语言？Scala 语言的优势在哪里？](https://zhuanlan.zhihu.com/p/150701210)

