###  抽象工厂模式(Abstract Factory Pattern)



[上一篇：工厂方法模式(Factory Method Pattern)](./工厂方法模式(Factory Method Pattern).md) 一篇我们介绍了工厂方法模式以及它的一些常见变种，这篇我们继续来了解一下抽象工厂模式。



#### 一、抽象工厂模式的定义

要想理解抽象工厂模式的精髓，首先我们应该认识一下什么是产品族：位于不同产品等级结构中，功能相关联的产品组成的家族。比如：奔驰SUV、奥迪SUV、宝马SUV，这三者处于不同产品（品牌）中，但是他们的都是SUV汽车，因此这三者就是同一产品族的，相反奔驰轿车就和它们三者不属于同一产品族。

抽象工厂模式定义：提供一个创建一系列相关或相互依赖对象的接口，而无须指定它们具体的类。

#### 二、抽象工厂模式的优点

- 1、隔离了具体类的生成，使得客户并不需要知道什么被创建
- 2、每次可以通过具体工厂类创建一个产品族中的多个对象，增加或者替换产品族比较方便，增加新的具体工厂和产品族很方便；

### 三、抽象工厂模式的应用场景

一个系统不应当依赖于产品类实例如何被创建、组合和表达的细节，这对于所有类型的工厂模式都是重要的。

系统中有多于一个的产品族，而每次只使用其中某一产品族。

属于同一个产品族的产品将在一起使用，这一约束必须在系统的设计中体现出来。

系统提供一个产品类的库，所有的产品以同样的接口出现，从而使客户端不依赖于具体实现。

### 四、抽象工厂模式的通用类图

![](https://pics5.baidu.com/feed/0823dd54564e9258017177343e9fbf5ecdbf4e43.jpeg?token=df1aee257ff25118f35cb593876c2018)

抽象工厂模式包含的角色和工厂方法的角色如出一辙：

* AbstractFactory(抽象工厂)：用于声明生成抽象产品的方法

* ProductFactory(具体工厂)：实现了抽象工厂声明的生成抽象产品的方法，生成一组具体产品，这些产品构成了一个产品族，每一个产品都位于某个产品等级结构中；

* AbstractProduct(抽象产品)：为每种产品声明接口，在抽象产品中定义了产品的抽象业务方法；

* Product(具体产品)：定义具体工厂生产的具体产品对象，实现抽象产品接口中定义的业务方法。



### 五、抽象工厂模式通用实现





#### 5.1 抽象产品

接口Car定义车的行为

```java
public interface Car {
    void drive();
}
```

抽象类BenzCar定义Benz车的数据

```java
public abstract class BenzCar implements Car {
    private String name;
    
    public BenzCar(String name){
        this.name=name;
    }

    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
}
```

抽象类BwmCar定义Bwm车的数据

```java
public abstract class BwmCar implements Car{
    private String name;

    public BwmCar(String name){
        this.name=name;
    }

    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
}
```



#### 5.2 具体产品

继承 BenzCar 分别实现 BenzSUVCar、BenzBusinessCar 表示Benz的两个产品族

```java
public class BenzSUVCar extends BenzCar{
    public BenzSUVCar(String name) {
        super(name);
    }

    @Override
    public void drive() {
        System.out.println(this.getName()+"----BenzSUVCar-----------------------");
    }
}

public class BenzBusinessCar extends BenzCar{
    public BenzBusinessCar(String name) {
        super(name);
    }

    @Override
    public void drive() {
        System.out.println(this.getName()+"----BenzBusinessCar-----------------------");
    }
}
```

同理，继承BwmCar分别实现BwmSUVCar、BwmBusinessCar 表示Bwm的两个产品族

```java
public class BwmSUVCar extends BwmCar{
    public BwmSUVCar(String name) {
        super(name);
    }

    @Override
    public void drive() {
        System.out.println(this.getName()+"----BwmSUVCar-----------------------");
    }
}

public class BwmBusinessCar extends BwmCar{
    public BwmBusinessCar(String name) {
        super(name);
    }

    @Override
    public void drive() {
        System.out.println(this.getName()+"----BwmBusinessCar-----------------------");
    }
}
```



#### 5.3 抽象工厂

接口CarFactory定义工厂的能力

```java
public interface CarFactory {
    //生产Benz车
    Car createBenzCar();

    //生产Bwm车
    Car createBwmCar();
}
```



#### 5.4 具体工厂

SUV 和 Business 是两个不同的产品族，因此我们需要为他们创建不同的工厂，如下 SUVFactory 用于生产SUV，BusinessFactory 用于生产 Business

```java
public class SUVFactory implements CarFactory{
    @Override
    public Car createBenzCar() {
        return new BenzSUVCar("奔驰GLC");
    }

    @Override
    public Car createBwmCar() {
        return new BwmSUVCar("宝马X5");
    }
}

public class BusinessFactory implements CarFactory{
    @Override
    public Car createBenzCar() {
        return new BenzBusinessCar("奔驰A级");
    }

    @Override
    public Car createBwmCar() {
        return new BwmBusinessCar("宝马1系");
    }
}
```



#### 5.5 client 调用

```java
public class Main {

    public static void main(String[] args) {
        //SUV工厂
        SUVFactory suvFactory = new SUVFactory();

        Car benzCar = suvFactory.createBenzCar();
        benzCar.drive();

        Car bwmCar = suvFactory.createBwmCar();
        bwmCar.drive();

        //Business 工厂
        BusinessFactory businessFactory = new BusinessFactory();
        
        Car businessBenzCar = businessFactory.createBenzCar();
        businessBenzCar.drive();

        Car businessBwmCar = businessFactory.createBwmCar();
        businessBwmCar.drive();
    }
}
```

运行程序最终打印如下：

<img src="img/%E6%88%AA%E5%B1%8F2022-02-16%20%E4%B8%8B%E5%8D%886.51.56.png" style="zoom:67%;" />