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

`ReentrantLock` 锁的unlock方法直接调用 `Sync`类的`release`方法，

![](http://image.easyblog.top/168001708651070b47562-26f3-4f90-8d94-fdbed3eb2e36.png)



https://tech.meituan.com/2019/12/05/aqs-theory-and-apply.html





## 3.5 J.U.C线程池详解











## 3.6 J.U.C并发集合详解









## 3.7 J.U.C原子类详解
