## JAVA 常见OOM场景及分析



![](http://image.easyblog.top/1671173578633ddac76d1-f952-4bbd-b8db-01067f9d0cfb.png)

Java 常见OOM场景有一下四种类型：

- [1.堆内存溢出]()
- [2.⽅法区(运⾏时常量池)和元空间溢出]()
- [3.直接内存溢出]()
- [4.栈内存溢出]()



## 1.堆内存溢出

堆内存溢出十分常见，⼤部分⼈都应该能想得到这⼀点，堆内存⽤来存储对象实例，我们只要不停的创建对象，并且保证GC Roots和对象之间有没有可达路径避免垃圾回收，那么在对象数量超过最⼤堆的⼤⼩限制后很快就能出现这个异常。
写⼀段代码测试⼀下，设置堆内存⼤⼩2M。

![](http://image.easyblog.top/1671173993168bc8ad9bb-30e3-40c1-b5c3-05428c50a30a.png)

测试代码：

```java
@Service
public class OOMDemoService {


    /**
     * 触发OOM：堆内存溢出
     */
    public void oomHeap(){
        List<UserBean> list=new ArrayList<>();
        while (true){
            list.add(new UserBean());
        }
    }
    
}
```

运⾏代码，很快能看见OOM异常出现:

![img](http://image.easyblog.top/167117483188276758606-4187-4ddf-a72a-329f5e4c3d9a.png)

⼀般的排查⽅式可以通过设置`-XX: +HeapDumpOnOutOfMemoryError`在发⽣异常时dump出当前的内存转储快照来分析，分析可以使⽤Eclipse Memory Analyzer(MAT)来分析，独⽴⽂件可以在[官⽹下载](https://www.eclipse.org/mat/downloads.php)。

另外如果使⽤的是IDEA的话，可以使⽤商业版JProfiler或者开源版本的JVM-Profiler，IDEA2018版本之后内置了分析⼯具，包括Flame Graph(⽕焰图)和Call Tree(调⽤树)功能。

**Flame Graph(⽕焰图)**

![](http://image.easyblog.top/16711780481927075a671-0ab4-46be-af75-d9f8e9e43ff6.png)

**Call Tree(调⽤树)**

![](http://image.easyblog.top/16711781099058983103d-9dfd-4374-93f1-8bbb6b47edc1.png)



## 2.⽅法区(运⾏时常量池)和元空间溢出

⽅法区和堆⼀样，是线程共享的区域，包含Class⽂件信息、运⾏时常量池、常量池，运⾏时常量池和常量池的主要区别是具备动态性，也就是不⼀定⾮要是在Class⽂件中的常量池中的内容才能进⼊运⾏时常量池，运⾏期间也可以可以将新的常量放⼊池中，⽐如String.intern()⽅法。

我们写⼀段代码验证⼀下String.intern()，同时我们设置`-XX:MetaspaceSize=50m -XX:MaxMetaspaceSize=50m `元空间⼤⼩。由于我使⽤的是1.8版本的JDK，⽽1.8版本之前⽅法区存在于永久代(PermGen)，1.8之后取消了永久代的概念，转为元空间(Metaspace)，如果是之前版本可以设置PermSize MaxPermSize永久代的⼤⼩。
```java
@Service
public class OOMDemoService {

    /**
     * 常量池溢出
     */
    public void oomMetaSpace() {
        String str = UUID.randomUUID().toString();
        List<String> stringList = new ArrayList<>();
        while (true) {
            str = str + str;
            stringList.add(str.intern());
        }
    }

}
```

调用上面方法，出现OOM异常：

![](http://image.easyblog.top/1671179558376207cc093-be10-492d-98d6-9e4d95d1dc73.png)

不过比较奇怪的是OOM提示是`Java head sapce`溢出了，这是为什么呢？

intern()本⾝是⼀个native⽅法，它的作⽤是：如果字符串常量池中已经包含⼀个等String对象的字符串，则返回代表池中这个字符串的String对象；否则，将String对象包含的字符串添加到常量池中，并且返回String对象的引⽤。
⽽在1.7版本之后，字符串常量池已经转移到堆区，所以会报出堆内存溢出的错误，如果1.7之前版本的话会看见PermGen space的报错。



## 3.直接内存溢出

直接内存并不是虚拟机运⾏时数据区域的⼀部分，并且不受堆内存的限制，但是受到机器内存⼤⼩的限制。常见的⽐如在NIO中可以使⽤native函数直接分配堆外内存就容易导致OOM的问题。
直接内存⼤⼩可以通过`-XX:MaxDirectMemorySize`指定，如果不指定，则默认与Java 堆最⼤值-Xmx⼀样。

由直接内存导致的内存溢出，⼀个明显的特征是在Dump⽂件中不会看见明显的异常，如果发现OOM之后Dump⽂件很⼩，⽽程序中⼜直 接或间接使⽤了NIO，那就可以考虑检查⼀下是不是这⽅⾯的原因。



## 4.栈内存溢出

栈是线程私有，它的⽣命周期和线程相同。每个⽅法在执⾏的同时都会创建⼀个栈帧⽤于存储局部变量表、操作数栈、动态链接、⽅法出⼝ 等信息，⽅法调⽤的过程就是栈帧⼊栈和出栈的过程。
在java虚拟机规范中，对虚拟机栈定义了两种异常：

* 如果线程请求的栈深度⼤于虚拟机所允许的深度，将抛出**StackOverflowError**异常
* 如果虚拟机栈可以动态扩展，并且扩展时⽆法申请到⾜够的内存，抛出**OutOfMemoryError**异常

先写⼀段代码测试⼀下，设置-Xss160k，-Xss代表每个线程的栈内存⼤⼩

```java
@Service
public class OOMDemoService {

    /**
     * 栈溢出
     * @param dep
     */
    public void oomStackOverflow(int dep){
        System.out.println("OomStackOverflow dep:"+dep);
        oomStackOverflow(dep+1);
    }

}
```

运⾏代码，很快能看见StackOverflow异常出现:

![](http://image.easyblog.top/1671180501041196f4e7b-18b3-4c1d-ae1e-f4723e0a7a89.png)