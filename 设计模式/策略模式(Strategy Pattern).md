### 策略模式(Strategy Pattern)



### 一、策略模式的定义

策略模式（Strategy Pattern）也叫做政策模式（Policy Pattern）其定义：定义一组算法，将他们封装起来，使它们可以相互替换。经常被用来解决在有多种算法相似的情况下，使用 if...else 所带来的复杂和难以维护。

### 二、策略模式的优缺点

#### 优点：

1. 算法直接可以相互替换。这是因为策略都实现策略接口。
2.  可以避免多重条件的情况出现。假设一个策略家族有N个成员，当一会需要策略A，另一会需要策略B，不使用策略模式的话，只能使用ifelse或switch语句实现，但是这样的程序不容易维护，可读性也比较差。使用策略模式后可以由其他模块确定策略。
3. 有良好的扩展性。增加一个策略，只需要实现策略接口，就这么简单。

#### 缺点：

　1. 子类膨胀，一个策略一个类，策略类的数量比较惊人。
 　2. 调用者必须知道所有具体策略的存在（调用一个策略还需要自己new出来）。 

### 三、策略模式的应用场景

1. 如果在一个系统里面有许多类，它们之间的区别仅在于它们的行为，那么使用策略模式可以动态地让一个对象在许多行为中选择一种行为。 
2. 一个系统需要动态地在几种算法中选择一种。 
3. 如果一个对象有很多的行为，如果不用恰当的模式，这些行为就只好使用多重的条件选择语句来实现。

### 四、策略模式的通用类图

![](https://pic002.cnblogs.com/images/2012/369936/2012011900511269.png)

* Strategy（抽象策略角色）：定义了每个算法必须有的算法和属性。
* StrategyA/StrategyB...（具体策略角色）：实现了抽象策略定义的接口。
* Context（封装角色）：将策略封装起来，屏蔽了调用者直接访问具体策略。



### 五、策略模式的通用实现

#### 5.1 抽象策略

```java
public interface Strategy {
    void operation();
}
```

#### 5.2 具体策略算法

实现Strategy接口，分别实现StrategyA、StrategyB 表示不同的两个策略

```java
public class StrategyA implements Strategy{
    @Override
    public void operation() {
        System.out.println("StrategyA执行了...");
    }
}


public class StrategyB implements Strategy{
    @Override
    public void operation() {
        System.out.println("StrategyB执行了...");
    }
}
```



#### 5.3 封装角色

```java
public class Context {

    private Strategy strategy = null;

    public Context(Strategy strategy) {
        this.strategy = strategy;
    }

    /*实现策略可以相互替换*/
    public void replaceStrategy(Strategy strategy) {
        this.strategy = strategy;
    }

    /*封装角色提供接口，让调用使用策略，屏蔽高层模块直接使用策略*/
    public void execute() {
        this.strategy.operation();
    }

}
```



#### 5.4 client调用示例

```java
public class Main {
    public static void main(String[] args) {
        Context context = new Context(new StrategyA());
        
        //策略A执行
        context.execute();
        
        context.replaceStrategy(new StrategyB());
        
        //策略B执行
        context.execute();
    }
}
```

运行程序最终打印如下：

![](img/%E6%88%AA%E5%B1%8F2022-02-16%20%E4%B8%8B%E5%8D%887.45.26.png)



### 六、实际案例

举个比较简单的例子帮助大家理解策略模式 ，比如现在有个需求要求"实现一个可以计算两个整数加减乘除计算的功能"

#### 实现一：不使用策略模式

```java
public class Calculator {
    /*代表加减乘除的字符串，用于判断使用什么计算策略*/
    public final static String ADD_SYMBOL = "+";
    public final static String SUB_SYMBOL = "-";
    public final static String MUL_SYMBOL = "*";
    public final static String DIV_SYMBOL = "/";

    /*策略选择函数，operator字符串参数用于判断，a、b两个参数使用的计算策略*/
    public int execute(int a, int b, String operator) {
        int result = 0;
        if (ADD_SYMBOL.equals(operator)) {
            result = this.add(a, b);
        } else if (SUB_SYMBOL.equals(operator)) {
            result = this.sub(a, b);
        } else if (MUL_SYMBOL.equals(operator)) {
            result = this.multi(a, b);
        } else if (DIV_SYMBOL.equals(operator)) {
            result = this.div(a, b);
        }

        return result;
    }

    /*加法功能实现*/
    private int add(int a, int b) {
        return a + b;
    }

    /*减法功能实现*/
    private int sub(int a, int b) {
        return a - b;
    }

    /*乘法功能实现*/
    private int multi(int a, int b) {
        return a * b;
    }

    /*除法功能实现*/
    private int div(int a, int b) {
        //只是为了说明问题，除数为0就不考虑了
        return a / b;
    }
}
```

这个实现比较容易想到，通过"operator"这个参数决定使用什么计算策略。这样的实现看上去完美的处理了需求，单还是有很多需要改进的地方：

* 首先，**扩展性很差**——若要添加一个求余的需求，除了改源代码旗本没其他办法了，添加一个需求改一次，这样还了得。
* 其次，**可读性不行**，别看这个程序看上去还比较清晰，要是考虑上异常、"operator"检查，功能变多后等等就不顺眼了。
* 最后，这个**类职责不清晰**，又要选择策略又要实现策略。

#### 实现二：使用策略模式

抽象策略角色（定义每个策略通用的行为）：

```java
public interface CalculateStrategy {
    //给两位整数做计算
    int calculate(int a, int b);
}
```

具体策略角色（实现抽象策略）：都继承 CalculateStrategy 分别实现加法、减法、乘法和除法策略

```java
public class AddCalculateStrategy implements CalculateStrategy{
    @Override
    public int calculate(int a, int b) {
        return a+b;
    }
}

public class SubCalculateStrategy implements CalculateStrategy{
    @Override
    public int calculate(int a, int b) {
        return a-b;
    }
}

public class MutiCalculateStrategy implements CalculateStrategy{
    @Override
    public int calculate(int a, int b) {
        return a*b;
    }
}

public class DivCalculateStrategy implements CalculateStrategy{
    @Override
    public int calculate(int a, int b) {
        //除0暂不考虑
        return a/b;
    }
}
```



封装角色：

```java
public class CalculateContext {
    private CalculateStrategy strategy = null;

    public CalculateContext(CalculateStrategy strategy) {
        this.strategy = strategy;
    }

    public void replaceStrategy(CalculateStrategy strategy) {
        this.strategy = strategy;
    }

    public int execute(int a, int b) {
        return strategy.calculate(a, b);
    }
}
```



Client调用示例：

```java
public class Main {
    public static void main(String[] args) {
        CalculateContext context = new CalculateContext(new AddCalculateStrategy());

        System.out.println("1+2="+context.execute(1, 2));

        context.replaceStrategy(new DivCalculateStrategy());

        System.out.println("100/10="+context.execute(100, 10));
    }
}
```

这是按照常规的策略模式实现加减乘除功能的。这样的实现基本上解决了第一种实现方式的缺点。增加一个功能，只需要实现接口 `CalculateStrategy`，扩展非常简单便捷。if 判断语句消失了，可读性大大提高。每个类的职责现在更清晰了。尽管如此，这个实现还是有一个非常大缺点，**这个缺点是该模式自身的缺点，就是调用者必须知道具体策略（这个缺点可以使用混合模式解决）**。具体请参考我的另一篇文章：[混合模式（工厂方法模式+策略模式+门面模式）](./混合模式（工厂方法模式+策略模式+门面模式）)

