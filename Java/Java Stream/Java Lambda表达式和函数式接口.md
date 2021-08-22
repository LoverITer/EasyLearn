###  Java Lambda表达式和函数式接口



### 1. Lambda表达式介绍

**Lambda 表达式本质是一种匿名函数**，简单地说，它是没有声明的方法，也没有访问修饰符、返回值声明和名字。你可以将其想做一种速记，在你需要使用某个方法的地方写上它。当某个方法只使用一次，而且定义很简短，使用这种速记替代之尤其有效，这样，你就不必在类中费力写声明与方法了。



![](http://image.easyblog.top/15944430236789033877e-1619-41b5-a683-12bbbdd13a7c.png)

Lambda表达式是Java8提供的一个新的特性，它支持Java也能进行简单的“函数式编程”。 它是一个匿名函数，Lambda表达式基于

数学中的λ演算得名，直接对应于其中的lambda抽象(lambda abstraction)，是一个匿名函数，即没有函数名的函数。

Java中Lambda表达式有一般由三部分组成，语法如下所示。

```java
()->{expression}
//或
(arg1,arg2...)->{expression}
```

- **第一部分：形式参数**，参数是提供给函数式接口方法所使用的；
- **第二部分：箭头符号 ** `->`；
- **第三部分：方法体**，这一块需要使用花括号 `{ }` 将方法体包裹起来，括号里可以是表达式和代码块 。
- **第四部分：返回值**，如果主体只有一个表达式返回值则编译器会自动返回值，大括号需要指定明表达式返回了一个数值。

此外，关于Lambda达式还需要知道这些：

* **参数的类型既可以明确声明，也可以根据上下文来推断**。例如：`(int a)`与`(a)`效果相同

* **所有参数需包含在圆括号内，参数之间用逗号相隔**。例如：`(a, b)` 或 `(int a, int b)` 或 `(String a, int b, float c)`

* **一个 Lambda 表达式可以有零个、一个或多个参数**。当参数集为空的时候，可以只写空圆括号，例如：`() -> 42` ，当只有一个参数，且其类型可推导时，圆括号 `()` 可省略。例如：`a -> return a*a`

* **一个Lambda 表达式的主体可包含零条或多条语句**。如果 Lambda 表达式的主体只有一条语句，花括号 `{}` 可省略。匿名函数的返回类型与该主体表达式一致，如果 Lambda 表达式的主体包含一条以上语句，则表达式必须包含在花括号{}中（形成代码块）。匿名函数的返回类型与代码块的返回类型一致，若没有返回则为空

  

### 2. 函数式接口

提到Lambda表达式，就绕不过另一个概念：**函数式接口(Functional Interface)**。函数式接口的概念也很简单：一个**有且仅有一个抽象方法**，但是可以有多个非抽象方法的接口就是一个函数式接口。

#### 2.1 @FunctionalInterface

`@FunctionalInterface`是Java8新引入的一个注解，正如它源码里的注释写到，该注解一般用在接口层面，且注解的接口要有且仅有一个抽象方法。具体就是说，注解在Interface上，且Interface里只能有一个抽象方法，可以有default方法。因为从语义上来讲，一个函数式接口需要通过一个**逻辑上的**方法表达一个单一函数。那理解这个单一就很重要了，单一不是说限制你一个interface里只有一个抽象方法，单是多个方法的其他方法需要是继承自Object的public方法，或者你要想绕过，就自己实现default。函数式接口自己本身一定是只有一个抽象方法。同时，如果是**Object类的public方法，也是不允许的**。官方的说明翻译如下：

> 如果一个接口I，I有一组抽象方法集合M，且这些方法都不是Object类的public签名方法，那么如果存在一个M中的方法m，满足：
>
> m的签名是所有M中方法签名的子签名。
>
> m对于M中的每个方法都是返回类型可替换的。 此时，接口I是一个函数式接口。

官网的概念比较晦涩难懂，举几个栗子吧。

**case 1**

如下case 1实现的接口是一个函数式接口吗？

```java
@FunctionalInterface
public interface Fun{
    void runTask();
}
```

没问题，当然是函数式接口！

**case 2**

如下case 2中的这个接口去掉了`@FunctionalInterface` 注解，还是函数式接口吗？

```java
public interface Fun{
    void runTask();
}
```

答案：也是函数式接口。

有些小伙伴可能会惊讶了，没关系，打开`@FunctionalInterface` 注解的源码会看到如下这么一段注释：

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210724233605.png)

