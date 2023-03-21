# Java面经-2023版 



# 一、Java基础

## 1.1 语法基础

### 面向对象特性？

面相对象三大特性：**封装、继承和多态**。

* 封装（Encapsulation）

  **封装是将数据和方法组合在一起，以防外部对象直接对数据进行访问和修改。封装使得对象内部实现对于外部来说不可见，可以隐藏对象的复杂性，提高安全性和可维护性**。

* 继承（Inheritance）

  **继承是指一个类可以从另一个类继承它的属性和方法。通过继承，子类可以重用父类的方法，减少重复的代码，并且可以添加或修改子类的特定功能**。

* 多态（Polymorphism）

  **多态是指同一个方法可以具有不同的行为。它使得程序可以处理不同类型的对象，提高代码的灵活性和可扩展性**。

  多态分为**编译时多态**和**运行时多态**：

  * 编译时多态：编译时多态是静态的，主要指的是**方法的重载**，他是根据方法参数列表的不同来区分不同的方法
* 运行时多态：运行时多态是动态的，只在运行时提过**动态绑定**实现具体类型确定的。Java实现多态有3个必要条件：**继承、重写和向上转型**。只用满足这3个条件，才能在同一个继承结构中使用统一的逻辑处理不同的对象，从而执行不同的行为。



### 对equals()和hashCode()的理解?

#### 为什么在重写 equals 方法的时候需要重写 hashCode 方法?

> equals方法经常会被用来判断两个对象是否相等，而hashCode方法主要用在容器类中，用于计算一个对象的hash值，通过hash值可以将对象映射到散列表上，并且可以通过hash值快速查找对象。
>
> 通常两个对象的equals方法相等，那么hashCode值一定相等；但是，两个对象的hashCode相等 ，equals值不一定相等。所以如果只重写了equals方法，没有重写hashCode方法，会导致即使equals方法返回true，但因为hashCode值不同而无法正确查找到对象。

#### 有没有可能两个不相等的对象有相同的 hashcode?

> 存在这种情况，这也是HashMap这种散列表出现哈希冲突的原因。

#### 两个相当的对象可以存在不同的hashcode？

> 相当的两个对象不能有不同的hashcode，他们必须存在一个唯一的hash值，以便hashmap可以将其正确的映射



### final、finalize 和 finally 的不同之处?

* final

  > final 是一个修饰符，可以用来修复变量，方法和类，修复变量时，变量不可被修改（基本数据类型，引用类型不用变引用但是引用内数据可变）；修复方法时方法不可被重写；修饰类时，类不可继承；

* finanlly

  > finanlly 是一个关键字，通常与try-catch块一起用于异常处理，它表示一定会被执行的逻辑

* finalize

  > finalize 是一个方法，Java 运行在执行垃圾回收之前调用finalize方法纸执行一些必要的清理工作。这个方法会由GC在在确定这个对象不被任何对象引用时，调用此对象的 finalize 方法，但是什么时候调用 JVM 并没有保证。



### String、StringBuffer与StringBuilder的区别？

* 可变和不可变

  > **String 是不可改变的字符串**，一但实例化一个String对象之后，它的值就不能被修改。每次修改String对象的值时，都会创建一个新的String对象，因此在对字符串进行大量修改时，使用String可能会导致性能问题，**StringBuffer 和 StringBuilder 都是可变字符串**，在频繁修改字符串的场景下应该考虑使用 StringBuffer 和 StringBuilder。

* 线程安全和不安全

  > **StringBuffer 是线程安全的，因为它的所有公共方法都加了synchorinzed关键字来保证线程安全**，因此，在多线程环境下操作字符串，应该考虑使用StringBuffer。
  >
  > **StringBuilder 是非线程安全的，由于没有线程同步的开销，StringBuilder的性能比StringBuffer要更好**，在单线程环境下操作字符串，应该考虑使用StringBuilder。



### 接口与抽象类的区别？

* 定义的修饰符不同：接口使用`interface`关键字，抽象类使用 `abstract class`；
* 抽象类可以有构造方法，接口不能有构造方法；
* 抽象类可以有普通成员变量，接口不能有普通成员变量；
* 抽象类和接口中都可以定义静态成员变量，但是抽象类的中静态成员变量可以被任意访问修饰符修饰，接口中的静态成员变量只能是 `public static final`

* 抽象类中可以定义静态方法，接口中不能定义静态方法
* 抽象类的方法的访问修饰符可以是`private、protected、public` ，接口中方法的访问修饰符如果是抽象方法只能是`abstract public`,JDK 8 之后允许存在 `default` 方法，JDK 9 之后运行存在 `private` 方法



## 1.2 泛型

### 什么是泛型？

