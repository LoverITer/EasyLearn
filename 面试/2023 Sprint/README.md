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

equals方法经常会被用来判断两个对象是否相等，而hashCode方法主要用在容器类中，用于计算一个对象的hash值，通过hash值可以将对象映射到散列表上，并且可以通过hash值快速查找对象。

通常两个对象的equals方法相等，那么hashCode值一定相等；但是，两个对象的hashCode相等 ，equals值不一定相等。所以如果只重写了equals方法，没有重写hashCode方法，会导致即使equals方法返回true，但因为hashCode值不同而无法正确查找到对象。

#### 有没有可能两个不相等的对象有相同的 hashcode?

存在这种情况，这也是HashMap这种散列表出现哈希冲突的原因。

#### 两个相当的对象可以存在不同的hashcode？

相当的两个对象不能有不同的hashcode，他们必须存在一个唯一的hash值，以便hashmap可以将其正确的映射



### final、finalize 和 finally 的不同之处?

* final

  `final` 是一个修饰符，可以用来修复变量，方法和类，修复变量时，变量不可被修改（基本数据类型，引用类型不用变引用但是引用内数据可变）；修复方法时方法不可被重写；修饰类时，类不可继承；

* finanlly

  `finanlly` 是一个关键字，通常与try-catch块一起用于异常处理，它表示一定会被执行的逻辑

* finalize

  `finalize` 是一个方法，Java 运行在执行垃圾回收之前调用finalize方法纸执行一些必要的清理工作。这个方法会由GC在在确定这个对象不被任何对象引用时，调用此对象的 finalize 方法，但是什么时候调用 JVM 并没有保证。



### String、StringBuffer与StringBuilder的区别？

* 可变和不可变

  **String 是不可改变的字符串**，一但实例化一个String对象之后，它的值就不能被修改。每次修改String对象的值时，都会创建一个新的String对象，因此在对字符串进行大量修改时，使用String可能会导致性能问题，**StringBuffer 和 StringBuilder 都是可变字符串**，在频繁修改字符串的场景下应该考虑使用 StringBuffer 和 StringBuilder。

* 线程安全和不安全

  **StringBuffer 是线程安全的，因为它的所有公共方法都加了synchorinzed关键字来保证线程安全**，因此，在多线程环境下操作字符串，应该考虑使用StringBuffer。
  
  **StringBuilder 是非线程安全的，由于没有线程同步的开销，StringBuilder的性能比StringBuffer要更好**，在单线程环境下操作字符串，应该考虑使用StringBuilder。



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



### Class Signature 保留泛型信息？

Q：Java中泛型在编译的类型会被擦除，为什么使用反射还是可以获取到泛型类型？

A：网上比较靠谱的回答：https://wiyi.org/type-erasure.html

总结：**当类的泛型信息被填写了实际的类型信息，这些类型信息会在编译时被记录在class signature**。比如下面例子：

```java
public class UserRepository implements CrudRepository<User> {

    private final LinkedList<User> users = new LinkedList<User>();

    @Override
    public User find() {
        return users.getFirst();
    }

    @Override
    public void add(User user) {
        users.add(user);
    }

    @Override
    public boolean contain(User user) {
        return users.contains(user);
    }
}
```

UserRepository实现了CrudRepository接口，且指定类型为User，这些类型在编译时就会被编码。看看字节码。

```java
//...忽略上面的
{
  public java.io.Serializable find();
    descriptor: ()Ljava/io/Serializable;
    flags: ACC_PUBLIC, ACC_BRIDGE, ACC_SYNTHETIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokevirtual #11                 // Method find:()Lorg/wiyi/generic/domain/User;
         4: areturn
      LineNumberTable:
        line 8: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lorg/wiyi/generic/UserRepository;
}
Signature: #37                          // Ljava/lang/Object;Lorg/wiyi/generic/CrudRepository<Lorg/wiyi/generic/domain/User;>;
SourceFile: "UserRepository.java"
```



这里Class Signature上明确包含了泛型的实际类型，这种情况下计算存在泛型擦除，我们也可以使用反射获取到泛型的实际类型。JDK提供了2个方法获取Class的Signature，分别是:

```java
Class.getGenericInterfaces();
Class.getGenericSuperclass();
```



### 反射的使用？

* Class获取类信息

  ```java
   @Test                                                                    
   public void classTest() throws Exception {                               
       // 获取Class对象的三种方式                                                    
       System.out.println("根据类名:  \t" + User.class);                        
       User user = new User();                                              
       System.out.println("根据对象:  \t" + user.getClass());                   
       System.out.println("根据全限定类名:\t" + Class.forName("org.example.User"));
       // 常用的方法                                                             
       Class<? extends User> userClass = user.getClass();                   
       System.out.println("获取全限定类名:\t" + userClass.getName());              
       System.out.println("获取类名:\t" + userClass.getSimpleName());           
       System.out.println("实例化:\t" + userClass.newInstance());              
   }                                                                        
  ```

  

* Constructor获取并操作构造器

  ```java
  //todo
  ```

  

* Field获取并操作属性

  ```java
   @Test                                                                                                                             
   public void fieldTest() throws IllegalAccessException {                                                                           
       User user = new User("张三", "Test address", 20);                                                                               
       Class<? extends User> userClass = user.getClass();                                                                            
       // 获取成员变量                                                                                                                     
       Field[] fields = userClass.getDeclaredFields();                                                                               
       for (Field field : fields) {                                                                                                  
           // 私有成员变量需要"暴力"访问                                                                                                         
           field.setAccessible(true);                                                                                                
                                                                                                                                     
           // 获取成员变量名和值                                                                                                              
           System.out.println(field.getName()+"="+field.get(user));                                                                  
       }                                                                                                                             
                                                                                                                                     
       Optional<Field> fieldOptional = Arrays.stream(fields).filter(field -> StringUtils.equals(field.getName(), "name")).findAny(); 
       fieldOptional.ifPresent((field)->{                                                                                            
           try {                                                                                                                     
               // 操作属性设置值                                                                                                            
               field.set(user,"李四");                                                                                                 
           } catch (IllegalAccessException e) {                                                                                      
               throw new RuntimeException(e);                                                                                        
           }                                                                                                                         
       });                                                                                                                           
       System.out.println(user);                                                                                                     
   }                                                                                                                                                                                           
  ```

  

* Method获取并操作方法

  ```java
   @Test                                                                                                                                       
   public void methodTest(){                                                                                                                   
       User user = new User("张三", "Test address", 20);                                                                                         
       Class<? extends User> userClass = user.getClass();                                                                                      
       // 获取成员方法                                                                                                                               
       Method[] declaredMethods = userClass.getDeclaredMethods();                                                                              
       for (Method method : declaredMethods) {                                                                                                 
           System.out.println("方法名："+method.getName());                                                                                        
           System.out.println("方法入参："+Arrays.toString(method.getParameters()));                                                                
           System.out.println("方法修饰符："+ Modifier.toString(method.getModifiers()));                                                             
           System.out.println("方法返回值："+method.getReturnType());                                                                                
           System.out.println("==========================");                                                                                   
       }                                                                                                                                       
                                                                                                                                               
       // 通过反射调用方法                                                                                                                             
       Optional<Method> toString = Arrays.stream(declaredMethods).filter(method -> StringUtils.equals(method.getName(), "toString")).findAny();
       toString.ifPresent(method -> {                                                                                                          
           Object invoke = null;                                                                                                               
           try {   
               // invoke 反射调用方法
               invoke = method.invoke(user, null);                                                                                             
           } catch (IllegalAccessException | InvocationTargetException e) {                                                                    
               throw new RuntimeException(e);                                                                                                  
           }                                                                                                                                   
           System.out.println(invoke);                                                                                                         
       });                                                                                                                                     
   }                                                                                                                                           
  ```




# 二、Java集合

Java 容器分为 `Collection` 和 `Map` 两大类，Collection 集合的子接口有 `List`、`Set` 和 `Queue`三种子接口。我们比较常用的是Set、List，**Map接口不是Collection的子接口**。

## 2.1 Java的集合体系

### 介绍集合体系？

Java 有三种类型的集合：`List`、`Set` 和 `Queue`：

* **List**

  List 是一种**元素有序且元素可能重复的容器**，有序是指**元素存入集合的顺序和取出的顺序一致**，重复是指在List集合中元素时可以重复出现的，元素可以为null，常见的List集合容器实现：ArrayList、LinkedList、Vector....

* **Set**

  Set是一种**元素无序且元素不重复的容器**，无序是指元素存入集合的顺序和取出的顺序可能不一致，不重复是指Set集合中元素不能重复粗周年，Set集合中元素也可以为null，常见的Set集合容器实现：HashSet、TreeSet、LinkedHashSet...

* **Queue**

  Queue 是一种**先进先出（FIFO）数据结构（队列）**，Queue 接口常见的实现有：LinkedList（一个基于链表实现的队列）、PriorityQueue（一个基于堆实现的优先级队列）、ArrayDeque（一个基于数组实现的双端队列）



### ArrayList的底层？

ArrayList 实现了List 接口，是一个基于数组实现的动态数组，程序可以在运行过程中动态的添加或删除元素，并根据需要自动调整容器容量大小。

ArrayList 底层数据结构是一个 **Object 数组**，数组**默认容量是10**，在默认构造方法中，ArrayList采用了类似懒加载的机制，当我们`new ArrayList()`之后底层的Object数组实际没有被实例化出来，真正实例化是在第一次调用add方法添加元素时初始化的。



### ArrayList扩容机制？

当我们向ArrayList中添加元素时会首先检查数组大小是否够用，如果不够用则会扩容，**默认的扩容大小是原数组的`1.5倍`，如果扩容1.5的大小还是没法满足添加元素所需的容量，就会使用传入的容量大小**。具体扩容机制如图：

