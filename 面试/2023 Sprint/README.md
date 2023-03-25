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



### 什么是进程?什么是线程？

* **进程**

进程可以理解为一个应用程序执行的实例（比如在windows下打开Word就启动了一个进程），进程是资源分配的最小单位，每个进程都有自己独立的地址空间，每启动一个进程，系统就会为它分配地址空间，建立数据表来维护代码段、堆栈段和数据段。**进程主要有数据、程序和程序控制块（PCB）组成，其中PCB是系统感知进程存在的唯一标志。**

* **线程**

线程是进程中的一个执行单元，一个进程中可以启动多个线程，并且一个进程中的多个线程可以共享此进程中的所用资源。每个线程都有自己独立的运行时桟和程序计数器，**线程是CPU调度的最小单位**。

### 并发和并行的概念？

- **并发（concurrent）**

  单核CPU 下，操作系统通过任务调度器，将CPU的时间片分给不同的线程使用，只是由于CPU的切换速度非常快（Windows下一个最小的时间片是15ms），让用户看上去是同步执行的，实际上还是串性执行的。**这种线程轮流使用CPU的方法叫并发。**

- **并行（parallel）**

  **多个cpu或者多台机器同时处理任务**，是真正意义上的同时执行。



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