> **Java中泛型是一种语言机制，它允许在编译时对代码中要操作的数据类型参数化**，使用泛型可以增强代码的类型安全性和可读性，同时也提高了代码的可重用性和灵活性。
>
> **使用泛型的主要目的是解决类和方法的重复代码问题**。在没有泛型的情况下，需要为每种数据类型都编写一个独立的类或方法来处理数据类型的差异，这会导致代码冗长且难以维护。

### 泛型如何定义？

Java中的泛型分为**类泛型**和**方法泛型**

* 类泛型定义示例

```java
interface Info<T>{        // 在接口上定义泛型  
    public T getVar() ; // 定义抽象方法，抽象方法的返回值就是泛型类型  
}  
class InfoImpl<T> implements Info<T>{   // 定义泛型接口的子类  
    private T var ;             // 定义属性  
    public InfoImpl(T var){     // 通过构造方法设置属性内容  
        this.setVar(var) ;    
    }  
    public void setVar(T var){  
        this.var = var ;  
    }  
    public T getVar(){  
        return this.var ;  
    }  
} 
public class GenericsDemo24{  
    public static void main(String arsg[]){  
        Info<String> i = null;        // 声明接口对象  
        i = new InfoImpl<String>("汤姆") ;  // 通过子类实例化对象  
        System.out.println("内容：" + i.getVar()) ;  
    }  
}  
```



* 方法泛型定义示例