![](http://image.easyblog.top/1679497843993519f67bb-e051-4b6e-9c0e-8a178c0a31f0.png)



### Vector （ 数组实现、 线程同步）

`Vector` 与 `ArrayList` 一样，也是通过数组实现的，不同的是**它支持线程安全的，即某一时刻只有一 个线程能够写 Vector，避免多线程同时写而引起的不一致性，但实现同步需要很高的花费，因此， 访问它比访问 ArrayList 慢**。



### LinkedList（双链表、双端队列

`LinkedList` 即实现了List接口，同时它也实现了Deque接口。它**使用链表结构存储数据，很适合数据的动态插入和删除。**具体的我们看看源码就清楚了：

![](http://image.easyblog.top/1679498607465c14aa86d-e75d-484a-a291-756f5c510ccd.png)

有两个Node类型的引用 first 和 last 分别指向链表的头和尾元素，Node类型的定义如下：

![](http://image.easyblog.top/1679498734102f7ca1841-c33b-413e-a0ec-2a34bd0456ed.png)

Node 类型的定义是一个典型的双链表的树结构样式，有一个`next引用指向下一个元素`，`prev引用指向上一个元素`，同时每个节点中都有一个元素 `item`



### ArrayList和LinkedList的区别？

1、ArrayList的实现是基于数组（可以动态扩容的数组），LinkedList的实现是基于双向链表。

2、对于随机访问，ArrayList优于LinkedList

3、对于插入和删除操作，LinkedList优于ArrayList

4、LinkedList比ArrayList更占内存，因为LinkedList的节点除了存储数据，还存储了两个引用，一个指向前一个元素，一个指向后一个元素。的效率更高，因为ArrayList是数组，所以在其中进行增删操作时，会对操作点之后所有数据的下标索引造成影响，需要进行数据的移动。



### 安全失败机制（fail-safe）和快速失败机制（fail-fast）

#### 快速失败机制 fail-fast

* **是什么？**

  **fail-fast是java集合的一种错误检测机制，在遍历集合的时候当对集合的结构进行修改（删除、增加）的时候，就会触发fail-fast。表现形式就是会抛出`ConcurrentmodificationException`异常**。

![](http://image.easyblog.top/1595933610575c4da05ab-05d7-4aca-95c6-9890668f4daf.png)

* **原理**

  **迭代器在遍历时直接访问集合中的内容，并且在遍历过程中使用一个int类型的变量modCount统计修改的次数，他表示的是集合在遍历之前已经修改过的次数，每当删除或添加元素之后都会把modCount+1，此时当调用hasNext()/next()方法时会判断modCount和exceptedModCount的值是否相等，如果相等就返回继续遍历，否则抛出ConcurrentModificationException，终止遍历。**

* **应用场景**

  java.util包下的集合类都使用了这种机制（Hashtable除外，它使用的是fail-safe）,不能在多线程下发生并发修改（迭代过程中被修改）算是一种安全机制吧。

* **如何避免快速失败机制/如何在遍历中增加或删除元素？**

  * （1）使用**iterator**遍历同时加上锁同步
  * （2）**使用线程安全的容器**，比如ArrayList 的替代是 `CopyOnWriteArrayList`

#### 安全失败机制 fail-safe

* **是什么？**

  **采用安全失败机制的集合容器，在遍历的时候不会直接在原来的内容量上遍历，而是复制一份在副本上遍历。由于遍历是在拷贝的副本上进行的，所以在遍历过程中对原集合所作的修改并不能被迭代器检测到，因此不会引发`ConcurrentmodificationException`**

* **应用场景**

  J.U.C包下都是安全失败机制容器，可以在多线程环境下并发的修改和访问

* **缺点**

  基于拷贝内容的优点是避免了Concurrent Modification Exception，但同样地，迭代器并不能访问到修改后的内容，即：迭代器遍历的是开始遍历那一刻拿到的集合拷贝，在遍历期间原集合发生的修改迭代器是不知道的。



## 2.2 Java Map

### jdk1.7 HashMap如何实现？

哈希表的实现方式主要有：开放地址法和链地址法，开放地址法就是当前key被占领了就找下一个位置，下一个位置的定位方案有线性探测再散列、平方探测再散列、随机探测再散列；链地址法就是把hash值相同的放在一个新的叫同义词链的链表里，链表头指针放到value位置。 HashMap采用了**链地址法**来解决哈希冲突。

#### 数据结构

具体的实现方式在jdk1.7和jdk1.8之后的实现是不同的，**jdk1.7之前采用了数组+链表的数据结构，为了进一步提高查询效率jdk1.8开始采用了数组+链表+红黑树的数据结构**

![img](http://image.easyblog.top/15875666326443df06e28-d331-40af-8d0c-a84654a10da0.png)



* **数组**：jdk1.7 HashMap的主干是一个Entry数组，Entry是HashMap的基本组成单元，每一个Entry包含一个key-value键值对，**HashMap默认初始数组大小是16**

  ```java
      /**
       * An empty table instance to share when the table is not inflated.
       */
      static final Entry<?,?>[] EMPTY_TABLE = {};
      /**
       * The table, resized as necessary. Length MUST Always be a power of two.
       */
      transient Entry<K,V>[] table = (Entry<K,V>[]) EMPTY_TABLE;
  ```

* **链表**：HashMap中链表是用来解决hash冲突的。每一个 Entry 中除了包含一个key-value键值对外，还包含一个指向下个Entry节点的引用，当同一个桶出现hash冲突时会将新的元素添加到前桶链表末尾节点。

#### hash方法

![](http://image.easyblog.top/16797288657161d692de0-648b-494c-a21e-030fa6dfdc68.png)                                                                   

**hash函数是一种将集合S转换为固定长度、不可逆的集合U的映射**，它的值一般是数字。 hash函数拥有无限的输入空间，但是输出空间却是有限的，这意味着hash函数一定会产生冲突，所以一个好的hash函数对于哈希表十分重要。

jdk 1.7 的hash函数入参是任意类型的key，如果key是String类型会通过stringHash32函数获取hash值，其他类型会通过key的hashCode通过和hashseed位计算获得hash值，hashseed是哈希表出事化时生成的一个随机值，每个HashMap实例的值不同。



#### indexFor方法

![img](http://image.easyblog.top/1679727543253d0217065-addc-4e6e-bbd8-7cc2565e7351.png)

有了key的映射hash值，需要找到元素插入的位置我们还需要一个方法来计算，`indexFor`就是来做这个事情的，计算方法也是使用了位运算，将**key的hash值和哈希表长度-1进行与运算**，要注意要这样干必须要保证**哈希表的哈希表的长度必须是非零2的幂**



#### put方法

![](http://image.easyblog.top/167972966524627dbb46e-b895-421f-b718-c9baffd078b2.png)

put方法可以向HashMap中添加元素

* 如果key是null，调用`putForNullKey`处理，方法会讲key是null的元素添加到哈希表index=0的位置
* key不是null，会首先使用`hash`方法获取key的hash值，然后使用`indexFor`方法获取key应该插入哈希表的位置，随后从链表第一个节点开始遍历，如果存在相等元素相等（**hash值相等并且当前节点的key==key或equals方法返回true**），则更新元素的值，否则使用`addEntry`方法添加新元素。



#### addEntry 方法

![](http://image.easyblog.top/167973263967082695f60-1fce-40a1-9d95-95e8447fd4f8.png)

添加之前会**首先检查当前哈希表的元素是否达到了阈值（默认值是哈希表大小的0.75），并且如果新增元素的桶位0号元素不是空则会触发哈希表的扩容机制`resize`**



jdk1.7添加新元素采用的是`头插法`，即新元素会插入在桶所在链表的头部，具体执行逻辑在`createEntry`方法中，它会把**新元素直接放在哈希表桶所在链表的第一个位置，然后把next指向之前的头**



#### resize方法

![](http://image.easyblog.top/16797331796096a96ef68-b69c-4ce0-85b0-90e9c610f92e.png)

jdk1.7扩容后的容量是之前哈希表容量的`2倍`，哈希表的容量大小是有上限的，当达到最大容量（`MAX_CAPACITY`，2^30）时不会再扩大哈希数组的大小。



扩容时会首先初始化一个新容量（`newCapacity`）大小的哈希数组，然后将旧哈希表中的数据移到新哈希表上，最后根据新容量计算并更新扩容阈值



#### get方法

![](http://image.easyblog.top/16797339946548a8f72d7-4c35-44ad-99f3-75dae79c0d55.png)



`get` 方法用于从HashMap中获取指定元素，首先会使用`hash`方法计算出key的hash值，然后使用`indexFor`方法计算出桶位，然后遍历桶所在位置的链表，和链表上每个节点比较，发现相等的节点返回，没有相等元素返回null



这里比较相等的方法是：**hash值相同，并且节点的key的地址相等或equals方法返回true**



#### jdk1.7 HashMap存在的问题

##### 扩容死循环

![](http://image.easyblog.top/1679734630929dedf1424-3497-46c5-897a-9718242fcd80.png)

两个线程A，B同时对HashMap进行resize()操作，在执行`transfer`方法的while循环时，若此时当前槽上的元素为`a-->b-->null`：

* 1、线程A执行到 `Entry<K,V> next = e.next;` 时发生阻塞，此时 e=a，next=b
* 2、线程B完整的执行了整段代码，此时新表newTable元素为b-->a-->null
* 3、线程A继续执行后面的代码(此时Entry已经被替换成了线程B扩容后的新表)，执行完一个循环之后，newTable变为了a<-->b，造成while(e!=null) 一直死循环，CPU飙升

 

##### 扩容数据丢失

同样在resize的transfer方法上

* 1、当前线程迁移过程中，其他线程新增的元素有可能落在已经遍历过的哈希槽上；在遍历完成之后，table数组引用指向了newTable，这时新增的元素就会丢失，被无情的垃圾回收。
* 2、如果多个线程同时执行resize，每个线程又都会new Entry[newCapacity]，此时这是线程内的局部变量，线程之前是不可见的。迁移完成后，resize的线程会给table线程共享变量，从而覆盖其他线程的操作，因此在被覆盖的new table上插入的数据会被丢弃掉。

 

### jdk1.8 HashMap如何实现？

#### 数据结构

**jdk1.8开始采用了数组+链表+红黑树的数据结构**

![img](http://image.easyblog.top/15875666522254a942394-5da9-4ede-bab9-d445eba81066.png)



与jdk1.7数据结构层面不同的是jdk1.8增加了红黑树，红黑树是为了解决当桶链表长度过长时也会降低哈希表效率的问题。红黑树（Red Black Tree）是一种**自平衡的二叉查找树**，但是每个节点上增加的了一个标记位用于存储节点的颜色（红色或黑色），红黑树的查找、删除、插入操作最坏的时间复杂度都可以达到O(lgn)。 红黑树是牺牲了严格的高度平衡的优越条件为代价，红黑树能够以O(log2 n)的时间复杂度进行搜索、插入、删除操作。此外，由于它的设计，任何不平衡都会在三次旋转之内解决。总的来说，红黑树有以下性质：

* **红黑树的节点有红色或黑色的；**
* **红黑树的根节点是黑色的；**
* **每个红色节点的两个子节点都为黑色；**
* **从任意节点出发，到其每个叶子节点的路径中包含相同数量的黑色节点**；
* **新加入红黑树的节点是红色节点**



#### hash方法

![](http://image.easyblog.top/1679736083687a58e7707-3396-41ec-8379-dcf9fb311e3c.png)

jdk1.8 HashMap的hash方法实现简化许多（得益于新的数据结构，不需要那么复杂的哈希函数了），**如果key是null，返回hash是0，否则获取key的hashCode与其高16位坐异或运算**。



#### put方法

jdk1.8 put方法直接调用了`putVal`方法

![](http://image.easyblog.top/167973643013603a0513b-b746-4d50-ad12-5632153ed56b.png)

putVal()方法的逻辑：

- （1）通过put()方法传入key-vlaue，并调用hash函数计算出key的hash值
- （2）调用到`putVal(...)`方法中，首先判断当前哈希表是否初始化了，如果没有初始化则会调用`resize()`方法执行初始化，否则进入下一步；
- （3）根据hash值计算出key-value键值对的槽位`（i=(n-1)&hash）`，如果槽位table[i]还没有元素，那就直接新添加的key-value键值对实例化成Node添加到哈希表中；如果槽位table[i]已经有元素了，则可能出现了哈希冲突，下面开始处理冲突；
- （4）如果发生了冲突，那就要还要判断此时处理冲突的情况是什么
  - 如果哈希表链表头部的key和待插入的新元素的key相等并且它们的equals()方法相等，那么就会执行元素值替换操作；
  - 否则如果此时槽位已近变成了红黑树了，那就调用`putTreeValue()`把值插入到红黑树中
  - 否则若此时的数据结构是链表，从链表头遍历每个结点，如果key相等并且equals()方法相等那就执行值替换操作，否则执行尾插，在链表尾部插入新元素，插入之后判断链表的长度是否大于等于树化阈值8，如果达到了树化阈值将会调用`treeifyBin()`执行树化逻辑，**最后能不能树化，还需要看当前哈希数组的长度是否大于最小树化容量64，如果达不到那就不会执行树化逻辑而是执行扩容`resize`逻辑**；

* （5）最后如果是执行了添加新元素的逻辑，哈希表的大小加1，并于扩容阈值比较，如果哈希表大小大于了扩容阈值则执行扩容逻辑 `resize`



#### resize方法

jdk1.8扩容关键逻辑，if 分支上面的代码主要是确定下来新的哈希表的容量`newCap`、新扩容阈值 `threshold` 和新的哈希数组，初始化新哈希数组完成后就要对老哈希表的数据进行迁移。

![](http://image.easyblog.top/167974092167887ecdacb-84fb-4729-8495-82104cf5237f.png)

`resize`数据迁移逻辑：

* 遍历老哈希表，没有元素的槽位不管，只处理有元素的槽位；处理有元素的槽位的逻辑分为三大部分：

  * 1）如果这个槽位没有后继结点了（没有形成链表），就重新计算元素移动到新的哈希表中位置并直接移动过去，重新计算桶位的算法还是：`e.hash&(newCap-1)`

    2）如果槽位有后继结点并且已近树化了（已经是红黑树了），那就将红黑树打散后，在新的哈希表中重新hash并散列到新的Node数组中

    3）如果槽位有后继结点但是没有树化（已经是链表了），那就还是用尾插法把元素移动到新的哈希表中，需要注意的是：

    - A：扩容后，**若`e.hash&oldCap=0`，那么元素在新hash数组的位置=原始位置(newTab[j] = loHead)**
    - B：扩容后，**若`e.hash&oldCap!=0`，那么元素在新hash数组的位置=原始位置下标+旧数组大小( newTab[j + oldCap] = hiHead)**。



#### get方法

![](http://image.easyblog.top/1679741982873f3c3f440-34f0-424a-a70a-805815f91069.png)

- 首先判断当前哈希表是否为空，如果为空直接返回null；
- 如果哈希表不空，就会计算key的桶位下标`table[(n-1)&hash]`，如果对应桶位元素为null会直接返回null；
- 如果哈希表不为空并且key在HashMap中存在，那就首先检查一下hash表中的这个元素是不是要找的值（判定的标准是equals方法返回true），如果是就直接返回vlaue,如果不是还会在判断一下是否存在后继结点，如果不存在直接返回null
- 如果存在后继结点，那么首先判断一下是红黑树还是链表，如果是红黑树那就到红黑树中取值，如果是链表那就从头开始遍历链表找到值后返回。





### Hashtable如何实现？

Hashtable 是 jdk1.0 开始就支持的哈希表实现，Java 2 重构的Hashtable实现了Map接口，因此，Hashtable现在集成到了集合框架中。与HashMap相比hashtable的差异点有以下几点：

* `Null key & Null value`：HashMap的key 和 value 都支持可以为null，而HashTable在遇到null时，会抛出NullPointerException 异常

* `线程安全性`:HashMap不是线程安全的，Hashtable是线程安全的，因为他的所有共有方法都是用`synchronized`关键字进行了同步

* HashMap 把 Hashtable 的 `contains` 方法去掉了，改成 `containsValue` 和 `containsKey`。因为 `contains` 方法容易让人引起误解。

* Hashtable的默认初始化大小是11，而HashMap的默认初始化大小是 16

* 当发生哈希冲突是 Hashtable 也采用了头插链表的方式插入新元素和jdk1.7之前的HasjMap方式类似，但是HashMap 在 jdk1.8 之后采用了尾插法。

* 哈希值和桶下边的计算方式不同，Hashtable 是取模运算，HashMap 是位运算，后者效率更高。

  * Hashtable 的计算方式是：

    ```java
    hash=key.hashCode()
    index=(hash & 0x7FFFFFFF) % tab.length  // 采用的是对哈希数组容量取模的方式获取下标 
    ```

  * HashMap的计算方式是：

    ```java
    hash=(key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16)
    index= hash & (tab.lenght-1)     //采用了hash值与哈希数组-1按位与的方式获取下标，这种方式效率更高
    ```

    



# 三、Java多线程

## 3.1 并发基础

### 多线程的出现是要解决什么问题的? 本质什么?

CPU、内存、I/O 设备的速度是有极大差异的，为了合理利用 CPU 的高性能，平衡这三者的速度差异，计算机体系结构、操作系统、编译程序都做出了贡献，主要体现为:

- CPU 增加了缓存，以均衡与内存的速度差异；// 导致可见性问题
- 操作系统增加了进程、线程，以分时复用 CPU，进而均衡 CPU 与 I/O 设备的速度差异；// 导致原子性问题
- 编译程序优化指令执行次序，使得缓存能够得到更加合理地利用。// 导致有序性问题



### 线程安全有哪些实现思路?

实现线程的安全的主要思路/方法有三种：互斥同步、非阻塞同步和无同步

#### 互斥同步

**互斥同步是实现线程安全最直接的方式，这种方式主要的实现方式就是控制对共享资源的访问进行互斥访问实现**，Java 中 的 `synchronized` 、`ReentrantLock` 以及各种`分布式锁`实现方案都是基于这种思想。



#### 非阻塞同步

互斥同步最主要的问题就是线程阻塞和唤醒所带来的性能问题，因此这种同步也称为阻塞同步。

互斥同步属于一种悲观的并发策略，总是认为只要不去做正确的同步措施，那就肯定会出现问题。无论共享数据是否真的会出现竞争，它都要进行加锁(这里讨论的是概念模型，实际上虚拟机会优化掉很大一部分不必要的加锁)、用户态核心态转换、维护锁计数器和检查是否有被阻塞的线程需要唤醒等操作。



**相应的非阻塞同步是一种乐观并发策略，这种实现方式主要是依靠类似版本号机制来实现**的。常见的非阻塞同步方式有以下几种：

##### CAS（Compare And Swap）

随着硬件指令集的发展，我们可以使用基于冲突检测的乐观并发策略: 先进行操作，如果没有其它线程争用共享数据，那操作就成功了，否则采取补偿措施(不断地重试，直到成功为止)。这种乐观的并发策略的许多实现都不需要将线程阻塞，因此这种同步操作称为非阻塞同步。

乐观锁需要操作和冲突检测这两个步骤具备原子性，这里就不能再使用互斥同步来保证了，只能靠硬件来完成（`lock cmpxchg`指令）。硬件支持的原子性操作最典型的是: 比较并交换(Compare-and-Swap，CAS)。**CAS 指令需要有 3 个操作数，分别是内存地址 V、旧的预期值 A 和 新值 B。当执行操作时，只有当 V 的值等于 A，才将 V 的值更新为 B，否则什么都不做**。



##### Atomic原子类

J.U.C 包里面的原子类 AtomicXxx，其中的 compareAndSet() 和 getAndIncrement() 等方法都使用了 Unsafe 类的 CAS 操作。以AtomicInteger为例：

以下代码是 `incrementAndGet()` 的源码，它调用了 `unsafe` 的 `getAndAddInt() `。

```java
public final int incrementAndGet() {
    return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
}
```

以下代码是 `getAndAddInt()` 源码，var1 指示对象内存地址，var2 指示该字段相对对象内存地址的偏移，var4 指示操作需要加的数值，这里为 1。通过 `getIntVolatile(var1, var2)` 得到旧的预期值，通过调用 `compareAndSwapInt()` 来进行 CAS 比较，如果该字段内存地址中的值等于 var5，那么就更新内存地址为 var1+var2 的变量为 var5+var4。

可以看到 getAndAddInt() 在一个循环中进行，发生冲突的做法是不断的进行重试。

```java
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

    return var5;
}
```





##### ABA问题

* **什么是ABA问题，为什么有ABA问题**

  所谓`ABA`问题，比如数值`i = 5`，`A`线程本来想改成10，在更改之前，`B`线程先将i先由5变成了6，再更新成5，但是`A`线程并不知道`i`发生过改变，仍然将`i`改成10。尽管最后的结果没有问题，但是整个过程还是不对的。

* **ABA问题的解决方法？**

  一般的解决方法是给共享数据加上一个版本号(version)，每次修该之前不仅要比较预期值和内存值是否一致，还要比较版本号是否一致，只有两者同时满足才可以修改，修改值之后将版本号同时更新。

  

  **Java中的解决方法是使用时间戳作为版本，比如AtomicStampedReference这个类中就使用了一个stamp字段来存储时间戳当做版本**



#### 无同步方案

要保证线程安全，并不是一定就要进行同步。如果一个方法本来就不涉及共享数据，那它自然就无须任何同步措施去保证正确性。

##### 栈封闭

多个线程访问同一个方法的局部变量时，不会出现线程安全问题，因为局部变量存储在虚拟机栈中，属于线程私有的，这里还有一个Java 内存分配的知识点：**栈上分配**。



##### 线程本地存储(Thread Local Storage)

如果一段代码中所需要的数据必须与其他代码共享，那就看看这些共享数据的代码是否能保证在同一个线程中执行。如果能保证，我们就可以把共享数据的可见范围限制在同一个线程之内，这样，无须同步也能保证线程之间不出现数据争用的问题。

符合这种特点的应用并不少见，大部分使用消费队列的架构模式(如“生产者-消费者”模式)都会将产品的消费过程尽量在一个线程中消费完。其中最重要的一个应用实例就是经典 Web 交互模型中的“一个请求对应一个服务器线程”(Thread-per-Request)的处理方式，这种处理方式的广泛应用使得很多 Web 服务端应用都可以使用线程本地存储来解决线程安全问题。

可以使用` java.lang.ThreadLocal` 类来实现线程本地存储功能。

为了理解 ThreadLocal，先看以下代码:

```java
public class ThreadLocalExample1 {
    public static void main(String[] args) {
        ThreadLocal threadLocal1 = new ThreadLocal();
        ThreadLocal threadLocal2 = new ThreadLocal();
        Thread thread1 = new Thread(() -> {
            threadLocal1.set(1);
            threadLocal2.set(1);
        });
        Thread thread2 = new Thread(() -> {
            threadLocal1.set(2);
            threadLocal2.set(2);
        });
        thread1.start();
        thread2.start();
    }
}
```

它所对应的底层结构图为:

![](https://pdai.tech/images/pics/3646544a-cb57-451d-9e03-d3c4f5e4434a.png)

每个 Thread 都有一个 `ThreadLocal.ThreadLocalMap` 对象，Thread 类中就定义了 ThreadLocal.ThreadLocalMap 成员。

```java
/* ThreadLocal values pertaining to this thread. This map is maintained
 * by the ThreadLocal class. */
ThreadLocal.ThreadLocalMap threadLocals = null;
```

当调用一个 ThreadLocal 的 `set(T value)` 方法时，先得到当前线程的 ThreadLocalMap 对象，然后将 `ThreadLocal->value `键值对插入到该 Map 中。

```java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
```

get() 方法类似。

```java
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}
```

**ThreadLocal 从理论上讲并不是用来解决多线程并发问题的，因为根本不存在多线程竞争**。

在一些场景 (尤其是使用线程池) 下，由于 ThreadLocal.ThreadLocalMap 的底层数据结构导致 ThreadLocal 有内存泄漏的情况，应该尽可能**在每次使用 ThreadLocal 后手动调用 remove()，以避免出现 ThreadLocal 经典的内存泄漏甚至是造成自身业务混乱的风险**。





### 什么是进程?什么是线程？

* **进程**

进程可以理解为一个应用程序执行的实例（比如在windows下打开Word就启动了一个进程），进程是资源分配的最小单位，每个进程都有自己独立的地址空间，每启动一个进程，系统就会为它分配地址空间，建立数据表来维护代码段、堆栈段和数据段。**进程主要有数据、程序和程序控制块（PCB）组成，其中PCB是系统感知进程存在的唯一标志。**

* **线程**

线程是进程中的一个执行单元，一个进程中可以启动多个线程，并且一个进程中的多个线程可以共享此进程中的所用资源。每个线程都有自己独立的运行时桟和程序计数器，**线程是CPU调度的最小单位**。



### 并发和并行的概念？

- **并发（concurrent）**

  ![img](https://pdai.tech/images/java/java-bingfa-2.png)

  单核CPU 下，操作系统通过任务调度器，将CPU的时间片分给不同的线程使用，只是由于CPU的切换速度非常快（Windows下一个最小的时间片是15ms），让用户看上去是同步执行的，实际上还是串性执行的。**这种线程轮流使用CPU的方法叫并发。**

- **并行（parallel）**

  ![img](https://pdai.tech/images/java/java-bingfa-1.png)

  **多个cpu或者多核处理器同时处理任务**，是真正意义上的同时执行。



### 线程有哪几种状态? 分别说明从一种状态到另一种状态转变有哪些方式?

Java中的线程有5种状态：**新建（New）、就绪（Runnable）、运行（Running）、阻塞（Blocked）、死亡（Dead）**

<img src="http://image.easyblog.top/1585712810812ab80dad3-d004-48ea-b160-3558796ca2b2.png" alt="img" style="zoom:50%;" />

- （1）**New**：当程序使用 **new 关键字创建了一个线程之后**，该线程就处于新建状态，此时仅由 JVM 为其分配内存，并初始化其成员变量的值
- （2）**Runnable**：线程被创建后，其他线程**调用了线程的start()方法**之后该线程处于这个状态。该状态的线程处于可执行线程池中，就绪状态的线程表示有权力去获取CPU时间片了，CPU时间片就是执行权。当线程拿到CPU时间片后就会马上执行run()方法，这时线程就进入了运行状态。
- （3）**Running**：线程获取到CPU时间片后线程执行任务的状态
- （4）**Blocked**：阻塞状态是由于当前执行中的线程因为某种原因放弃CPU使用权，暂时停止运行的状态。处于阻塞状态的线程必须再次切换到Runnable才能再次获取到CPU时间片。阻塞的类型可以分为三种：
  - **等待阻塞**：运行中的线程执行了wait()方法，JVM会把该线程放入等待队列（waiting queue）
  - **同步阻塞**：运行中的线程尝试获取一个对象的对象锁时，当发现这把锁正在被其他线程使用，那么JVM就会把该线程放入锁池(lock pool )
  - **其他阻塞**：运行中的线程调用了sleep()、join()方法或者发出了I/O请求时，JVM会把该线程置为阻塞状态。当sleep()状态超时、join()等待线程终止或者超时、或者I/O处理完毕时，线程重新转入就绪状态。
- （5）**Dead**：当执行中的线程执行完线程任务或执行过程中发生了Error或Exception线程都会死亡。



### Java创建线程的方式有哪些？

Java创建线程有三种方式：

* 继承`Thread`类
* 实现 `Runable` 接口
* 实现 `Callable` 接口



需要注意的是，实现 `Runnable` 和 `Callable` 接口的类只能当做一个可以在线程中运行的任务，不是真正意义上的线程，因此最后还需要通过 Thread 来调用。可以说任务是通过线程驱动从而执行的。



#### 继承Thread类

**第一种方法是将类声明为 Thread 的子类。该子类应重写 Thread 类的 `run` 方法，然后在run方法里实现相应的逻辑代码**

```java
public class ThreadTest {

    public static void main(String[] args) {
        Task task = new Task();
        task.setName("test-thread");
        task.start();
    }

    static class Task extends Thread{

        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName());
        }
    }

}
```



#### 实现 Runable 接口

**第二种方法是实现Runnable接口，并实现`run`方法，相比继承Thread类创建线程的好处是以实现接口的方式创建线程可以对类进行更好的扩展，该类可以继承其他类来扩展自身需求，相比第一种方式更加灵活，扩展性强。**

```java
public class RunnableTest{

    public static void main(String[] args){
        Thread t=new Thread(new Task(),"test-thread");
        t.start();
    }

    static class Task implements Runnable{
        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName());
        }
    }
}
```



实现 Callable 接口

**第三种方法，如果你想要在线程执行完毕之后得到带有返回值的线程则实现Callable接口。我们只需要实现Callable接口的`call()`方法，而且call方法不仅有返回值，还可以抛出异常。在jdk1.5中提供了Future接口来表示call()方法的返回值，并且提供了一个FutureTask实现类。**

```java
public class CallableTest {

    public static void main(String[] args) {
        /**
         * FutureTask实现了Runnable接口，因此FutureTask也就相当于是一个Runnable实现类
         */
        FutureTask<Integer> ft = new FutureTask<Integer>(new Task());
        Thread thread = new Thread(ft, "callable-test");
        thread.start();
        try {
            //FutureTask的get()方法会阻塞
            System.out.println(ft.get());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }

    static class Task implements Callable<Integer> {

        /**
         * 实现call()方法
         *
         * @return
         * @throws Exception
         */
        public Integer call() throws Exception {
            int ret = 0;
            for (int i = 0; i < 100; i++) {
                ret += i & (i - 1);
            }
            return ret;
        }
    }

}
```

##### 



### Java线程间通信的方式有哪些？

多个线程一起工作解决某个问题时，如果某些部分必须在其它部分之前完成，那么就需要对线程进行协调，即让线程之间通信。



#### 多线程同步

**这种方式，本质上就是“共享内存”式的通信。多个线程需要访问同一个共享变量，谁拿到了锁（获得了访问权限），谁就可以执行**。

```java
public class MyObject {

    synchronized public void methodA() {
        //do something....
    }

    synchronized public void methodB() {
        //do some other thing
    }
}

@Data
public class ThreadA extends Thread {

    private MyObject object;
  
    
    @Override
    public void run() {
        super.run();
        object.methodA();
    }
}

@Data
public class ThreadB extends Thread {

    private MyObject object;

    @Override
    public void run() {
        super.run();
        object.methodB();
    }
}

public class Test {
    public static void main(String[] args) {
        MyObject object = new MyObject();

        //线程A与线程B 持有的是同一个对象:object
        ThreadA a = new ThreadA(object);
        ThreadB b = new ThreadB(object);
        a.start();
        b.start();
    }
}
```

由于线程A和线程B持有同一个MyObject类的对象object，尽管这两个线程需要调用不同的方法，但是它们是同步执行的，如上示例中：线程B需要等待线程A执行完了methodA()方法之后，它才能执行methodB()方法。这样，线程A和线程B就实现了通信。





#### wait/notify机制

这是一种等待/通知机制，简单的说，等待/通知机制就是一个【线程A】等待，一个【线程B】通知（线程A可以不用再等待了）。比如生产者和消费者模型，消费者等待生产者生产资源，这是等待，生产者生产好资源通知等待的消费者去消费，这是通知。

**等待/通知机制的经典范式（模板）**

（1）等待方（消费者）需遵循如下原则：

- 获取共享对象锁
- 如果条件不满足，那么调用对象的wait()方法，被通知后仍然要检查条件
- 条件满足则执行对应逻辑

```java
synchronized(对象){
   while(条件不满足){
       对象.wait();
   }
   对应的逻辑;
}
```

（2）通知方（生产者）需遵循如下原则：

- 获得共享对象锁
- 改变条件
- 通知所有等待该对象锁的线程

```java
synchronized(对象){
    改变条件;
    对象.notifyAll();
}
```



等待/通知的相关方法是任意Java对象都具备的，因为这些方法被定义在java.lang.Object类中

```java
wait()     //代用该方法的线程会进入到WAITING状态，并且当前线程会被放置到等待队列中，只有等待另外线程的唤醒或被中断返回，调用wait()方法后线程会释放对象锁，同时该方法必须在同步方法或同步块中使用，否则会抛出IllegalMonitorStateException异常。
wait(long)     //超时等待一段时间，时间单位是ms,如果没有被唤醒就超时返回
wait(long,int)   //更精细的超时等待，精确到ns
notify()        //唤醒一个在等待对象锁的其他线程，如果有多个线程在等待该对象锁，那么会由线程规划器随机唤醒一个线程，此方法也必须在同步方法或同步块中使用，否则会抛出IllegalMonitorStateException异常。
notifyAll()     //唤醒所有等待此对象锁的线程

注意！notify()或notifyAll()在调用之后，等待线程不会立即从WAITING状态立即变为RUNNING状态，而是需要等到调动notify()或notifyAll()的方法释放对象锁之后才会从WAITING状态返回。
```





#### await/signal

`java.util.concurrent` 类库中提供了 `Condition` 类来实现线程之间的协调，可以在 Condition 上调用 `await() `方法使线程等待，其它线程调用 `signal()` 或 `signalAll()` 方法唤醒等待的线程。相比于 wait() 这种等待方式，await() 可以指定等待的条件，因此更加灵活。



### as-if-serial语义和happens-before语义？

#### **as-if-serial语义**

无论如何重排序，单线程内程序的执行结果不能被改变。编译器、runtime和处理器都必须遵循as-if-serial语义。为了遵守这条语义，编译器和处理器不会对存在数据依赖关系的操纵进行重排序，反之，就会被重排序。

#### **happens-before语义**

JSR-133使用happens-before的概念来阐述操作之间的内存可见性。在JMM中，如果一个操作执行的结果需要对另一个操作可见，

那么这2个操作之间必须要存在happens-before关系。这里提到的2个操作既可以是一个线程之内，也可以是不同线程之间。

与程序员密切相关的happens-before规则如下： 
1、程序顺序规则：一个线程中的每个操作，happens-before于线程中的任意后续操作。 
2、监视器锁规则：一个锁的解锁，happens-before于随后对这个锁的加锁。 
3、volatile变量规则：对一个volatile域的写，happens-before于任意后续对这个volatile域的读。 
4、传递性：如果A happens-before B，且Bhappens-before C，那么Ahappens-before C。 
需要注意的是： 
  两个操作之间具有happens-before关系，并不意味着前一个操作必须要在后一个操作之前执行！ 
happens-before仅仅要求前一个操作（执行的结果）对后一个操作可见，且前一个操作按顺序排在第 
二个操作之前。 
  一个happens-before规则对应一个或多个编译器和处理器重排序规则。对于Java程序员来说， 
happens-before规则简单易懂，它避免Java程序员为了理解JMM提供的内存可见性保证而去 

学习复杂的重排序以及这些规则的具体实现方法。



### JMM内存模型

JMM就是 `Java Memory Model` 的缩写，中文名即Java内存模型。因为在不同的硬件生产商和不同的操作系统下，内存的访问有一定的差异，所以会造成相同的代码运行在不同的系统上会出现各种问题。所以**java内存模型(JMM)屏蔽掉各种硬件和操作系统的内存访问差异，以实现让java程序在各种平台下都能达到一致的并发效果。**

Java内存模型将内存分为**`主内存`**和 **`工作内存`**两个部分，并且规定所有的变量都存储在主内存中，包括实例变量、静态变量，但是不包括局部变量和方法参数。每个线程都有自己的工作内存，**线程工作内存保存了该线程用到的变量和主内存的副本拷贝，线程对变量的操作都在工作内存中进行。线程不能直接读写内存中的变量**。

**不同的线程之间也无法访问对方工作内存中的变量。线程之间变量值的传递均需要通过主内存来完成**。

 **主内存和本地内存结构**

从抽象的角度来看，JMM定义了线程和主内存之间的抽象关系：线程之间的共享变量存储在主内存（main memory）中，每个线程都有一个私有的本地内存（local memory），本地内存中存储了该线程以读/写共享变量的副本。本地内存是JMM的一个抽象概念，并不真实存在。本地内存它涵盖了缓存，写缓冲区，寄存器以及其他的硬件和编译器优化之后的一个数据存放位置。

<img src="https://img-blog.csdn.net/20160507135725155" alt="è¿éåå¾çæè¿°" style="zoom:60%;" />

**JMM定义了什么？**

整个Java内存模型实际上是围绕着三个特征建立起来的。分别是：原子性，可见性，有序性。

**原子性**

原子性指的是一个操作是不可分割，不可中断的，一个线程在执行时不会被其他线程干扰。

**面试官拿笔写了段代码，下面这几句代码能保证原子性吗**？

```java
int i = 2;
int j = i;
i++;
i = i + 1;
```

第一句是基本类型赋值操作，必定是原子性操作。

第二句先读取i的值，再赋值到j，两步操作，不能保证原子性。

第三和第四句其实是等效的，先读取i的值，再+1，最后赋值到i，三步操作了，不能保证原子性。

JMM只能保证基本的原子性，如果要保证一个代码块的原子性，提供了monitorenter 和 moniterexit 两个字节码指令，也就是 synchronized 关键字。因此在 synchronized 块之间的操作都是原子性的。

**可见性**

可见性指当一个线程修改共享变量的值，其他线程能够立即知道被修改了。**Java是利用volatile关键字来提供可见性的**。 当变量被volatile修饰时，这个变量被修改后会立刻刷新到主内存，当其它线程需要读取该变量时，会去主内存中读取新值。而普通变量则不能保证这一点。

**除了volatile关键字之外，final和synchronized也能实现可见性**。

synchronized的原理是，在执行完，进入unlock之前，必须将共享变量同步到主内存中。

final修饰的字段，一旦初始化完成，如果没有对象逸出（指对象为初始化完成就可以被别的线程使用），那么对于其他线程都是可见的。

**有序性**

在Java中，**可以使用synchronized或者volatile保证多线程之间操作的有序性**。实现原理有些区别：

volatile关键字是使用内存屏障达到禁止指令重排序，以保证有序性。

synchronized的原理是，一个线程lock之后，必须unlock后，其他线程才可以重新lock，使得被synchronized包住的代码块在多线程之间是串行执行的。

**JMM的八种内存交互操作？**

<img src="http://image.easyblog.top/160109608021984ce7f47-7c02-405a-9f24-a737f6d0eb9a.png" alt="img" style="zoom:67%;" />

1. lock(锁定)，作用于**主内存**中的变量，把变量标识为线程独占的状态。
2. read(读取)，作用于**主内存**的变量，把变量的值从主内存传输到线程的工作内存中，以便下一步的load操作使用。
3. load(加载)，作用于**工作内存**的变量，把read操作主存的变量放入到工作内存的变量副本中。
4. use(使用)，作用于**工作内存**的变量，把工作内存中的变量传输到执行引擎，每当虚拟机遇到一个需要使用到变量的值的字节码指令时将会执行这个操作。
5. assign(赋值)，作用于**工作内存**的变量，它把一个从执行引擎中接受到的值赋值给工作内存的变量副本中，每当虚拟机遇到一个给变量赋值的字节码指令时将会执行这个操作。
6. store(存储)，作用于**工作内存**的变量，它把一个从工作内存中一个变量的值传送到主内存中，以便后续的write使用。
7. write(写入)：作用于**主内存**中的变量，它把store操作从工作内存中得到的变量的值放入主内存的变量中。
8. unlock(解锁)：作用于**主内存**的变量，它把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定。

再补充一下JMM对8种内存交互操作制定的规则：

- 不允许read、load、store、write操作之一单独出现，也就是read操作后必须load，store操作后必须write。
- 不允许线程丢弃他最近的assign操作，即工作内存中的变量数据改变了之后，必须告知主存。
- 不允许线程将没有assign的数据从工作内存同步到主内存。
- 一个新的变量必须在主内存中诞生，不允许工作内存直接使用一个未被初始化的变量。就是对变量实施use、store操作之前，必须经过load和assign操作。
- 一个变量同一时间只能有一个线程对其进行lock操作。多次lock之后，必须执行相同次数unlock才可以解锁。
- 如果对一个变量进行lock操作，会清空所有工作内存中此变量的值。在执行引擎使用这个变量前，必须重新load或assign操作初始化变量的值。
- 如果一个变量没有被lock，就不能对其进行unlock操作。也不能unlock一个被其他线程锁住的变量。
- 一个线程对一个变量进行unlock操作之前，必须先把此变量同步回主内存。



## 3.2 Java并发关键字

### synchronized详解

#### synchronized的三种使用方式？

synchronized是Java语言内置的锁，可以用于代码块和方法（包括静态方法和普通成员方法）

* **synchronized代码块（对象锁）**

  <img src="https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210122201241.png" style="zoom:67%;" />

* **synchronzied修饰实例方法（方法锁）**

  <img src="https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210122201225.png" style="zoom:67%;" />

  在这种情况下多线程并发访问实例方法时，如果其他线程调用同一个对象的被 `synchronized` 修饰的方法，就会被阻塞。相当于把锁记录在这个方法对应的对象上（相当于synchronized(this){...}）。

* **synchronzied修饰静态方法（类锁）**

  <img src="https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210122201233.png" style="zoom:67%;" />

  这种情况下进行多线程并发访问时，如果其他线程也是调用属于同一类的被 `synchronized` 修饰的静态方法，就会被阻塞。相当于把锁信息记录在这个方法对应的Class对象上。又由于Class的相关数据存储在永久带 PermGen（jdk1.8 则是 metaspace），永久代是全局共享的，因此静态方法锁相当于类的一个全局锁 。因此当一个线程访问类中的静态方法时，其他线程无法再访问这个类中的任何成员。



#### synchronized的实现原理？

`synchronized`关键字是Java并发编程中线程同步的常用手段之一，其作用有三个:

  1. **互斥性**：确保线程互斥的访问同步代，锁自动释放，多个线程操作同个代码块或函数必须排队获得锁，
 2. **可见性**：保证共享变量的修改能够及时可见，获得锁的线程操作完毕后会将所数据刷新到共享内存区
 3. **有序性**：不解决重排序，但保证有序性



##### （1）互斥性—加锁和释放锁的原理

* **同步代码块**

  <img src="http://image.easyblog.top/1597050921017c784e919-4a57-4974-9f1e-87740aad79b0.png" alt="img" style="zoom:40%;" />

其实我们使用`javap`命令反编译synchronized之后就可以发现**同步代码块在字节码层面通过`monitorenter`和`monitorexit`两条字节码指令实现的**。

- 当进入方法执行到**monitorenter**指令时，当前线程会尝试获取Monitor的所有权，成为这个Monitor的Owner，获取成功后继续执行临界区中的代码，执行结果会执行**monitorexit**指令，表示释放锁；如果获取锁失败那就会进入阻塞队列中等待；

- 如果当前线程已经是此Monitor的Owner了，再次执行到monitorenter那就直接进入，并将进入次数+1

- 同理，当执行完monitorexit，对应的进入数就-1，直到为0，才可以被其他线程持有

细心的同学可能会发现，**为什么在反编译后的字节码中出现了两次`monitorexit`指令**？

答：程序正常退出，得用一个 `monitorexit` 吧，如果中间出现异常，正常执行的`monitorexit`可能就无法执行到，锁会一直无法释放。所以编译器会为同步代码块添加了一个隐式的 `try-finally` 异常处理，在 `finally` 中会调用 `monitorexit` 命令最终释放锁。



* **同步方法**

<img src="http://image.easyblog.top/15970498789810ce65221-08a0-4b1a-950c-cbf81517dee4.png" alt="img" style="zoom:45%;" />

**方法的同步：在方法常量表中记录一个ACC_SYNCHRONIZED访问标记，调用指令会检查方法的常量表中是否设置了ACC_SYNCHORINZED标记 ,如果设置了这个标志，执行线程就需要先获取Monitor，获取成功之后才能执行方法，最后方法执行完毕释放Monitor。** 在方法执行期间，执行线程持有了该Monitor之后，其他线程就无法再获取了。如果在执行同步方法期间发生了异常，并且方法内部无法处理此异常，那么同步方法将会在抛出异常后自动释放Monitor。



##### （2）**可重入原理：加锁次数计数器**

`Monitorenter`和`Monitorexit`指令，会让对象在执行，使其锁计数器加1或者减1。每一个对象在同一时间只与一个monitor(锁)相关联，而一个monitor在同一时间只能被一个线程获得，一个对象在尝试获得与这个对象相关联的Monitor锁的所有权的时候，monitorenter指令会发生如下3中情况之一：

- monitor计数器为0，意味着目前还没有被获得，那这个线程就会立刻获得然后把锁计数器+1，一旦+1，别的线程再想获取，就需要等待
- 如果这个monitor已经拿到了这个锁的所有权，又重入了这把锁，那锁计数器就会累加，变成2，并且随着重入的次数，会一直累加
- 这把锁已经被别的线程获取了，等待锁释放

`monitorexit指令`：释放对于monitor的所有权，释放过程很简单，就是讲monitor的计数器减1，如果减完以后，计数器不是0，则代表刚才是重入进来的，当前线程还继续持有这把锁的所有权，如果计数器变成0，则代表当前线程不再拥有该monitor的所有权，即释放锁。



##### （3）可见性：内存模型和happens-before规则

synchronized锁，其本质就是锁的对象，所以了解原理之前先介绍下java对象。下图展示的是Java 对象在内存中的内存结构：

![](http://image.easyblog.top/167983082075625554f3f-e8c3-4d13-89cc-a091042a33c8.png)

可以看到，内存中的对象一般由三部分组成，分别是**对象头、对象实际数据和对齐填充**。 

* **对象头**包含 Mark Word、Class Pointer。如果是数组还有一个Length 字段。
  * **Mark Word 记录了对象的hashCode、锁信息，垃圾回收信息等。** 

  * **Class Pointer 用于指向对象对应的 Class 对象（其对应的元数据对象）的内存地址。**
  * **Length只适用于对象是数组时，它保存了该数组的长度信息。**

* 对象实际数据包括了对象的所有成员变量，其大小由各个成员变量的大小决定。

* 对齐填充表示最后一部分的填充字节位，JVM要求Java对象大小必须是8字节的整数倍，因此如果对象的大小满足条件将会填充，这部分不包含有用信息。 

`synchronized` 锁使用的就是对象头的 Mark Word 字段中的一部分。Mark Word 中的某些字段发生变化，就可以代表锁不同的状态。

在jdk1.6之前，synchronized锁实现的都是重量级锁，锁的标识位是10，其中指针指向的是monitor对象（监视器锁）的起始位置，每个对象都有一个与之关联的Monitor对象，当Monirot被某个线程持有后，它便处于锁定状态。





#### synchronized 锁升级过程

JVM中`monitorenter`和`monitorexit`字节码依赖于底层的操作系统的`Mutex Lock`来实现的，但是由于使用`Mutex Lock`需要将当前线程挂起并从用户态切换到内核态来执行，这种切换的代价是非常昂贵的；然而在现实中的大部分情况下，同步方法是运行在单线程环境(无锁竞争环境)如果每次都调用`Mutex Lock`那么将严重的影响程序的性能。**不过在jdk1.6中对锁的实现引入了大量的优化，如锁粗化(Lock Coarsening)、锁消除(Lock Elimination)、轻量级锁(Lightweight Locking)、偏向锁(Biased Locking)、适应性自旋(Adaptive Spinning)等技术来减少锁操作的开销**。

* **无锁到偏向锁**

  在大多数情况下，锁不仅不存在竞争，而且总是由同一个线程多次获得，因此为了减少同一线程获取锁的代价引入了偏向锁。

  首先检测是否为可偏向的状态，如果处于可偏向的状态，会测试当前Mark Word的线程ID是否指向自己，如果是，不需要再次获取锁，直接执行同步代码。

  如果线程ID不是自己的线程ID，就会通过CAS获取锁，获取成功代表当前偏向锁不存在竞争，获取失败，这说明当前偏向锁存在锁竞争，这时候就会先启动偏向锁撤销。

  **锁撤销流程**：当出现竞争，持有锁的线程会等待一个全局安全点阻塞，遍历线程栈，查看是否有被锁对象的锁记录（Lock Record）,如果有Lock Record，需要修复锁记录和Mark Word,使其变成无锁状态，将是否为偏向锁状态置为0，然后开始进行轻量级锁加锁流程。

  

* **偏向锁升级为轻量级锁**

  如果有不止一个线程在交替着使用一个锁对象，那么synchronized锁对象就会升级为轻量级锁。

  * **首先线程在自己的桟帧中产生创建一块锁记录空间，并把对象头中的Mark Word复制到锁记录中，被称为Displaced Mark Word。**

  * **然后线程尝试使用CAS操作将对象头中的Mark Word替换为指向锁记录的指针。如果成功，当前线程获得锁，随后将锁标志设为00；如果CAS操作失败，表示其他线程竞争锁，当前线程将进入到自旋获取锁的过程。**


自旋是一个耗费处理器资源的行为，因此当自旋一定次数后，锁就会升级为重量级锁，当前线程就会被阻塞，自旋默认的次数是10次，可以通过JVM参数`-X:preBlockSpin`来修改。

**在jdk1.6中引入了自适应的自旋锁。自适应的锁的自旋时间不是固定的，他会根据上一次在同一锁上的自旋时间以及拥有者的状态来决定。如果在同一个自旋锁对象上，自旋等待成功获取了锁并且正在运行，那么JVM就会认为这次自旋也很有可能再次成功，进而允许自旋等待持续更长时间。如果对于某个锁，自旋成功很少获取到锁，那么JVM就会省略自旋过程，以避免浪费处理器资源。**

* **轻量级锁升级成重量级锁**

  当多个线程请求获取轻量级锁的时候，轻量级锁就会膨胀为**重量级锁**，重量级锁就是指当一个线程获得锁之后，其余等待这个锁的线程都将进入到阻塞状态。重量级锁是操作系统层面的锁，此时线程的调度都将由操作系统负责，因此这就会引起频繁的上下文切换，导致线程被频繁的唤醒和挂起，使得程序性能下降。重量级锁的底层实现在JVM层面是通过对象内部的监视器（monitor）实现的。

  

### volatile详解

#### volatile的作用？

volatile是java虚拟机提供的轻量级同步机制，主要特点有三个：

1. **volatile可以保证不同线程对一个变量的可见性**，即一个线程修改了被voaltile修饰的变量后，这新值对其他线程来说是立即可见的;
2. **volatile可以禁止指令重排序**，保证了有序性；
3. volatile不能保证完全的原子性，只能保证单次的读/写操作具有原子性



##### 可见性

可见性问题主要指一个线程修改了共享变量值，而另一个线程却看不到。引起可见性问题的主要原因是每个线程拥有自己的一个高速缓存区——线程工作内存。如下图所示是Java内存模型示意图（JMM）：

<img src="http://image.easyblog.top/15971084794948dee08b1-8488-4fcc-88ff-1ba161da1a39.png" alt="img" style="zoom:50%;" />

JMM将JVM内存抽象为**主内存**和**工作内存**。主内存是所有的线程共享的内存区域；工作内存是线程私有的。JMM规定：

- **所有的变量都存储在主内存中(虚拟机内存的一部分)，对于所有线程都是共享的**。
- 每个线程都有自己的工作内存，**工作内存中保存的是该线程使用到的变量副本（该副本就是主内存中该变量的一份拷贝）**，线程对变量的所有操作都必须在工作内存中进行，而不能直接读写主内存中的变量。
- **线程之间无法直接访问对方的工作内存中的变量**，线程间变量的传递均需要通过主内存来完成



比如，存在两个线程A、B，同时从主线程中获取一个对象（i = 25），某一刻，A、B的工作线程中i都是25，A效率比较高，片刻，改完之后，马上将i更新到了主内存，但是此时B是完全没有办法i发生了变化，仍然用i做一些操作。**问题就发生了，B线程没有办法马上感知到变量的变化！！**



##### 禁止指令重排序

计算机在底层执行程序的时候，为了提高效率，经常会对指令做重排序，一般重排序分为三种

1. **编译器优化的重排序**。编译器在不改变单线程程序语义的前提下，可以重新安排语句的执行顺序；
2. **指令并行的重排**。如果不存在数据依赖性，处理器可以改变语句对应机器指令的执行顺序，处理器的重排序又被称为**乱序执行（out-of-order execution,OOE）技术**；
3. **内存系统的重排**。由于处理器使⽤缓存和读/写缓冲区，这使得加载和存储操作看上去可能是在乱序执⾏的。

单线程下，无论怎么样重排序，最后执行的结果都一致的，并且指令重排遵循基本的数据依赖原则，数据需要先声明再计算（`as-if-serial语义`）；多线程下，线程交替执行，由于编译器存在优化重排，两个线程中使用的变量能够保证一致性是无法确定的，结果无法预测。

volatile禁止指令重排序的原理是利用内存屏障来实现，通过插入内存屏障禁止在内存屏障前后的指令执行重排序的优化。



##### 原子性

volatile只能保证单次读/写的原子性，但是无法保证全局的原子性，因为基于上述的Java的内存模型的存在，修改一个变量`i`的值并不是一步操作，过程可以分为三步：

1. 从主内存中读取值，加载到工作内存
2. 工作内存中对i进行自增
3. 自增完成之后再写回主内存。

每个线程获取主内存中的值修改，然后再写回主内存，多个线程执行的时候，存在很多情况的写值的覆盖。



#### volatile的实现原理？

##### volatile可见性实现

###### 内存屏障（Memory Barrier）

volatile 变量的内存可见性是基于**内存屏障(`Memory Barrier`)**实现:

- 内存屏障，又称内存栅栏，是一个 CPU 指令：`lock cmpxchg `。
- 在程序运行时，为了提高执行性能，编译器和处理器会对指令进行重排序，JMM 为了保证在不同的编译器和 CPU 上有相同的结果，通过插入特定类型的内存屏障来禁止特定类型的编译器重排序和处理器重排序，插入一条内存屏障会告诉编译器和 CPU：不管什么指令都不能和这条 Memory Barrier 指令重排序。

lock 前缀的指令在多核处理器下会引发两件事情:

- 将当前处理器缓存行的数据写回到系统内存。
- 写回内存的操作会使在其他 CPU 里缓存了该内存地址的数据无效。

为了提高处理速度，处理器不直接和内存进行通信，而是先将系统内存的数据读到内部缓存(L1，L2 或其他)后再进行操作，但操作完不知道何时会写到内存。

如果对声明了 volatile 的变量进行写操作，JVM 就会向处理器发送一条 lock 前缀的指令，将这个变量所在缓存行的数据写回到系统内存。

为了保证各个处理器的缓存是一致的，实现了`缓存一致性协议(MESI)`，每个处理器通过嗅探在总线上传播的数据来检查自己缓存的值是不是过期了，当处理器发现自己缓存行对应的内存地址被修改，就会将当前处理器的缓存行设置成无效状态，当处理器对这个数据进行修改操作的时候，会重新从系统内存中把数据读到处理器缓存里。

所有多核处理器下还会完成：当处理器发现本地缓存失效后，就会从内存中重读该变量数据，即可以获取当前最新值。

volatile 变量通过这样的机制就使得每个线程都能获得该变量的最新值。



###### lock 指令

在 Pentium 和早期的 IA-32 处理器中，lock 前缀会使处理器执行当前指令时产生一个 `LOCK#` 信号，会对总线进行锁定，其它 CPU 对内存的读写请求都会被阻塞，直到锁释放。 后来的处理器，加锁操作是由高速缓存锁代替总线锁来处理。 因为锁总线的开销比较大，锁总线期间其他 CPU 没法访问内存。 这种场景多缓存的数据一致可以通过缓存一致性协议(MESI)来保证。



###### 缓存一致性协议（MESI）

缓存是分段(line)的，一个段对应一块存储空间，称之为`缓存行`，它是 CPU 缓存中可分配的最小存储单元，大小 32 字节、64 字节、128 字节不等，这与 CPU 架构有关，通常来说是 64 字节。 `LOCK#` 因为锁总线效率太低，因此使用了多组缓存。 为了使其行为看起来如同一组缓存那样。因而设计了 **缓存一致性协议**。 缓存一致性协议有多种，但是日常处理的大多数计算机设备都属于 `" 嗅探(snooping)" 协议`。 所有内存的传输都发生在一条共享的总线上，而所有的处理器都能看到这条总线。 缓存本身是独立的，但是内存是共享资源，所有的内存访问都要经过仲裁(同一个指令周期中，只有一个 CPU 缓存可以读写内存)。 CPU 缓存不仅仅在做内存传输的时候才与总线打交道，而是不停在嗅探总线上发生的数据交换，跟踪其他缓存在做什么。 当一个缓存代表它所属的处理器去读写内存时，其它处理器都会得到通知，它们以此来使自己的缓存保持同步。 只要某个处理器写内存，其它处理器马上知道这块内存在它们的缓存段中已经失效。



##### volatile禁止重排序

为了性能优化，JMM 在不改变正确语义的前提下，会允许编译器和处理器对指令序列进行重排序。JMM 提供了内存屏障阻止这种重排序。

Java 编译器会在生成指令系列时在适当的位置会插入内存屏障指令来禁止特定类型的处理器重排序。

JMM 会针对编译器制定 volatile 重排序规则表。

![img](https://pdai.tech/images/thread/java-thread-x-key-volatile-2.png)

" NO " 表示禁止重排序。

为了实现 volatile 内存语义时，编译器在生成字节码时，会在指令序列中插入内存屏障来禁止特定类型的处理器重排序。

对于编译器来说，发现一个最优布置来最小化插入屏障的总数几乎是不可能的，为此，JMM 采取了保守的策略。

<img src="http://image.easyblog.top/15971265900601c1b0648-b4c5-4ca8-8f5a-e415a9f3cf26.png" alt="img" style="zoom:50%;" /><img src="http://image.easyblog.top/1597126244876a33e5514-0a25-4eef-b9ca-e4a018f84e28.png" alt="img" style="zoom:50%;" />

- 在每个 volatile 写操作的前面插入一个 StoreStore 屏障。
- 在每个 volatile 写操作的后面插入一个 StoreLoad 屏障。
- 在每个 volatile 读操作的后面插入一个 LoadLoad 屏障。
- 在每个 volatile 读操作的后面插入一个 LoadStore 屏障。



## 3.3 J.U.C 全局

### JUC框架包含几个部分?

![image](https://pdai.tech/images/thread/java-thread-x-juc-overview-1-u.png)

JUC框架包主要包含5大部分：

1. Lock框架和Tools类，锁和并发工具(把图中这两个放到一起理解)
2. Collections 并发集合
3. Atomic 原子类
4. Executor 线程池



### Lock框架和Tools哪些核心的类?

![](https://pdai.tech/images/thread/java-thread-x-juc-overview-lock.png)

- **接口: Condition**， Condition为接口类型，它将 Object 监视器方法(wait、notify 和 notifyAll)分解成截然不同的对象，以便通过将这些对象与任意 Lock 实现组合使用，为每个对象提供多个等待 set (wait-set)。其中，Lock 替代了 synchronized 方法和语句的使用，Condition 替代了 Object 监视器方法的使用。可以通过await(),signal()来休眠/唤醒线程。
- **接口: Lock**，Lock为接口类型，Lock实现提供了比使用synchronized方法和语句可获得的更广泛的锁定操作。此实现允许更灵活的结构，可以具有差别很大的属性，可以支持多个相关的Condition对象。
- **接口ReadWriteLock** ReadWriteLock为接口类型， 维护了一对相关的锁，一个用于只读操作，另一个用于写入操作。只要没有 writer，读取锁可以由多个 reader 线程同时保持。写入锁是独占的。
- **抽象类: AbstractOwnableSynchonizer** AbstractOwnableSynchonizer为抽象类，可以由线程以独占方式拥有的同步器。此类为创建锁和相关同步器(伴随着所有权的概念)提供了基础。AbstractOwnableSynchronizer 类本身不管理或使用此信息。但是，子类和工具可以使用适当维护的值帮助控制和监视访问以及提供诊断。
- **抽象类(long): AbstractQueuedLongSynchronizer** AbstractQueuedLongSynchronizer为抽象类，以 long 形式维护同步状态的一个 AbstractQueuedSynchronizer 版本。此类具有的结构、属性和方法与 AbstractQueuedSynchronizer 完全相同，但所有与状态相关的参数和结果都定义为 long 而不是 int。当创建需要 64 位状态的多级别锁和屏障等同步器时，此类很有用。
- **核心抽象类(int): AbstractQueuedSynchronizer** AbstractQueuedSynchronizer为抽象类，其为实现依赖于先进先出 (FIFO) 等待队列的阻塞锁和相关同步器(信号量、事件，等等)提供一个框架。此类的设计目标是成为依靠单个原子 int 值来表示状态的大多数同步器的一个有用基础。
- **锁常用类: LockSupport** LockSupport为常用类，用来创建锁和其他同步类的基本线程阻塞原语。LockSupport的功能和"Thread中的 Thread.suspend()和Thread.resume()有点类似"，LockSupport中的park() 和 unpark() 的作用分别是阻塞线程和解除阻塞线程。但是park()和unpark()不会遇到“Thread.suspend 和 Thread.resume所可能引发的死锁”问题。
- **锁常用类: ReentrantLock** ReentrantLock为常用类，它是一个可重入的互斥锁 Lock，它具有与使用 synchronized 方法和语句所访问的隐式监视器锁相同的一些基本行为和语义，但功能更强大。
- **锁常用类: ReentrantReadWriteLock** ReentrantReadWriteLock是读写锁接口ReadWriteLock的实现类，它包括Lock子类ReadLock和WriteLock。ReadLock是共享锁，WriteLock是独占锁。
- **锁常用类: StampedLock** 它是java8在java.util.concurrent.locks新增的一个API。StampedLock控制锁有三种模式(写，读，乐观读)，一个StampedLock状态是由版本和模式两个部分组成，锁获取方法返回一个数字作为票据stamp，它用相应的锁状态表示并控制访问，数字0表示没有写锁被授权访问。在读锁上分为悲观锁和乐观锁。
- **工具常用类: CountDownLatch** CountDownLatch为常用类，它是一个同步辅助类，在完成一组正在其他线程中执行的操作之前，它允许一个或多个线程一直等待。
- **工具常用类: CyclicBarrier** CyclicBarrier为常用类，其是一个同步辅助类，它允许一组线程互相等待，直到到达某个公共屏障点 (common barrier point)。在涉及一组固定大小的线程的程序中，这些线程必须不时地互相等待，此时 CyclicBarrier 很有用。因为该 barrier 在释放等待线程后可以重用，所以称它为循环 的 barrier。
- **工具常用类: Phaser** Phaser是JDK 7新增的一个同步辅助类，它可以实现CyclicBarrier和CountDownLatch类似的功能，而且它支持对任务的动态调整，并支持分层结构来达到更高的吞吐量。
- **工具常用类: Semaphore** Semaphore为常用类，其是一个计数信号量，从概念上讲，信号量维护了一个许可集。如有必要，在许可可用前会阻塞每一个 acquire()，然后再获取该许可。每个 release() 添加一个许可，从而可能释放一个正在阻塞的获取者。但是，不使用实际的许可对象，Semaphore 只对可用许可的号码进行计数，并采取相应的行动。通常用于限制可以访问某些资源(物理或逻辑的)的线程数目。
- **工具常用类: Exchanger** Exchanger是用于线程协作的工具类, 主要用于两个线程之间的数据交换。它提供一个同步点，在这个同步点，两个线程可以交换彼此的数据。这两个线程通过exchange()方法交换数据，当一个线程先执行exchange()方法后，它会一直等待第二个线程也执行exchange()方法，当这两个线程到达同步点时，这两个线程就可以交换数据了。



### JUC并发集合哪些核心的类?

![image](https://pdai.tech/images/thread/java-thread-x-juc-overview-2.png)

- **ArrayBlockingQueue：** 一个由数组支持的有界阻塞队列。此队列按 FIFO(先进先出)原则对元素进行排序。队列的头部 是在队列中存在时间最长的元素。队列的尾部 是在队列中存在时间最短的元素。新元素插入到队列的尾部，队列获取操作则是从队列头部开始获得元素。
- **LinkedBlockingQueue：** 一个基于已链接节点的、范围任意的 blocking queue。此队列按 FIFO(先进先出)排序元素。队列的头部 是在队列中时间最长的元素。队列的尾部 是在队列中时间最短的元素。新元素插入到队列的尾部，并且队列获取操作会获得位于队列头部的元素。链接队列的吞吐量通常要高于基于数组的队列，但是在大多数并发应用程序中，其可预知的性能要低。
- **LinkedBlockingDeque：** 一个基于已链接节点的、任选范围的阻塞双端队列。
- **ConcurrentLinkedQueue：** 一个基于链接节点的无界线程安全队列。此队列按照 FIFO(先进先出)原则对元素进行排序。队列的头部 是队列中时间最长的元素。队列的尾部 是队列中时间最短的元素。新的元素插入到队列的尾部，队列获取操作从队列头部获得元素。当多个线程共享访问一个公共 collection 时，ConcurrentLinkedQueue 是一个恰当的选择。此队列不允许使用 null 元素。
- **ConcurrentLinkedDeque：** 是双向链表实现的无界队列，该队列同时支持FIFO和FILO两种操作方式。
- **DelayQueue：** 延时无界阻塞队列，使用Lock机制实现并发访问。队列里只允许放可以“延期”的元素，队列中的head是最先“到期”的元素。如果队里中没有元素到“到期”，那么就算队列中有元素也不能获取到。
- **PriorityBlockingQueue：** 无界优先级阻塞队列，使用Lock机制实现并发访问。priorityQueue的线程安全版，不允许存放null值，依赖于comparable的排序，不允许存放不可比较的对象类型。
- **SynchronousQueue：** 没有容量的同步队列，通过CAS实现并发访问，支持FIFO和FILO。
- **LinkedTransferQueue：** JDK 7新增，单向链表实现的无界阻塞队列，通过CAS实现并发访问，队列元素使用 FIFO(先进先出)方式。LinkedTransferQueue可以说是ConcurrentLinkedQueue、SynchronousQueue(公平模式)和LinkedBlockingQueue的超集, 它不仅仅综合了这几个类的功能，同时也提供了更高效的实现。
- **CopyOnWriteArrayList：** ArrayList 的一个线程安全的变体，其中所有可变操作(add、set 等等)都是通过对底层数组进行一次新的复制来实现的。这一般需要很大的开销，但是当遍历操作的数量大大超过可变操作的数量时，这种方法可能比其他替代方法更 有效。在不能或不想进行同步遍历，但又需要从并发线程中排除冲突时，它也很有用。
- **CopyOnWriteArraySet：** 对其所有操作使用内部CopyOnWriteArrayList的Set。即将所有操作转发至CopyOnWriteArayList来进行操作，能够保证线程安全。在add时，会调用addIfAbsent，由于每次add时都要进行数组遍历，因此性能会略低于CopyOnWriteArrayList。
- **ConcurrentSkipListSet：** 一个基于ConcurrentSkipListMap 的可缩放并发 NavigableSet 实现。set 的元素可以根据它们的自然顺序进行排序，也可以根据创建 set 时所提供的 Comparator 进行排序，具体取决于使用的构造方法。
- **ConcurrentHashMap：** 是线程安全HashMap的。ConcurrentHashMap在JDK 7之前是通过Lock和segment(分段锁)实现，JDK 8 之后改为CAS+synchronized来保证并发安全。
- **ConcurrentSkipListMap：** 线程安全的有序的哈希表(相当于线程安全的TreeMap)；映射可以根据键的自然顺序进行排序，也可以根据创建映射时所提供的 Comparator 进行排序，具体取决于使用的构造方法。



### JUC原子类哪些核心的类?

![](https://image.easyblog.top/%E6%88%AA%E5%B1%8F2023-03-26%20%E4%B8%8B%E5%8D%8810.11.06.png)

- 原子更新基本类型
  - AtomicBoolean: 原子更新布尔类型。
  - AtomicInteger: 原子更新整型。
  - AtomicLong: 原子更新长整型。
- 原子更新数组
  - AtomicIntegerArray: 原子更新整型数组里的元素。
  - AtomicLongArray: 原子更新长整型数组里的元素。
  - AtomicReferenceArray: 原子更新引用类型数组里的元素。
- 原子更新引用类型
  - AtomicIntegerFieldUpdater: 原子更新整型的字段的更新器。
  - AtomicLongFieldUpdater: 原子更新长整型字段的更新器。
  - AtomicStampedFieldUpdater: 原子更新带有版本号的引用类型。
  - AtomicReferenceFieldUpdater: 上面已经说过此处不在赘述
- 原子更新字段类
  - AtomicReference: 原子更新引用类型。
  - AtomicStampedReference: 原子更新引用类型, 内部使用Pair来存储元素值及其版本号。
  - AtomicMarkableReferce: 原子更新带有标记位的引用类型。



### JUC线程池哪些核心的类?

![img](https://pdai.tech/images/thread/java-thread-x-juc-executors-1.png)

- **接口: Executor** Executor接口提供一种将任务提交与每个任务将如何运行的机制(包括线程使用的细节、调度等)分离开来的方法。通常使用 Executor 而不是显式地创建线程。
- **ExecutorService** ExecutorService继承自Executor接口，ExecutorService提供了管理终止的方法，以及可为跟踪一个或多个异步任务执行状况而生成 Future 的方法。 可以关闭 ExecutorService，这将导致其停止接受新任务。关闭后，执行程序将最后终止，这时没有任务在执行，也没有任务在等待执行，并且无法提交新任务。
- **ScheduledExecutorService** ScheduledExecutorService继承自ExecutorService接口，可安排在给定的延迟后运行或定期执行的命令。
- **AbstractExecutorService** AbstractExecutorService继承自ExecutorService接口，其提供 ExecutorService 执行方法的默认实现。此类使用 newTaskFor 返回的 RunnableFuture 实现 submit、invokeAny 和 invokeAll 方法，默认情况下，RunnableFuture 是此包中提供的 FutureTask 类。
- **FutureTask** FutureTask 为 Future 提供了基础实现，如获取任务执行结果(get)和取消任务(cancel)等。如果任务尚未完成，获取任务执行结果时将会阻塞。一旦执行结束，任务就不能被重启或取消(除非使用runAndReset执行计算)。FutureTask 常用来封装 Callable 和 Runnable，也可以作为一个任务提交到线程池中执行。除了作为一个独立的类之外，此类也提供了一些功能性函数供我们创建自定义 task 类使用。FutureTask 的线程安全由CAS来保证。
- **核心: ThreadPoolExecutor** ThreadPoolExecutor实现了AbstractExecutorService接口，也是一个 ExecutorService，它使用可能的几个池线程之一执行每个提交的任务，通常使用 Executors 工厂方法配置。 线程池可以解决两个不同问题: 由于减少了每个任务调用的开销，它们通常可以在执行大量异步任务时提供增强的性能，并且还可以提供绑定和管理资源(包括执行任务集时使用的线程)的方法。每个 ThreadPoolExecutor 还维护着一些基本的统计数据，如完成的任务数。
- **核心: ScheduledThreadExecutor** ScheduledThreadPoolExecutor实现ScheduledExecutorService接口，可安排在给定的延迟后运行命令，或者定期执行命令。需要多个辅助线程时，或者要求 ThreadPoolExecutor 具有额外的灵活性或功能时，此类要优于 Timer。
- **核心: Fork/Join框架** ForkJoinPool 是JDK 7加入的一个线程池类。Fork/Join 技术是分治算法(Divide-and-Conquer)的并行实现，它是一项可以获得良好的并行性能的简单且高效的设计技术。目的是为了帮助我们更好地利用多处理器带来的好处，使用所有可用的运算能力来提升应用的性能。
- **工具类: Executors** Executors是一个工具类，用其可以创建ExecutorService、ScheduledExecutorService、ThreadFactory、Callable等对象。它的使用融入到了ThreadPoolExecutor, ScheduledThreadExecutor和ForkJoinPool中。



## 3.4 J.U.C锁详解

### 什么是AQS?

`AQS（AbstractQueuedSynchronizer）`是 Java 并发包中一个提供原子性状态管理的基础工具类。AQS 可以用来构建同步器，例如 `ReentrantLock`、`Semaphore`、`CountDownLatch` 等。它是一个抽象类，使用了`模板方法`设计模式，允许子类通过继承和重写其内部方法来实现自定义的同步器。

AQS 内部维护了一个等待队列和一个int类型的同步状态。等待队列用于存储等待获取同步状态的线程，并按照先进先出的顺序进行排队；同步状态则表示当前同步器的状态，可以被多个线程并发获取和释放。AQS 提供了 acquire() 和 release() 方法来获取和释放同步状态，通过实现这些方法可以定义不同的同步逻辑。

AQS 内部使用了 CAS（Compare-And-Swap）操作来实现同步状态的原子性修改，同时也提供了一些支持扩展的方法，例如 tryAcquire()、tryRelease()、tryAcquireShared()、tryReleaseShared() 等，可以用于实现不同种类的同步器。AQS 的实现方式非常灵活，可以通过继承它来实现各种同步器，例如独占锁、共享锁等。



### AQS的核心思想是什么? 它是怎么实现的? 底层数据结构等

#### AQS数据结构

![](http://image.easyblog.top/167992250739443fe0755-e3bb-4c4c-9b1a-f3730fbc1cfb.png)

AQS（AbstractQueuedSynchronizer）的核心设计原理是基于一个**先进先出的等待队列**和一个**共享的状态变量**，通过线程的竞争和等待来实现对共享状态的管理和同步

![img](https://p0.meituan.net/travelcube/7132e4cef44c26f62835b197b239147b18062.png)

AQS 的等待队列是一个双向链表，用于存储等待获取同步状态的线程，并按照先进先出的顺序进行排队。队列中的每个节点包含了一个线程引用和一个等待状态（例如是否被阻塞、是否被中断等）。线程获取同步状态时，如果发现当前状态已经被其他线程占用，则该线程会被加入到等待队列中，并被阻塞。当状态被释放时，AQS 会从等待队列中唤醒下一个等待线程。

每个Node节点中有以下几个重要的属性：

![](http://image.easyblog.top/16799220634494109f2a8-bf7b-4b7e-95c2-6c83124e92e9.png)

AQS中有两种锁模式：

* SHARED： 表示线程以共享的模式等待锁
* EXCLUSIVE：表示线程正在以独占的方式等待锁

waitStatus有下面几个枚举值：

* 0：当一个Node被初始化的时候的默认值
* CANCELLED（1）：表示线程获取锁的请求已经取消了
* CONDITION（-2）：表示节点在等待队列中，节点线程等待唤醒
* PROPAGATE（-3）：当前线程处在SHARED情况下，该字段才会使用
* SIGNAL（-1）：表示线程已经准备好了，就等资源释放了



### AQS有哪些核心的方法?

AQS提供了大量用于自定义同步器实现的Protected方法。自定义同步器实现的相关方法也只是为了通过修改State字段来实现多线程的独占模式或者共享模式。自定义同步器需要实现以下方法：

![](http://image.easyblog.top/16800049061287c39c48c-a122-4958-8f9b-8c1dc12e642c.png)

一般来说，自定义同步器要么是独占方式，要么是共享方式，它们也只需实现 `tryAcquire/tryRelease `或 `tryAcquireShared/tryReleaseShared` 中的一种即可。AQS也支持自定义同步器同时实现独占和共享两种方式，如ReentrantReadWriteLock。ReentrantLock是独占锁，所以实现了 `tryAcquire/tryRelease`。

以非公平锁为例，这里主要阐述一下非公平锁与AQS之间方法的关联之处，具体每一处核心方法的作用会在文章后面详细进行阐述。

![img](https://p1.meituan.net/travelcube/b8b53a70984668bc68653efe9531573e78636.png)

为了帮助大家理解ReentrantLock和AQS之间方法的交互过程，以非公平锁为例，我们将加锁和解锁的交互流程单独拎出来强调一下，以便于对后续内容的理解。

![img](https://p1.meituan.net/travelcube/7aadb272069d871bdee8bf3a218eed8136919.png)

加锁：

- 通过ReentrantLock的加锁方法Lock进行加锁操作。
- 会调用到内部类Sync的Lock方法，由于Sync#lock是抽象方法，根据ReentrantLock初始化选择的公平锁和非公平锁，执行相关内部类的Lock方法，本质上都会执行AQS的Acquire方法。
- AQS的Acquire方法会执行tryAcquire方法，但是由于tryAcquire需要自定义同步器实现，因此执行了ReentrantLock中的tryAcquire方法，由于ReentrantLock是通过公平锁和非公平锁内部类实现的tryAcquire方法，因此会根据锁类型不同，执行不同的tryAcquire。
- tryAcquire是获取锁逻辑，获取失败后，会执行框架AQS的后续逻辑，跟ReentrantLock自定义同步器无关。

解锁：

- 通过ReentrantLock的解锁方法Unlock进行解锁。
- Unlock会调用内部类Sync的Release方法，该方法继承于AQS。
- Release中会调用tryRelease方法，tryRelease需要自定义同步器实现，tryRelease只在ReentrantLock中的Sync实现，因此可以看出，释放锁的过程，并不区分是否为公平锁。
- 释放成功后，所有处理由AQS框架完成，与自定义同步器无关。

### ReentrantLock 加锁和解锁原理？

`ReentrantLock` 是AQS比较典型的应用，ReentrantLock有公平锁和非公平锁两种模式，底层机制都是类似的，这里以非公平锁为例进行分析加锁和解锁的原理，进而理解AQS的原理。

#### 加锁

##### lock

ReentrantLock 中获取锁的方法是 `lock` 方法，方法调用了内部类`Sync`的`lock` 方法，`Sync` 类是`AQS`的子类同时是一个抽象类，在AQS中`Sync`有两种实现`FairSync` 和 `NonFairSync`，即 公平锁 和 非公平锁。这里我们看一下非公平锁的加锁实现。

![](http://image.easyblog.top/16800104595274d0aaa87-f5ae-4b45-97c3-09cff7bc776f.png)

* 首先，当前线程会尝试使用 CAS 的方式修改AQS中state的值从0修改为1，如果修改成功，设置当前线程为线程持有者
* 如果修改state失败，则进入`acquire`方法



##### acquire

![](http://image.easyblog.top/16800110226652b659bf8-1854-4b96-8348-6648849da10b.png)

AQS 在这里使用了模板方法设计模式，`tryAcquire`方法是需要子类来实现的方法，方法返回boolean值，返回true表示获取锁成功，此时AQS会讲当前线程设置为锁持有者并且设置为可以中断的，如果没有获取锁成功，则通过 `addWaiter` 和 `acquireQueued` 将当前线程添加到等待队列末尾。



##### tryAcquire（nonfairTryAcquire）—获取锁

`tryAcquire` 方法由 AQS 子类实现，这里就是 `NonfairSync`的 `tryAcquire` 实现，具体逻辑在 父类Sync的`nonfairTryAcquire`方法中做了实现

![](http://image.easyblog.top/1680011405428e3ce9d9c-e707-4ccf-b771-73c6a7c6541e.png)

方法中当前线程根据当前state值不同做出不同动作：

* 当 **state=0**，再次尝试使用CAS的方式将state的值从0修改为1，如果设置成功就表示加锁成功，则将自己设置为锁持有者

* 当 **state!=0**，这里 ReentrantLock 实现了重入锁的逻辑， 当当前线程判断锁持有者是自己则直接将state的值+1，表示锁的加锁次数

* 最后，获取锁成功会返回true，否则返回false



##### addWaiter—线程加入等待队列

当执行`acquire(1)`时，会通过`tryAcquire(1)`获取锁。在这种情况下，如果获取锁失败，就会调用`addWaiter(Node.EXCLUSIVE)`以独占模式将当前线程加入到等待队列中去。

![](http://image.easyblog.top/16800127570002dddfc52-4058-42ae-8470-b495decdb709.png)

主要逻辑如下：

* 首先通过当前的线程和锁模式新建一个等待队列节点。
* 将变量pred指向等待队列尾指针tail
* 如果pred指向尾不是Null，则将新创建的Node节点的prev指向pred，随后使用 CAS 的方式将等待队列的tail指针指向新加入的Node节点，设置成功将pred的next节点设置为新加入的Node节点
* 如果Pred指针是Null（说明等待队列中没有元素），或者当前Pred指针和Tail指向的位置不同（说明被别的线程已经修改），就需要看一下`enq`的方法。



##### enq—自旋加入等到队列

![](http://image.easyblog.top/1680013504809ec585612-0ced-47ee-a812-c188118c71d8.png)

方法是一个死循环，会不断重复做下面两件事：

* 获取当前等待队列tail节点，如果tail节点是Null，表示等待队列还没有初始化，则需要进行初始化，此时会初始化等待队列的头和尾节点指向**同一个空节点**
* 如果tail节点不是Null，表示当前等待队列是有初始化过的，则会不断重试将enq入参的node节点（当前线程包装的等待队列节点）设置为当前等待队列的尾节点，直到设置成功。



##### acquireQueued—自旋获取锁

经过 `addWaiter` 方法，未成功获取锁的线程已经被添加到等待队列尾部，下面就需要**让线程在等待队列中自旋不断地获取锁，直到获取成功或者不再需要获取（中断）**。

![](http://image.easyblog.top/16800145946440797ca73-7cf3-426a-8411-1ccbb38bf7eb.png)

方法主体是死循环，主要逻辑：

* 获取当前节点的前驱节点
  * **如果当前节点是头结点 并且 节点获取锁成功，则将head指针移动到当前节点，并将当前节点设置为虚节点**
  * **如果当前节点是头结点 并且 节点获取锁未成功**，会判断当前节点是否需要阻塞：
    * 如果当前节点需要阻塞，则调用 `LockSupport.part(this)`方法阻塞当前方法
    * 如果当前节点不需要阻塞，则重新尝试调用 `tryAcquire`重新获取锁



上述方法的流程图如下：



![img](https://p0.meituan.net/travelcube/c124b76dcbefb9bdc778458064703d1135485.png)

从上图可以看出**，跳出当前循环的条件是当“前置节点是头结点，且当前线程获取锁成功”**。但为了防止因死循环导致CPU资源被浪费，我们会判断前置节点的状态来决定是否要将当前线程挂起，具体挂起流程用流程图表示如下（`shouldParkAfterFailedAcquire`流程）：

![img](https://p0.meituan.net/travelcube/9af16e2481ad85f38ca322a225ae737535740.png)



#### 解锁

前面我们已经剖析了加锁过程中的基本流程，接下来再对解锁的基本流程进行分析。由于ReentrantLock在解锁的时候，并不区分公平锁和非公平锁，所以我们直接看解锁的源码：

##### unlock/release—释放锁

`ReentrantLock` 锁的`unlock`方法直接调用 `Sync`类的`release`方法，所以直接看 `release`方法

![](http://image.easyblog.top/168001708651070b47562-26f3-4f90-8d94-fdbed3eb2e36.png)

* 自定义的tryRelease如果返回true，说明该锁没有被任何线程持有（锁被释放）
* 如果锁被释放，判断当前头结点不为空并且头结点的waitStatus不是初始化节点情况，则调用 `unparkSuccessor`方法唤醒后继节点



**这里的判断条件为什么是 `h != null && h.waitStatus != 0` ？**

> 1. h == null Head还没初始化。初始情况下，head == null，第一个节点入队，Head会被初始化一个虚拟节点。所以说，这里如果还没来得及入队，就会出现head == null 的情况。
> 2. h != null && waitStatus == 0 表明后继节点对应的线程仍在运行中，不需要唤醒。
> 3. h != null && waitStatus < 0 表明后继节点可能被阻塞了，需要唤醒。



##### tryRelease—释放锁

在ReentrantLock里面的公平锁和非公平锁的父类Sync定义了可重入锁的释放锁机制 `tryRelease`。

![](https://image.easyblog.top/%E6%88%AA%E5%B1%8F2023-03-29%20%E4%B8%8B%E5%8D%888.30.34.png)

方法返回boolean值，表示当前线程是否释放锁成功，true 表示已释放，false表示没有释放，具体逻辑如下：

* 获取当前锁state值，并将锁重入次数减一
* 如果当前线程不是持有锁的线程，则报错
* 如果当前线程是持有锁的线程，并且state值是0了，则将当前独占锁所有线程设置为null，并更新state



##### unparkSuccessor—唤醒后继结点

![](https://image.easyblog.top/%E6%88%AA%E5%B1%8F2023-03-29%20%E4%B8%8B%E5%8D%889.31.05.png)

* 获取头结点`waitStatus`，如果头结点等待状态 `waitStatus < 0`  ，说明头结点 
* 获取当前节点的下一个节点
  * 如果当前节点的下个节点不为空，而且等待状态`waitStatus<=0`，就把当前节点的下一个节点唤醒
  * 如果下个节点是null或者下个节点被cancelled，就找到队列最开始的非cancelled的节点
    * 寻找的方式是从尾部开始往前遍历等待队列，直到找到队列第一个waitStatus<0的节点，
    * 如果找到这样的节点，则将此节点唤醒



**为什么要从后往前找第一个非Cancelled的节点呢？**

> 这里可以回到 `addWaiter` AQS将线程加入等到队列的操作说起，**节点入队并不是原子操作**，也就是说，node.prev = pred; compareAndSetTail(pred, node) 这两个地方可以看作Tail入队的原子操作，但是此时pred.next = node;还没执行，如果这个时候执行了unparkSuccessor方法，就没办法从前往后找了，所以需要从后往前找。还有一点原因，在产生CANCELLED状态节点的时候，先断开的是Next指针，Prev指针并未断开，因此也是必须要从后往前遍历才能够遍历完全部的Node。
>
> 
>
> **综上所述，如果是从前往后找，由于极端情况下入队的非原子操作和CANCELLED节点产生过程中断开Next指针的操作，可能会导致无法遍历所有的节点**。







### ReentrantLock/Lock 锁 和 synchronized 锁的区别？

* （1）**synchronized是Java的关键字是jvm层面上的互斥锁，ReentrantLock/Lock 通过类实现的一个互斥锁**
* （2）**使用synchronized时，当线程执行完同步代码之后或者出现异常之后jvm会自动让线程释放锁；但是ReentrantLock/Lock 在使用的时候必须手动显示的在finally块中释放锁,如果没这样做，容易产生死锁**
* （3）**ReentrantLock/Lock 可以让等待锁的线程响应中断，而synchronized却不行，使用synchronized时，等待的线程会一直等待下去，不能够响应中断；**
* （4）**通过 ReentrantLock/Lock 可以知道有没有成功获取锁，而synchronized却无法办到**
* （5）**ReentrantLock/Lock 可以提高多个线程进行读操作的效率。从性能上来说，如果竞争资源不激烈，两者的性能是差不多的，而当竞争资源非常激烈时（即有大量线程同时竞争），此时Lock的性能要远远优于synchronized**。



### CountDownLatch原理简介？

CountDownLatch是一个同步工具类，用来协调多个线程之间的同步。这个工具通常用来控制线程等待，它可以让某一个线程等待直到倒计时结束，再开始执行。

**CountDownLatch 的两种应用场景**：

1. 某一线程在开始运行前等待n个线程执行完毕。将 CountDownLatch 的计数器初始化为n ：`new CountDownLatch(n)`，每当一个任务线程执行完毕，就将计数器减1 `countdownlatch.countDown()`，当计数器的值变为0时，在 `CountDownLatch上 await()` 的线程就会被唤醒。一个典型应用场景就是启动一个服务时，主线程需要等待多个组件加载完毕，之后再继续执行。

2. 实现多个线程开始执行任务的最大并行性。注意是并行性，不是并发，强调的是多个线程在某一时刻同时开始执行。类似于赛跑，将多个线程放到起点，等待发令枪响，然后同时开跑。做法是初始化一个共享的 `CountDownLatch` 对象，将其计数器初始化为 1 ：`new CountDownLatch(1)`，多个线程在开始执行任务前首先 `coundownlatch.await()`，当主线程调用 countDown() 时，计数器变为0，多个线程同时被唤醒。

**CountDownLatch 的不足**

CountDownLatch是一次性的，计数器的值只能在构造方法中初始化一次，之后没有任何机制再次对其设置值，当CountDownLatch使用完毕后，它不能再次被使用。



### CycleBarrier原理简介？

CyclicBarrier 和 CountDownLatch 非常类似，它也可以实现线程间的等待，但是它的功能比 CountDownLatch 更加复杂和强大。主要应用场景和 CountDownLatch 类似

**CyclicBarrier 的应用场景**

CyclicBarrier 可以用于多线程计算数据，最后合并计算结果的应用场景。比如我们用一个Excel保存了用户所有银行流水，每个Sheet保存一个帐户近一年的每笔银行流水，现在需要统计用户的日均银行流水，先用多线程处理每个sheet里的银行流水，都执行完之后，得到每个sheet的日均银行流水，最后，再用barrierAction用这些线程的计算结果，计算出整个Excel的日均银行流水。





### Semaphore原理简介？

Semaphore（信号量）， synchronized 和 ReentrantLock 都是一次只允许一个线程访问某个资源，**Semaphore(信号量)可以指定多个线程同时访问某个资源**。Semaphore 有两种模式，公平模式和非公平模式。

- **公平模式：** 调用acquire的顺序就是获取许可证的顺序，遵循FIFO；
- **非公平模式：** 抢占式的。



## 3.5 J.U.C线程池详解

### 为什么要有线程池?

线程池能够对线程进行统一分配，调优和监控:

- **降低资源消耗**。通过重复利用已创建的线程降低线程创建和销毁造成的消耗 
- **提高响应速度**。当任务到达时，任务可以不需要等到线程创建就能立即执行。 
- **提高线程的可管理性**。线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。



### Java实现和管理线程池有哪些方式? 

1. 使用`java.util.concurrent.Executors`工厂类创建线程池。这个工厂类提供了几种预定义的线程池实现，如`newFixedThreadPool`，`newCachedThreadPool`和`newSingleThreadExecutor`等，可以根据应用场景选择不同的线程池。
2. 自定义ThreadPoolExecutor线程池，这是一种比较灵活的方式，可以通过`ThreadPoolExecutor`的构造函数来指定线程池的核心线程数、最大线程数、队列容量等参数。
3. 可以使用`ScheduledThreadPoolExecutor`来实现定时任务调度。ScheduledThreadPoolExecutor是ThreadPoolExecutor的一个子类，可以用来定时执行任务或周期性地执行任务。
4. Java 8引入了`CompletableFuture`类，可以使用它的异步特性来实现线程池的管理。
5. Java 9引入了新的工厂类，如Flow类和SubmissionPublisher类，可以实现异步任务的处理和结果处理。



### ThreadPoolExecutor有哪些核心的配置参数? 请简要说明

**线程池的构造方法**

```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler) {
    //省略.....
}
```

- 1、**corePoolSize**：线程池中核心线程个数

- 2、**maximunPoolSize**：线程池最大允许的线程个数（这里有一个救急线程的概念，就是非核心线程，它的数量是maximunPoolSize-corePoolSize）

- 3、**keepAliveTime**：这个参数是给非核心线程设定的，他表示非核心线程可以空闲时间，超过这个时间就会被回收

- 4、**unit**：空闲时间的单位

- 5、**workQueue**：任务队列，它是一个阻塞队列，用于存放等待执行的任务。

- 6、**threadFactory**：线程工厂，用于创建线程以及可以给线程起一个有意义的名字

- 7、**RejectedExecutionHandler**：当任务队列和线程池都处于满负荷运行时，新提交的任务应该如何处理的策略，称为**饱和策略**。Java中提供的策略有以下4种：

  > （1）ThreadPoolExecutor.AbortPolicy：直接抛出异常，这是默认的策略
  >
  > （2）ThreadPoolExecutor.CallerRunsPolicy：让调用者来执行该任务
  >
  > （3）ThreadPoolExecutor.DisCardPolicy：不做任何处理，直接把任务丢掉，也不抛出任何异常
  >
  > （4）ThreadPoolExecutor.DisCardOldestPolicy：丢弃任务队列头部的任务，然后尝试执行当前任务

  当然，我们也可以根据我们的业务场景通过实现`RejectExeptionPolicy`接口实现自定义策略。



### ThreadPoolExecutor线程池任务提交的流程

![img](http://image.easyblog.top/158636140301387db9ed6-7a20-42db-94d6-5377024a8cc7.jpg)

* （1）首先会判断运行中的线程数是否小于corePoolSize，如果小于，则直接创建新的线程执行任务。
* （2）如果运行中的线程大于corePoolSize，判断workQueue任务队列是否已经满了，如果还没有满，则将任务放到任务队列中排队等待被处理；
* （3）如果workQueue任务队列满了，则判断当前线程数是否大于线程池的最大线程数，如果没有大于最大线程数则创建新的线程处理任务；
* （4）如果运行中的线程数大于最大线程数，则通过拒绝策略处理新提交的任务。在ThreadPoolExecutor中提供了4种拒绝策略：AbortPolicy：直接抛出异常，这是默认的策略；CallerRunsPolicy：让调用者来执行该任务；DisCardPolicy：不做任何处理，直接把任务丢掉，也不抛出任何异常；DisCardOldestPolicy：丢弃任务队列头部的任务，然后尝试执行当前任务



### 线程池中任务是如何关闭的?

关闭线程池可以使用线程池的`shutdown` 或 `shutdownNow`方法

- shutdown

将线程池里的线程状态设置成SHUTDOWN状态, 然后中断所有没有正在执行任务的线程.

- shutdownNow

将线程池里的线程状态设置成STOP状态, 然后停止所有正在执行或暂停任务的线程. 只要调用这两个关闭方法中的任意一个, isShutDown() 返回true. 当所有任务都成功关闭了, isTerminated()返回true.



### 如何合理配置线程池参数？

要想合理的配置线程池，就必须首先分析任务特性，可以从以下几个角度来进行分析：

1. 任务的性质：CPU密集型任务，IO密集型任务和混合型任务。
2. 任务的优先级：高，中和低。
3. 任务的执行时间：长，中和短。
4. 任务的依赖性：是否依赖其他系统资源，如数据库连接。

任务性质不同的任务可以用不同规模的线程池分开处理。CPU密集型任务配置尽可能少的线程数量，如配置**Ncpu+1**个线程的线程池。IO密集型任务则由于需要等待IO操作，线程并不是一直在执行任务，则配置尽可能多的线程，如**2xNcpu**。混合型的任务，如果可以拆分，则将其拆分成一个CPU密集型任务和一个IO密集型任务，只要这两个任务执行的时间相差不是太大，那么分解后执行的吞吐率要高于串行执行的吞吐率，如果这两个任务执行时间相差太大，则没必要进行分解。我们可以通过`Runtime.getRuntime().availableProcessors()`方法获得当前设备的CPU个数。

优先级不同的任务可以使用优先级队列PriorityBlockingQueue来处理。它可以让优先级高的任务先得到执行，需要注意的是如果一直有优先级高的任务提交到队列里，那么优先级低的任务可能永远不能执行。

执行时间不同的任务可以交给不同规模的线程池来处理，或者也可以使用优先级队列，让执行时间短的任务先执行。

依赖数据库连接池的任务，因为线程提交SQL后需要等待数据库返回结果，如果等待的时间越长CPU空闲时间就越长，那么线程数应该设置越大，这样才能更好的利用CPU。

并且，阻塞队列**最好是使用有界队列**，如果采用无界队列的话，一旦任务积压在阻塞队列中的话就会占用过多的内存资源，甚至会使得系统崩溃。



### ScheduledThreadPoolExecutor有什么样的数据结构，核心内部类和抽象类?

![img](https://pdai.tech/images/thread/java-thread-x-stpe-1.png)

ScheduledThreadPoolExecutor继承自 `ThreadPoolExecutor`。ScheduledThreadPoolExecutor 内部构造了两个内部类 `ScheduledFutureTask` 和 `DelayedWorkQueue`:

- `ScheduledFutureTask`: 继承了FutureTask，是一个异步运算任务；最上层分别实现了Runnable、Future、Delayed接口，说明它是一个可以延迟执行的异步运算任务。
- `DelayedWorkQueue`: 这是 ScheduledThreadPoolExecutor 为存储周期或延迟任务专门定义的一个延迟队列，继承了 AbstractQueue，为了契合 ThreadPoolExecutor 也实现了 BlockingQueue 接口。它内部只允许存储 RunnableScheduledFuture 类型的任务。与 DelayQueue 的不同之处就是它只允许存放 RunnableScheduledFuture 对象，并且自己实现了二叉堆(DelayQueue 是利用了 PriorityQueue 的二叉堆结构)。





### ScheduledThreadPoolExecutor相比ThreadPoolExecutor有哪些特性?

ScheduledThreadPoolExecutor继承自 ThreadPoolExecutor，为任务提供延迟或周期执行，属于线程池的一种。和 ThreadPoolExecutor 相比，它还具有以下几种特性:

- 使用专门的任务类型—ScheduledFutureTask 来执行周期任务，也可以接收不需要时间调度的任务(这些任务通过 ExecutorService 来执行)。
- 使用专门的存储队列—DelayedWorkQueue 来存储任务，DelayedWorkQueue 是无界延迟队列DelayQueue 的一种。相比ThreadPoolExecutor也简化了执行机制(delayedExecute方法，后面单独分析)。
- 支持可选的run-after-shutdown参数，在池被关闭(shutdown)之后支持可选的逻辑来决定是否继续运行周期或延迟任务。并且当任务(重新)提交操作与 shutdown 操作重叠时，复查逻辑也不相同。





### Fork/Join主要用来解决什么样的问题?

Fork/Join是一种**并行编程模型**，它主要**用于解决计算密集型问题**，即需要进行大量计算的任务。

该模型基于“分而治之”的策略，通过将一个大任务划分为许多小任务，然后并行地执行这些小任务，最终将它们的结果合并成一个最终结果。这个过程通常被称为“fork-join”。

这种并行计算模型非常适合于多核处理器或分布式系统中的并行计算。一些典型的应用包括图像处理、数据分析、并行搜索和排序算法等。



### Fork/Join框架主要包含哪三个模块? 模块之间的关系是怎么样的?

Fork/Join框架主要包含三个模块:

- `ForkJoinTask`：我们要使用ForkJoin框架，必须首先创建一个ForkJoin任务。它提供在任务中执行fork()和join()操作的机制，通常情况下我们不需要直接继承ForkJoinTask类，而只需要继承它的子类，Fork/Join框架提供了以下两个子类：
  - `RecursiveAction`：是一种ForkJoinTask的子类，用于表示不需要返回结果的任务，即只需要执行一些操作，而不需要返回结果。
  - `RecursiveTask`：是一种ForkJoinTask的子类，用于表示需要返回结果的任务，即需要执行一些操作，并返回一个结果。
- `ForkJoinPool`：是Fork/Join框架的核心，它管理着一组线程池，并提供了线程池的创建、启动、关闭和维护等操作。ForkJoinTask需要通过ForkJoinPool来执行，任务分割出的子任务会添加到当前工作线程所维护的双端队列中，进入队列的头部。当一个工作线程的队列里暂时没有任务时，它会随机从其他工作线程的队列的尾部获取一个任务。
- `ForkJoinWorkerThread`：是Fork/Join框架中的工作线程，它负责执行具体的任务，并将结果返回给主线程。

这三者的关系是: ForkJoinPool可以通过池中的ForkJoinWorkerThread来处理ForkJoinTask任务。



### ForkJoinPool类继承关系?

![img](https://pdai.tech/images/thread/java-thread-x-forkjoin-1.png)

内部类介绍:

- `ForkJoinWorkerThreadFactory`: 内部线程工厂接口，用于创建工作线程ForkJoinWorkerThread
- `DefaultForkJoinWorkerThreadFactory`: ForkJoinWorkerThreadFactory 的默认实现类
- `InnocuousForkJoinWorkerThreadFactory`: 实现了 ForkJoinWorkerThreadFactory，无许可线程工厂，当系统变量中有系统安全管理相关属性时，默认使用这个工厂创建工作线程。
- `EmptyTask`: 内部占位类，用于替换队列中 join 的任务。
- `ManagedBlocker`: 为 ForkJoinPool 中的任务提供扩展管理并行数的接口，一般用在可能会阻塞的任务(如在 Phaser 中用于等待 phase 到下一个generation)。
- `WorkQueue`: ForkJoinPool 的核心数据结构，本质上是work-stealing 模式的双端任务队列，内部存放 ForkJoinTask 对象任务，使用 @Contented 注解修饰防止伪共享。
  - 工作线程在运行中产生新的任务(通常是因为调用了 fork())时，此时可以把 WorkQueue 的数据结构视为一个栈，新的任务会放入栈顶(top 位)；工作线程在处理自己工作队列的任务时，按照 LIFO 的顺序。
  - 工作线程在处理自己的工作队列同时，会尝试窃取一个任务(可能是来自于刚刚提交到 pool 的任务，或是来自于其他工作线程的队列任务)，此时可以把 WorkQueue 的数据结构视为一个 FIFO 的队列，窃取的任务位于其他线程的工作队列的队首(base位)。
- 伪共享状态: 缓存系统中是以缓存行(cache line)为单位存储的。缓存行是2的整数幂个连续字节，一般为32-256个字节。最常见的缓存行大小是64个字节。当多线程修改互相独立的变量时，如果这些变量共享同一个缓存行，就会无意中影响彼此的性能，这就是伪共享。



### ForkJoinTask抽象类继承关系?

![img](https://pdai.tech/images/thread/java-thread-x-forkjoin-4.png)

ForkJoinTask 实现了 Future 接口，它也是一个可取消的异步运算任务，实际上ForkJoinTask 是 Future 的轻量级实现，主要用在纯粹是计算的函数式任务或者操作完全独立的对象计算任务。fork 是主运行方法，用于异步执行；而 join 方法在任务结果计算完毕之后才会运行，用来合并或返回计算结果。 其内部类都比较简单，ExceptionNode 是用于存储任务执行期间的异常信息的单向链表；其余四个类是为 Runnable/Callable 任务提供的适配器类，用于把 Runnable/Callable 转化为 ForkJoinTask 类型的任务(因为 ForkJoinPool 只可以运行 ForkJoinTask 类型的任务)。



### Fork/Join 框架的执行流程/运行机制是怎么样的?

![img](https://pdai.tech/images/thread/java-thread-x-forkjoin-5.png)



### Fork/Join的分治思想和work-stealing 实现方式?

#### 分治算法(Divide-and-Conquer)

分治算法(Divide-and-Conquer)把任务递归的拆分为各个子任务，这样可以更好的利用系统资源，尽可能的使用所有可用的计算能力来提升应用性能。首先看一下 Fork/Join 框架的任务运行机制:

![img](https://pdai.tech/images/thread/java-thread-x-forkjoin-2.png)

#### work-stealing(工作窃取)算法

工作窃取算法（work-stealing）是一种多线程任务调度算法，在工作窃取算法中，每个线程都维护一个任务队列（也称为工作队列），线程会从自己的任务队列中取出任务执行。当一个线程执行完自己的任务队列中的所有任务时，它会从其他线程的任务队列中“窃取”一些任务来执行，以保证所有线程的任务负载均衡。

工作窃取算法的主要思想是：当一个线程的任务执行完毕时，它会从其他线程的任务队列中随机选择一个任务“窃取”，并将这个任务添加到自己的任务队列中。这样可以避免某个线程的任务队列中出现过多的任务，而其他线程的任务队列却很空闲的情况。

工作窃取算法的优点在于：

1. 它可以充分利用多核处理器的计算资源，提高程序的并发性能。
2. 它可以避免线程之间的竞争，提高线程的执行效率。
3. 它可以动态地调整任务的分配策略，从而避免任务的负载不平衡。



**Tips：**

> 在 ForkJoinPool 中，线程池中每个工作线程(ForkJoinWorkerThread)都对应一个任务队列(WorkQueue)，工作线程优先处理来自自身队列的任务(LIFO或FIFO顺序，参数 mode 决定)，然后以FIFO的顺序随机窃取其他队列中的任务。
>
> 具体思路如下:
>
> * 每个线程都有自己的一个WorkQueue，该工作队列是一个双端队列。
> * 队列支持三个功能push、pop、poll
> * push/pop只能被队列的所有者线程调用，而poll可以被其他线程调用。
> * 划分的子任务调用fork时，都会被push到自己的队列中。
> * 默认情况下，工作线程从自己的双端队列获出任务并执行。
> * 当自己的队列为空时，线程随机从另一个线程的队列末尾调用poll方法窃取任务。

## 3.6 J.U.C并发集合详解

### ConcurrentHashMap在JDK1.7和JDK1.8中实现有什么差别? 

- `HashTable` : 使用了synchronized关键字对put等操作进行加锁;
- `ConcurrentHashMap JDK1.7`: 使用分段锁机制实现;
- `ConcurrentHashMap JDK1.8`: 则使用数组+链表+红黑树数据结构和CAS原子操作实现;



### ConcurrentHashMap JDK1.7实现的原理是什么?

在JDK1.5~1.7版本，Java使用了分段锁机制实现ConcurrentHashMap.

简而言之，ConcurrentHashMap在对象中保存了一个Segment数组，即将整个Hash表划分为多个分段；而每个Segment元素，它通过继承 ReentrantLock 来进行加锁，所以每次需要加锁的操作锁住的是一个 segment，这样只要保证每个 Segment 是线程安全的，也就实现了全局的线程安全；这样，在执行put操作时首先根据hash算法定位到元素属于哪个Segment，然后对该Segment加锁即可。因此，ConcurrentHashMap在多线程并发编程中可是实现多线程put操作。



#### 数据结构

jdk1.7 的ConcurrentHashMap的底层数据结构是**数组+链表**，ConcurrentHashMap是主要由**Segment**数组结构和 **HashEntry**数组结构组成。如下图所示。

![img](https://pdai.tech/images/thread/java-thread-x-concurrent-hashmap-1.png)

整个ConcurrentHashMap 由一个个 **Segment** 组成，Segment 代表“部分” 或 “一段”的意思，所以很多地方都会将其描述为分段锁。

简单的理解，ConcurrentHashMap 是一个 Segment数组，Segment 通过继承 ReentrantLock来进行加锁，所以每次需要加锁的操作锁住的是一个 Segment，这样只要保证每个 Segment是线程安全的，也就实现了全局的线程安全性。每个Segment中维护了一个HashEntry数组，它用于存储键值对。

- **Segment继承了ReentrantLock，是一个可重入的锁，它在ConcurretnHashMap中扮演锁的角色**；
- **HashEntry则用于存储键值对**。它们之间的关系是：一个ConcurrentHashMap包含了一个Segment数组，一个Segment里维护了一个HashEntry数组，HashEntry数组和HashMap结构类似，是一种**数组+链表**的结构，当一个线程需要对HashEntry中的元素修改的时候，必须先获得Segment锁。



#### **Segment内部类**

首先先来熟悉一下Segment，Segment是ConcurrentHashMap的内部类，它在ConcurrentHashMap就是扮演锁的角色，主要组成如下：

<img src="http://image.easyblog.top/1596449481856c291c686-a3ec-4bf4-bb47-464da2d52590.png" alt="img" style="zoom:67%;" />

重要的即使那个HashEntry，HashEntry是一个和HashMap类似的数据结构，这里值的注意的是他被volatile修饰了。这里复习一下volatile的特性：

（1）可以保证共享变量在多线程之间的内存可见性，即一个线程修改了共享变量的值对于其他线程是这个修改后的是可见的；

（2）可以禁止指令重排序；

（3）volatile可以保证对单次读写操作的原子性；

这里使用volatile修饰HashEntry数组的目的当然是为了保证内存的可见性问题。为什么需要保证HashEntry数组的内存可见性呢？

在往下看源码就会发现，jdk1.7以前的CHM的get操作是不需要加锁的，即**可以并发的读**，只有在写数据的时候才需要加锁，因此为了保证不同线程之前对于一个共享变量的数据一致，使用volatile再好不过。



#### ConcurrentHashMap 初始化

* `initialCapacity`：初始容量，这个值指的是整个 ConcurrentHashMap的初始容量，实际操作的时候需要平均分给每个 Segment。

* `loadFactor`：负载因子，之前我们说了，Segment 数组不可以扩容，所以这个负载因子是给每个 Segment 内部使用的。

* `concurrencyLevel`：并行级别、并发数、Segment 数。默认是16，也就是说 ConcurrentHashMap 默认有16个 Segment，所以理论上，最多同时支持16个线程并发写。这个值可以在初始化的时候设置为其他值，但是一旦初始化以后，它就不可以扩容了。最大值受MAX_SEGMENTS控制，为65535个。

`new ConcurrentHashMap()` 无参进行初始化以后：

（1）Segment数组的长度是16，并且segments[0]初始化了，其他segments[i]还是null，在使用之前需要调用ensureSegment()方法执行初始化

（2）segments[i]的默认大小是2，也即HashEntry的默认大小是2，负载因子为0.75，得出初始阈值为1.5，也就插入第一个元素不会触发扩容，插入第二个进行一次扩容。

**定位Segment**

ConcurrentHashMap使用了分段锁Segment来维护不同段的数据，那么在插入和获取元素的时候，必须先通过算法首先定位到Segment上之后才可以在具体的HashEntry用类似HashMap找元素的方法来定位一个元素。



#### put()—添加元素操作

<img src="http://image.easyblog.top/1596464636809e8a0c99a-19f7-4b02-a40e-df25afb7672f.png" alt="img" style="zoom:67%;" />

**put流程梳理：** 

（1）首先检查value，如果value==null，抛出NPE；

（2）根据hash值定位到Segment，定位segments[j]的算法是：**j=(hash>>>sgmentShift)&sgmentMask**，其实j就是hash值的低4位

（3）获得到Segment后判断是否为null,如果是null，表示还没有初始化，那先调用ensureSegment()方法初始化Segment

（4）最后，调用Segment对象的put方法存入键值对。

* 4.1、首先通过父类ReentrantLock的tryLock()方法对每个Segment加锁，上锁成功之后继续执行，否则调用scanAndLockForPut()方法不断的调用tryLock()尝试获取锁，尝试的次数和CPU核数有关，单核CPU值尝试1次，多核CPU最多重试64次，如果重试结束还没有获取锁，那就阻塞等待锁； 
* 4.2、确定HashEntry[i]的下标。`index = (tab.length - 1) & hash`; 
* 4.3、如果当前HashEntry没有初始化，先初始化，并将键值对插入；
* 4.4、已经初始化，遍历HashEntry链，有重复的，新值覆盖旧值，否则，插入； 
* 4.5、**插入之后，判断是否需要扩容**，扩容：Segment[]是不能扩容的，只能扩容一个个Segment中的HashEntry[]的大小为原来的两倍



#### get()—获取元素操作

<img src="http://image.easyblog.top/15965148764775c348927-bf62-4713-8a0f-d8685f5d8f24.png" alt="img" style="zoom:50%;" />

**get操作流程**： 

（1）计算 hash值，

（2）通过hash值找到 Segment数组中的下标。

（3）再次根据 hash 找到每Segment内部的 HashEntry[]数组的下标。

（4）遍历该数组位置处的链表，直到找到相等（内存地址相同或两个元素的hash值相同并且equals方法返回true）的 key。

get 是不需要加锁的：原因是get方法是将共享变量（table）定义为volatile，让被修饰的变量对多个线程可见(即其中一个线程对变量进行了修改，会同步到主内存，其他线程也能获取到最新值)，允许一写多读的作。

Segment的get操作实现非常简单和高效。**先经过一次hash()，然后使用散列值运算定位到Segment，在定位到聚义的元素的过程。**

get操作的高效是因为整个get操作不需要加锁，为什么他不需要加锁呢？是**因为get方法中使用的共享变量都被定义成了volatile类型**，比如：统计当前Segment大小的count，和用于存储key-value的HashEntry。定义成volatile的变量能够在多个线程之间保证内存可见性，如果与多个线程读，它读取到的值一定是当前这个变量的最新值。



#### size()—获取元素个数

size()方法就是求当前ConcurrentHashMap中元素实际个数的方法，它是**弱一致性的**。方法的逻辑大致是：**首先他会使用不加锁的模式去尝试计算 ConcurrentHashMap 的 size，比较前后两次计算的结果，结果一致就认为当前没有元素加入或删除，计算的结果是准确的，然后返回此结果；如果尝试了三次之后结果还是不一致，它就会给每个 Segment 加上锁，然后计算 ConcurrentHashMap 的 size 返回。**

<img src="https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210121205801.png" style="zoom:80%;" />





### ConcurrentHashMap JDK1.8实现的原理是什么?

JDK1.8的实现已经摒弃了Segment的概念，而是直接使用 **数组+链表+红黑树** 的数据结构来实现的，**并发控制使用的是CAS+synchronized**，jdk1.8版本的ConcurrentHashMap看起来就像是优化过之后线程安全的HashMap,虽然在JDK1.8中还能看到Segment的身影，但是已经简化了属性，只是为了兼容旧版本。下图是jdk1.8中ConcurrentHashMap的结构示意图：

<img src="http://image.easyblog.top/15964512103041ffc1840-4e2e-49d0-b0f8-2d94c2726d63.jpg" alt="img"/>

#### put()—添加元素操作

**put()方法流程梳理：**

（1）首先我们通过源码看到，jdk1.8中ConcurrentHashMap不允许key或value为null，如果你违反了就会报NPE。

（2）之后获取到hash值，在之后会首先判断当前hash表是否被初始化了，因为jdk1.8中ConcurrentHashMap是懒惰初始化的，只有第一次put的时候才会初始化。初始化会调用initTable方法，该方法中使用CAS设置seizeCtl的值来保证线程安全。

（3）如果哈希表已经初始化了，然后计算key的哈希桶的位置，如果在这个位置还是null，那就直接使用CAS设置新节点到桶的这个位置即可；定位桶的算法还是**i=(tab.length-1)&hash**

（4）如果检查发现当前槽位的**头结点的hash值==-1**（标记为是ForwardingNode结点了），表示当前hash表正在扩容中，此时其他线程过去帮忙扩容;

（5）如果当前hash表存在、(n-1)&hash 位子有元素了并且没有在扩容，那就是发生了哈希冲突，解决冲突之前首先对链表的头结点/红黑树的root结点上锁

- 5.1、如果key的桶的位置还是链表，那就在链表的**尾部插入新的元素**，再次过程中还会统计链表的结点个数，存储在bitCount中，用于在插入之后判断是否需要扩容以及检查是否相同key的结点存在，如果存在那就执行值替换逻辑；
- 5.2、如果key的桶的位置已经是红黑树了，那就把新的元素插入红黑树中

（6）插入完毕之后会判断binCount的值是否已经大于等于树化阈值（TREEIFY_THRESHOLD=8）了，如果是，那就把链表转换为红黑树，不过最后转不转得成还要看哈希表的容量是不是大于MIN_TREEIFY_CAPACITY（64），如果没有达到只会扩容



#### initTable()—初始化哈希表

在put过程中如果是第一次put，此时hash表还没有初始化，因此需要首先初始化哈希表。初始化哈希表会调用initTable()方法，这个方法会使用CAS加锁，然后实例化一个hash表

![](http://image.easyblog.top/168025111787210ca06a8-bf1e-4e9b-b2a5-90fd37970e04.png)



#### get()—获取元素操作

get()方法没有加锁，即可以并发的读，敢这么做重要是因为Node数组被volatile被修饰了，volatile保证了共享变量在多线程下的内存可见性，即其他线程只要一修改Node数组中的数据，get操作的线程马上就可以知道修改的数据。

<img src="http://image.easyblog.top/15965397939966be09e6d-d8da-495f-b0ea-bc64c50fc780.png" alt="img" />

get()方法逻辑梳理：

（1）计算出key的hash值；

（2）根据 hash 值找到哈希槽的位置：**(n - 1) & hash**

（3）根据该位置处头节点的性质进行查找：

* 1）如果该位置为 null，那么直接返回 null 就可以了
* 2）如果该位置处的节点刚好就是我们需要的，返回该节点的值即可
* 3）如果该位置节点的 hash 值小于 0，说明正在扩容，或者是红黑树，后面调用 find 接口，程序会根据上下文执行具体的方法。
* 4）如果以上 3 条都不满足，那就是链表，进行遍历比对即可



#### size()—获取元素的个数

<img src="https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210121203722.png"/>

 

#### transfer()—扩容操作

扩容的逻辑比较复杂，这里不再贴出源码了，我们来捋一下关键流程就好了：

- 首先扩容会传进来两个参数：旧哈希表和新哈希表（`Node<K,V>[] tab, Node<K,V>[] nextTab`），在第一个线程调用这个方法的时候新哈希表nextTab=null
- **当线程判断nextTab=null之后，会new一个是旧hash表2倍大小的新哈希表**（`Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1];`）
- **创建新哈希表完成之后就是元素结点的搬迁工作了**。这个过程中值的注意的是，**当一个线程把旧哈希表上桶中的元素搬迁到新的哈希表上之后或者这个桶中没有元素，它用一个ForwardingNode类型的结点挂在这个桶下，以此告诉其他线程这个桶位置已经被处理了，这样可以避免其他线程get或put时出现错误**
- **还需要注意的是，在移动桶中元素的过程中，还是需要对链表和红黑树分别处理**，前者的判断依据是头结点的hash值大于等于0，后者的判断依据是头结点的hash值小于0并且头结点是TreeBin类的实例。



### CopyOnWriteArrayList的实现原理?

`CopyOnWriteArrayList` 是 Java 并发包中的一个线程安全的 List 实现，它的特点是写操作是通过复制一份新的数组来实现的，因此可以保证写操作的线程安全性，而读操作则可以并发执行。

具体实现原理如下：

1. 内部使用一个 `volatile` 修饰的数组来存储元素。在初始化时，`CopyOnWriteArrayList` 会创建一个空数组作为初始数组。当添加元素时，会将新元素添加到数组的末尾。
2. 在进行写操作时，`CopyOnWriteArrayList` 首先会获取一个 `ReentrantLock` 锁，确保同一时刻只有一个线程进行写操作。
3. 接着，会创建一个新的数组，并将原数组的元素复制到新数组中。因为新数组与原数组是不同的对象，所以写操作不会影响原数组。
4. 写操作完成之后，`CopyOnWriteArrayList `会将新数组替换掉旧数组。由于 `volatile` 关键字的作用，所有线程都能立即看到数组的变化。
5. 在进行读操作时，`CopyOnWriteArrayList` 直接读取当前数组的元素。因为当前数组不会发生变化，所以读操作是线程安全的。由于读操作与写操作是分离的，所以 `CopyOnWriteArrayList `可以支持高并发的读操作。

总之，`CopyOnWriteArrayList` 是一种适用于读多写少的场景的线程安全 List 实现，通过写时复制的方式保证写操作的线程安全性，同时支持高并发的读操作。



### CopyOnWriteArrayList有何缺陷，说说其应用场景?

CopyOnWriteArrayList 有几个缺点：

- 由于写操作的时候，需要拷贝数组，会消耗内存，如果原数组的内容比较多的情况下，可能导致young gc或者full gc
- 不能用于实时读的场景，像拷贝数组、新增元素都需要时间，所以调用一个set操作后，读取到数据可能还是旧的,虽然CopyOnWriteArrayList 能做到最终一致性,但是还是没法满足实时性要求；



### 阻塞队列有几种，分别都有哪些特性？

阻塞队列（BlockingQueue）是一个支持两个附加操作的队列。这两个附加的操作是：在队列为空时，获取元素的线程会等待队列变为非空。当队列满时，存储元素的线程会等待队列可用。阻塞队列常用于生产者和消费者的场景，生产者是往队列里添加元素的线程，消费者是从队列里拿元素的线程。阻塞队列就是生产者存放元素的容器，而消费者也只从容器里拿元素。

<img src="http://image.easyblog.top/1587444510867628c9aac-b66a-488b-8165-365de883ebf5.jpg" alt="img" style="zoom:30%;" />

#### (1)、ArrayListBlockingQueue

ArrayListBlockingQueue是一个用数组实现的有界阻塞队列，此队列按照先进先出（FIFO）的原则对元素进行排序。支持公平锁和非公平锁（默认情况下是非公平锁）。

```java
public ArrayBlockingQueue(int capacity) {
    //默认使用的是非公平锁
    this(capacity, false);
}
//通过boolean类型的参数可以控制使用公平锁还是非公平锁
public ArrayBlockingQueue(int capacity, boolean fair) {
    if (capacity <= 0)
        throw new IllegalArgumentException();
    this.items = new Object[capacity];
    lock = new ReentrantLock(fair);
    notEmpty = lock.newCondition();
    notFull =  lock.newCondition();
}
```

#### (2)、LinkedBlockingQueue

LinkedBlockingQueue是一个用**单链表**实现的有界阻塞队列。此队列的默认长度和最大长度为`Integer.MAX_VALUE`。此队列按照先进先出的原则对元素进行排序。

#### (3)、PriorityBlockingQueue

一个支持线程优先级排序的无界队列，默认自然序进行排序，也可以自定义实现compareTo()方法来指定元素排序规则，不能保证同优先级元素的顺序。

#### (4)、DelayQueue

DelayQueue是一个支持延时获取元素的无界阻塞队列。队列基于PriorityBlockingQueue实现。队列中的元素必须实现Delayed接口，在创建元素是可以指定多久才能从队列中获取当前元素。DelayQueue可以用于 **缓存系统设计** 和 **定时任务调度** 这样的应用场景。

#### (5)、SynchronousQueue

SynchronousQueue是一个不存储元素的阻塞队列。每一个put操作必须等待一个take操作，否则不能继续添加元素。它支持公平访问队列。默认情况下线程采用非公共策略访问队列。**当使用公平锁的时候，等待的线程会采用先进先出的顺序访问队列**。

```java
//默认使用非公平锁
public SynchronousQueue() {
    this(false);
}
//通过boolean类型的参数可以控制使用公平锁还是非公平锁
public SynchronousQueue(boolean fair) {
    transferer = fair ? new TransferQueue<E>() : new TransferStack<E>();
}
```

#### ( 6)、LinkedTransferQueue

LinkedTransferQueue是一个由链表实现的无界阻塞Transfer队列。相对于其他阻塞队列，LinkedTransferQueue多了transfer和tryTransfer方法

#### (7)、LinkedBlockingDeque

LinkedBlockingDeque是一个由**双链表**实现的双向阻塞队列。队列头部和尾部都可以添加和移除元素，多线程并发时，可以将锁的竞争最多降到一半。



## 3.7 J.U.C原子类详解

### 什么是CAS?

CAS的全称为`Compare-And-Swap`，直译就是对比交换。是一条CPU的原子指令，其作用是让CPU先进行比较两个值是否相等，然后原子地更新某个位置的值，经过调查发现，其实现方式是基于硬件平台的汇编指令，就是说CAS是靠硬件实现的，JVM只是封装了汇编调用，那些AtomicInteger类便是使用了这些封装后的接口。   简单解释：CAS操作需要输入两个数值，一个旧值(期望操作前的值)和一个新值，在操作期间先比较下在旧值有没有发生变化，如果没有发生变化，才交换成新值，发生了变化则不交换。

CAS操作是原子性的，所以多线程并发使用CAS更新数据时，可以不使用锁。JDK中大量使用了CAS来更新数据而防止加锁(synchronized 重量级锁)来保持原子更新。

相信sql大家都熟悉，类似sql中的条件更新一样：update set id=3 from table where id=2。因为单条sql执行具有原子性，如果有多个线程同时执行此sql语句，只有一条能更新成功。



### CAS会有哪些问题?

CAS 方式为乐观锁，synchronized 为悲观锁。因此使用 CAS 解决并发问题通常情况下性能更优。

但使用 CAS 方式也会有几个问题：

- ABA问题

因为CAS需要在操作值的时候，检查值有没有发生变化，比如没有发生变化则更新，但是如果一个值原来是A，变成了B，又变成了A，那么使用CAS进行检查时则会发现它的值没有发生变化，但是实际上却变化了。

ABA问题的解决思路就是使用版本号。在变量前面追加上版本号，每次变量更新的时候把版本号加1，那么A->B->A就会变成1A->2B->3A。

从Java 1.5开始，JDK的Atomic包里提供了一个类AtomicStampedReference来解决ABA问题。这个类的compareAndSet方法的作用是首先检查当前引用是否等于预期引用，并且检查当前标志是否等于预期标志，如果全部相等，则以原子方式将该引用和该标志的值设置为给定的更新值。

- 循环时间长开销大

自旋CAS如果长时间不成功，会给CPU带来非常大的执行开销。如果JVM能支持处理器提供的pause指令，那么效率会有一定的提升。pause指令有两个作用：第一，它可以延迟流水线执行命令(de-pipeline)，使CPU不会消耗过多的执行资源，延迟的时间取决于具体实现的版本，在一些处理器上延迟时间是零；第二，它可以避免在退出循环的时候因内存顺序冲突(Memory Order Violation)而引起CPU流水线被清空(CPU Pipeline Flush)，从而提高CPU的执行效率。

- 只能保证一个共享变量的原子操作

当对一个共享变量执行操作时，我们可以使用循环CAS的方式来保证原子操作，但是对多个共享变量操作时，循环CAS就无法保证操作的原子性，这个时候就可以用锁。



### 阐述你对Unsafe类的理解?

UnSafe类总体功能：

![img](https://pdai.tech/images/thread/java-thread-x-atomicinteger-unsafe.png)

Unsafe类可以用来直接访问和操作Java虚拟机中的内存，这包括直接修改对象的内部状态、分配内存、释放内存以及执行CAS（Compare-and-Swap）操作等。

Unsafe类还可以用来执行本地方法调用，这允许Java代码与本地C或C++代码进行交互。





# 四、Java IO

## 4.1 IO 基础

### 如何从数据传输方式理解IO流？

从数据传输方式或者说是运输方式角度看，可以将 IO 类分为 `字节流` 和 `字符流`:

1. **`字节流`**, 字节流读取单个字节，字符流读取单个字符(一个字符根据编码的不同，对应的字节也不同，如 UTF-8 编码中文汉字是 3 个字节，GBK编码中文汉字是 2 个字节。)
2. **`字符流`**, 字节流用来处理二进制文件(图片、MP3、视频文件)，字符流用来处理文本文件(可以看做是特殊的二进制文件，使用了某种编码，人可以阅读)。

针对不同字节流和字符流，Java提供了对应的IO类体系：

* 字节流

![img](https://pdai.tech/images/io/java-io-category-1.png)

* 字符流

![img](https://pdai.tech/images/io/java-io-category-2.png)

### Java IO设计上使用了什么设计模式？

**装饰者模式**： 所谓装饰，就是把这个装饰者套在被装饰者之上，从而动态扩展被装饰者的功能。

- 以 InputStream 为例
  - InputStream 是抽象组件；
  - FileInputStream 是 InputStream 的子类，属于具体组件，提供了字节流的输入操作；
  - FilterInputStream 属于抽象装饰者，装饰者用于装饰组件，为组件提供额外的功能。例如 BufferedInputStream 为 FileInputStream 提供缓存的功能。

![image](https://pdai.tech/images/pics/DP-Decorator-java.io.png)

实例化一个具有缓存功能的字节流对象时，只需要在 FileInputStream 对象上再套一层 BufferedInputStream 对象即可。

```java
FileInputStream fileInputStream = new FileInputStream(filePath);
BufferedInputStream bufferedInputStream = new BufferedInputStream(fileInputStream);
```

DataInputStream 装饰者提供了对更多数据类型进行输入的操作，比如 int、double 等基本类型。



## 4.2 5种IO模型

### 什么是阻塞？什么是同步？

- **阻塞IO 和 非阻塞IO**

这两个概念是**程序级别**的。主要描述的是程序请求操作系统IO操作后，如果IO资源没有准备好，那么程序该如何处理的问题: 前者等待；后者继续执行(并且使用线程一直轮询，直到有IO资源准备好了)

- **同步IO 和 非同步IO**

这两个概念是**操作系统级别**的。主要描述的是操作系统在收到程序请求IO操作后，如果IO资源没有准备好，该如何响应程序的问题: 前者不响应，直到IO资源准备好以后；后者返回一个标记(好让程序和自己知道以后的数据往哪里通知)，当IO资源准备好以后，再用事件机制返回给程序。



### 什么是Linux的IO模型？

网络IO的本质是socket的读取，socket在linux系统被抽象为流，IO可以理解为对流的操作。刚才说了，对于一次IO访问（以read举例），**数据会先被拷贝到操作系统内核的缓冲区中，然后才会从操作系统内核的缓冲区拷贝到应用程序的地址空间**。所以说，当一个read操作发生时，它会经历两个阶段：

- 第一阶段：等待数据准备 (Waiting for the data to be ready)。
- 第二阶段：将数据从内核拷贝到进程中 (Copying the data from the kernel to the process)。

对于socket流而言，

- 第一步：通常涉及等待网络上的数据分组到达，然后被复制到内核的某个缓冲区。
- 第二步：把数据从内核缓冲区复制到应用进程缓冲区。

网络应用需要处理的无非就是两大类问题，网络IO，数据计算。相对于后者，网络IO的延迟，给应用带来的性能瓶颈大于后者。网络IO的模型大致有如下几种：

1. 同步阻塞IO（bloking IO）
2. 同步非阻塞IO（non-blocking IO）
3. 多路复用IO（multiplexing IO）
4. 信号驱动式IO（signal-driven IO）
5. 异步IO（asynchronous IO）

![img](https://pdai.tech/images/io/java-io-compare.png)

#### 什么是同步阻塞IO？

应用进程被阻塞，直到数据复制到应用进程缓冲区中才返回。

![img](https://pdai.tech/images/io/java-io-model-0.png)



#### 什么是同步非阻塞IO？

应用进程执行系统调用之后，内核返回一个错误码。应用进程可以继续执行，但是需要不断的执行系统调用来获知 I/O 是否完成，这种方式称为轮询(polling)。

![img](https://pdai.tech/images/io/java-io-model-1.png)

#### 什么是多路复用IO？

系统调用可能是由多个任务组成的，所以可以拆成多个任务，这就是多路复用。

使用 `select` 或者 `poll` 等待数据，并且可以等待多个套接字中的任何一个变为可读，这一过程会被阻塞，当某一个套接字可读时返回。之后再使用 `recvfrom` 把数据从内核复制到进程中。

它可以让单个进程具有处理多个 I/O 事件的能力。又被称为 Event Driven I/O，即事件驱动 I/O。

![img](https://pdai.tech/images/io/java-io-model-2.png)



##### 有哪些多路复用IO实现？

目前流程的多路复用IO实现主要包括四种: `select`、`poll`、`epoll`、`kqueue`。下表是他们的一些重要特性的比较:

| IO模型 | 相对性能 | 关键思路         | 操作系统      | JAVA支持情况                                                 |
| ------ | -------- | ---------------- | ------------- | ------------------------------------------------------------ |
| select | 较高     | Reactor          | windows/Linux | 支持,Reactor模式(反应器设计模式)。Linux操作系统的 kernels 2.4内核版本之前，默认使用select；而目前windows下对同步IO的支持，都是select模型 |
| poll   | 较高     | Reactor          | Linux         | Linux下的JAVA NIO框架，Linux kernels 2.6内核版本之前使用poll进行支持。也是使用的Reactor模式 |
| epoll  | 高       | Reactor/Proactor | Linux         | Linux kernels 2.6内核版本及以后使用epoll进行支持；Linux kernels 2.6内核版本之前使用poll进行支持；另外一定注意，由于Linux下没有Windows下的IOCP技术提供真正的 异步IO 支持，所以Linux下使用epoll模拟异步IO |
| kqueue | 高       | Proactor         | Linux         | 目前JAVA的版本不支持                                         |

多路复用IO技术最适用的是“高并发”场景，所谓高并发是指1毫秒内至少同时有上千个连接请求准备好。其他情况下多路复用IO技术发挥不出来它的优势。另一方面，使用JAVA NIO进行功能实现，相对于传统的Socket套接字实现要复杂一些，所以实际应用中，需要根据自己的业务需求进行技术选择。



#### 什么是信号驱动IO？

应用进程使用 `sigaction` 系统调用，内核立即返回，应用进程可以继续执行，也就是说等待数据阶段应用进程是非阻塞的。内核在数据到达时向应用进程发送 SIGIO 信号，应用进程收到之后在信号处理程序中调用 `recvfrom `将数据从内核复制到应用进程中。

相比于非阻塞式 I/O 的轮询方式，信号驱动 I/O 的 CPU 利用率更高。

![img](https://pdai.tech/images/io/java-io-model-3.png)



#### 什么是异步IO？

相对于同步IO，异步IO不是顺序执行。用户进程进行`aio_read`系统调用之后，无论内核数据是否准备好，都会直接返回给用户进程，然后用户态进程可以去做别的事情。等到socket数据准备好了，内核直接复制数据给进程，然后从内核向进程发送通知。IO两个阶段，进程都是非阻塞的。

![img](https://pdai.tech/images/io/java-io-model-4.png)

Linux提供了AIO库函数实现异步，但是用的很少。目前有很多开源的异步IO库，例如`libevent`、`libev`、`libuv`



### 什么是Reactor模型？

大多数网络框架都是基于Reactor模型进行设计和开发，Reactor模型基于事件驱动，特别适合处理海量的I/O事件。

 

#### **传统的IO模型**？

这种模式是传统设计，每一个请求到来时，大致都会按照：请求读取->请求解码->服务执行->编码响应->发送答复 这个流程去处理。

![img](https://pdai.tech/images/io/java-io-reactor-1.png)

服务器会分配一个线程去处理，如果请求暴涨起来，那么意味着需要更多的线程来处理该请求。若请求出现暴涨，线程池的工作线程数量满载那么其它请求就会出现等待或者被抛弃。若每个小任务都可以使用非阻塞的模式，然后基于异步回调模式。这样就大大提高系统的吞吐量，这便引入了Reactor模型。



#### Reactor 模型？

- **Reactor模型中定义的三种角色**：

1. **Reactor**：负责监听和分配事件，将I/O事件分派给对应的Handler。新的事件包含连接建立就绪、读就绪、写就绪等。
2. **Acceptor**：处理客户端新连接，并分派请求到处理器链中。
3. **Handler**：将自身与事件绑定，执行非阻塞读/写任务，完成channel的读入，完成处理业务逻辑后，负责将结果写出channel。可用资源池来管理。



##### **单Reactor单线程模型**

Reactor线程负责多路分离套接字，accept新连接，并分派请求到handler。Redis使用单Reactor单进程的模型。

![img](https://pdai.tech/images/io/java-io-reactor-2.png)

消息处理流程：

1. Reactor对象通过select监控连接事件，收到事件后通过dispatch进行转发。
2. 如果是连接建立的事件，则由acceptor接受连接，并创建handler处理后续事件。
3. 如果不是建立连接事件，则Reactor会分发调用Handler来响应。
4. handler会完成read->业务处理->send的完整业务流程。



##### 单Reactor多线程模型

将handler的处理池化。

![img](https://pdai.tech/images/io/java-io-reactor-3.png)



##### **多Reactor多线程模型**

主从Reactor模型： 主Reactor用于响应连接请求，从Reactor用于处理IO操作请求，读写分离了。

![img](https://pdai.tech/images/io/java-io-reactor-4.png)



### 什么是Java NIO？

Java NIO主要有三大核心部分：Channel(通道)，Buffer(缓冲区), Selector。**传统IO基于字节流和字符流进行操作**，而**NIO基于Channel和Buffer(缓冲区)进行操作**，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中。Selector(选择区)用于监听多个通道的事件（比如：连接打开，数据到达）。因此，单个线程可以监听多个数据通道。

NIO和传统IO（一下简称IO）之间第一个最大的区别是，IO是面向流的，NIO是面向缓冲区的。

![img](https://pdai.tech/images/io/java-io-nio-x.png)

1. `Channel`：表示一个可以进行读写操作的对象，类似于传统的流，但可以同时进行读写操作。
2. `Buffer`：用于存储数据，可以在 Channel 和应用程序之间传递数据。
3. `Selector`：用于监控多个 Channel 的状态，可以同时处理多个 Channel 的 I/O 操作，从而实现非阻塞 I/O。

当一个 Channel 注册到 Selector 中时，Selector 会开始监听该 Channel 的状态。如果 Channel 可读或可写，Selector 就会通知应用程序进行读写操作。此时应用程序可以使用 Buffer 读取或写入数据，然后再将 Buffer 写入到 Channel 中。

由于 Selector 可以同时监控多个 Channel 的状态，因此可以实现高效的多路复用，从而处理大量的并发连接。与传统的阻塞 I/O 不同，NIO 可以实现非阻塞的 I/O 操作，当没有数据可读或可写时，程序不会被阻塞，可以更高效地利用系统资源，同时可以更好地处理并发连接。



## 4.3 零拷贝

### 传统的IO存在什么问题？为什么引入零拷贝的？

如果服务端要提供文件传输的功能，我们能想到的最简单的方式是：将磁盘上的文件读取出来，然后通过网络协议发送给客户端。

传统 I/O 的工作方式是，数据读取和写入是从用户空间到内核空间来回复制，而内核空间的数据是通过操作系统层面的 I/O 接口从磁盘读取（`read()`）或写入（`write()`）。

![img](https://pdai.tech/images/io/java-io-copy-3.png)

首先，**期间共发生了 4 次用户态与内核态的上下文切换**，因为发生了两次系统调用，一次是 read() ，一次是 write()，每次系统调用都得先从用户态切换到内核态，等内核完成任务后，再从内核态切换回用户态。

上下文切换到成本并不小，一次切换需要耗时几十纳秒到几微秒，虽然时间看上去很短，但是在高并发的场景下，这类时间容易被累积和放大，从而影响系统的性能。

其次，还发生了 **4 次数据拷贝**，其中**两次是 DMA 的拷贝**，另外**两次则是通过 CPU 拷贝**的，下面说一下这个过程：

- **第一次拷贝**，把磁盘上的数据拷贝到操作系统内核的缓冲区里，这个拷贝的过程是通过 DMA 搬运的。
- **第二次拷贝**，把内核缓冲区的数据拷贝到用户的缓冲区里，于是我们应用程序就可以使用这部分数据了，这个拷贝到过程是由 CPU 完成的。
- **第三次拷贝**，把刚才拷贝到用户的缓冲区里的数据，再拷贝到内核的 socket 的缓冲区里，这个过程依然还是由 CPU 搬运的。
- **第四次拷贝**，把内核的 socket 缓冲区里的数据，拷贝到网卡的缓冲区里，这个过程又是由 DMA 搬运的。

回过头看这个文件传输的过程，我们只是搬运一份数据，结果却搬运了 4 次，过多的数据拷贝无疑会消耗 CPU 资源，大大降低了系统性能。

这种简单又传统的文件传输方式，存在冗余的上文切换和数据拷贝，在高并发系统里是非常糟糕的，多了很多不必要的开销，会严重影响系统性能。

所以，**要想提高文件传输的性能，就需要减少「用户态与内核态的上下文切换」和「内存拷贝」的次数**。



### mmap + write怎么实现的零拷贝？

在前面我们知道，read() 系统调用的过程中会把内核缓冲区的数据拷贝到用户的缓冲区里，于是为了减少这一步开销，我们可以用 mmap() 替换 read() 系统调用函数。

mmap() 系统调用函数会直接把内核缓冲区里的数据「映射」到用户空间，这样，操作系统内核与用户空间就不需要再进行任何的数据拷贝操作。

![img](https://pdai.tech/images/io/java-io-copy-4.png)



具体过程如下：

- 应用进程调用了 mmap() 后，DMA 会把磁盘的数据拷贝到内核的缓冲区里。接着，应用进程跟操作系统内核「共享」这个缓冲区；
- 应用进程再调用 write()，操作系统直接将内核缓冲区的数据拷贝到 socket 缓冲区中，这一切都发生在内核态，由 CPU 来搬运数据；
- 最后，把内核的 socket 缓冲区里的数据，拷贝到网卡的缓冲区里，这个过程是由 DMA 搬运的。

我们可以得知，通过使用 mmap() 来代替 read()， 可以减少一次数据拷贝的过程。

但这还不是最理想的零拷贝，因为仍然需要通过 CPU 把内核缓冲区的数据拷贝到 socket 缓冲区里，而且仍然需要 4 次上下文切换，因为系统调用还是 2 次。



### sendfile怎么实现的零拷贝？

在 Linux 内核版本 2.1 中，提供了一个专门发送文件的系统调用函数 sendfile()，函数形式如下：

```c
#include <sys/socket.h>
ssize_t sendfile(int out_fd, int in_fd, off_t *offset, size_t count);
```

它的前两个参数分别是`目的端`和`源端`的文件描述符，后面两个参数是`源端的偏移量`和`复制数据的长度`，返回值是实际复制数据的长度。

首先，它可以替代前面的 read() 和 write() 这两个系统调用，这样就可以减少一次系统调用，也就减少了 2 次上下文切换的开销。

其次，该系统调用，可以直接把内核缓冲区里的数据拷贝到 socket 缓冲区里，不再拷贝到用户态，这样就只有 2 次上下文切换，和 3 次数据拷贝。如下图：

![img](https://pdai.tech/images/io/java-io-copy-5.png)

但是这还不是真正的零拷贝技术，如果网卡支持 SG-DMA（**The Scatter-Gather Direct Memory Access**）技术（和普通的 DMA 有所不同），我们可以进一步减少通过 CPU 把内核缓冲区里的数据拷贝到 socket 缓冲区的过程。

你可以在你的 Linux 系统通过下面这个命令，查看网卡是否支持 scatter-gather 特性：

```sh
$ ethtool -k eth0 | grep scatter-gather
scatter-gather: on
```

于是，从 Linux 内核 2.4 版本开始起，对于支持网卡支持 SG-DMA 技术的情况下， sendfile() 系统调用的过程发生了点变化，具体过程如下：

- 第一步，通过 DMA 将磁盘上的数据拷贝到内核缓冲区里；
- 第二步，缓冲区描述符和数据长度传到 socket 缓冲区，这样网卡的 SG-DMA 控制器就可以直接将内核缓存中的数据拷贝到网卡的缓冲区里，此过程不需要将数据从操作系统内核缓冲区拷贝到 socket 缓冲区中，这样就减少了一次数据拷贝；

所以，这个过程之中，只进行了 2 次数据拷贝，如下图：

![img](https://pdai.tech/images/io/java-io-copy-6.png)

这就是所谓的**零拷贝（Zero-copy）技术，因为我们没有在内存层面去拷贝数据，也就是说全程没有通过 CPU 来搬运数据，所有的数据都是通过 DMA 来进行传输的**。

零拷贝技术的文件传输方式相比传统文件传输的方式，**减少了 2 次上下文切换和数据拷贝次数，只需要 2 次上下文切换和数据拷贝次数，就可以完成文件的传输，而且 2 次的数据拷贝过程，都不需要通过 CPU，2 次都是由 DMA 来搬运**。



# 五、JVM和调优

## 5.1 类加载机制

### 类加载生命周期？

类从被加载到虚拟机内存中开始，到卸载出内存为止，他的整个生命周期包括：**加载（Loading）**、**验证（Verify）**、**准备（Prepare）**、**解析（Resolve）**、**初始化（Initialization）**、**使用（Using）** 和**卸载（Unloading）**7个阶段。其中验证、准备和解析可以合起来统称为链接（Linking）。各个阶段的发生顺序：

<img src="http://image.easyblog.top/15813276429581b8a5e4a-b04d-4965-8ce5-ae24b2948fd1.jpg" alt="img"/>

#### **加载**

**（1）类加载时机**

Java虚拟机中并没有明确规定类的加载时机，这个不同的虚拟机产品会有不同的实现。但是Java虚拟机规范中对于类的初始化阶段有明确的规定，当出现以下5种情况时必须立即对类进行初始化（间接说明了类的5种加载的时机）：

1. 当使用new关键字实例化对象、读取或设置一个类的静态字段、调用一个类的静态方法的时候
2. 使用java.lang.reflect包的方法对类进行反射调用的时候，如果类还没有初始化过，则需要初始化
3. 当初始化一个类的时候如果发现他的父类还没有初始化，那么需要先初始化父类
4. 当虚拟机启动的时候，用户指定的执行主类（含有main方法的那个类）会优先初始化这个类
5. 当使用动态语言支持时如果一个java.lang.invoke.MethodHandle实例最后的解析结果为RET_getStatic、RET_putStatic、RET_invokeStatic的方法句柄，并且这个方法句柄所在的类没有进行初始化，则需要先初始化。

**（2）类加载流程**

在加载阶段，JVM完成下面3件事：

1. 通过一个类的全限定名来获取定义此类的二进制字节流
2. 将字节流所代表的静态存储结构转化为方法区的运行时数据结构
3. **在内存中生成一个代表这个类的java.lang.Class对象**，作为方法区这个类的各种数据的访问接口

总结一下，加载阶段JVM的工作就是：**将类.class字节码文件中的二进制数据读入到内存中，将其放在运行时数据区的方法区内，然后在堆区创建一个java.lang.Class对象，用来封装类在方法区中的数据结构**



#### 链接

链接阶段具体有三个过程：**验证、准备和解析**。 

**（1）验证阶段**： **验证是链接的第一步，这一步的作用是为了确保class文件中字节流包含的信息符合虚拟机的要求，并且不会危害虚拟机的自生安全**。

> ​    验证大致上会完成下面4个阶段的检查动作：**文件格式验证**、**元数据验证**、**字节码验证**和**符号引用验证**。如果输入的字节流不符合class文件格式的约定，JVM就会抛出一个java.lang.VerifyError异常或其子类异常。

**Tips**：虚拟机的类加载机制中，验证阶段绝对是非常重要的一个阶段，但是它并不是必要的阶段。如果我们的程序（无论是我们自己写的还是第三方的）都已经被反复使用和验证的情况下，那么在真正运行的时候就可以考虑使用`-Xverify:none`来关闭大部分的类验证措施，以缩短类加载的时间。

**（2）准备阶段**

准备阶段是**JVM正式为类变量分配内存并为类变量设置初始值的阶段**，这些变量所使用的内存将在方法区中进行分配。**不过要清楚几点是**：

1）这个阶段给类变量设置的初始值并不是变量后面有程序员指定的值，而是统一设置为零值。正真指定的值这时是存放在类构造器<clinit>()方法中，例如下面的例子：

```css
 public static int a=100;   //在准备阶段变量a会在方法区中分得内存并被赋初值为0，之后又会在初始化阶段初始化为100
```

（2）即被static修饰同时又被final修饰的常量由于在编译的时候就被分配值了，准备阶段只会显示的初始化，也即准备阶段不会管这些常量的。 

（3）这里只会为一个类中的类变量分配内存并初始化零值，实例变量将会在对象实例化的时候随对象一起分>配在Java堆中。

**（3）解析阶段**

 解析阶段是**JVM将常量池的符号引用转换为直接引用的过程**。解析操作主要针对的类或接口、字段、类方法、接口方法、方法类型、方法句柄和调用点限定7类符号限定引用进行。



**初始化（Initialization）**

初始化阶段是类加载过程的最后一个阶段，到了这一步才正真开始执行类中定义的Java程序代码。**初始化阶段就是系统给类变量赋指定值并且执行静态代码块的阶段，或者说初始化阶段是执行类构造器`<clinit>()`方法的过程。**关于 **`<clinit>()`** 方法我们还需要知道下面几点：

1. `<clinit>()`方法是由javac编译器自动收集类中所有类变量的赋值动作和静态代码块中的语句合并而来，不需要人为定义。
2. `<clinit>()`方法虽然叫类构造器，但它与类的构造函数（类的构造函数在虚拟机视角下是`<init>()`方法）不同，他不需要显示的调用父类构造器，虚拟机会保证在子类的`<clinit>()`方法执行前，父类的`<clinit>()`方法已经执行完毕。
3. `<clinit>()`方法不是必须的，如果一个类中即没有静态语句块，也没有类变量，那么编译器就可以不为这个类生成`<clinit>()`方法。
4. 虚拟机会保证一个类的`<clinit>()`方法在多线程环境中被正确地加锁、同步。如果有多个线程同时去初始化一个类，那么只会有一个线程去执行这个类的`<clinit>()`方法，其他的线程都会阻塞。



### 类加载器的层次？

从JVM的角度来讲，只存在两种不同的类加载器：引导类加载器（Bootstrap ClassLoader）和用户自定义类加载器。之所以这样划分是因为引导类加载器是使用C++实现的，它是虚拟机的一部分，而其他的加载器（比如扩展类加载器、应用类加载器......）都是使用Java语言实现的，独立于虚拟机外部，并且都直接或间接的继承自`java.lang.ClassLoader`这个抽象类。

但是从Java开发人员的角度来看，JVM的类加载器可以细分为以下几种：

**（1）启动类加载器（Bootstrap ClassLoader）**

* 1）启动类加载器使用C++语言实现，它是JVM的组成部分 
* 2）它用来加载Java的核心类库（`$JAVA_HOME/jre/lib/rt.jar`、`resoures.jar`，`sun.boot.class.path`路径下的内容），用于提供JVM自身需要的类（大致就是以java、javax、sun开头的类库） 
* 3）由于是使用C++实现的，因此它不继承自java.lang.Classloader，也没有父加载器，并且启动类加载器无法被Java程序直接引用，如果尝试获取启动类加载器，那么一定返回的是null

**（2）扩展类加载器（Extension ClassLoader）**

由Java语言实现，具体的实现在`sun.misc.Launcher$ExtClassLoader`这个内部类中，他派生与ClassLoader，主要负责加载`$JAVA_HOME/jre/lib/ext`目录中的类库，或者被java.ext.dirs系统变量所指定的路径中的所有类库。

**（3）应用类加载器（Application ClassLoder）**

由`sun.misc.Launcher$AppClassLoader`实现。由于这个类加载器是ClassLoader中getSystemClassLaoder()方法的返回值，因此也称其为系统类加载器。它一般负责加载用户路径（ClassPath）上的类库，我们自己写的类一般情况下就是通过这个类加载器加载的。下图时ClassLoader、ExtClassLoader、AppClassloader之间在语言层面的继承关系



### 双亲委派机制？

<img src="http://image.easyblog.top/15813186368441321d601-da6e-4b34-a8cf-1e7dc59fb779.jpg" alt="img" style="zoom:40%;" />

#### 双亲委派机制工作原理

* 1）如果一个类加载器收到了类加载的请求，他不会自己立即去加载，而是把这个加载请求委托给父级加载器执行加载请求；
* 2）如果一个父级加载器还存在父级加载器，则进一步向上委托，依次递归，请求最终会传达到最顶层的启动类加载器；
* 3）如果父级加载器可以完成加载任务，就成功返回，倘若父级加载器无法完成加载任务，则它的子级类加载器才会尝试自己加载，如果还是不行在给子级加载器的子级加载器去加载，这就是双亲委派机制。

**需要注意的是**： 1. 双亲委派模型示意图所展示的不是几种类加载器的继承关系，而是他们在加载一个类的时候的委托关系（优先级关系），而这些类本质上也并不存在继承关系，示意图中所展示的只是一种层级（阶级）关系 2. 一个类如果所有的类加载器都加载失败，那么系统就会抛出`ClassNotFoundException`异常

另外，双亲委派机制实在ClassLoader类的loadClass()方法中实现的，因此如果不想使用算清委派机制，可以重写这个方法，但是一般不建议重写该方法。



#### 双亲委派机制的好处（为什么需要双亲委派机制？）

* 1）可以避免一个类被重复加载 
* 2）可以保护程序安全，尤其是核心API不会遭到随意篡改



### 如何破坏双亲委派模型

重写ClassLoader类的`loadClass()`方法，别重写`findClass`方法，因为`loadClass`是核心入口，将其重写成自定义逻辑即可破坏双亲委派模型。



### 如何自定义类加载器

只需要继承`java.lang.Classloader`类，然后覆盖他的`findClass(String name)`方法即可，该方法根据参数指定的类名称，返回对应 的Class对象的引用。

**自定义类加载器示例**

**自定义类加载器 CustomClassLoader：** 继承ClassLoader，重写findClass()方法，在findClass方法中使用文件IO将需要加载的类读取到byte数组中就OK了，之后交给defineClass方法处理就可以了。

```java
public class CustomClassLoader extends ClassLoader {

    //类的路径
    private String classPath;

    public CustomClassLoader(String classPath) {
        this.classPath = classPath;
    }

    //name表示这个类的类的全限定名
    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        byte[] bytes = null;
        InputStream is = null;
		//将类名中的.转换为路径符号/
        name = name.replaceAll("\\.", "/");
        ByteArrayOutputStream out = new ByteArrayOutputStream();
        try {
		    //读取class文件的流
            is = new FileInputStream(new File(classPath+"/"+name+".class"));
            int len = 0;
            while (-1 != (len = is.read())) {
                out.write(len);
            }
            bytes = out.toByteArray();
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            try {
                assert is != null;
                is.close();
                out.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        if(bytes==null){
            throw new ClassNotFoundException();
        }
        return defineClass(name,bytes,0,bytes.length);
    }
}
```

**测试代码**

```java
public class AppTest {

    @Test
    public void test(){
        try {
            //Car E:\IDE Java Files\LeetCode\algorithm\src\main\java\algorithm\greedy\Car.java
            CustomClassLoader classLoader = new CustomClassLoader("E:\\IDE Java Files\\LeetCode\\algorithm\\src\\main\\java");
            Class clazz = classLoader.loadClass("algorithm.greedy.Car");
            Object o = clazz.newInstance();
            Method print = clazz.getDeclaredMethod("print", null);
            print.invoke(o, null);
           /* Car car = new Car();
            car.print();*/
        }catch (Exception e){
            e.printStackTrace();
        }
    }

}
```



### 热部署原理

采取破坏双亲委派模型的手段来实现热部署，默认的`loadClass()`方法先找缓存，你改了class字节码也不会热加载，所以自定义ClassLoader，去掉找缓存那部分，直接就去加载，也就是每次都重新加载。



## 5.2 JVM内存结构

### JVM内存整体结构？

<img src="https://image.easyblog.top/jvm-pic.jpg"  />

Java 虚拟机定义了若干种程序运行期间会使用到的运行时数据区，其中有一些会随着虚拟机启动而创建，随着虚拟机退出而销毁。另外一些则是与线程一一对应的，这些与线程一一对应的数据区域会随着线程开始和结束而创建和销毁。

- **线程私有**：`程序计数器`、`虚拟机栈`、`本地方法栈`
- **线程共享**：`堆`、`方法区`, `堆外内存`（Java7的永久代或JDK8的元空间、代码缓存）



### 什么是程序计数器？

程序计数器（`Program Conputer Register`）这是一块较小的内存空间，可以看做是当前线程所执行的字节码的行号指示器，在虚拟机的概念模型里，字节码解释器的工作就是通过改变这个计数器的值来选取下一条需要执行的字节码指令，分支、循环、跳转、异常处理、线程恢复等基础功能都需要依赖这个计数器来完成。

**①、线程私有**

Java虚拟机支持多线程，是通过线程轮流切换并分配处理器执行时间的方式来实现的，在任一确定的时刻，一个处理器只会执行一条线程中的指令，因此为了线程切换后能恢复到正确的执行位置，每条线程都需要一个独立的程序计数器。因此线程启动时，JVM 会为每个线程分配一个PC寄存器（Program Conter，也称程序计数器）。

**②、记录当前字节码指令执行地址**

如果当前线程执行的是一个Java方法，这个计数器记录的是正在执行的虚拟机字节码指令的地址；如果正在执行的是 Native 方法，则这个计数器值为空（Undefined）。

**③、不抛 OutOfMemoryError 异常**

程序计数器的空间大小不会随着程序执行而改变，始终只是保存一个 returnAdress 类型的数据或者一个与平台相关的本地指针的值。所以该区域是Java运行时内存区域中唯一一个Java虚拟机规范中没有规定任何 OutOfMemoryError 情况的区域。





> **PC寄存器为什么会被设定为线程私有的？**
>
> 多线程在一个特定的时间段内只会执行其中某一个线程方法，CPU会不停的做任务切换，这样必然会导致经常中断或恢复。为了能够准确的记录各个线程正在执行的当前字节码指令地址，所以为每个线程都分配了一个PC寄存器，每个线程都独立计算，不会互相影响。



### 什么是虚拟机栈？

Java虚拟机栈（`Java Virtual Machine stack`），这块区域也是线程私有的，与线程同时创建，用于存储栈帧。Java 每个方法执行的时候都会同时创建一个栈帧（Stack Frame）用于存储局部变量表、操作栈、动态链接、方法出口等信息，每一个方法被调用直至执行完成的过程，就对应着一个栈帧在虚拟机栈中从入栈到出栈的过程。

<img src="https://images2018.cnblogs.com/blog/1120165/201807/1120165-20180722152410736-914758638.png" alt="img"/>

**①、线程私有**

随线程创建而创建，声明周期和线程保持一致。

**②、由栈帧组成**

线程每个方法被执行的时候都会创建一个栈帧，用于存储**局部变量表、操作栈、动态链接、方法出口**等信息，每一个方法被调用直至执行完成的过程，就对应着一个栈帧在虚拟机栈中从入栈到出栈的过程。

**③、抛出 StackOverflowError 和 OutOfMemoryError 异常**

如果线程请求的栈深度大于虚拟机所允许的深度，将抛出 StackOverflowError 异常；如果虚拟机栈可以动态扩展，当扩展时无法申请到足够的内存时会抛出 OutOfMemoryError 异常。







### 什么是本地方法栈？

本地方法栈（Native Method Stacks）作用和虚拟机栈类型，虚拟机栈执行的是Java方法，本地方法栈执行的是 Native 方法，本地方法栈也会抛出抛出 StackOverflowError 和 OutOfMemoryError 异常。

注意：由于虚拟机规范并没有对本地方法栈中的方法使用语言、使用方式和数据结构强制规定，因此具体的虚拟机可以自由实现它。上图我们也给出在 HotSpot 虚拟机中，本地方法栈和虚拟机栈合为一体了。



### 什么是Java堆？

Java堆是Java虚拟机所管理内存最大、被所有线程共享的一块区域，目的是用来存放对象，基本上所有的对象实例和数组都在堆上分配（不是绝对）。Java堆也是垃圾回收器管理的主要区域。

**①、线程共享**

堆存放的对象，某个线程修改了对象属性，另外一个线程从堆中获取的该对象是修改后的对象，为什么堆要设计成线程共享呢？

我们可以假设堆是线程私有的，很显然一个系统创建的对象会有很多，而且有些对象会比较大，如果设计成线程私有的，那么如果有很多线程同时工作，那么都必须给他们分配相应的私有内存，我相信内存很快就撑爆了，很显然将堆设计为线程共享是最好不过了，不过凡事都具有两面性，线程共享的设计这也带来了**多线程并发资源冲突问题**，关于这个问题由于不是本系列博客的主旨，这里就不做详细介绍了。

**②、存放对象**

基本上所有的对象实例和数组都要在堆上进行分配，但是随着 JIT 编译器的发展和逃逸分析技术的成熟，栈上分配、标量替换等优化技术会导致对象不一定在堆上进行分配。

**③、垃圾收集**

　　Java堆也被称为“GC堆”，是垃圾回收器的主要操作内存区域。当前垃圾回收器都是使用的分代收集算法，所以Java堆还可以分为：新生代和老年代，而新生代又可以分为 Eden 空间、From Survivor 空间、To Survivor空间。这是为了更好的回收内存，关于垃圾回收算法在后续博客会详细介绍。

<img src="http://image.easyblog.top/1581945296692366e8ad8-8995-4e99-9631-e5e9e10b617b.png" alt="img"  />

**④、抛出 OutOfMemoryError 异常**

　　根据Java虚拟机规范，Java堆可以处于物理上不连续的内存空间中，只要逻辑上连续即可，实现时既可以实现成固定大小，也可以是扩展的。如果在堆中没有完成实例分配，并且堆也无法扩展，将抛出OutOfMemoryError 异常。



### 堆区内存是怎么细分的？

对于大多数应用，Java 堆是 Java 虚拟机管理的内存中最大的一块，被所有线程共享。此内存区域的唯一目的就是存放对象实例，几乎所有的对象实例以及数据都在这里分配内存。

为了进行高效的垃圾回收，虚拟机把堆内存**逻辑上**划分成三块区域（分代的唯一理由就是优化 GC 性能）：

1. 新生带（年轻代）：新对象和没达到一定年龄的对象都在新生代
2. 老年代（养老区）：被长时间使用的对象，老年代的内存空间应该要比年轻代更大

<img src="http://image.easyblog.top/1581945296692366e8ad8-8995-4e99-9631-e5e9e10b617b.png" alt="img"  />

Java 虚拟机规范规定，Java 堆可以是处于物理上不连续的内存空间中，只要逻辑上是连续的即可，像磁盘空间一样。实现时，既可以是固定大小，也可以是可扩展的，主流虚拟机都是可扩展的（通过 `-Xmx` 和 `-Xms` 控制），如果堆中没有完成实例分配，并且堆无法再扩展时，就会抛出 `OutOfMemoryError` 异常。

- **年轻代 (Young Generation)**

年轻代是所有新对象创建的地方。当填充年轻代时，执行垃圾收集。这种垃圾收集称为 **Minor GC**。年轻一代被分为三个部分——伊甸园（**Eden Memory**）和两个幸存区（**Survivor Memory**，被称为from/to或s0/s1），默认比例是`8:1:1`

1. 大多数新创建的对象都位于 Eden 内存空间中
2. 当 Eden 空间被对象填充时，执行**Minor GC**，并将所有幸存者对象移动到一个幸存者空间中
3. Minor GC 检查幸存者对象，并将它们移动到另一个幸存者空间。所以每次，一个幸存者空间总是空的
4. 经过多次 GC 循环后存活下来的对象被移动到老年代。通常，这是通过设置年轻一代对象的年龄阈值来实现的，然后他们才有资格提升到老一代

- **老年代(Old Generation)**

旧的一代内存包含那些经过许多轮小型 GC 后仍然存活的对象。通常，垃圾收集是在老年代内存满时执行的。老年代垃圾收集称为 主GC（Major GC），通常需要更长的时间。

大对象直接进入老年代（大对象是指需要大量连续内存空间的对象）。这样做的目的是避免在 Eden 区和两个Survivor 区之间发生大量的内存拷贝



### JVM中对象的分配过程?

![](http://image.easyblog.top/1680337604178b58041a8-1c86-410b-8294-06f3eddc2924.png)

对象分配过程：

- 1）依据**逃逸分析**，判断是否能栈上分配？
  - 如果可以，使用标量替换方式，把对象分配到`VM Stack`中。如果 线程销毁或方法调用结束后，自动销毁，不需要 GC 回收器 介入。
  - 否则，继续下一步。
- 2）判断是否大对象？
  - 如果是，直接分配到堆上 `Old Generation` 老年代上。如果对象变为垃圾后，由老年代GC 收集器（比如 Parallel Old, CMS, G1）回收。
  - 否则，继续下一步。
- 3）判断是否可以在 `TLAB`中分配？
  - 如果是，在 `TLAB`中分配堆上`Eden`区。
  - 否则，在 `TLAB`外堆上的`Eden`区分配。



### 什么是逃逸分析？栈上分配？

逃逸分析（`Escape Analysis`）简单来讲就是，Java Hotspot **虚拟机通过分析新创建对象的使用范围，并决定是否在 Java 堆上分配内存的一项技术**。

逃逸分析的 JVM 参数如下：

- 开启逃逸分析：`-XX:+DoEscapeAnalysis`
- 关闭逃逸分析：`-XX:-DoEscapeAnalysis`
- 显示分析结果：`-XX:+PrintEscapeAnalysis`

逃逸分析技术在 Java SE 6u23+ 开始支持，并默认设置为启用状态，可以不用额外加这个参数。

针对上面第三点，当一个对象没有逃逸时，可以得到以下几个虚拟机的优化。

#### 锁消除

我们知道线程同步锁是非常牺牲性能的，当编译器确定当前对象只有当前线程使用，那么就会移除该对象的同步锁。

例如，StringBuffer 和 Vector 都是用 synchronized 修饰线程安全的，但大部分情况下，它们都只是在当前线程中用到，这样编译器就会优化移除掉这些锁操作。

锁消除的 JVM 参数如下：

- 开启锁消除：`-XX:+EliminateLocks`
- 关闭锁消除：`-XX:-EliminateLocks`

锁消除在 JDK8 中都是默认开启的，并且锁消除都要建立在**逃逸分析**的基础上。

#### 标量替换

首先要明白`标量`和`聚合量`，**基础类型和对象的引用可以理解为标量，它们不能被进一步分解。而能被进一步分解的量就是聚合量**，比如：对象。

对象是聚合量，它又可以被进一步分解成标量，将其成员变量分解为分散的变量，这就叫做`标量替换`。

这样，如果一个对象没有发生逃逸，那压根就不用创建它，只会在栈或者寄存器上创建它用到的成员标量，节省了内存空间，也提升了应用程序性能。

标量替换的 JVM 参数如下：

- 开启标量替换：`-XX:+EliminateAllocations`
- 关闭标量替换：`-XX:-EliminateAllocations`
- 显示标量替换详情：`-XX:+PrintEliminateAllocations`

标量替换同样在 JDK8 中都是默认开启的，并且都要建立在逃逸分析的基础上。

#### 栈上分配

当对象没有发生逃逸时，该对象就可以通过标量替换分解成成员标量分配在栈内存中，和方法的生命周期一致，随着栈帧出栈时销毁，减少了 GC 压力，提高了应用程序性能。





### 什么是 TLAB ?

TLAB（Thread Local Allocation Buffer）是虚拟机在堆内存的Eden划分出来的一块专用空间线程专属。在虚拟机的TLAB功能启动的情况下，在线程初始化时，虚拟机会为每个线程分配一块TLAB空间，只给当前线程使用，这样每个线程都单独拥有一个空间，如需要分配内存，在自己的空间上分配，在不存在竞争的情况大大提升分配效率。



### 为什么会有TLAB?

TLAB的存在可以提高Java程序的并发性能和对象分配的效率，降低垃圾回收的负担，从而提高整个程序的性能：

1. `减少线程之间的竞争`：在没有TLAB的情况下，所有线程都需要竞争堆内存中的空闲空间进行对象分配，这会增加线程之间的竞争，影响程序的并发性能。有了TLAB，每个线程都有自己的内存区域进行对象分配，可以避免线程之间的竞争。
2. `减少垃圾回收的负担`：在没有TLAB的情况下，每次对象分配都会增加垃圾回收的负担。因为垃圾回收器需要遍历整个堆内存才能找到不再被使用的对象进行回收。有了TLAB，对象分配发生在线程私有的内存区域中，不需要遍历整个堆内存，从而减少了垃圾回收的负担。
3. `提高对象分配的效率`：由于TLAB是线程私有的，所以对象分配可以在TLAB中进行，不需要对整个堆内存进行扫描和分配，可以减少内存分配时的锁竞争和系统调用等操作，从而提高对象分配的效率。



### JVM中对象在堆中的生命周期?

1. 在 JVM 内存模型的堆中，堆被划分为新生代和老年代 
   - 新生代又被进一步划分为 **Eden区** 和 **Survivor区**，Survivor 区由 **From Survivor** 和 **To Survivor** 组成
2. 当创建一个对象时，对象会被优先分配到新生代的 Eden 区 
   - 此时 JVM 会给对象定义一个**对象年轻计数器**（`-XX:MaxTenuringThreshold`）
3. 当 Eden 空间不足时，JVM 将执行新生代的垃圾回收（Minor GC） 
   - JVM 会把存活的对象转移到 Survivor 中，并且对象年龄 +1
   - 对象在 Survivor 中同样也会经历 Minor GC，每经历一次 Minor GC，对象年龄都会+1
4. 如果分配的对象超过了`-XX:PetenureSizeThreshold`，对象会**直接被分配到老年代**



### **什么是方法区？**

方法区（Method Area）用来**存储已被Java虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据**。

方法区也称为“永久代”，这是因为垃圾回收器对方法区的垃圾回收比较少，主要是针对常量池的回收以及对类型的卸载，回收条件比较苛刻。经常会导致对此内存未完全回收而导致内存泄露，最后当方法区无法满足内存分配时，将抛出 OutOfMemoryError 异常。

在JDK1.8 的 HotSpot 虚拟机中，已经去掉了方法区的概念，用 `Metaspace` 代替，并且将其移到了本地内存来规划了。



### **什么是运行时常量池？**

在Java虚拟机规范中，运行时常量池（Runtime Constant Pool）用于存放编译期生成的各种字面量和符号引用，是**方法区**的一部分。但是Java虚拟机规范对其没有做任何细节的要求，所以不同虚拟机实现商可以按照自己的需求来实现该区域，比如在 **HotSpot** 虚拟机实现中，就将运行时常量池移到了堆中。

**①、存放字 面量、符号引用、直接引用**

通常来说，该区域除了保存Class文件中描述的引用外，还会把翻译出来的直接引用也存储在运行时常量池，并且Java语言并不要求常量一定只能在编译器产生，运行期间也可能将常量放入池中，比如String类的intern()方法，**当调用intern方法时，如果池中已经包含一个与该`String`确定的字符串相同`equals(Object)`的字符串，则返回该字符串。否则，将此String对象添加到池中，并返回此对象的引用。**。

**②、抛出 OutOfMemoryError 异常**

运行时常量池是方法区的一部分，会受到方法区内存的限制，当常量池无法申请到内存时，会抛出该异常。



### 什么是直接内存？

直接内存（Direct Memory）并不是虚拟机运行时数据区的一部分，它也不是Java虚拟机规范定义的内存区域。我们可以看到在 HotSpot 中，就将方法区移除了，用元数据区来代替，并且将元数据区从虚拟机运行时数据区移除了，转到了本地内存中，也就是说这块区域是受本机物理内存的限制，当申请的内存超过了本机物理内存，才会抛出 OutOfMemoryError 异常。

直接内存也是受本机物理内存的限制，在JDK1.4中新加入的 NIO（new input/output）类，引入了一种基于通道（Channel）与缓冲区（Buffer）的 I/O 方式，它可以使用 Native 函数库直接分配堆外内存，然后通过一个存储在Java堆里面的 DirectByteBuffer 对象作为这块内存的引用操作，这样避免了在Java堆和Native堆中来回复制数据，显著提高性能。



### Java对象在内存中的存储布局

在HotSpot虚拟机中对象在内存中的存储布局可以分为：**对象头、实例数据和对齐填**充3部分。

<img src="http://image.easyblog.top/159427348170717f20231-8e84-46f2-8ae5-b4fed58c5c9f.png" alt="img" />

#### **对象头（Object Header）**

对象头在HotSpot虚拟机中被分为了两部分，**一部分官方称为Mark Word;另一部分是类型指针**。如果对象是一个Java数组，那在对象头中还有一块用于记录数组长度的数据。

<img src="http://image.easyblog.top/15822769154174f52765f-4a90-4384-aed8-2f530eb98f2b.png" alt="img" style="zoom:80%;" />

* 第一部分**Mark Word用于存储对象自身的运行时数据**，如哈希码（HashCode）、GC 分代年龄、锁状态标志、线程持有的锁、偏向线程 ID、对象分代年龄等信息。
* 第二部分是**类型指针，即对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例**。在不开启类指针压缩（`-XX:+UseCompressedClassPointers`）的情况下是8字节。压缩后变为4字节，默认压缩。 通过命令：`java -XX:+PrintCommandLineFlags -version` 查看classPointer是否开启压缩

#### **实例数据（Instance Data）**

 实例数据部分是对象真正存储的有效信息，也是在程序代码中所定义的各种类型的字段内容。     这部分的存储顺序会受到虚拟机分配策略参数（FieldsAllocationStyle）和字段在 Java 源码中定义顺序的影响。

#### **对齐填充**

由于HotSpot VM的自动内存管理系统要求对象的起始地址必须是8字节的整数倍，即：Java对象的大小必须是8字节的整数倍，而对象头的大小正好是8字节的整数倍，所以当对象的实例数据没有对齐的时候，就需要对齐填充来补全。因此对齐填充并不是必须存在的，也没有特殊的含义，它仅仅起到了占位符的作用。



### 一个Object对象占用多少个字节内存?

会占用16个字节。比如`Object obj = new Object();`

因为obj引用占用栈的4个字节，new出来的对象占用堆中的8个字节，4+8=12，但是对象要求都是8的倍数，所以对象的字节对齐（Padding）部分会补齐4个字节，也就是占用16个 字节。

再比如：

```java
public class NewObj {
    int count;
    boolean flag;
    Object obj;
}
NewObj obj = new NewObj();
```

这个对象大小为：对象头8字节+int类型4字节+boolean类型1字节+对象的引用4字节=17字节，需要8的倍数，所以字节对齐需要补充7个字节，也就是这段程序占用24字节。

## 5.3 GC垃圾回收

### 对象怎么定位

目前主流的对象访问方式有使用**句柄** 和 **直接指针**两种方式。

**（1）句柄访问**     

在Java堆内存中划分一块内存专门用来作为句柄池，JVM桟的局部变量表的reference存储对象的句柄地址，而句柄中保存了对象实例数据与类型数据的具体地址，如下图所示：

<img src="http://image.easyblog.top/1582280468345e2700388-2407-422f-8c39-d74306d2a90e.jpg" alt="enter image description here" />

使用句柄方式最大的好处就是reference中存储的是稳定的句柄地址，在对象被移动（垃圾收集时移动对象是非常普遍的行为）时只会改变句柄中的实例数据指针，而reference本身不需要被修改。

**（2）直接指针访问**

​    此时reference直接存储对象在堆中的地址，不再需要句柄池，这样做最大的好处是提高了性能，因为它节省了一次指针定位的时间开销。然而这也是HotSpot VM所选择的对象访问方式。

<img src="http://image.easyblog.top/1582280393660ad3239df-e018-4404-8174-319e5ceb4e47.jpg" alt="直接指针访问" />



### 如何判断对象是否能被回收？

#### **引用计数算法**

这种算法是这样的：给每一个创建的对象增加一个引用计数器，每当有一个地方引用它时，这个计数器就加1；而当引用失效时，这个计数器就减1。当这个引用计数器值为0时，也就是说这个对象没有任何地方在使用它了，那么这就是一个无效的对象，便可以进行垃圾回收了。这种算法实现简单，而且效率也很高。但是**Java没有采用该算法来进行垃圾回收**，因为这种算法无法解决对象之间的循环引用问题。

#### 可达性分析法

这里直接给出结论：**在主流的商用程序中（Java，C#），都是使用根搜索算法（GC Roots Tracing）来判定对象是否存活。** 

该算法思路：通过一系列名为“GC Roots” 的对象作为终点，当一个对象到GC Roots 之间无法通过引用到达时，那么该对象便可以进行回收了。

<img src="http://image.easyblog.top/1582340634188cbd9a85a-2a5a-48ef-acf6-bca177d9caad.jpg" alt="img" />

如上图所示，GC Roots到Object1、Object2、Object3、Object4都是可达的（可达意为GC Roots沿着某条路径（引用链）一定可以找到该对象），但是GC Roots到Object5、Object6、Object7没有可达的引用链，因此它们将会被判断为可回收的对象。

- 优点：更加精确和严谨，可以分析出循环数据结构相互引用的情况；
- 缺点：实现比较复杂、需要分析大量数据，消耗大量时间、分析过程需要GC停顿（引用关系不能发生变化），即停顿所有Java执行线程（称为"Stop The World"，是垃圾回收重点关注的问题）。

在Java语言中可以作为GC Roots的对象有以下几种：

（1）**虚拟机桟桟帧中的局部变量表所引用的对象。** 

（2）**方法区中类的静态的属性所引用的对象**。 

（3）**方法区中常量引用的对象**。 

（4）**本地方法桟中本地方法所引用的对象**。



### finalize方法回收对象两次标记过程

**当一个对象被GC Roots标记为不可用之后，要宣告一个对象的死亡还要至少经过两次标记**：

（1）当一个对象通过可达性分析发现CG Roots与他不可达，那他会被第一次标记并且进行第一次筛选，筛选的条件是此对象的finalize()方法是否有必要执行。当对象的finalize方法没有重写或已经被执行过了，那么将不会再执行，那么此对象就会立即被宣告死亡。

（2）如果这个对象有必要执行 finalize() 方法，那么该**对象将会被放置在一个由虚拟机自动建立、低优先级，名为 F-Queue 队列中，GC会对F-Queue进行第二次标记**，如果对象在finalize() 方法中成功拯救了自己（比如重新与GC Roots建立连接），那么第二次标记时，就会将该对象移除即将回收的集合，否则就会被回收。



### 对象的引用类型有哪几种，分别介绍下？

#### **强引用**

类似`Object obj=new Object()`这样的引用方式就是强引用，**只有所有的GC Roots对象都不通过强引用引用该对象的时候，该对象才能被垃圾回收**。日常开发中，使用最多的可能就是强引用了吧！

#### **软引用**

对于只有软引用的对象而言，当系统内存空间足够时，它不会被系统回收；如果内存空间不足了，就会被回收。软引用可以使用`SoftReference<V>`类来实现并且可以配合引用队列来释放软引用自身

使用场景：软引用可以用于存放一些重要性不是很强又不能随便让清除的对象，比如图片编辑器、视屏编辑器、缓存等

#### **弱引用**

对于只有弱引用的对象而言，当系统垃圾回收机制运行时，不管系统内存是否足够，该对象所占用的内存一定会被回收。在不需要的时候，让JVM自动帮我们处理掉它们。弱引用可以使用`WeakReference<V>`类来实现。同理可以配合引用队列来释放弱引用自身

使用场景：ThreadLocal中的ThreadLocalMap中的key和ThreadLocal对象之间就是弱引用；WeakHashMap类中的key

#### **虚引用**

如果一个对象只有一个虚引用时，那么它和没有引用的效果大致相同。

使用场景：用于对象销毁前的一些操作，比如说资源释放等



### 垃圾收集算法有哪些？

垃圾回收涉及到大量的程序细节，而且各个平台的虚拟机操作内存的方式也不一样，但是他们进行垃圾回收的算法是通用的，所以这里我们也只介绍几种通用算法。

#### 标记-清除算法

<img src="https://img2018.cnblogs.com/blog/1120165/201907/1120165-20190701213406016-286379646.png" alt="img" style="zoom:60%;" />

**算法实现**：分为标记-清除两个阶段，首先根据上面的根搜索算法标记出所有需要回收的对象，在标记完成后，然后在统一回收掉所有被标记的对象。

**缺点**：

* 1、`效率低`：标记和清除这两个过程的效率都不高。

* 2、`容易产生内存碎片`：因为内存的申请通常不是连续的，那么清除一些对象后，那么就会产生大量不连续的内存碎片，而碎片太多时，当有个大对象需要分配内存时，便会造成没有足够的连续内存分配而提前触发垃圾回收，甚至直接抛出OutOfMemoryExecption。



#### 复制算法

为了解决标记-清除算法的两个缺点，复制算法诞生了。

<img src="https://img2018.cnblogs.com/blog/1120165/201907/1120165-20190701215336656-48891683.png" alt="img" style="zoom:60%;" />

**算法实现**：将可用内存按容量划分为大小相等的两块区域，每次只使用其中一块，当这一块的内存用完了，就将还活着的对象复制到另一块区域上，然后再把已使用过的内存空间一次性清理掉。

**优点**：每次都是只对其中一块内存进行回收，不用考虑内存碎片的问题，而且分配内存时，只需要移动堆顶指针，按顺序进行分配即可，简单高效。

**缺点**：将内存分为两块，但是每次只能使用一块，也就是说，机器的一半内存是闲置的，这资源浪费有点严重。并且如果对象存活率较高，每次都需要复制大量的对象，效率也会变得很低。



#### 标记-整理算法

上面我们说到复制算法的缺点——会浪费一半的内存，并且对象存活率较高时，会有过多的复制操作，效率低下。

如果对象存活率很高，基本上不会进行垃圾回收时，标记-整理算法诞生了。

<img src="https://img2018.cnblogs.com/blog/1120165/201907/1120165-20190701215456537-787486222.png" alt="img" style="zoom:50%;" />

**算法实现**：首先标记出所有存活的对象，然后让所有存活对象向一端进行移动，最后直接清理到端边界以外的内存。

**局限性**：只有对象存活率很高的情况下，使用该算法才会效率较高。



### 分代收集是怎么工作的？

以上三种垃圾回收算法是JVM垃圾回收器最常使用的三种算法模型，基于这些算法，JVM中有一个**分代回收模型**。

模型实现：根据对象的存活周期不同将内存分为几块，然后不同的区域采用不同的回收算法。

1、对于存活周期较短，每次都有大批对象死亡，只有少量存活的区域，采用复制算法，因为只需要付出少量存活对象的复制成本即可完成收集；

2、对于存活周期较长，没有额外空间进行分配担保的区域，采用标记-整理算法，或者标记-清除算法。

比如，对于 HotSpot 虚拟机，它将堆空间分为如下两块区域：

<img src="http://image.easyblog.top/1581945296692366e8ad8-8995-4e99-9631-e5e9e10b617b.png" alt="img" style="zoom:80%;" />

堆有新生代和老年代两块区域组成，而新生代区域又分为三个部分，分别是 Eden,From Surivor,To Survivor ,比例是8:1:1。

新生代一般采用复制算法，每次使用一块Eden区和一块Survivor区，当进行垃圾回收时，将Eden和一块Survivor区域的所有存活对象复制到另一块Survivor区域，然后清理到刚存放对象的区域，依次循环。

老年代一般采用标记-清除或者标记-整理算法，根据使用的垃圾回收器来进行判断。

至于为什么要这样，这是由于内存分配的机制导致的，**新生代存的基本上都是朝生夕死的对象，而老年代存放的都是存活率很高的对象**。



### Java垃圾回收器有哪些？

<img src="http://image.easyblog.top/1594299535371015eb6ac-5b51-4881-ad53-3a8d983adb26.jpeg" alt="深入理解JVM—垃圾回收器（Grabage Collector）进阶篇" style="zoom:50%;" />

1. **Serial 收集器**

![image](https://pdai.tech/images/pics/22fda4ae-4dd5-489d-ab10-9ebfdad22ae0.jpg)

Serial 是单线程的收集器，只会使用一个线程进行垃圾收集工作。

它的优点是简单高效，对于单个 CPU 环境来说，由于没有线程交互的开销，因此拥有最高的单线程收集效率。

它是 Client 模式下的默认新生代收集器，因为在用户的桌面应用场景下，分配给虚拟机管理的内存一般来说不会很大。Serial 收集器收集几十兆甚至一两百兆的新生代停顿时间可以控制在一百多毫秒以内，只要不是太频繁，这点停顿是可以接受的。



2. **ParNew 收集器**

![image](https://pdai.tech/images/pics/81538cd5-1bcf-4e31-86e5-e198df1e013b.jpg)

它是 Serial 收集器的多线程版本。

是 Server 模式下的虚拟机首选新生代收集器，除了性能原因外，主要是因为除了 Serial 收集器，只有它能与 CMS 收集器配合工作。

默认开启的线程数量与 CPU 数量相同，可以使用 -XX:ParallelGCThreads 参数来设置线程数。



3. **Parallel Scavenge 收集器**

与 ParNew 一样是多线程收集器。

其它收集器关注点是尽可能缩短垃圾收集时用户线程的停顿时间，而它的目标是达到一个可控制的吞吐量，它被称为“吞吐量优先”收集器。这里的吞吐量指 CPU 用于运行用户代码的时间占总时间的比值。

停顿时间越短就越适合需要与用户交互的程序，良好的响应速度能提升用户体验。而高吞吐量则可以高效率地利用 CPU 时间，尽快完成程序的运算任务，主要适合在后台运算而不需要太多交互的任务。

缩短停顿时间是以牺牲吞吐量和新生代空间来换取的: 新生代空间变小，垃圾回收变得频繁，导致吞吐量下降。

可以通过一个开关参数打开 GC 自适应的调节策略(GC Ergonomics)，就不需要手动指定新生代的大小(-Xmn)、Eden 和 Survivor 区的比例、晋升老年代对象年龄等细节参数了。虚拟机会根据当前系统的运行情况收集性能监控信息，动态调整这些参数以提供最合适的停顿时间或者最大的吞吐量。



4. **Serial Old 收集器**

![image](https://pdai.tech/images/pics/08f32fd3-f736-4a67-81ca-295b2a7972f2.jpg)

Serial Old 是 Serial 收集器的老年代版本，也是给 Client 模式下的虚拟机使用。如果用在 Server 模式下，它有两大用途:

- 在 JDK 1.5 以及之前版本(Parallel Old 诞生以前)中与 Parallel Scavenge 收集器搭配使用。
- 作为 CMS 收集器的后备预案，在并发收集发生 Concurrent Mode Failure 时使用。



5. **Parallel Old 收集器**

![image](https://pdai.tech/images/pics/278fe431-af88-4a95-a895-9c3b80117de3.jpg)

Parallel Old 是 Parallel Scavenge 收集器的老年代版本。

在注重吞吐量以及 CPU 资源敏感的场合，都可以优先考虑 Parallel Scavenge 加 Parallel Old 收集器。



6. **CMS 收集器**

![image](https://pdai.tech/images/pics/62e77997-6957-4b68-8d12-bfd609bb2c68.jpg)

`CMS(Concurrent Mark Sweep)`，Mark Sweep 指的是标记 - 清除算法。

分为以下四个流程:

- 初始标记: 仅仅只是标记一下 GC Roots 能直接关联到的对象，速度很快，需要停顿。
- 并发标记: 进行 GC Roots Tracing 的过程，它在整个回收过程中耗时最长，不需要停顿。
- 重新标记: 为了修正并发标记期间因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，需要停顿。
- 并发清除: 不需要停顿。

在整个过程中耗时最长的并发标记和并发清除过程中，收集器线程都可以与用户线程一起工作，不需要进行停顿。

具有以下缺点:

- 吞吐量低: 低停顿时间是以牺牲吞吐量为代价的，导致 CPU 利用率不够高。
- 无法处理浮动垃圾，可能出现 Concurrent Mode Failure。浮动垃圾是指并发清除阶段由于用户线程继续运行而产生的垃圾，这部分垃圾只能到下一次 GC 时才能进行回收。由于浮动垃圾的存在，因此需要预留出一部分内存，意味着 CMS 收集不能像其它收集器那样等待老年代快满的时候再回收。如果预留的内存不够存放浮动垃圾，就会出现 Concurrent Mode Failure，这时虚拟机将临时启用 Serial Old 来替代 CMS。
- 标记 - 清除算法导致的空间碎片，往往出现老年代空间剩余，但无法找到足够大连续空间来分配当前对象，不得不提前触发一次 Full GC。



7. **G1 收集器**

`G1(Garbage-First)`，它是一款面向服务端应用的垃圾收集器，在多 CPU 和大内存的场景下有很好的性能。HotSpot 开发团队赋予它的使命是未来可以替换掉 CMS 收集器。

堆被分为新生代和老年代，其它收集器进行收集的范围都是整个新生代或者老年代，而 **G1 可以直接对新生代和老年代一起回收**。

G1 把堆划分成多个大小相等的独立区域(Region)，新生代和老年代不再物理隔离。

![image](https://pdai.tech/images/pics/9bbddeeb-e939-41f0-8e8e-2b1a0aa7e0a7.png)

**通过引入 Region 的概念，从而将原来的一整块内存空间划分成多个的小空间，使得每个小空间可以单独进行垃圾回收**。这种划分方法带来了很大的灵活性，使得可预测的停顿时间模型成为可能。通过记录每个 Region 垃圾回收时间以及回收所获得的空间(这两个值是通过过去回收的经验获得)，并维护一个优先列表，每次根据允许的收集时间，优先回收价值最大的 Region。

每个 Region 都有一个 `Remembered Set`，用来记录该 Region 对象的引用对象所在的 Region。通过使用 Remembered Set，在做可达性分析的时候就可以避免全堆扫描。

![image](https://pdai.tech/images/pics/f99ee771-c56f-47fb-9148-c0036695b5fe.jpg)

如果不计算维护 Remembered Set 的操作，G1 收集器的运作大致可划分为以下几个步骤:

- 初始标记
- 并发标记
- 最终标记: 为了修正在并发标记期间因用户程序继续运作而导致标记产生变动的那一部分标记记录，虚拟机将这段时间对象变化记录在线程的 `Remembered Set Logs` 里面，最终标记阶段需要把 Remembered Set Logs 的数据合并到 Remembered Set 中。这阶段需要停顿线程，但是可并行执行。
- 筛选回收: 首先对各个 Region 中的回收价值和成本进行排序，根据用户所期望的 GC 停顿时间来制定回收计划。此阶段其实也可以做到与用户程序一起并发执行，但是因为只回收一部分 Region，时间是用户可控制的，而且停顿用户线程将大幅度提高收集效率。

具备如下特点:

- 空间整合: 整体来看是基于“标记 - 整理”算法实现的收集器，从局部(两个 Region 之间)上来看是基于“复制”算法实现的，这意味着运行期间不会产生内存空间碎片。
- 可预测的停顿: 能让使用者明确指定在一个长度为 M 毫秒的时间片段内，消耗在 GC 上的时间不得超过 N 毫秒。



8. **ZGC 收集器**

`ZGC（Z Garbage Collector）`是一种由Oracle公司开发的低延迟、可扩展、并发的垃圾回收器。它可以在几毫秒内完成垃圾回收操作，并且可以扩展到非常大的堆大小，适用于需要更快响应时间和更大堆大小的服务器应用程序。

ZGC的主要特点如下：

1. 低延迟：ZGC的最大特点是可以在非常短的时间内完成垃圾回收操作，最大的停顿时间通常不超过10ms，即使在非常大的堆大小下也可以保持较低的停顿时间。
2. 可扩展：ZGC可以支持非常大的堆大小，最大可以支持数TB的堆大小。它也支持动态调整堆大小和线程数量，以适应不同的应用程序需求。
3. 并发：ZGC可以在多个CPU核心上并发执行垃圾回收操作，并且可以与应用程序并发执行。这意味着在执行垃圾回收时不会暂停所有的用户线程，因此可以保证应用程序的响应时间。
4. 可靠性：ZGC具有强大的内存管理能力，可以有效地处理各种内存管理问题，例如内存泄漏、堆内碎片等。



ZGC采用了以下几种技术来实现低延迟和高吞吐量的垃圾回收：

1. **并发压缩**：ZGC采用了一种名为“并发压缩”的技术来执行垃圾回收操作。它会将垃圾对象标记为“死亡”状态，并将它们从堆中删除。同时，它会在后台执行压缩操作，将存活对象移动到连续的内存区域，从而减少堆内碎片。
2. **内存染色**：ZGC采用了一种名为“内存染色”的技术来识别可回收的内存区域。它会使用一组内存标记来标记每个内存区域的状态，从而能够快速地识别可回收的内存区域。
3. **并发执行**：ZGC采用了一种名为“并发执行”的技术来在多个CPU核心上并发执行垃圾回收操作。这样可以保证在执行垃圾回收时不会暂停所有的用户线程，从而能够提高应用程序的响应时间。



### 详细介绍一下CMS收集器？

CMS全称叫`Concurrent Mark Sweep`，从名字就能看出来这是一个并发收集并且采用`标记-清除`算法的垃圾收集器。一种以获取最短回收停顿时间为目标的**老年代收集器**

CMS 收集器的内存回收过程是与用户线程一起并发执行的，可以搭配 `ParNew 收集器`（多线程，新生代，复制算法）与 `Serial Old 收集器`（单线程，老年代，标记-整理算法）使用。

CMS的工作流程分为：`初始标记`、`并发标记`、`重新标记`、`并发清除`四个阶段

![](https://pdai.tech/images/pics/62e77997-6957-4b68-8d12-bfd609bb2c68.jpg)

#### **初始标记(有STW)**

只标记` GC Roots` ，所以速度很快，但是仍存在 `Stop The World `问题。

#### **并发标记**

进行 `GC Roots Tracing `的过程，也就是从`GC ROOT`开始进行可达性分析，找出并标记`存活对象`，此时是和用户线程并发执行的，不会`STW`。

#### **重新标记(有STW)**

因为并发标记是和用户线程并发执行的，在并发标记的过程中用户线程有可能又会新产生垃圾，或者已经被标记为垃圾的对象又重新被引用。所以需要修正并发标记期间因用户程序继续运行而导致标记产生变动的那一部分对象的标记记录，此过程存在STW

需要注意的是，重新标记过程中不会从`GC ROOT`开始去重新扫描，而是将并发标记过程中用户线程新产生的垃圾进行标记。

#### **并发清除**

此阶段对`垃圾对象`进行清除回收。由于此时是和用户线程并发运行，所以在清除的过程中，可能仍然会有新的垃圾产生，这些垃圾就叫`[浮动垃圾]`。针对于这种情况，在并发清理时产生的垃圾，会放到下一轮垃圾回收的时候被回收。

如果当用户需要存入一个很大的对象时，新生代放不下去，老年代由于浮动垃圾过多，就会退化为 Serial Old 收集器，将老年代垃圾进行标记-整理，当然这也是很耗费时间的！



#### CMS收集器的缺点

1. **内存碎片问题**：由于 CMS 采用的是标记-清除算法，这意味着在垃圾回收过程中会产生大量的内存碎片，从而使得堆内存变得不连续，可能会导致频繁的Full GC操作。当剩余内存不能满足程序运行要求时，系统将会出现 `Concurrent Mode Failure`， CMS 会退回 Serial Old 回收器进行垃圾清除，此时的性能将会被降低。
2. **remark阶段停顿时间会很长的问题**：CMS回收垃圾的四个阶段中，最花费时间的一个阶段是重新标记阶段，而且此过程需要停止应用程序线程，如果堆大小非常大，这个停顿时间可能会非常长。
3. **浮动垃圾**：在并发清除的过程中，在标记阶段有些对象被GC ROOT引用着，但是在并发清除的时候由于GC线程和用户线程是并发执行的，所以这些对象没有被GC ROOT引用了。那么这些对象就变成了垃圾（`也就是浮动垃圾`）。由于在标记阶段这些对象已经被认为不是垃圾了。所以垃圾收集器在这次无法清理这些对象。这些垃圾会在下一个周期被回收；
4. **concurrent mode failure**：执行CMS GC的过程中，同时业务线程也在运行，当年轻带空间满了，执行`YGC`时，需要将存活的对象放入到老年代，而此时老年代空间不足，这时CMS还没有机会回收老年带产生的，或者在做Minor GC的时候，新生代救助空间放不下，需要放入老年代，而老年代也放不下而产生的。
5. **promotion failed**：这个问题是指，在进行Minor GC时，Survivor空间不足，对象只能放入老年代，而此时老年代也放不下造成的，多数是由于老年代有足够的空闲空间，但是由于碎片较多，新生代要转移到老年带的对象比较大,找不到一段连续区域存放这个对象导致的



#### CMS收集器缺点解决

1. **内存碎片问题：**针对这个问题，我们需要用到这个参数：`-XX:CMSFullGCsBeforeCompaction=n` 意思是说在上一次CMS并发GC执行过后，到底还要再执行多少次`full GC`才会做压缩。默认是0，也就是在默认配置下每次CMS GC顶不住了而要转入full GC的时候都会做压缩。

2. **concurrent mode failure：**解决这个问题其实很简单，只需要设置两个参数即可

   `-XX:+UseCMSInitiatingOccupancyOnly -XX:CMSInitiatingOccupancyFraction=60`：是指设定CMS在对内存占用率达到60%的时候开始GC。

3. **remark阶段停顿时间会很长的问题**：解决只需要加入`-XX:+CMSScavengeBeforeRemark`。在执行remark操作之前先做一次`Young GC`，目的在于减少年轻代对老年代的无效引用，降低remark时的开销。



### 详细介绍一下G1收集器？

`G1(Garbage-First)`是一个分代的，增量的，并行与并发的`标记-复制`垃圾回收器。它的设计目标是为了适应现在不断扩大的内存和不断增加的处理器数量，进一步降低暂停时间（pause time），同时兼顾良好的吞吐量。G1回收器和CMS比起来，有以下不同：

- G1垃圾回收器是**compacting**的，因此其回收得到的空间是连续的。这避免了CMS回收器因为不连续空间所造成的问题。如需要更大的堆空间，更多的floating garbage。连续空间意味着G1垃圾回收器可以不必采用空闲链表的内存分配方式，而可以直接采用bump-the-pointer的方式；
- G1回收器的内存与CMS回收器要求的内存模型有极大的不同。G1将内存划分一个个固定大小的region，每个region可以是年轻代、老年代的一个。**内存的回收是以region作为基本单位的**；
- G1还有一个及其重要的特性：**软实时**（soft real-time）。所谓的实时垃圾回收，是指在要求的时间内完成垃圾回收。“软实时”则是指，用户可以指定垃圾回收时间的限时，G1会努力在这个时限内完成垃圾回收，但是G1并不担保每次都能在这个时限内完成垃圾回收。通过设定一个合理的目标，可以让达到90%以上的垃圾回收时间都在这个时限内。



#### G1内存模型

##### **分区Region**

G1将堆空间分为一个一个方形区域（Region）。可通过参数`-XX:G1HeapRegionSize`设置单个Region的大小，取值范围`1M-32M`，2的n次幂。默认为堆内存/2048，即一共`2048`个Region。

一段连续的空间，根据需要可扮演不同角色，可以是年轻代的伊甸区（E）、幸存区（S），也可以是老年代（O）。
除了E、S、O，还有一种代号为H的Region，是以往算法里没有的概念。H代表Humongous，即大对象。当一个对象的大小超过了Region的一半，则视为大对象，如果超过了整个Region的大小，将使用连续N个H Region来存放。虽说Humongous跳出了年轻代和老年代的概念，但G1的大多数行为都把它视为老年代的一部分。

![](http://image.easyblog.top/1680414845636c8d37673-4acd-45b6-a55e-e0b6f8c43908.png)



##### **卡片Card**

![img](https://pdai.tech/images/java/java-jvm-gc-g1-1.jpeg)

在**每个分区内部又被分成了若干个大小为512 Byte卡片(Card)，这是分配堆内存最小可用粒度**，所有分区的卡片记录在全局卡片表(`Global Card Table`)中，分配的对象会占用物理上连续的若干个卡片，当查找对分区内对象的引用时便可通过记录卡片来查找该引用对象(见RSet)。每次对内存的回收，都是对指定分区的卡片进行处理。



##### **Remember Set(RSet)**

 在串行和并行收集器中，GC时是通过整堆扫描来确定对象是否处于可达路径中。然而G1为了避免STW式的整堆扫描，为每个分区各自分配了一个 `RSet（Remembered Set）`，它内部类似于一个反向指针，记录了其它 Region 对当前 Region 的引用情况，这样就带来一个极大的好处：**回收某个Region时，不需要执行全堆扫描，只需扫描它的 RSet 就可以找到外部引用，来确定引用本分区内的对象是否存活，进而确定本分区内的对象存活情况**，而这些引用就是 initial mark 的根之一。



##### Collection Set(CSet)

![img](https://pdai.tech/images/java/java-jvm-gc-g1-4.jpeg)

收集集合(CSet)代表每次GC暂停时回收的一系列目标分区。在任意一次收集暂停中，CSet所有分区都会被释放，内部存活的对象都会被转移到分配的空闲分区中。因此无论是年轻代收集，还是混合收集，工作的机制都是一致的。年轻代收集CSet只容纳年轻代分区，而混合收集会通过**启发式算法**，在老年代候选回收分区中，筛选出回收收益最高的分区添加到CSet中。

候选老年代分区的CSet准入条件，可以通过活跃度阈值`-XX:G1MixedGCLiveThresholdPercent`(默认85%)进行设置，从而拦截那些回收开销巨大的对象；同时，每次混合收集可以包含候选老年代分区，可根据CSet对堆的总大小占比`-XX:G1OldCSetRegionThresholdPercent`(默认10%)设置数量上限。

由上述可知，G1的收集都是根据CSet进行操作的，年轻代收集与混合收集没有明显的不同，最大的区别在于两种收集的触发条件。



##### 三色标记法

CMS和G1在并发标记时使用的是同一个算法：三色标记法，三色标记算法是并发收集阶段的重要算法，它是描述追踪式回收器的一种有用的方法，利用它可以推演回收器的正确性。 首先，我们将对象分成三种类型的：

- `黑色`：根对象，或者该对象与它的子对象都被扫描了
- `灰色`：对象本身被扫描，但还没扫描完该对象中的子对象
- `白色`：未被扫描对象，扫描完成所有对象之后，最终为白色的为不可达对象，即垃圾对象



###### 三色标记算法的流程

（1）根对象被置为黑色，子对象被置为灰色

![img](https://img-blog.csdnimg.cn/407e09d645cf4c8e88e8f1bfe99caf35.png)



（2）继续由灰色遍历，将已扫描了子对象的对象置为黑色。

![img](https://img-blog.csdnimg.cn/6947dea02172413da0fcf4bbcb492902.png)

（3）遍历了所有可达的对象后，所有可达的对象都变成了黑色。不可达的对象即为白色，需要被清理。

![img](https://img-blog.csdnimg.cn/a6dac8a9a04748ba961e4aa886299a2f.png)

###### 三色标记算法的漏标问题

在标记阶段，应用线程也在执行，那么对象的指针就有可能改变。这样的话，我们就会遇到一个问题：对象丢失问题。我们看下面一种情况，当垃圾收集器扫描到下面情况时：

![img](https://img-blog.csdnimg.cn/8f2bd57ef29a4827ac70c085ef4e94ed.png)

这时候应用程序执行了以下操作：

```java
A.c=C
B.c=null
```

这样，对象的状态图变成如下情形：

![img](https://img-blog.csdnimg.cn/3bcf6a14bba24bf688bd37bd8000d63d.png)

此时C是白色，被认为是垃圾需要清理掉，显然这是不合理的。CMS 和 G1 都存在这样的问题，而 CMS 和 G1 的两种对这个问题页有两种不同实现方式：

* （1）CMS采用的是**增量更新**（`incremental update`）：关注引用的增加，只要在写屏障里发现要有一个白对象的引用被赋值到一个黑对象的字段里，那就把这个白对象变成灰色的，即插入的时候记录下来。
* （2）G1 使用的是 **STAB**（`snapshot-at-the-beginning`）的方式：关注引用的删除，当灰–>白引用消失时，要把这个引用推到GC的堆栈，保证白还能被GC扫描到。



> **为什么G1采用SATB而不用incremental update**？
>
> 因为采用incremental update把黑色重新标记为灰色后，之前扫描过的还要再扫描一遍，效率太低。G1有RSet与SATB相配合。Card Table里记录了RSet，RSet里记录了其他对象指向自己的引用，这样就不需要再扫描其他区域，只要扫描RSet就可以了。
>
> 也就是说 灰色–>白色 引用消失时，如果没有 黑色–>白色，引用会被push到堆栈，下次扫描时拿到这个引用，由于有RSet的存在，不需要扫描整个堆去查找指向白色的引用，效率比较高。SATB配合RSet浑然天成。





#### GC模式

G1的GC模式有`Young GC`、`Mixed GC`、`Full GC`。

##### Young GC

除了大对象在H Region分配空间，一般对象都在E Region分配。当所有E Region被耗尽，触发一次Young GC，执行完后，活跃对象会被复制到S Region或晋升到O Region，空闲的E Region加入空闲列表。
![](http://image.easyblog.top/16804205350906875c5aa-344f-4319-8598-e5c226fe7c36.png)

##### Mixed GC

![](http://image.easyblog.top/1680420668877f6d6fbb1-2e6f-40a3-ba1e-2c424164b75b.png)

G1可以回收堆空间的任何部分，组成回收集（Collection Set、CSet）来回收，衡量标准不再是属于哪个分代，而是哪块内存中垃圾数量最多，回收收益最大，这就是Mixed GC。
Mixed GC回收所有E Region、S Region和部分的O Region及H Region。
可通过参数`-XX:InitiatingHeapOccupancyPercent`设置阈值，当O Region占整个堆空间百分比达到此阈值，触发一次Mixed GC，包括以下几步：

![image](https://pdai.tech/images/pics/f99ee771-c56f-47fb-9148-c0036695b5fe.jpg)

* `初始标记（Initial Mark）`：标记出所有 GC Roots 节点以及直接可达的对象，这一阶段需STW，但是耗时很短。
* `并发标记（Concurrent Mark）`：采用三色标记法，标记整个堆空间的可访问的对象。此阶段不会STW，与应程序同时运行。此阶段时间长，约占整个Mixed GC的80%
* `最终标记（Remark）`：重新标记阶段是为了修正在并发标记期间，因应用程序继续运作而导致标记产生变动的那一部分标记记录，就是去处理剩下的 SATB日志缓冲区和所有更新，找出所有未被访问的存活对象。
* `垃圾清除（Cleanup）`：该阶段主要是排序各个 Region 的回收价值和成本，并根据用户所期望的GC停顿时间来制定回收计划。此阶段会STW。需要注意的是这个阶段并不会实际去做垃圾的回收，也不会执行存活对象的拷贝，清除阶段执行的详细操作有以下几点：
  * **RSet梳理**：启发式算法会根据活跃度和RSet尺寸对分区定义不同等级，同时RSet梳理也有助于发现无用的引用。
  * **整理堆分区**：为混合收集识别回收收益高（基于释放空间和暂停目标）的老年代分区集合；
  * **识别所有空闲分区**：即发现无存活对象的分区，该分区可在清除阶段直接回收，无需等待下次收集周期。



##### Full GC

当 G1 无法在堆空间中申请新的分区时，G1便会触发**担保机制**，执行一次STW式的、单线程的 Full GC，Full GC会对整堆做**标记清除和压缩**，最后将只包含纯粹的存活对象。

使用参数`-XX:G1ReservePercent`可以设置保留空间来应对晋升模式下的异常情况，默认值10%，最大占用整堆50%，更大也无意义。

G1在以下场景中会触发 Full GC，同时会在日志中记录to-space-exhausted以及Evacuation Failure：

* （1）从年轻代分区拷贝存活对象时，无法找到可用的空闲分区
* （2）从老年代分区转移存活对象时，无法找到可用的空闲分区
* （3）分配巨型对象时在老年代无法找到足够的连续分区





##### **G1 垃圾回收流程小结**

**Young CG 和 Mixed GC，是G1回收空间的主要活动。当应用开始运行时，堆内存可用空间还比较大，只会在年轻代满时，触发年轻代收集；随着老年代内存增长，当到达 IHOP 阈值 -XX:InitiatingHeapOccupancyPercent(老年代占整堆比，默认45%) 时，G1开始着手准备收集老年代空间。首先经历并发标记周期，识别出高收益的老年代分区。但随后G1并不会马上开启一次混合收集，而是让应用线程先运行一段时间，等待触发一次年轻代收集，在这次STW中，G1将开始整理混合收集周期。接着再次让应用线程运行，当接下来的几次年轻代收集时，将会有老年代分区加入到CSet中，即触发混合收集，这些连续多次的混合收集称为混合收集**。

**当MixedGC执行之后还是无法申请到新的分区时，触发G1的担保机制，G1执行一次STW式的、单线程的 Full GC，Full GC会对整堆做标记清除和压缩，最后将只包含纯粹的存活对象。由于G1的应用场景往往堆内存都比较大，所以Full GC的收集代价非常昂贵，应该避免Full GC的发生**。



#### G1 收集器的缺点

* （1）如果停顿时间过短的话，可能导致每次选出的回收集只占堆内存很小一部分，收集器收集的速度逐渐跟不上分配器的分配速度，进而导致垃圾慢慢堆积，最终造成堆空间占满，引发Full GC 反而降低性能。
* （2）G1 无论是在垃圾收集产生的内存占用还是程序运行时的额外执行负载都要比CMS要高
* （3）CMS在小内存应用上大概率由于 G1。所以小内存的情况下使用CMS收集器，大内存的情况下可以使用G1收集器（G1收集器6GB以上）





### 详细介绍一下ZGC收集器？

ZGC 收集器是一款`基于 Region 内存布局`的，(暂时) 不设分代的，使用了`读屏障`、`染色指针`和`内存多重映射`等技术来实现可并发的`标记-整理`算法的，以低延迟为首要目标的一款垃圾收集器。



#### ZGC内部布局

与 Shenandoah 和 G1一样，ZGC 也采用基于 Region 的堆内存布局，但与它们不同的是 ， ZGC 的 Region 具 有 动 态 性 (动态创建和销毁 ， 以及动态的区域容量大小)。在 x64硬件平台下 ， ZGC 的 Region 可以具有大、中、小三类容量(如下图所示):

- **小型 Region (Small Region )：**容量固定为 2M， 存放小于 256K 的对象。
- **中型 Region (Medium Region)：**容量固定为 32M，放置大于等于256K但小于4M的对象。
- **大型 Region (Large Region):** 容量不固定，可以动态变化，但必须为2MB 的整数倍，用于放置 4MB或以上的大对象。

![](https://image.easyblog.top/%E6%88%AA%E5%B1%8F2023-04-09%20%E4%B8%8B%E5%8D%882.27.59.png)



#### NUMA-aware

和NUMA有个对应的概念叫UMA，NUMA是在UMA上面改进的，ZGC能自动感知NUMA架构并充分利用NUMA架构特性的，下面具体描述：

* `UMA`：全称`Uniform Memory Access Architecture`(统一内存访问体系结构)，看到下图我们可以看出，UMA是多核CPU同时访问一块内存，这样的话就有内存的竞争，就会有锁，对应的就会有效率的降低，当CPU的核数越多，竞争就越激烈。
* `NUMA`：全称`Non Uniform Memory AccessArchitecture`(非统一内存访问体系结构)，如图所示，它会将内存划分为相等大小的区域，每一块CPU会优先访问某一块内存，这样冲突就降低了很多，当自己优先的这块内存已被用完，它可以去访问其它内存块。

![](http://image.easyblog.top/1681022471270dd2eef93-58ba-43c3-9763-87d564b085f4.png)



#### 染色指针(Colored Pointer)

染色指针，ZGC 的核心设计之一。以前的垃圾收集器的 GC 信息都保存在对象头中，而 ZGC 的 GC 信息保存在指针中(直接把标记信息记录在对象的引用指针上)。

![](http://image.easyblog.top/1681022713837b430f8e3-7dde-47c1-9a34-26a033a991ff.png)

每个对象有一个64位指针，这64位被分为：

- 18位：预留给以后使用。
- 1位：Finalizable标识，此位与并发引用处理有关，它表示这个对象只能通过finalizer才能访问(finalizer:object基类的一个空方法，如果被重写则会在GC之前调用该方法，该方法会且只会被调用一次)。
- 1位：Remapped 标识，设置此位的值后，对象未指向relocation set中(relocation set表示需要GC的Region集合)，有值就说明对象被gc复制过。
- 1位：Marked1标识。
- 1位：Marked0标识，和上面的Marked1都是标记对象用于辅助GC。
- 42位：对象的地址(所以它可以支持2^42=4T内存）



#### 读屏脏

由于ZGC采用的是复制整理的方式进行GC，很有可能在对象的位置改变之后指针位置尚未更新时程序调用了该对象，那么此时在程序需要并行的获取该对象的引用时，ZGC就会对该对象的指针进行读取，判断Remapped标识，如果标识为该对象位于本次需要清理的region区中，该对象则会有内存地址变化，会在指针中将新的引用地址替换原有对象的引用地址，然后再进行返回。 如此，使用读屏障便解决了并发GC的对象读取问题。





#### ZGC的工作过程？

![](http://image.easyblog.top/168102335122813c7ae75-5a0c-464a-b755-99549505b45a.png)

ZGC工作过程整体可以分为四个阶段：

* `并发标记（Concurrent Mark）`：与G1、Shenandoah一样，**并发标记是遍历对象图做可达性分析的阶段**，前后也要经过类似于G1、Shenandoah的初始标记、最终标记(尽管ZGC中的名字不叫这些)的短暂停顿，而且这些停顿阶段所做的事情在目标上也是相类似的。与G1、Shenandoah不同的是，ZGC的标记是在指针上而不是在对象上进行的，标记阶段会更新染色指针中的Marked 0、Marked 1标志位。

* `并发预备重分配（Concurrent Prepare for Relocate）`：**这个阶段需要根据特定的查询条件统计得出本次收集过程要清理哪些Region，将这些Region组成重分配集(Relocation Set)**。重分配集与G1收集器的回收集(Collection Set)还是有区别的，ZGC划分Region的目的并非为了像G1那样做收益优先的增量回收。相反，ZGC每次回收都会扫描所有的Region，用范围更大的扫描成本换取省去G1中记忆集的维护成本。因此，ZGC的重分配集只是决定了里面的存活对象会被重新复制到其他的Region中，里面 的Region会被释放，而并不能说回收行为就只是针对这个集合里面的Region进行，因为标记过程是针对全堆的。此外，在JDK 12的ZGC中开始支持的类卸载以及弱引用的处理，也是在这个阶段中完成的。

* `并发重分配（Concurrent Relocate）`：重分配是ZGC执行过程中的核心阶段，**这个过程要把重分配集中的存活对象复制到新的Region上，并为重分配集中的每个Region维护一个`转发表(Forward Table)`，记录从旧对象到新对象的转向关系**。得益于染色指针的支持，ZGC收集器能仅从引用上就明确得知一个对象是否处于重分配集之中，如果用户线程此时并发访问了位于重分配集中的对象，这次访问将会被预置的内存屏障所截获，然后立即根据Region上的转发表记录将访问转发到新复制的对象上，并同时修正更新该引用的值，使其直接指向新对象，ZGC将这种行为称为指针的`“自愈”(Self-Healing)能力`。
* `并发重映射（Concurrent Remap）`：**重映射所做的就是修正整个堆中指向重分配集中旧对象的所有引用**，这一点从目标角度看是与 Shenandoah 并发引用更新阶段一样的，但是 ZGC 的并发重映射并不是一个必须要“迫切”去完成的任务，因为前面说过，即使是旧引用，它也是可以自愈的，最多只是第一次使用时多一次转发和修正操作。重映射清理这些旧引用的主要目的是为了不变慢(还有清理结束后可以释放转发表这样的附带收益)，所以说这并不是很“迫切”。因此，ZGC 很巧妙地把并发重映射阶段要做的工作，合并到了下一次垃圾收集循环中的并发标记阶段里去完成，反正它们都是要遍历所有对象的，这样合并就节省了一次遍历对象的开销。一旦所有指针都被修正之后，原来记录新旧对象关系的转发表就可以释放掉了。

## 5.4 JVM调优和问题排查







# 六、Spring

## 6.1 Spring IOC

### 谈谈对于 Spring IoC 的了解？

`IOC（Inversion Of Controll，控制反转）`是一种设计思想，就是将原本在程序中手动创建对象的控制权，交给IOC容器来管理，并由IOC容器完成对象的注入。这样可以很大程度上简化应用的开发，把应用从复杂的依赖关系中解放出来。IOC容器就像是一个工厂一样，当我们需要创建一个对象的时候，只需要配置好配置文件/注解即可，完全不用考虑对象是如何被创建出来的。





### 什么是Spring Bean?

简单来说，Bean 代指的就是那些被 IoC 容器所管理的对象。

我们需要告诉 IoC 容器帮助我们管理哪些对象，这个是通过配置元数据来定义的。配置元数据可以是 XML 文件、注解或者 Java 配置类。



### 将一个类声明为 Bean 的注解有哪些?

- `@Component` ：通用的注解，可标注任意类为 `Spring` 组件。如果一个 Bean 不知道属于哪个层，可以使用`@Component` 注解标注。
- `@Repository` : 对应持久层即 Dao 层，主要用于数据库相关操作。
- `@Service` : 对应服务层，主要涉及一些复杂的逻辑，需要用到 Dao 层。
- `@Controller` : 对应 Spring MVC 控制层，主要用于接受用户请求并调用 `Service` 层返回数据给前端页面。



### @Component 和 @Bean 的区别是什么？

- `@Component` 注解作用于类，而`@Bean`注解作用于方法。
- `@Component`通常是通过类路径扫描来自动侦测以及自动装配到 Spring 容器中（我们可以使用 `@ComponentScan` 注解定义要扫描的路径从中找出标识了需要装配的类自动装配到 Spring 的 bean 容器中）。`@Bean` 注解通常是我们在标有该注解的方法中定义产生这个 bean,`@Bean`告诉了 Spring 这是某个类的实例，当我需要用它的时候还给我。
- `@Bean` 注解比 `@Component` 注解的自定义性更强，而且很多地方我们只能通过 `@Bean` 注解来注册 bean。比如当我们引用第三方库中的类需要装配到 `Spring`容器时，则只能通过 `@Bean`来实现。



### 注入 Bean 的注解有哪些？

Spring 内置的 `@Autowired` 以及 JDK 内置的 `@Resource` 和 `@Inject` 都可以用于注入 Bean。

| Annotaion    | Package                            | Source       |
| ------------ | ---------------------------------- | ------------ |
| `@Autowired` | `org.springframework.bean.factory` | Spring 2.5+  |
| `@Resource`  | `javax.annotation`                 | Java JSR-250 |
| `@Inject`    | `javax.inject`                     | Java JSR-330 |

`@Autowired` 和`@Resource`使用的比较多一些。



### @Autowired 和 @Resource 注解的区别

- `@Autowired` 是 Spring 提供的注解，`@Resource` 是 JDK 提供的注解。
- `Autowired` 默认的注入方式为`byType`（根据类型进行匹配），`@Resource`默认注入方式为 `byName`（根据名称进行匹配）。
- 当一个接口存在多个实现类的情况下，`@Autowired` 和`@Resource`都需要通过名称才能正确匹配到对应的 Bean。`Autowired` 可以通过 `@Qualifier` 注解来显式指定名称，`@Resource`可以通过 `name` 属性来显式指定名称。



### Bean 的作用域有哪些?

Spring 中 Bean 的作用域通常有下面几种：

- **`singleton`** : IoC 容器中只有唯一的 bean 实例。Spring 中的 bean 默认都是单例的，是对单例设计模式的应用。
- **`prototype`** : 每次获取都会创建一个新的 bean 实例。也就是说，连续 `getBean()` 两次，得到的是不同的 Bean 实例。
- **`request`** （仅 Web 应用可用）: 每一次 HTTP 请求都会产生一个新的 bean（请求 bean），该 bean 仅在当前 HTTP request 内有效。
- **`session`** （仅 Web 应用可用） : 每一次来自新 session 的 HTTP 请求都会产生一个新的 bean（会话 bean），该 bean 仅在当前 HTTP session 内有效。
- **`application/global-session`** （仅 Web 应用可用）： 每个 Web 应用在启动时创建一个 Bean（应用 Bean），该 bean 仅在当前应用启动时间内有效。
- **`websocket`** （仅 Web 应用可用）：每一次 WebSocket 会话产生一个新的 bean。



**如何配置Bean的作用域呢？**

```java
@Bean
@Scope(value = ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public Person personPrototype() {
    return new Person();
}
```



### 单例 Bean 的线程安全问题？

对于单例Bean，所有线程都共享一个单例实例Bean，因此是存在资源的竞争。

如果单例Bean 是一个无状态Bean，也就是线程中的操作不会对Bean的成员执行**「查询」**以外的操作，那么这个单例Bean是线程安全的。比如Spring mvc 的 Controller、Service、Dao等，这些Bean大多是无状态的，只关注于方法本身。

对于Spring 单例 Bean的下车安全，常见的有两种解决办法：

1. 在 Bean 中尽量避免定义可变的成员变量。
2. 在类中定义一个 `ThreadLocal` 成员变量，将需要的可变成员变量保存在 `ThreadLocal` 中（推荐的一种方式）。



> **spring单例，为什么controller、service和dao确能保证线程安全？**
>
> Controller、Service、DAO本身并不是线程安全，如果只是调用里面的方法，而且多线程调用一个实例的方法，会在内存中复制变量，这是自己的线程的工作内存，是安全的。（这一点参考JVM栈的共工作原理）



### Spring的Bean生命周期?

Spring的Bean生命周期概括起来可以分为4大阶段：

1. **实例化（Instantiation）**
2. **属性赋值（Populate）**
3. **初始化（Initialization）**
4. **销毁（Destruction）**

![Spring Bean 生命周期](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/2/15/1704860a4de235aa~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.awebp)

#### 1.  **实例化**

通过反射API创建Bean实例，此阶段最终获得到的是一个不完整的Bean，因为Bean的属性均未设置值；



#### 2.**属性赋值**

调用`set()`方法设置Bean属性值



#### 3.**初始化**

这一阶段步骤较多，不过也主要分为以下5个步骤：

##### 3.1 检查Aware相关接口并设置相关依赖

* 如果 Bean 实现了 `BeanNameAware` 接口，调用 `setBeanName()`方法，传入 Bean 的名字。
* 如果 Bean 实现了 `BeanClassLoaderAware` 接口，调用 `setBeanClassLoader()`方法，传入 `ClassLoader`对象的实例。
* 如果 Bean 实现了 `BeanFactoryAware` 接口，调用 `setBeanFactory()`方法，传入 `BeanFactory`对象的实例。

##### 3.2 BeanPostProcessor进行前置处理

如果有和加载这个 Bean 的 Spring 容器相关的 `BeanPostProcessor` 对象，执行`postProcessBeforeInitialization()` 方法



##### 3.3 InitializingBean初始化

如果 Bean 实现了`InitializingBean`接口，执行`afterPropertiesSet()`方法。



##### 3.4 自定义init-method初始化

如果 Bean 在配置文件中的定义包含 init-method 属性，执行指定的方法。



##### 3.5 BeanPostProcessor进行后置处理

如果有和加载这个 Bean 的 Spring 容器相关的 `BeanPostProcessor` 对象，执行`postProcessAfterInitialization()` 方法



#### 4.销毁

* 当要销毁 Bean 的时候，如果 Bean 实现了 `DisposableBean` 接口，执行 `destroy()` 方法。
* 当要销毁 Bean 的时候，如果 Bean 在配置文件中的定义包含 `destroy-method` 属性，执行指定的方法。





结合代码来直观的看下，在 `doCreateBean()` 方法中能看到依次执行了这 4 个阶段：

```java
// AbstractAutowireCapableBeanFactory.java
protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final @Nullable Object[] args)
    throws BeanCreationException {

    // 1. 实例化
    BeanWrapper instanceWrapper = null;
    if (instanceWrapper == null) {
        instanceWrapper = createBeanInstance(beanName, mbd, args);
    }
    
    Object exposedObject = bean;
    try {
        // 2. 属性赋值
        populateBean(beanName, mbd, instanceWrapper);
        // 3. 初始化
        exposedObject = initializeBean(beanName, exposedObject, mbd);
    }

    // 4. 销毁-注册回调接口
    try {
        registerDisposableBeanIfNecessary(beanName, bean, mbd);
    }

    return exposedObject;
}
```



重点关注初始化阶段的流程，在`initializeBean`方法中我们看到如过程（注释的序号对应图中序号）：

```java
// AbstractAutowireCapableBeanFactory.java
protected Object initializeBean(final String beanName, final Object bean, @Nullable RootBeanDefinition mbd) {
  // 3. 检查 Aware 相关接口并设置相关依赖
		if (System.getSecurityManager() != null) {
			AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
				invokeAwareMethods(beanName, bean);
				return null;
			}, getAccessControlContext());
		}
		else {
			invokeAwareMethods(beanName, bean);
		}

		Object wrappedBean = bean;
		if (mbd == null || !mbd.isSynthetic()) {
      // 4. BeanPostProcessor前置处理
			wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
		}

		try {
      // 5. 若实现 InitializingBean 接口，调用 afterPropertiesSet() 方法
      // 6. 若配置自定义的 init-method方法，则执行
			invokeInitMethods(beanName, wrappedBean, mbd);
		}
		catch (Throwable ex) {
			throw new BeanCreationException(
					(mbd != null ? mbd.getResourceDescription() : null),
					beanName, "Invocation of init method failed", ex);
		}
		if (mbd == null || !mbd.isSynthetic()) {
      // 7. BeanPostProcessor后置处理
			wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
		}

		return wrappedBean;
	}
```

在 `invokInitMethods() `方法中会检查 InitializingBean 接口和 init-method 方法

```java
// AbstractAutowireCapableBeanFactory.java
protected void invokeInitMethods(String beanName, final Object bean, @Nullable RootBeanDefinition mbd)
			throws Throwable {
   
    //检查是否实现了InitializingBean接口，如果是，则调用afterPropertiesSet方法进行初始化
		boolean isInitializingBean = (bean instanceof InitializingBean);
		if (isInitializingBean && (mbd == null || !mbd.isExternallyManagedInitMethod("afterPropertiesSet"))) {
			if (logger.isTraceEnabled()) {
				logger.trace("Invoking afterPropertiesSet() on bean with name '" + beanName + "'");
			}
			if (System.getSecurityManager() != null) {
				try {
					AccessController.doPrivileged((PrivilegedExceptionAction<Object>) () -> {
						((InitializingBean) bean).afterPropertiesSet();
						return null;
					}, getAccessControlContext());
				}
				catch (PrivilegedActionException pae) {
					throw pae.getException();
				}
			}
			else {
				((InitializingBean) bean).afterPropertiesSet();
			}
		}

    //检查是否实现了自定义初始化方法
		if (mbd != null && bean.getClass() != NullBean.class) {
			String initMethodName = mbd.getInitMethodName();
			if (StringUtils.hasLength(initMethodName) &&
					!(isInitializingBean && "afterPropertiesSet".equals(initMethodName)) &&
					!mbd.isExternallyManagedInitMethod(initMethodName)) {
				invokeCustomInitMethod(beanName, bean, mbd);
			}
		}
	}
```





### Spring的Bean生命周期各节点扩展点？

#### 1.Aware接口

若 Spring 检测到 bean 实现了 Aware 接口，则会为其注入相应的依赖。所以**通过让bean 实现 Aware 接口，则能在 bean 中获得相应的 Spring 容器资源**。Spring 中提供的 Aware 接口有：

1. `BeanNameAware`：注入当前 bean 对应 beanName；
2. `BeanClassLoaderAware`：注入加载当前 bean 的 ClassLoader；
3. `BeanFactoryAware`：注入 当前BeanFactory容器 的引用。

以上是针对 BeanFactory 类型的容器，而对于 ApplicationContext 类型的容器，也提供了 Aware 接口，只不过这些 Aware 接口的注入实现，是通过 BeanPostProcessor 的方式注入的，但其作用仍是注入依赖。

1. `EnvironmentAware`：注入 Enviroment，一般用于获取配置属性；
2. `EmbeddedValueResolverAware`：注入 EmbeddedValueResolver（Spring EL解析器），一般用于参数解析；
3. `ApplicationContextAware`（ResourceLoader、ApplicationEventPublisherAware、MessageSourceAware）：注入 ApplicationContext 容器本身。



#### 2.BeanPostProcessor

BeanPostProcessor 是 Spring 为**修改 bean**提供的强大扩展点，其可作用于容器中所有 bean，常用场景有：

1. 对于标记接口的实现类，进行自定义处理。例如1中所说的`ApplicationContextAwareProcessor`，为其注入相应依赖；再举个例子，自定义对实现解密接口的类，将对其属性进行解密处理；
2. 为当前对象提供代理实现。例如 Spring AOP 功能，生成对象的代理类，然后返回。



#### 3.InitializingBean 和 init-method

InitializingBean 和 init-method 是 Spring 为 **bean 初始化**提供的扩展点。

##### InitializingBean

InitializingBean接口 的定义如下：

```csharp
public interface InitializingBean {
	void afterPropertiesSet() throws Exception;
}
```

Bean对象需要继承InitializingBean接口，并重写 afterPropertiesSet() 方法写初始化逻辑。



##### init-method

指定 init-method 方法，指定初始化方法：

使用xml配置文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="demo" class="com.chaycao.Demo" init-method="init()"/>
    
</beans>
```

使用注解方式：

```java
@Bean(initMethod="init")
public Bean createBean(){
  //...
}
```



### 谈谈对循环依赖的理解？

循环依赖顾名思义就是对象之间互相引用了对方，比如如下示例：

```java
public class CircularDependencyTest {


    public static void main(String[] args) {
        CircularBeanA circularBeanA = new CircularBeanA();
        System.out.println(circularBeanA);
    }


}


class CircularBeanA {
    private CircularBeanB circularBeanB = new CircularBeanB();

}


class CircularBeanB {
    private CircularBeanA circularBeanA = new CircularBeanA();
}
```



上面代码中`CircularBeanA` 引用了 `CircularBeanB`，同时`CircularBeanB` 引用了 `CircularBeanA`，此时我们在`main`方法中实例化`CircularBeanA`，执行方法就会看到**栈溢出**报错

![](http://image.easyblog.top/16814419445271906890f-c752-49bc-886d-668b8e0cdbe0.png)



#### 为什么会报栈溢出呢？

先来分析为什么缓存能解决循环依赖。

上文分析得到，如下图所示，之所以产生循环依赖的问题，主要是：`A创建时--->需要B---->B去创建--->需要A`，从而产生了循环

<img src="http://image.easyblog.top/16814430696042a1f278c-39ac-4cb9-bb23-e552d0dfae4f.png" style="zoom:40%;" />

#### 解决循环依赖思路？

通过分析我们知道，造成循环依赖的问题就在于对象之间的依赖关系死循环了，因此在创建对象实例时会造成构造方法栈溢出，那么打破这里死循环的思路就是主动打破这里的死循环，即加缓存，专业名词叫**提前暴露**，它的具体思路如下图所示：

![](http://image.easyblog.top/16814500218987fa52c63-eda5-4f16-bbb1-5dd22fa6d5fe.png)

我们可以加入一个缓存，这个缓存中存放对象实例半成品（未设置属性值的对象），然后A对象创建，在进行依赖注入之前先把A放入到缓存中，放入缓存后，再进行依赖注入，此时A的Bean依赖了B的Bean，如果B的Bean不存在，则需要创建B的Bean，而创建B的Bean的过程和A一样，也是先创建一个B的原始对象，然后把B的原始对象提早暴露出来放入缓存中，然后在对B的原始对象进行依赖注入A，此时能从缓存中拿到A的原始对象（虽然是A的原始对象，还不是最终的Bean），B的原始对象依赖注入完了之后，B的生命周期结束，那么A的生命周期也能结束。

我们用这样的思路改善最初的代码，如下：

```java
public class CircularDependencyTest2 {

    // 保存实例化过程中的半成品对象
    private static final Map<String, Object> singletonObjets = new ConcurrentHashMap<>();

    public static void main(String[] args) throws InstantiationException, IllegalAccessException {
        System.out.println(getBean(CircularBeanA2.class));
        System.out.println(getBean(CircularBeanB2.class));
    }


    private static <T> T getBean(Class<T> clazz) throws InstantiationException, IllegalAccessException {
        // 获取bean name
        String beanName = clazz.getSimpleName().toLowerCase();
        // 缓存中存在就返回bean
        if (singletonObjets.containsKey(beanName)) {
            return (T) singletonObjets.get(beanName);
        }
        // 缓存中不存在就实例化bean，并将其放入缓存
        T obj = clazz.newInstance();
        singletonObjets.put(beanName, obj);
        // 获取对象属性
        Field[] fields = obj.getClass().getDeclaredFields();
        // 遍历属性设置值
        for (Field field : fields) {
            field.setAccessible(true);
            Class<?> fieldType = field.getType();
            field.set(obj, getBean(fieldType));
        }
        return obj;
    }

}

@Setter
@Getter
class CircularBeanA2 {
    private CircularBeanB2 circularBeanB;

}

@Setter
@Getter
class CircularBeanB2 {
    private CircularBeanA2 circularBeanA;
}
```



上面的代码中，我们不再使用构造器注入的方式实例化对象，而是使用设值注入，因为通过分析我们知道**构造器注入的方式造成的循环依赖是无解的，上面`getBean`方法就是以set值的方式来初始化对象的，此外我们还需要一个Bean缓存来存储提前暴露的半成品对象，如果缓存中存在需要的bean就直接返回，否则实例化对象并将其放入缓存中，然后获取其所有属性遍历并set值。**



### Spring到底解决了哪种情况下的循环依赖？

针对Spring中Bean对象的各种场景，支持的方式不一样：

![](http://image.easyblog.top/16814568250317f6ef999-6704-49c4-a118-0d8da8fa57da.png)

* 对于单例模式：
  * 如果是构造器注入，Spring 无法解决循环依赖，但是Spring会检测出循环依赖并报错，检测的原理就是通过对象提前暴露缓存实现的。
  * 如果是设值注入，Spring 会通过三级缓存提前暴露对象来解决循环依赖
* 对于原型模式：无论那种模式都无法直接通过Spring得到支持，不过设置注入可以使用`@Lazy`使用懒加载的方式实现解决循环依赖问题。







### Spring循环依赖三级缓存的问题？

#### Spring解决循环依赖为什么需要三级缓存？

从上面这个分析过程中可以得出，只需要一个缓存就能解决循环依赖了，那么为什么Spring中还需要三级缓存呢？

**思考这样一个场景**：如果A的原始对象注入给B的属性之后，A的原始对象进行了AOP产生了一个代理对象，此时就会出现，对于A而言，它的Bean对象其实应该是AOP之后的代理对象，而B的a属性对应的并不是AOP之后的代理对象，这就产生了冲突。造成冲突的本质根源是B依赖的A和最终的A不是同一个对象。

如何处理的，就是利用了第三级缓存**`singletonFactories`**。

首先，`singletonFactories`中存的是某个beanName对应的ObjectFactory，在bean的生命周期中，生成完原始对象之后，就会构造一个ObjectFactory存入`singletonFactories`中。这个ObjectFactory是一个函数式接口，所以支持Lambda表达式：**`() -> getEarlyBeanReference(beanName, mbd, bean)`**。

上面的Lambda表达式就是一个ObjectFactory，执行该Lambda表达式就会去执行getEarlyBeanReference方法，而该方法如下：

```java
protected Object getEarlyBeanReference(String beanName, RootBeanDefinition mbd, Object bean) {
	Object exposedObject = bean;
	if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
		for (BeanPostProcessor bp : getBeanPostProcessors()) {
			if (bp instanceof SmartInstantiationAwareBeanPostProcessor) {
				SmartInstantiationAwareBeanPostProcessor ibp = (SmartInstantiationAwareBeanPostProcessor) bp;
				exposedObject = ibp.getEarlyBeanReference(exposedObject, beanName);
			}
		}
	}
	return exposedObject;
}
```

该方法会去执行`SmartInstantiationAwareBeanPostProcessor`中的`getEarlyBeanReference`方法，而这个接口下的实现类中只有两个类实现了这个方法，一个是`AbstractAutoProxyCreator`，一个是`InstantiationAwareBeanPostProcessorAdapter`，它的实现如下：

```java
// InstantiationAwareBeanPostProcessorAdapter
@Override
public Object getEarlyBeanReference(Object bean, String beanName) throws BeansException {
	return bean;
}

// AbstractAutoProxyCreator
@Override
public Object getEarlyBeanReference(Object bean, String beanName) {
	Object cacheKey = getCacheKey(bean.getClass(), beanName);
	this.earlyProxyReferences.put(cacheKey, bean);
	return wrapIfNecessary(bean, beanName, cacheKey);
}
```

`getEarlyBeanReference()`主要作恶以下几件事：

* 首先得到一个cachekey，cachekey就是beanName。

* 然后把beanName和bean（这是原始对象）存入`earlyProxyReferences`中

* 调用wrapIfNecessary进行AOP，得到一个代理对象。



所以很明显，在整个Spring中，默认就只有AbstractAutoProxyCreator真正意义上实现了getEarlyBeanReference 方法，而该类就是用来进行AOP的。



#### getEarlyBeanReference的调用时机

![](http://image.easyblog.top/1681565309333dfd6f912-73d5-4801-8f20-2e72cc4aa9c9.png)

根据上面的分析，A原始对象在创建的时候会向Spring的第三级缓存`singletonFactories`中保存一个`ObjectFactory`，其实就是lambda表达式，需要注意存入singletonFactories时并不会执行lambda表达式，也就是不会执行`getEarlyBeanReference`方法。



在创建B时需要A，此时会从根据beanName从singletonFactories获取到一个ObjectFactory，然后执行ObjectFactory，也就是**执行getEarlyBeanReference方法，此时会得到一个A原始对象经过AOP之后的代理对象，然后把该代理对象放入`earlySingletonObjects`中，注意此时并没有把代理对象放入singletonObjects中**，那什么时候放入到singletonObjects中呢？

此时，我们只获得了A原始对象的代理对象，这个对象还不是完整的，因为A对象还没有进行属性填充，所以此时不能直接把A对象的代理对象直接放入singletonObjets，所以只能暂时现A这种状态的对象的代理对象放入到earlySingletonObjects缓存中，这样当其他对象依赖A时就可以直接从earlySingletonObjects获取到A原始对象的代理对象了，并且是A的同一个代理对象。



当B创建完成后A继续它的生命周期，A在完成属性注入后，会按照它本身的逻辑去进行AOP，而此时我们知道A原始对象已经经历过了AOP，所以对于A本身而言，不会再去进行AOP了。

>  **怎么判断一个对象是否经历过了AOP呢**？
>
> 会利用上文提到的`earlyProxyReferences`，在AbstractAutoProxyCreator的postProcessAfterInitialization方法中，会去判断当前beanName是否在earlyProxyReferences，如果在则表示已经提前进行过AOP了，无需再次进行AOP。



对于A而言，进行了AOP的判断后，以及BeanPostProcessor的执行之后，就需要把A对应的对象放入singletonObjects中了，但是我们知道，应该是要A的代理对象放入singletonObjects中，所以此时需要从earlySingletonObjects中得到代理对象，然后入singletonObjects中。



**总结**：

*  **Spring 能够解决循环依赖的情况**：主bean通过属性或者setter方法注入所依赖的bean，不是通过构造函数注入。

* 循环依赖的解决：三级缓存实现，即`singletonObjects`，`earlySingletonObjects`和`singletonFactories`。
  1. ` singletonObjects`：缓存某个beanName对应的经过了完整生命周期的bean
  2. `earlySingletonObjects`：缓存提前拿原始对象进行了AOP之后得到的代理对象，原始对象还没有进行属性注入和后续的BeanPostProcessor等生命周期
  3. `singletonFactories`：缓存的是一个ObjectFactory，主要用来去生成原始对象进行了AOP之后得到的代理对象，在每个Bean的生成过程中，都会提前暴露一个工厂，这个工厂可能用到，也可能用不到，如果没有出现循环依赖依赖本bean，那么这个工厂无用，本bean按照自己的生命周期执行，执行完后直接把本bean放入singletonObjects中即可，如果出现了循环依赖依赖了本bean，则另外那个bean执行ObjectFactory提交得到一个AOP之后的代理对象(如果有AOP的话，如果无需AOP，则直接得到一个原始对象)。
  4. 其实还有一个缓存，就是`earlyProxyReferences`，它用来记录某个原始对象是否进行过AOP了。





### BeanFactory 和 FactoryBean的区别？

在Spring中有BeanFactory和FactoryBean这2个接口，从名字来看很相似，比较容易搞混。

#### BeanFactory

`BeanFactory` 是一个接口，它是Spring中工厂的顶层规范，是Spring容器的核心接口，它定义了`getBean()`、`containsBean()`、`isSignleton()`等一系列管理Bean的方法，Spring 容器都是它的具体实现，例如：

* DefaultListableBeanFactory
* XmlBeanFactory
* ApplicationContext



#### FactoryBean

`FactoryBean` 也是一个接口，它是一个Bean，但又不仅仅是一个Bean。**它是一个能够生产或修饰对象生成的工厂Bean，类似设计模式中的工程模式和装饰器模式，它能在需要的时候生产一个对象，且不仅仅限于它自身，它能返回任何Bean的实例**。

如下是FactoryBean的定义：

```java
public interface FactoryBean<T> {

	//从工厂中获取bean
	@Nullable
	T getObject() throws Exception;

	//获取Bean工厂创建的对象的类型
	@Nullable
	Class<?> getObjectType();

	//Bean工厂创建的对象是否是单例模式
	default boolean isSingleton() {
		return true;
	}
}
```

从它的接口定义中可以看出，`FactoryBean`表现出的是一个工厂的职责。**即一个Bean A如果实现了BeanFactory接口，那么A久变成了一个工厂，根据A的名字获取的实际实际上是工厂调用`getObject()`返回的对象，而不是A本身，如果要获取工厂A自身的实例，那么需要在名称前面加上'`&`'符号**



**说了这么多，为什么要有`FactoryBean`这个东西呢，有什么具体的作用吗？**

FactoryBean在Spring中最为典型的一个应用就是用来**创建AOP的代理对象**。

AOP实际上是Spring在运行时创建了一个代理对象，也就是说这个对象，是我们在运行时创建的，而不是一开始就定义好的，这很符合工厂方法模式。更形象地说，AOP代理对象通过Java的反射机制，在运行时创建了一个代理对象，在代理对象的目标方法中根据业务要求织入了相应的方法。这个对象在Spring中就是`ProxyFactoryBean`。



**总结**

* **BeanFactory 和 FactoryBean 都是工厂，但FactoryBean本质上还是一个Bean，也归BeanFactory管理**。
* **BeanFactory 是Spring容器的顶层接口，而FactoryBean更像是用户自定义的工厂接口**。



## 6.2 Spring AOP

### 谈谈对 AOP 的了解？

`AOP（Aspect-Oriented Programming，面向切面编程）`能够将那些与业务无关，却为业务模块所共同调用的逻辑或责任（例如事务处理、日志管理、权限控制等）封装起来，便于减少系统的重复代码，降低模块间的耦合度，并有利于未来的可扩展性和可维护性。**AOP的核心思想是基于代理思想，对目标对象创建代理对象，在不修改原目标对象的情况下，通过代理对象，调用增强功能代码，从而对原有业务方法的功能进行增强**。

Spring AOP是基于动态代理的，如果需要代理的对象实现了某个接口，那么Spring AOP就会使用`JDK动态代理`去创建代理对象了；如果需要代理的对象没后接口，只有具体的类就无法使用JDK动态代理，此时Spring AOP会使用`CGLib动态代理`来创建代理对象。

![SpringAOPProcess](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/230ae587a322d6e4d09510161987d346.jpeg)

当然也可以使用`AspectJ`，Spring AOP中已经集成了AspectJ，AspectJ应该算得上是Java生态系统中最完整的AOP框架了。使用AOP之后我们可以把一些通用功能抽象出来，在需要用到的地方直接使用即可，这样可以大大简化代码量。我们需要增加新功能也方便，提高了系统的扩展性。日志功能、事务管理和权限管理等场景都用到了AOP。



### AOP 有哪些实现方式？

实现 AOP 的技术，主要分为两大类：

- **静态代理**

   指使用 AOP 框架提供的命令进行编译，从而在编译阶段就可生成 AOP 代理类，因此也称为编译时增强； 

  - 编译时编织（特殊编译器实现），典型的实现比如：AspectJ
  - 类加载时编织（特殊的类加载器实现）。

- **动态代理**

   在运行时在内存中“临时”生成 AOP 动态代理类，因此也被称为运行时增强。 

  - JDK 动态代理 
    - JDK Proxy 是 Java 语言自带的功能，无需通过加载第三方类实现；
    - Java 对 JDK Proxy 提供了稳定的支持，并且会持续的升级和更新，Java 8 版本中的 JDK Proxy 性能相比于之前版本提升了很多；
    - JDK Proxy 是通过拦截器加反射的方式实现的；
    - JDK Proxy 只能代理实现接口的类；
    - JDK Proxy 实现和调用起来比较简单；
  - CGLIB 
    - CGLib 是第三方提供的工具，基于 ASM 实现的，性能比较高；
    - CGLib 无需通过接口来实现，它是针对类实现代理，主要是对指定的类生成一个子类，它是通过实现子类的方式来完成调用的。



### AspectJ 定义的通知类型有哪些？

- **Before**（前置通知）：目标对象的方法调用之前触发
- **After** （后置通知）：目标对象的方法调用之后触发
- **AfterReturning**（返回通知）：目标对象的方法调用完成，在返回结果值之后触发
- **AfterThrowing**（异常通知） ：目标对象的方法运行中抛出 / 触发异常后触发。AfterReturning 和 AfterThrowing 两者互斥。如果方法调用成功无异常，则会有返回值；如果方法抛出了异常，则不会有返回值。
- **Around** （环绕通知）：编程式控制目标对象的方法调用。环绕通知是所有通知类型中可操作范围最大的一种，因为它可以直接拿到目标对象，以及要执行的方法，所以环绕通知可以任意的在目标对象的方法调用前后搞事，甚至不调用目标对象的方法



### 谈谈你对CGLib的理解？

JDK 动态代理机制只能代理实现接口的类，一般没有实现接口的类不能进行代理。使用 CGLib 实现动态代理，完全不受代理类必须实现接口的限制。

CGLib 的原理是对指定目标类生成一个子类，并覆盖其中方法实现增强，但因为采用的是继承，所以不能对 final 修饰的类进行代理。



### Spring AOP的原理/如何生成代理类的？



## 6.3 Spring MVC

### 谈谈对Spring MVC的了解？

MVC 是模型(Model)、视图(View)、控制器(Controller)的简写，其核心思想是通过将业务逻辑、数据、显示分离来组织代码。

![img](https://oss.javaguide.cn/java-guide-blog/image-20210809181452421.png)

MVC 是一种设计模式，Spring MVC 是一款很优秀的 MVC 框架。Spring MVC 可以帮助我们进行更简洁的 Web 层的开发，并且它天生与 Spring 框架集成。Spring MVC 下我们一般把后端项目分为 Service 层（处理业务）、Dao 层（数据库操作）、Entity 层（实体类）、Controller 层(控制层，返回数据给前台页面)。



### Spring MVC 的核心组件有哪些？

Spring合并组件有以下5个：

- **`DispatcherServlet`** ：**核心的中央处理器**，负责接收请求、分发，并给予客户端响应。
- **`HandlerMapping`** ：**处理器映射器**，根据 uri 去匹配查找能处理的 `Handler` ，并会将请求涉及到的拦截器和 `Handler` 一起封装。
- **`HandlerAdapter`** ：**处理器适配器**，根据 `HandlerMapping` 找到的 `Handler` ，适配执行对应的 `Handler`；
- **`Handler`** ：**请求处理器**，处理实际请求的处理器，通常就是程序员编写的controller方法。
- **`ViewResolver`** ：**视图解析器**，根据 `Handler` 返回的逻辑视图 / 视图，解析并渲染真正的视图，并传递给 `DispatcherServlet` 响应客户端。



### Spring MVC的工作流程？

![img](http://image.easyblog.top/1597936663684c9fe5f02-c484-4866-baf1-88e4a152b2a6.png)

**流程说明（重要）：**

1. 客户端（浏览器）发出请求，`DispathcherServlet`获取到请求

2. `DispatcherServlet`根据请求信息调用 `HandlerMapping`。`HandlerMapping` 根据 uri 去匹配查找能处理的 `Handler`（也就是我们平常说的 `Controller` 控制器） ，并会将请求涉及到的`拦截器`和 `Handler` 一起封装成`执行链(HandlerExecutionChain)`返回。
3. `DispatcherServlet` 调用 `HandlerAdapter`适配执行 `Handler` 。
4. `Handler` 完成对用户请求的处理后，会返回一个 `ModelAndView` 对象给`DispatcherServlet`， `ModelAndView` 顾名思义，包含了数据模型以及相应的视图的信息。`Model` 是返回的数据对象，`View` 是个逻辑上的 `View`。
5. `ViewResolver` 会根据逻辑 `View` 查找实际的 `View`，比如当 Spring MVC 接收到 application/json 类型的响应请求时，它会查找合适的视图解析器，根据配置文件中的配置，找到适合处理该类型响应的视图解析器。Spring MVC 默认使用的是 `MappingJackson2JsonView` 视图解析器，它会将 Java 对象转换成 JSON 字符串并返回。
6. `DispatcherServlet` 进行视图渲染 （视图渲染将模型数据(在ModelAndView对象中)填充到response域）
7. `DispatcherServlet`返回响应给请求者（浏览器）



### 统一异常处理怎么做？

推荐使用注解的方式统一异常处理，具体会使用到 `@ControllerAdvice` + `@ExceptionHandler` 这两个注解 。

```java
@ControllerAdvice
@ResponseBody
public class GlobalExceptionHandler {

    @ExceptionHandler(BaseException.class)
    public ResponseEntity<?> handleAppException(BaseException ex, HttpServletRequest request) {
      //......
    }

    @ExceptionHandler(value = ResourceNotFoundException.class)
    public ResponseEntity<ErrorReponse> handleResourceNotFoundException(ResourceNotFoundException ex, HttpServletRequest request) {
      //......
    }
}
```

这种异常处理方式下，会给所有或者指定的 `Controller` 织入异常处理的逻辑（AOP），当 `Controller` 中的方法抛出异常的时候，由被`@ExceptionHandler` 注解修饰的方法进行处理。



## 6.4 Spring 事务

### Spring 管理事务的方式有几种/事务实现方式？

- **编程式事务** ： 在代码中硬编码(不推荐使用) ，通过 `TransactionTemplate`或者 `TransactionManager` 手动管理事务，实际应用中很少使用，但是对于理解 Spring 事务管理原理有帮助。

  ```java
  @Service
  public class TransactionDemo {
      
      @Autowired
      private TransactionTemplate transactionTemplate;
  
    
      // 手动管理事务
      public void programmaticUpdate() {
          // 这里也可以使用Lambda表达式
          transactionTemplate.execute(new TransactionCallbackWithoutResult() {
              protected void doInTransactionWithoutResult(TransactionStatus status) {
                  updateOperation1();
                  updateOperation2();
              }
          });
      }
  }
  ```

  

- **声明式事务** ： 在 XML 配置文件中配置或者基于注解 `@Transactional` ， 实际是通过 AOP 实现，其本质是通过AOP功能，对方法前后进行拦截，将事务处理的功能编织到拦截的方法中，也就是在目标方法开始之前启动一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务。

  ```java
  @Service
  public class TransactionDemo {
      
  
      @Transactional   // 基于注解管理事务
      public void programmaticUpdate() {
          updateOperation1();
          updateOperation2();
      }
  }
  ```

  

### Spring 事务中哪几种事务传播行为?

**事务传播行为是为了解决业务层方法之间互相调用的事务问题**。

当事务方法被另一个事务方法调用时，必须指定事务应该如何传播。例如：方法可能继续在现有事务中运行，也可能开启一个新事务，并在自己的事务中运行。

正确的事务传播行为可能的值如下:

**1.`TransactionDefinition.PROPAGATION_REQUIRED`**

使用的最多的一个事务传播行为，我们平时经常使用的`@Transactional`注解默认使用就是这个事务传播行为。如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。

**`2.TransactionDefinition.PROPAGATION_REQUIRES_NEW`**

创建一个新的事务，如果当前存在事务，则把当前事务挂起。也就是说不管外部方法是否开启事务，`Propagation.REQUIRES_NEW`修饰的内部方法会新开启自己的事务，且开启的事务相互独立，互不干扰。

**3.`TransactionDefinition.PROPAGATION_NESTED`**

如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于`TransactionDefinition.PROPAGATION_REQUIRED`。

**4.`TransactionDefinition.PROPAGATION_MANDATORY`**

如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。（mandatory：强制性）



除了以上4个事务隔离级别，以下 3 种事务传播行为，事务将不会发生回滚：

- **`TransactionDefinition.PROPAGATION_SUPPORTS`**: 如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
- **`TransactionDefinition.PROPAGATION_NOT_SUPPORTED`**: 以非事务方式运行，如果当前存在事务，则把当前事务挂起。
- **`TransactionDefinition.PROPAGATION_NEVER`**: 以非事务方式运行，如果当前存在事务，则抛出异常。



### Spring 事务中的隔离级别有哪几种?

事物的隔离级别（isolation level）定义了一个事务可能受其他并发事务影响的程度。 在并发事务中，经常会引起以下问题：

* **脏读（Dirty reads）**——脏读发生在一个事务读取了另一个事务改写但尚未提交的数据时。如果改写在稍后被回滚了，那么第一个事务获取的数据就是无效的。

- **不可重复读（Nonrepeatable read）**——不可重复读发生在一个事务执行相同的查询两次或两次以上，但是每次都得到不同的数据时。这通常是因为另一个并发事务在两次查询期间进行了更新。
- **幻读（Phantom read）**——幻读与不可重复读类似。它发生在一个事务（T1）读取了几行数据，接着另一个并发事务（T2）插入了一些数据时。在随后的查询中，第一个事务（T1）就会发现多了一些原本不存在的记录。

​    所以为了解决这些问题，引入了数据库的事务隔离级别的概念。Spring定义的事务隔离级别和数据库中定义的事务隔离级别是对应的，具体如下：

| 隔离级别                     | 含义                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| `ISOLATION_DEFAULT`          | 使用后端数据库默认的隔离级别，MySQL 默认采用的 `REPEATABLE_READ` 隔离级别 Oracle 默认采用的 `READ_COMMITTED` 隔离级别. |
| `ISOLATION_READ_UNCOMMITTED` | 最低的隔离级别，允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读 |
| `ISOLATION_READ_COMMITTED`   | 允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生 |
| `ISOLATION_REPEATABLE_READ`  | 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生 |
| `ISOLATION_SERIALIZABLE`     | 最高的隔离级别，完全服从ACID的隔离级别，确保阻止脏读、不可重复读以及幻读，也是最慢的事务隔离级别，因为它通常是通过完全锁定事务相关的数据库表来实现的 |



### Spring 事务常见问题

#### （1）大事务导致事务超时

上周有一个功能，包含FTP文件上传及数据库操作，不知道是不是网络的原因，FTP把文件传输到对方服务器的时间特别长，导致后续的写库操作无法执行。后面是通过将功能拆分，不在事务内部执行文件上传，移到外层就行。



#### （2）事务失效

##### （2.1）非public方法失效

在源码内部对于非public方式执行返回null,不支持对非public的事务支持

![img](https://segmentfault.com/img/bVcSxIO)

##### （2.2）rollbackFor配置为默认值

将rollbackFor配置为默认值后，抛出非检查性异常时，事务无法回滚

##### （2.3）使用了不支持事务的引擎

在MySQL中支持事务的引擎是`innodb`



### Spring 事务注解的本质？

**Spring 事务注解 `@Transactional` 注解的逻辑主要是依赖Spring AOP原理实现，其本质是通过AOP功能，对方法前后进行拦截，将事务处理的功能编织到拦截的方法中，也就是在目标方法开始之前启动一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务**。

我们了解Spring事务注解的运行原理，实际就是去了解事务注解动态代理类的生成过程，这个过程主要分为两个步骤：

1. 向Spring容器注册事务相关的切面逻辑
2. 根据切面逻辑生成动态代理（本质是Spring Aop生成代理的原理）

下面围绕这两点来看下Spring声明事务的实现原理：

#### @EnableTransactionManagement工作原理（加载事务切面逻辑）

Spring 切面逻辑里有三个概念：

* `Pointcut`：负责告诉spring容器哪个类需要增强
* `Advise`：具体的切面逻辑，这里就是根据异常进行commit或者回滚的相关逻辑
* `Advisor`：封装了`Advise`和`Pointcut`的类



我们开启Spring的声明式事务所使用的`@EnableTransactionManagement`注解，本质上就是增加了一个Advisor，当我们使用`@EnableTransactionManagement`注解来开启Spring事务时，该注解代理的功能就是向Spring容器中添加了两个Bean：

- **AutoProxyRegistrar**
- **ProxyTransactionManagementConfiguration**

##### AutoProxyRegistrar

AutoProxyRegistrar主要的作用是向Spring容器中注册了一个`InfrastructureAdvisorAutoProxyCreator`的Bean。而`InfrastructureAdvisorAutoProxyCreator`继承了`AbstractAdvisorAutoProxyCreator`，所以这个类的主要作用就是开启自动代理的作用，也就是一个BeanPostProcessor，会在初始化后步骤中去寻找Advisor类型的Bean，并判断当前某个Bean是否有匹配的Advisor，是否需要利用动态代理产生一个代理对象。



##### ProxyTransactionManagementConfiguration

`ProxyTransactionManagementConfiguration`是一个配置类，它又定义了另外三个bean：

- **BeanFactoryTransactionAttributeSourceAdvisor**：一个Advisor，该类实现了`PointcutAdvisor`接口
- **AnnotationTransactionAttributeSource**：相当于`BeanFactoryTransactionAttributeSourceAdvisor`中的Pointcut，`AnnotationTransactionAttributeSource`就是用来判断某个类上是否存在`@Transactional`注解，或者判断某个方法上是否存在`@Transactional`注解的。
- **TransactionInterceptor**：相当于`BeanFactoryTransactionAttributeSourceAdvisor`中的Advice，TransactionInterceptor就是代理逻辑，当某个类中存在`@Transactional`注解时，到时就产生一个代理对象作为Bean，代理对象在执行某个方法时，最终就会进入到TransactionInterceptor的`invoke()`方法。

![](http://image.easyblog.top/168164797905397945a0e-07ef-4f96-98e1-efe1d8a9ee3b.png)



到这里，Spring容器加载了需要实现事务相关切面的关键的三个对象，其中`Pointcut`的匹配逻辑就是看这个方法有没有被`@Transactional`注解标注，最终会调用到`SpringTransactionAnnotationParser`类的`parseTransactionAnnotation`方法里。

![](http://image.easyblog.top/1681649836547bc8df1d3-c5cf-4274-893e-a556e9399471.png)



##### TransactionInterceptor 事务代理逻辑

这里主要看下事务的核心逻辑，这个核心逻辑就在实现了`Advise`接口的`TransactionInterceptor`类的`invoke`方法里，这里看下这里的源码：

```java
public Object invoke(MethodInvocation invocation) throws Throwable {
   Class<?> targetClass = (invocation.getThis() != null ? AopUtils.getTargetClass(invocation.getThis()) : null);
    // 调用事务逻辑
    return invokeWithinTransaction(invocation.getMethod(), targetClass, invocation::proceed);
}

@Nullable
protected Object invokeWithinTransaction(Method method, @Nullable Class<?> targetClass, TransactionAspectSupport.InvocationCallback invocation) throws Throwable {
  TransactionAttributeSource tas = this.getTransactionAttributeSource();
  // 获取改方法上的事务配置，包括传播级别、异常信息等配置
  TransactionAttribute txAttr = tas != null ? tas.getTransactionAttribute(method, targetClass) : null;
  // 事务管理器，负责生成事务上下文信息，比如开启事务、获取数据库链接等逻辑
  TransactionManager tm = this.determineTransactionManager(txAttr);
  ...
  PlatformTransactionManager ptm = this.asPlatformTransactionManager(tm);
  String joinpointIdentification = this.methodIdentification(method, targetClass, txAttr);
  // 根据传播级别配置，看是否需要新建事务
  TransactionAspectSupport.TransactionInfo txInfo = this.createTransactionIfNecessary(ptm, txAttr, joinpointIdentification);

  Object retVal;
  // 通过try catch捕获异常来实现回滚逻辑
  try {
  // 调用真正的dao层逻辑
      retVal = invocation.proceedWithInvocation();
  } catch (Throwable var18) {
  // 根据@Transactional配置的异常来决定是否回滚
      this.completeTransactionAfterThrowing(txInfo, var18);
      throw var18;
  } finally {
  // 结束当前的事务，信息是保存在ThreadLocal里
      this.cleanupTransactionInfo(txInfo);
  }

  if (retVal != null && vavrPresent && TransactionAspectSupport.VavrDelegate.isVavrTry(retVal)) {
      TransactionStatus status = txInfo.getTransactionStatus();
      if (status != null && txAttr != null) {
          retVal = TransactionAspectSupport.VavrDelegate.evaluateTryFailure(retVal, txAttr, status);
      }
  }
  // 没有异常时，执行commit操作
  this.commitTransactionAfterReturning(txInfo);
  return retVal;
  ...
  
}
```

**总结下核心步骤如下**：

网上大佬画的Spring事务原理流程图：https://www.processon.com/view/link/5fab6edf1e0853569633cc06

通过动态代理为标注了`@Transactional`注解的方法增加切面逻辑，而事务的上下文包括数据库链接都是通过`ThreadLocal`来传递，在这个切面逻辑里主要做这几个事情：

1. 获取方法上标注的注解的元数据，包括传播级别、异常配置等信息
2. 通过`ThreadLocal`获取事务上下文，检查是否已经激活事务
3. 如果已经激活事务，则根据传播级别配置，看是否需要新建事务（如果新建事务，会生成一个新的事务上下文对象`TransactionInfo`，并将上一个事务上下文赋值到新上下文的oldTransactionInfo属性上）代码位置在`TransactionAspectSupport`类的`prepareTransactionInfo`方法里的`bindToThread`方法里
4. 开启事务，先通过数据库连接池获取链接，关闭链接的`autocommit`，然后在`try catch`里反射执行真正的`DAO`操作，通过异常情况来决定是`commit`还是`rollback`









# 七、Spring Boot

### 什么是SpringBoot?

Spring Boot 是 Spring 开源组织下的子项目，**是 Spring 组件一站式解决方案，主要是简化了使用 Spring 的难度，简省了繁重的配置，提供了各种启动器，开发者能快速上手**。

* `独立运行`

  Spring Boot 而且内嵌了各种 servlet 容器，Tomcat、Jetty 等，现在不再需要打成war 包部署到容器中，Spring Boot 只要打成一个可执行的 jar 包就能独立运行，所有的依赖包都在一个 jar 包内。

* `简化配置`

  spring-boot-starter-web 启动器自动依赖其他组件，简少了 maven 的配置。

* `自动配置`

  Spring Boot 能根据当前类路径下的类、jar 包来自动配置 bean，如添加一个 spring

  boot-starter-web 启动器就能拥有 web 的功能，无需其他配置。

* `无代码生成和XML配置`

  Spring Boot 配置过程中无代码生成，也无需 XML 配置文件就能完成所有配置工作，这一切都是借助于条件注解完成的，这也是 Spring4.x 的核心功能之一。



### SpringBoot核心注解有哪些？

启动类上面的注解是`@SpringBootApplication`，他也是SpringBoot的核心注解，主要组合包含了以下3个注解：

- `@SpringBootConfiguration`：组合了@Configuration注解，实现配置文件的功能；
- `@EnableAutoConfiguration`：打开自动配置的功能，也可以关闭某个自动配置的选项，如关闭数据源自动配置的功能：
- `@SpringBootApplication(exclude={DataSourceAutoConfiguration.class})`；
- `@ComponentScan`：Spring组件扫描。



### SpringBoot自动配置原理？

每个SpringBoot应用入口类（启动类）定义头上都会有一个`@SpringBootApplication`注解，这个注解由三个核心注解构成：`@SpringBootConfiguration`、`@EnableAutoConfiguration` 和`@CompmentScan`

![](http://image.easyblog.top/1681653947090365dc787-7805-4124-9c99-9c4acf6e5bc2.png)



#### @SpringBootConfiguration

`@SpringBootConfiguration`，标注这是一个配置类，作用和`@Configuration`类似，@Configuration` 注解的作用是将其作为一个配置类，来配置 `Spring` 的上下文，相当于 `Spring` 的 `XML` 配置文件中的` `<beans>`



#### @ComponentScan

这个的作用就是 **扫描指定路径下的组件**，并加入到 **`IOC` 容器** 中,相当于Spring的 XML配置文件中的 `<context:component-scan/>`



#### @EnableAutoConfiguration

![](http://image.easyblog.top/16816543581332b372ec7-a794-45c2-9281-9a367c12c1c5.png)

`@EnableAutoConfiguration`：**实现自动导入的核心注解**。它也是一个复合注解，重要的注解有两个：`@AutoConfigurationPackage` 和 `@Import(AutoConfigurationImportSelector.class)`

##### @AutoConfigurationPackage

![](http://image.easyblog.top/16816546343042c4da6f5-6527-4b9f-a7f1-4f5492a26e9f.png)

从名字就可以看出它是一个 **自动配置包路径** 的注解，它的作用是当没有配置 `basePackages` 或者 `basePackageClasses` 时，这个类就会自动**将该注解所在的包作为基本路径进行注册**



##### @Import(AutoConfigurationImportSelector.class)

它使用`@Imoprt`导入了一个类—`AutoConfigurationImportSelector`，这个类是`ImportSelector`接口的一个实现类，接口中定义有一个`selectImports`方法，AutoConfigurationImportSelector在方法实现中会加载项目classpah下所有`META-INF/spring.factories`配置，并且依次的去创建这个集合中的所有的类的全路径对象。

**（1）process**

这个方法在获取这些 `Import` 类时会被调用，其中 `getAutoConfigurationEntry` 方法就是从`META-INF/spring.factories`配置加载自动配置类的步骤。

![](http://image.easyblog.top/16816551490585ef37f50-6a3d-4654-a12f-151353c1a6fd.png)



**（2）getAutoConfigurationEntry**

获取自动配置类实体

![](http://image.easyblog.top/1681655379085551d56eb-5b6f-42cd-9b09-e8653c689897.png)



**（3）getCandidateConfiguration**

![](http://image.easyblog.top/16816556068628040d21e-752e-42f3-a1a0-e7ffe07ac20b.png)

来到这里，可以发现它调用到这个 `SpringFactoriesLoader`，这里SpringBoot使用了Spring提供的  `SPI` 机制，进入方法内部可以发现它加载的是  `META-INF/spring.factories` 这个文件 ,相比 `java` 的  `META-INF/services` ，有以下的不同点：

1. 从名字上就可以发现很大的不同（ 一个是 `factories` 文件，一个是以**接口全名**命名的文件 ）。
2. `spring.factories`  以一个聚合的作用，把相应的接口和实现类以  **key = value** 形式展现在 `spring.factories` 文件中。
3. `spring.factories`  中的所有配置项会加载到我们的缓存中，以  `Map<String,List<String>>` 形式存储，但不是所有的都会被实例化，被加载到 `IOC` 容器中，除了必要的类外（ `EventPublishingRunListener` 等 ），还有**满足特定条件**下的自动配置类会被加载到 `IOC` 容器中



#### 总结

1.  `SpringBoot` 的自动装配很重要的一点就是，就是要在配置类上开启 `@EnableAutoConfiguration` 或者 `@SpringBootApplication`  注解，来让自动配置**生效**。

2. 自动配置的核心是 `Spring` 的 `SPI` 机制 ，以及**组件选择器**`AutoConfigurationImportSelector`，具体是通过其中的 `getAutoConfigurationEntry` 方法来获取 `SPI` 中的自动配置类并进行过滤，最后通过 `processImports` 将配置类加载到 `IOC` 容器中，完成自动配置。



### SpringBoot启动过程？

SpringBoot拼接极简的配置和一键启动，在极短的时间内就俘获了大量Javaer的“芳心”，那么SpringBoot带给我们这么多便利的背后，它都做了些什么，让我们就跟随spirng boot的整个启动流程一探究竟。

![](http://image.easyblog.top/16817163148422ef91829-4906-43eb-b610-de794ce5126e.png)

#### 1.初始化Spring应用

![](http://image.easyblog.top/168171805373176064d2c-3c37-4161-a131-26851efff3e0.png)

SpringBoot程序启动第一步会`new`一个`SpringBootApplication`对象，在构造方法中主要干了下面几件事情：

##### 设置应用类型

这个过程非常重要，直接决定了项目的类型，应用类型分为三种，都在`WebApplicationType`这个枚举类中，如下：

1. `NONE`：顾名思义，什么都没有，正常流程走，不额外的启动`web容器`，比如`Tomcat`。
2. `SERVLET`：基于`servlet`的web程序，需要启动内嵌的`servlet`web容器，比如`Tomcat`。
3. `REACTIVE`：基于`reactive`的web程序，需要启动内嵌`reactive`web容器。

判断的依据很简单，就是加载对应的类，比如加载了`DispatcherServlet`等则会判断是`Servlet`的web程序。源码如下：

![](http://image.easyblog.top/1681718157562ad08e6f4-b741-4b9f-99fc-2e3d0752c3f4.png)



##### 设置初始化器（Initializer）

初始化器`ApplicationContextInitializer`主要用于`IOC`容器刷新之前初始化一些组件，比如`ServletContextApplicationContextInitializer`。

SpringBoot获取初始化器是使用了`Spring SPI机制`，即SpringBoot会使用`SpringFactoriesLoader`从spring-boot-autoconfigure包下的`/META-INFO/spring.factories`文件中加载配置的初始化器

##### 设置监听器（Listener）

监听器（`ApplicationListener`）这个概念在`Spring`中就已经存在，主要用于监听特定的事件(`ApplicationEvent`)，比如IOC容器刷新、容器关闭等。

`Spring Boot`扩展了`ApplicationEvent`构建了`SpringApplicationEvent`这个抽象类，主要用于`Spring Boot`启动过程中触发的事件，比如程序启动中、程序启动完成等。如下图：

![img](https://ask.qcloudimg.com/http-save/yehe-1346475/31ozdjsftd.png?imageView2/2/w/2560/h/7000)



监听器的获取和初始化器的获取方式类似，都是借助了`Spring SPI机制`从spring-boot-autoconfigure包下的`/META-INFO/spring.factories`文件中加载配置的监听器。



#### 2.执行run方法

上面分析了`SpringApplication`的构建过程，一切都做好了铺垫，现在到了启动的过程了。

##### **获取、启动运行过程监听器**

运行过程监听器—`SpringApplicationRunListeners`，是用来监听应用程序启动过程的，接口的各个方法含义如下：

```java
public interface SpringApplicationRunListener {

    // 在run()方法开始执行时，该方法就立即被调用，可用于在初始化最早期时做一些工作
    void starting();
    // 当environment构建完成，ApplicationContext创建之前，该方法被调用
    void environmentPrepared(ConfigurableEnvironment environment);
    // 当ApplicationContext构建完成时，该方法被调用
    void contextPrepared(ConfigurableApplicationContext context);
    // 在ApplicationContext完成加载，但没有被刷新前，该方法被调用
    void contextLoaded(ConfigurableApplicationContext context);
    // 在ApplicationContext刷新并启动后，CommandLineRunners和ApplicationRunner未被调用前，该方法被调用
    void started(ConfigurableApplicationContext context);
    // 在run()方法执行完成前该方法被调用
    void running(ConfigurableApplicationContext context);
    // 当应用运行出错时该方法被调用
    void failed(ConfigurableApplicationContext context, Throwable exception);
}
```



运行过程监听器的获取和初始化其实还是借助Spring SPI机制，通过调用了`loadFactoryNames()`方法从`spring.factories`文件中获取值，最终注入的是`EventPublishingRunListener`这个实现类，创建实例过程肯定是通过反射了，因此我们看看它的构造方法，如下图：

![img](https://ask.qcloudimg.com/http-save/yehe-1346475/rkoun94vgu.png?imageView2/2/w/2560/h/7000)

这个运行监听器内部有一个事件广播器（`SimpleApplicationEventMulticaster`），主要用来广播特定的事件(`SpringApplicationEvent`)来触发特定的监听器`ApplicationListener`。



获取到监听器之后启动监听器就简单了，直接调用`starting()`方法，方法内部广播了`ApplicationStartingEvent`事件，这些事件可以在SpringBoot启动早期做一些初始化工作。



##### 环境构建

这一步主要用于加载系统配置以及用户的自定义配置(`application.properties`)，源码如下，在`run()`方法中：

![](http://image.easyblog.top/1681722520494a4e199c2-f762-4f58-817d-50acf9f5beaf.png)

`prepareEnvironment`方法内部广播了`ApplicationEnvironmentPreparedEvent`事件



##### 创建IOC容器

这一步主要根据`webApplicationType`决定创建IOC的类型，一般`Servlet`项目会创建的是`AnnotationConfigServletWebServerApplicationContext`。

> 注意，这一步仅仅是创建了`IOC容器`，没有其他操作。

![](http://image.easyblog.top/1681722704314f88c32c9-554b-403b-bf2a-8b5c7a391aa7.png)



##### IOC容器前置处理

这一步真是在刷新容器之前做准备，其中有一个非常关键的操作：将启动类注入容器，为后续的自动化配置奠定基础

![](http://image.easyblog.top/168172299286388f28fe6-6e0e-45d7-a68f-22d8b7ce79f9.png)



##### 刷新容器

SpringBoot启动的核心操作，





##### IOC容器后置操作







总结



### SpringBoot如何自定义starter?
