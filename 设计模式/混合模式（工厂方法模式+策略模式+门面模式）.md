### 混编模式（策略模式+工厂方法模式+门面模式）



### 一、引言

在 [策略模式(Strategy Pattern)](./策略模式(Strategy Pattern)) 一文我们了解并学习了策略模式相关知识点，我们使用策略模式可以定义一组算法类（此处的算法主要指业务范畴不同对不同业务逻辑的处理），将每个算法分别封装起来，让它们可以互相替换。策略模式这样做的好处在于可以使算法的变化独立于使用它们的客户端，但也正如它的优点一样，策略模式单独使用时的缺点也十分明显：（1）子类膨胀，一个策略一个类，策略类的数量比较惊人；（2）调用者必须知道所有具体策略的存在（调用一个策略还需要自己new出来）。这两个缺点导致了策略模式在快速的迭代的生成系统中无法单独使用，因为业务变化非常快，还不如直接if-else来的快。

其实，仔细分析策略模式的两个缺点就会发现，导致策略模式难以使用的根源在于调用者如要想调用一个策略接口下的某个策略实现时必须要知道这个策略存在，然后并将其实例化后调用处理逻辑，这个实例化策略实现的过程其实我们可以让工厂模式来帮忙，这里具体策略就是产品，工厂负责根据调用方的某个约束条件生产对应的策略实例并返回给调用端，之后调用端只用调用目标方法即可。



下面我们来实现一个具有加减乘除的简单计算器演示并一步步优化上面的过程。



### 二、策略模式提供策略实现

**抽象计算策略**：

```java
public interface Strategy {
    /**
     * 定义了计算策略所拥有的算法
     */
    int calculate(int a, int b);
}
```

实现抽象计算策略实现具体计算策略：

**（1）加法策略 AddStrategy**

```java
public class AddStrategy implements Strategy {
    @Override
    public int calculate(int a, int b) {
        return a + b;
    }
}
```

**（2）减法策略 SubStrategy**

```java
public class SubStrategy implements Strategy {
    @Override
    public int calculate(int a, int b) {
        return a - b;
    }
}
```

**（3）乘法策略 MultiStrategy**

```java
public class MultiStrategy implements Strategy {
    @Override
    public int calculate(int a, int b) {
        return a * b;
    }
}
```

**（4）除法策略 DivStrategy**

```java
public class DivStrategy implements Strategy {
    @Override
    public int calculate(int a, int b) {
        //这里关注设计，暂不考虑除0异常
        return a / b;
    }
}
```



此时还需要一个封装策略的门面，让策略可以互换：

```java
public class StrategyContext {

    private Strategy strategy = null;

    public StrategyContext(Strategy strategy) {
        strategy = strategy;
    }

    /**
     * 让策略可以更换
     */
    public void setStrategy(Strategy strategy) {
        this.strategy = strategy;
    }


    /**
     * @param a
     * @param b
     * @return
     */
    public int execute(int a, int b) {
        //委托给具体策略处理业务逻辑
        return strategy.calculate(a, b);
    }
}
```



### 三、策略工厂的实现生产策略

正如前文所说，为了避免策略模式必须要将具体的策略暴露给高层模块的缺点，我们可以借助工厂来生成策略，有个工厂，高层模块只需要一个约束条件就可以获得需要的策略。

（1）抽象出工厂的职责，使工厂也可以独立扩展

```java
public interface Factory {
    /**
     * 定义一个生成策略的接口，其参数还可以使用一个配置文件来实现约束条件，这里使用了枚举
     */
    Strategy createStrategy(StrategyEnum strategyEnum);
}
```

（2）实现工厂接口获取具体工厂类，此类就要职责就是根据约束条件生产客户端需要的策略

```java
public class SimpleStrategyFactory implements Factory {
    @Override
    public Strategy createStrategy(StrategyEnum strategyEnum) {
        Strategy strategy = null;
        try {
            String className = strategyEnum.getClassName();
            Class<?> clazz = Class.forName(className);
            strategy = (Strategy) clazz.newInstance();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return strategy;
    }
}
```