![img](https://pdai.tech/images/java/java-basic-generic-4.png)

定义泛型方法必须要在方法返回值前面使用`<T>` 来标准这是一个泛型方法，持有一个泛型 `T`，然后才可以用泛型T作为方法的返回值



### 泛型的上限和下限？

在使用泛型时我们限定泛型参数只能是某个类的子类或某个类的父类。

* **指定泛型上限**

  ```java
  class Info<T extends Number>{    // 此处泛型只能是数字类型
      private T var ;        // 定义泛型变量
      public void setVar(T var){
          this.var = var ;
      }
      public T getVar(){
          return this.var ;
      }
      public String toString(){    // 直接打印
          return this.var.toString() ;
      }
  }
  ```

* 指定泛型下限

  ```java
  class Info<? super String>{   // 只能接收String或Object类型的泛型
      private T var ;        // 定义泛型变量
      public void setVar(T var){
          this.var = var ;
      }
      public T getVar(){
          return this.var ;
      }
      public String toString(){    // 直接打印
          return this.var.toString() ;
      }
  }
  ```

  

### 如何理解Java中的泛型是伪泛型？

> Java中泛型特性是JDK1.5加入的，因此**为了兼容之前的版本，Java在语法上支持泛型，但在编译的时候使用了泛型擦除（Type Erasure）**。具体来说，泛型类会在编译的时候擦除为原始类型，泛型参数会被替换为其上限，如果没有指定上限则指定为Object，例如`List<String>` 在编译时会被擦除为`List` ，原始类型为 `Object`



## 1.3 注解

### 什么是注解？

Java中注解是**一种特殊标记（元数据）**，这些标记可以在编译、类加载、运⾏时被读取，并执⾏相对应的处理。

Java中提供了4个元注解，`@Target`、`@Inherited`、`@Retention`、`@Documented`，它们是可以用来定义其他注解的注解，分别表示的含义如下：

* `@Target`：表示注解可以被标记的位置，比如 `ElementType.PARAMETER` 表示可以标记在参数上
* `@Retention`：表示标明注解被保留的阶段，比如 `RetentionPolicy.RUNTIME` 表示在运行时保留
* `@Inherited`：表示注解是否可以被继承，即注解知否可以标注在其他注解上
* `@Documented`：表示是否生成javadoc文档



注意：**注解本身只是一种标记或元数据，注解本身不会对程序的运行时行为产生直接影响，需要通过程序来读取和处理注解信息**。



### 自定义注解？

定义注解的关键字是 `@interface` eg，定义 `@DLock` 注解

```java
@Target({ElementType.TYPE})
@Retention({RetentionPolicy.RUNTIME})
public @interface DLock{
  
}
```



注解定义本身没有难度，难点在于如何处理注解的逻辑，对于注解逻辑的处理分为**运行时处理逻辑和编译时处理逻辑**：

* **运行时处理逻辑**

如果想在**运行时处理一些注解逻辑时，我们可以指定注解的Retention为RUNTIME，然后结合反射、AOP等机制获取到反射动态处理一些逻辑。**

```java
// spring环境下定义注解行为
@Around("@annotaion(top.easyblog.titan.aspect.DLock)")
public Object doAround(ProceedingJoinPoint pjp) throws Throwable {
  
  // do someting
}
```



* **编译时处理逻辑**

如果你想要**在编译期间处理注解相关的逻辑，需要指定注解的Retention为SOURCE或CLASS这两个级别，并且需要继承 `AbstractProcessor` 并实现 `process` ⽅法**，项目里常用的lombok就是这个的实现原理。

具体底层原理是JAVA源代码在编译为字节码的过程中有一步**注解抽象语法树解析**，这一步中会解析注解，然后做处理的逻辑，具体的逻辑就是我们定义在具体 XxxxProcessor 里面的 process方法中

![](http://image.easyblog.top/1679324887761903f48cb-b6a3-458a-834e-4ae3a26cfa71.png)



## 1.4 异常

### Java异常体系？

![](https://cdn.learnku.com/uploads/images/202107/12/83656/9unTT81Zpt.png!large)

Java将系统的所有异常包装成对应的异常类型（对象），所有异常的基类是 `Throwable`，又分为 `Error`  和 `Exception` 两大子异常类型。

* Error

  Error 是有JVM虚拟机抛出来的异常，描述的系统内部的一些崩溃，例如Java虚拟机崩溃。这种情况仅凭程序自身是无法处理的，在程序中也不会对Error异常进行捕捉和抛出，常见的Error：`OutOfMemoryError、StackOverflowError

* Excetion

  Exception 是在程序中可以处理的异常，又分为 `Checked Exception` 和 `RuntimeException` 两类异常。

  * Checked Exception 是一类受检查的异常，这类异常是必须显式解决的，如果不解决就会出现编译异常，常见的受检查异常比如：`IOException`、`SQLException`、`FileNotFoundException`....
  * RuntimeException 是运行时异常，是在代码运行过程中发生的异常，一般是是代码错误引起的异常，程序中可以处理显式这些异常，也可以不处理。常见的的运行时异常如：`NullPointerException`、`ClassCastException`、`ClassNotFoundException`、`ArrayIndexOutOfBoundException`、`ArithmeticException`



### 关键字 throw 和 throws 的区别

* throws 用于申明异常，如果一个受检查异常没有在方法中被处理，那么必须使用throws来申明异常，throws 关键字放在方法签名的尾部，eg：

  ```java
  public void doSometing() throws IOException{
  
  
  }

* throw 用于在代码关键节点数据状态非法，程序无法继续向下执行时手动抛出异常

  ```java
  public void doSometing(boolean status) {
  
    if(!status){
      throw new IllegalArgumentException("参数异常");
    }
  
    ...deal biz
  }
  ```

  

###  try-with-resource？

try-with-resources 是 Java7 映引入的一个特性，它可以管理实现了`AutoCloseable`接口的资源实现自动关闭资源操作，当程序try块执行完成之后，会帮助我们自动关闭资源而不再需要我们手动显式的关闭资源。

```java
public void doSometing(File file){
  
  try(
     FileInputStream fis=new FileInputStream(file);
     FileOutputStream fos=new FileOutputStream(file);
  ){
    //deal biz...
  }catch(IOException e){
    //deal exception...
  }
  
}
```





### 异常底层原理？

* 代码层面

  代码层面，异常的实现原理就和**责任链模式**类似，当发生异常后异常会沿着调用栈一直往上抛，直到遇到可以处理此异常的地方，直到抛到main还没被处理则停止继续往上抛，JVM会将程序终止并打印调用栈。

* JVM层面

  JVM层面的实现原理我们需要深入字节码，如果一段代码中包含try-catch块，则编译器在编译的时候会给这段代码的字节码生成一个`异常表（Exception Table）`,异常表里记录了从哪一行到哪一行发生异常后跳转到哪一行处理的详细逻辑。如下是一个例子：

  ```java
  public static void simpleTryCatch() {
     try {
         testNPE();
     } catch (Exception e) {
         e.printStackTrace();
     }
  }
  ```

  编译如上代码获得class文件后使用javap来分析这段代码

  ```java
  //javap -c Main
   public static void simpleTryCatch();
      Code:
         0: invokestatic  #3                  // Method testNPE:()V
         3: goto          11
         6: astore_0
         7: aload_0
         8: invokevirtual #5                  // Method java/lang/Exception.printStackTrace:()V
        11: return
      Exception table:
         from    to  target type
             0     3     6   Class java/lang/Exception
  ```

  在上面代码中可以看到 Exception table，异常表中包含了一个或多个异常处理者(Exception Handler)的信息，这些信息包含如下：

  - **from** 可能发生异常的起始点
  - **to** 可能发生异常的结束点
  - **target** 上述from和to之前发生异常后的异常处理者的位置
  - **type** 异常处理者处理的异常的类信息

> Exception table:
>        from    to  target type
>            0     3     6   Class java/lang/Exception
>
> 所以如上异常表的含义就是：第0~3行可能会发生异常，当发生异常后跳转到第6行处理，异常类型是 java.lang.Exception



## 1.5 反射

### 什么是反射？

**反射是Java提供的一种机制，利用这种机制我们可以在程序运行的时候动态获取类的信息，访问和操作类的属性、方法和构造器，利用反射还可以在运行时动态实例化对象、调用方法以及修改属性值等操作。反射机制给Java程序的编写提供了极大的灵活性，但是也需要更高的运行时开销**。