翻译过来大致意思就是说：

> 无论接口声明中是否存在 @FunctionalInterface 注释，编译器都会将满足函数式接口定义的任何接口视为函数式接口。

**case 3**

上面都是接口中有方法定义的场景，接下来看看一下几种情况是不是函数式接口：

* [1] 定义的接口中没有任何方法定义的情况

```java
@FunctionalInterface
public interface Fun{
}
```

* [2] 定义的方法中有方法定义，但是和Object类中的public方法签名重复了

```java
@FunctionalInterface
public interface CalculationFun {
    boolean equals(Object obj);
}

```

* [3] 定义的方法和Object类的非public方法签名重复了

```java
@FunctionalInterface
public interface CalculationFun {
    Object clone();
}
```

这三种情况，哪些是函数式的接口，哪些不是函数式接口？

其实，真正理解了函数式接口的定义了，这个问题不难回答，[1] [2]都不是函数式接口，他们都会报同样的错误`No target method found`。只有[3]是函数式接口。

[1]报错的原因很简单，明确说了只有一个抽象方法嘛，没有肯定就会报错；[2]报错的原因前面也有提到过，就是Object类中的public也是不允许在函数接口中有同样方法签名的抽象方法出现的，`equals(Object obj)`是Object类的public方法，而`clone()`则是Object类的protected方法，因此这就是同样是Object中的方法，`equals(Object obj)`不行，但是`clone()`却可以的原因。

所以总结一句话就是：

**函数式接口，有且仅有一个抽象方法，Object的public方法除外，并且判断一个接口是不是函数式接口的标准并不是这个接口有没有在头上标注`@FunctionalInterface`注解**。

其实，在Java8 以前，已有大量函数式接口形式的接口，只是没有强制声明。例如`Runnable`、`Callable`、`ActionListener`、`InvocationHandler`、`FileFilter` ......。

在Java8 新增加的函数接口在`java.util.function` 包下，里面新增了很多内置函数式接口用来支持 Java的函数式编程，该包中的函数式接口主要包含四大类：`Function`、`Predicate`、`Supplier`、`Consumer`。下面依次展开详细说明一下。 

#### 2.2 Function 接口

`java.util.function.Function<T, R>` 接口定义了一个叫作`apply`的方法，它接受一个泛型T的对象，并返回一个泛型R的对象，如果需要把一个输入映射为另一种形式的输出的操作（比如给字符串加密），那么就可以使用函数式接口，该接口通常也被称为**功能性接口**。其接口以及主要方法的定义如下所示。

```java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
    ...
}
```

函数意为将参数T传递给一个函数，返回R。即：**R=Function(T)** 

同时该接口默认实现了3个default方法，分别是**compose**、**andThen**和**identity**，对应的函数表达为：

* compose对应**V=Function(ParamFunction(T))**，体现嵌套关系；
* andThen对应**V=ParamFunction(Function(T))**，转换了嵌套的顺序；
* 还有identity对应了一个传递自身的函数调用对应**Function(T)=T**。

从这里看出来，compose和andThen对于两个函数f和g来说，**f.compose(g)等价于g.andThen(f)**。看个例子： 

```java
public class FunctionTest {

    public static void main(String[] args) {
        //定义两个函数
        Function<Integer, Integer> add = x -> x + 1;
        Function<Integer, Integer> multiply = x -> x * 2;

        int x=2;
        System.out.println("when x= "+x+",f(x)=x+1,then f(x)="+add.apply(x));
        System.out.println("when x= "+x+",g(x)=2x,then g(x)="+multiply.apply(x));
        System.out.println("when x= "+x+",f(x)=x+1,g(x)=2x,then f(g(x))="+add.compose(multiply).apply(x));
        System.out.println("when x= "+x+",f(x)=x+1,then g(f(x))="+add.andThen(multiply).apply(x));
    }
}
```

上面的代码输出如下图所示。

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210725004832.png)

**高阶函数** 

只是普通的lambda表达式，其能力有限。我们会希望引入更强大的函数能力——高阶函数，可以定义任意同类计算的函数。 
比如这个函数定义，参数是z，返回值是一个Function，这个Function本身又接受另一个参数y，返回z+y。于是我们可以根据这个函数，定义任意加法函数： 