现在这个简单计算器基本已经实现完成了，我们可以在客户端尝试调用一下：

```java
public class Client {
    public static void main(String[] args) {
        //1.获取工厂
        Factory factory = new SimpleStrategyFactory();
        //2.获取策略
        Strategy strategy = factory.createStrategy(StrategyEnum.ADD);
        //3.封装
        StrategyContext strategyContext = new StrategyContext(strategy);
        //4.计算
        System.out.println(strategyContext.execute(100, 200));

      
        //切换策略
        strategyContext.setStrategy(factory.createStrategy(StrategyEnum.MULTI));
        System.out.println(strategyContext.execute(100, 200));

    }
}
```

程序执行结果如下：

![](img/%E6%88%AA%E5%B1%8F2022-02-24%20%E4%B8%8B%E5%8D%882.23.59.png)



### 四、计算器门面封装调用逻辑

通过上面Client的调用发现每次进行计算的步骤为：获取工厂、获取策略、封装策略、计算结果。每次调用都这样写比较麻烦，高层模块为了计算一个结果还需要记住执行顺序，这时候我们可以使用门面模式来屏蔽子系统的复杂性，为高层模块提供一个计算接口即可。

```java
/**
* 门面角色不参与子系统的逻辑，所以对子系统进行了一次封装
*/
public class FacadeContext {

    private final Factory factory = new SimpleStrategyFactory();
    private final StrategyContext context = new StrategyContext(null);


    public int calculate(int a, int b, StrategyEnum strategyEnum) {
        context.setStrategy(factory.createStrategy(strategyEnum));
        return context.execute(a, b);
    }
}
```

计算器的门面，简单的委托类，为高层提供一个访问子系统的接口，使高层模块不再依赖子系统

```Java
public class CalculatorFacade {

    private final FacadeContext context = new FacadeContext();


    public int exec(int a, int b, StrategyEnum strategyEnum) {
        return context.calculate(a, b, strategyEnum);
    }

}
```



现在来看看站在Client调用方的调用是多么优雅：

```java
public class Client {
    public static void main(String[] args) {

        CalculatorFacade calculatorFacade = new CalculatorFacade();
        //加法
        System.out.println(calculatorFacade.exec(1, 2, StrategyEnum.ADD));
        //减法
        System.out.println(calculatorFacade.exec(200, 100, StrategyEnum.SUB));
        //乘法 
        System.out.println(calculatorFacade.exec(2, 10, StrategyEnum.MULTI));
         //除法
        System.out.println(calculatorFacade.exec(50, 5, StrategyEnum.DIV));
    }
}
```



**附：StrategyEnum枚举代码：**

```java
public enum StrategyEnum {

    ADD("org.example.design.factory.mix.strategy.AddStrategy"),
    SUB("org.example.design.factory.mix.strategy.SubStrategy"),
    MULTI("org.example.design.factory.mix.strategy.MultiStrategy"),
    DIV("org.example.design.factory.mix.strategy.DivStrategy");

    private final String className;

    StrategyEnum(String className) {
        this.className = className;
    }

    public String getClassName() {
        return className;
    }
}
```

程序执行结果如下：

![](img/%E6%88%AA%E5%B1%8F2022-02-24%20%E4%B8%8B%E5%8D%882.44.00.png)



以上是混合使用三种模式的一个简单例子，可以看出灵活搭配模式能让系统更健壮，灵活性更高，扩展性更强。在上述例子添加策略非常非常的容易，只需要继承Strategy接口即可，然后在枚举中增加对应的策略类名即可，高层代码一点也不用改变。



### 参考

* [策略模式——（+简单工厂模式+反射）](https://www.cnblogs.com/yulinfeng/p/5891161.html)
* [混合模式（工厂方法模式+策略模式+门面模式）](https://www.cnblogs.com/otomedaybreak/archive/2012/01/20/2327879.html)

