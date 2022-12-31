### 职责链模式(Chain Of Responsibility Pattern)



### 一、职责链模式的定义

责任链模式（Chain of Responsibility）定义：**使多个对象都有机会处理请求，从而避免请求的发送者和接受者之间的耦合关系。将这些对象连成一条链，并沿着这条链传递该请求，直到有对象能够处理它**。

理解职责链模式，我们可以先将关注点放在“链”字上，很容易联想到链式结构，举个生活中常见的例子，比如公司请假，根据请假时长的不同，需要递交到的级别也不同，请1天假，给直系领导报告即可，请1~3天假，则需要报告给直系领导的上级，请1周假，则需要更高级别的领导审核......这种层级递进的关系就是一种链式结构，也就是职责链模式的的应用场景。

### 二、职责链式模式的优缺点

#### 优点

1. 将请求和处理分开，接收者和发送者都没有对方的明确信息，且链中的对象也并不知道链的结构，结果是责任链可简化对象的相互连接，它们仅需保持一个指向其后继者的引用，而不需要保持它所有的候选继承者，大大的降低了耦合度。
2. 可以随时增加或者修改处理一个请求的结构，增加了给对象指派职责的灵活性

#### 缺点

1. 性能会受到影响，特别是在链比较长的时候
2. 调试不方便。采用了类似递归的方式，调试时逻辑可能比较复杂
3. 不能保证请求一定被处理：每个职责对象都只负责自己的部分，这样就可以出现某个请求，即使把整个链走完，都没有职责对象处理它。这就需要提供默认处理，并且注意构造链的有效性。

### 三、职责链模式的应用场景

遇到以下场景，可以考虑使用职责链模式优化代码架构，提升系统扩展性和可维护性：

1. 有多个对象可以处理同一个请求，具体哪个对象处理该请求待运行时刻再确定，客户端只需将请求提交到链上，而无须关心请求的处理对象是谁以及它是如何处理的。
2. 在不明确指定接收者的情况下，向多个对象中的一个提交一个请求。
3. 可动态指定一组对象处理请求，客户端可以动态创建职责链来处理请求，还可以改变链中处理者之间的先后次序。

**应用实例**：

1. **Java的异常处理机制**：try-catch语句中catch一系列的异常，当发生异常后JVM沿着catch链逐个尝试，知道某个catch将异常处理掉；

2. **JS 中的冒泡事件**：当点击下面的`A`，`A`在`B`里面，`B`在`C`里面， 那么点击事件将会沿着：`A`-> `B` -> `C` 会依次传到各级元素

   <img src="http://image.easyblog.top/%E6%88%AA%E5%B1%8F2022-02-25%20%E4%B8%8B%E5%8D%881.36.39.png" style="width:60%;" />

3. `SpringMVC` 中，我们有时候会定义一些拦截器，对请求进行预处理，也就是请求过来的时候，会依次经历拦截器，通过拦截器之后才会进入我们的处理业务逻辑代码；同理，在Servlet中的Fliter也有类似的机制。

### 四、 职责链模式的通用类图

<img src="http://image.easyblog.top/1-0-5768536-5768538.png" alt="1-0" style="width:60%;" />

* **抽象处理者(Handler)角色**：定义出一个处理请求的接口。如果需要，接口可以定义 出一个方法以设定和返回对下家的引用。这个角色通常由一个Java抽象类或者Java接口实现。上图中Handler类的聚合关系给出了具体子类对下家的引用，抽象方法handleRequest()规范了子类处理请求的操作。
* **具体处理者(ConcreteHandler)角色**：具体处理者接到请求后，可以选择将请求处理掉，或者将请求传给下家。由于具体处理者持有下家的引用，因此，如果需要，具体处理者可以访问下家

### 五、职责链模式的通用实现

日常开发中我们肯定使用过日志，常见的日志组件比如Logback、Log4以及它的升级版log4j2......日志在处理日志信息时就是一条职责链，按照日志等级高低，不同级别的日志可以打印不同级别的日志信息，下面我们将用职责链实现这一过程。

<img src="http://image.easyblog.top/case-0-5772114.png" style="width:90%;" />

#### 5.1 抽象处理角色

职责链抽象处理角色：`AbstractLogger` ，需要维护：

* （1）下一个处理者的引用 successor，如果为空表示链处理到头了
* （2）对客户端暴露一个可以直接调用的业务方法，此方法要求可以自动将对请求的处理沿着链传递下去并且在符合要求的时候调用业务逻辑队请求进行处理，相对应的就是这里的`log`  和 `write` 方法，`log` 方法由此抽象角色实现并维护，`write` 方法由具体Handler自己实现。

```java
public abstract class AbstractLogger {

    public static int DEBUG = 1;
    public static int INFO = 2;
    public static int ERROR = 3;

    //责任链中的下一个处理者
    protected AbstractLogger successor;
    protected int level;

    public AbstractLogger(int level) {
        this.level = level;
    }

    public void setSuccessor(AbstractLogger successor) {
        this.successor = successor;
    }

    /**
     * 客户端调用的方法
     *
     * @param level
     * @param message
     */
    public void log(int level, String message) {
        if (this.level == level) {
            write(message);
        }
        if (this.successor != null) {
            successor.log(level, message);
        }
    }

    /**
     * 真正的业务处理逻辑，由各个Handler自己实现
     *
     * @param message
     */
    abstract protected void write(String message);
}
```



#### 5.2 具体处理角色

继承抽象处理者，分别实现了 InfoLogger、DebugLogger 和 ErrorLogger，实现如下：

```java
//处理INFO级别信息
public class InfoLogger extends AbstractLogger {
    
    public InfoLogger(){
        super(AbstractLogger.INFO);
    }
    
    @Override
    protected void write(String message) {
        System.out.println("[INFO] " + message);
    }
}

//处理DEBUG级别信息
public class DebugLogger extends AbstractLogger {
    
    public DebugLogger() {
        super(AbstractLogger.DEBUG);
    }

    @Override
    protected void write(String message) {
        System.out.println("[DEBUG] " + message);
    }
}

//处理ERROR级别信息
public class ErrorLogger extends AbstractLogger {
    
    public ErrorLogger() {
        super(AbstractLogger.ERROR);
    }

    @Override
    protected void write(String message) {
        System.err.println("[DEBUG] " + message);
    }
}
```



#### 5.3 Client调用示例

```java
public class Client {

    public static void main(String[] args) {
        AbstractLogger logger = new InfoLogger();
        AbstractLogger debugLogger = new DebugLogger();
        AbstractLogger errorLogger = new ErrorLogger();

        logger.setSuccessor(debugLogger);
        debugLogger.setSuccessor(errorLogger);


        logger.log(AbstractLogger.INFO,"这是一条INFO日志");
        logger.log(AbstractLogger.DEBUG,"这是一条DEBUG日志");
        logger.log(AbstractLogger.ERROR,"这是一条ERROR日志");
    }
}
```

程序执行结果如下：

![](http://image.easyblog.top/%E6%88%AA%E5%B1%8F2022-02-25%20%E4%B8%8B%E5%8D%883.14.06.png)





### 参考

* [深入理解设计模式（12）：职责链模式](https://www.cnblogs.com/xuwendong/p/9814425.html)
* [简说设计模式——职责链模式](https://www.cnblogs.com/adamjwh/p/10932216.html)