```java
public class FunctionTest {

    public static void main(String[] args) {

        Function<Integer, Function<Integer, Integer>> addMaker = x -> y -> x + y;
        //任意数+3的加法器
        Function<Integer, Integer> add3 = addMaker.apply(3);
        //任意数+5的加法器
        Function<Integer, Integer> add5 = addMaker.apply(5);

        //6+3
        System.out.println(add3.apply(6));
        //10+5
        System.out.println(add5.apply(10));
    }
}
```

高阶函数接受一个函数作为参数，结果返回另一个函数，所以是典型的**函数到函数的映射**。 `BiFunction` 提供了**二元函数的一个接口声明**，举例来说： 

```java
public class FunctionTest {

    public static void main(String[] args) {
        BiFunction<Integer,Integer,Integer> multiply=(x,y)->x*y;
        //计算3*5
        System.out.println("when x= "+x+",y="+y+",f(z)=x*y,then f(z)="+multiply.apply(3,5));
    }
}
```

其结果将是：when x= 3,y=5,f(z)=x*y,then f(z)=15

二元函数没有compose能力，只是默认实现了andThen。 有了一元和二元函数，那么可以通过组合扩展出更多的函数可能。 

Function接口相关的接口包括：

|      Interface       |              functional method               | 说明                                                     |
| :------------------: | :------------------------------------------: | :------------------------------------------------------- |
|   LongFunction<R>    |             R apply(long value)              |                                                          |
|      Double<R>       |            R apply(double value)             |                                                          |
|   ToIntFunction<T>   |           int applyAsInt(T value)            | 以下三个接口，指定了返回参数类型，接收参数类型为泛型T    |
|  ToLongFunction<T>   |          long applyAsLong(T value)           |                                                          |
| ToDoubleFunction<T>  |        double applyAsDouble(T value)         |                                                          |
|  IntToLongFunction   |         long applyAsLong(int value)          | 以下六个接口，既指定了接收参数类型，也指定了返回参数类型 |
| IntToDoubleFunction  |        double applyAsLong(int value)         |                                                          |
|  LongToIntFunction   |         int applyAsLong(long value)          |                                                          |
| LongToDoubleFunction |        double applyAsLong(long value)        |                                                          |
| DoubleToIntFunction  |        int applyAsLong(double value)         |                                                          |
| DoubleToLongFunction |        long applyAsLong(double value)        |                                                          |
|   UnaryOperator<T>   |                 T apply(T t)                 | 特殊的Function，接收参数类型和返回参数类型一样           |
|   IntUnaryOperator   |     int applyAsInt(int left, int right)      | 以下三个接口，指定了接收参数和返回参数类型，并且都一样   |
|  LongUnaryOperator   |    long applyAsInt(long left, long right)    |                                                          |
| DoubleUnaryOperator  | double applyAsInt(double left, double right) |                                                          |



最后，简单实现一下如何使用Function如何实现给字符串加密的。

```java
public class FunctionTest2 {
    public static void main(String[] args) {
        Function<String, String> sha1Encoder = (str) -> {
            if (null == str || str.length() == 0) {
                throw new IllegalArgumentException();
            }
            return EncryptUtils.getInstance().SHA1(str);
        };

        System.out.println(sha1Encoder.apply("12345678"));
    }
}
```

实现的加密效果如下。

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210725113314.png)



#### 2.3 Consumer 接口

`java.util.function.Consumer<T>` 接口定义了一个名叫 `accept `的抽象方法，它接受泛型T，没有返回值(void)。如果需要访问类型 T 的对象，并对其执行某些操作，可以使用这个接口，通常称为**消费性接口**。其接口以及主要方法的定义如下所示。

```java
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
}
```

这个函数接口其实可以使用上面`Function<T,R>` 来代替，但是对于一些纯粹消费型的函数，Function一定需要一个泛型参数作为返回值类型，这时Function就一直返回一个无用的值。比如下面的例子，如果没有Consumer，类似的行为使用Function表达就一定需要一个无用的返回值。 

```java
public class ConsumerTest {

    public static void main(String[] args) {
        Consumer<Integer> consumer= System.out::println;
        consumer.accept(100);

        //实现了同样的功能，但是这个实现总会有无用的返回值
        Function<Integer,Integer> function=(x)->{
            System.out.println(x);
            return x;
        };
        function.apply(100);
    }

}
```

**其他的Consumer接口：**

| interface      | functional method         | 说明                               |
| -------------- | ------------------------- | ---------------------------------- |
| IntConsumer    | void accept(int value)    | 以下三个类，接收一个指定类型的参数 |
| LongConsumer   | void accept(long value)   |                                    |
| DoubleConsumer | void accept(double value) |                                    |



#### 2.4 Supplier 接口

`java.util.function.Supplier<T>` 接口定义了一个 `get` 抽象方法，它没有参数，返回一个泛型T的对象，这类似于一个工厂方法,通常称为**功能性接口**。其接口以及方法的定义如下所示。

```java
@FunctionalInterface
public interface Supplier<T> {
    T get();
}
```

这个抽象方法的声明，同Consumer相反，是一个只声明了返回值，不需要参数的函数。也就是说Supplier其实表达的不是从一个参数空间到结果空间的映射能力，而是表达一种生成能力，因为我们常见的场景中不止是要消费（Consumer）或者是简单的映（Function），还包括了new（创建）这个动作。而Supplier就表达了这种能力。 

如下示例是一个产出指定个数可以指定范围的的随机数字,放入集合中的方法。

```java
public class SupplierTest {
    public static void main(String[] args) {
        //这里使用Supplier的好处就是我们可以以参数的形式自定义产生数字的范围，之后一切交给Supplier，而不用写很多方法
        List<Integer> integerList = getList(10, () -> (int) (Math.random() * 1000));
        integerList.forEach(System.out::println);

        List<Integer> integerList2 = getList(10, () -> (int) (Math.random() * 5000));
        integerList2.forEach(System.out::println);
    }

    /**
     * 产生指定数量的指定范围的数字并添加到集合中
     *
     * @param num
     * @param supplier
     * @return
     */
    public static List<Integer> getList(int num, Supplier<Integer> supplier) {
        List<Integer> numList = new ArrayList<>();
        for (int i = 0; i < num; i++) {
            numList.add(supplier.get());
        }
        return numList;
    }
}
```



**其他的Supplier接口**：

| interface       | functional method      | 说明                       |
| --------------- | ---------------------- | -------------------------- |
| BooleanSupplier | boolean getAsBoolean() | 以下三个接口，返回指定类型 |
| IntSupplier     | int getAsInt()         |                            |
| LongSupplier    | long getAsLong()       |                            |
| DoubleSupplier  | double getAsDouble()   |                            |



#### 2.5 Predicate 接口

`java.util.function.Predicate<T>`  接口定义了一个名叫 `test `的抽象方法，它接受泛型 T 对象，并返回一个boolean值。在对类型 T进行断言判断时，可以使用这个接口，通常称为**断言性接口**。 其实，Predicate在Stream中有应用，Stream的filter方法就是接受Predicate作为入参的。这个在[Java流式编程]() 这篇教程中有过深入分析。其接口以及主要方法的定义如下所示。

```java
@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);
}
```

使用这个接口，我呢可以实现对参数的校验验证等操作，比如对字符串进行判空的操作。

```java
public class PredicateTest {
    public static void main(String[] args) {
        //测试字符串是否为空，如果为空返回true，不为空返回false
        Predicate<String> predicate=(str)->null==str||str.replaceAll("^\\s+$","").length()==0;

        //test
        System.out.println(predicate.test("hello,world!"));
        System.out.println(predicate.test(null));
        System.out.println(predicate.test(""));
        System.out.println(predicate.test("   "));
        System.out.println(predicate.test("   hello,world!"));
        System.out.println(predicate.test("hello,world!    "));
    }
}
```

实现的效果如下。

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210725115024.png)



**其他的Predicate接口**：

| interface        | functional method          | 说明                             |
| ---------------- | -------------------------- | -------------------------------- |
| IntPredicate     | boolean test(int value)    | 以下三个接口，接收指定类型的参数 |
| LongPredicate    | boolean test(long value)   |                                  |
| DoublePredicate  | boolean test(double value) |                                  |
| BiPredicate<T,U> | boolean test(T t, U u)     | 接收两个泛型参数，分别为T，U     |