### 工厂方法模式(Factory Method Pattern)



设计模式中 工厂模式可以分为三类：

- **简单工厂模式（Simple Factory）**
- **工厂方法模式（Factory Method）**
- **抽象工厂模式（Abstract Factory）**

这三种模式从上到下逐步抽象，并且更具一般性。
GOF 在《设计模式》一书中将工厂模式分为两类：工厂方法模式（Factory Method）与抽象工厂模式（Abstract Factory）。将简单工厂模式（Simple Factory）看为工厂方法模式的一种特例，两者归为一类。
这三种工厂模式在设计模式的分类中都属于**创建型模式**。



#### 一、工厂方法模式的定义

工厂方法模式(Factory Method Pattern)其定义：定义一个用于创建对象的接口，让子类决定实例化哪一个类。工厂方法使一个类的实例化延迟到其子类。



#### 二、工厂方法模式的优点

1. 工厂方法模式可以降低模块间的耦合性，使用工厂方法模式创建一个对象，不再需要知道创建该对象的艰辛过程和必要信息，只需要提供一个产品的约束条件（例如，类名或约束字符串）就可以获取需要的对象。
2. 工厂方法有良好的扩展性。
3. 屏蔽了产品类，调用者只需要关系产品类的接口。因为产品类的具体对象实例是由工厂产生的。



### 三、工厂方法模式的应用场景

1. 工厂方法模式是自己new一个对象的替代方案，但不是绝对的。增加工厂会增加系统的复杂性。
2.  需要灵活、有良好可扩展性。因为工厂方法模式屏蔽了产品，调用者只会关系产品的接口而不是实现。在不改变接口的情况下，增加一种产品仅仅只需要实现产品的接口就可以了，例如，邮件系统有三种协议（POP3、IMPA、HTTP），这三种产品只需要实现一个共同的接口（例如：IMail），若以后出现了新协议，新协议只需要实现这个接口，该邮件系统就完美支持新协议了。



### 四、工厂方法的通用类图

![](http://image.easyblog.top/2012011718381225.png)

* **Product角色（抽象产品角色）**：定义了产品的共性，实现事物最抽象的定义
* **Creator角色（抽象工厂角色）**：定义了一个用于创建对象的接口，具体如何创建对象是有其子类（具体工厂）实现
* **ConcreteCreatorProduct（具体产品）**：实现了抽象产品所定义的接口
* **ConcreteCreatorCreator（具体工厂）**：实现了创建具体产品的接口



### 五、工厂方法通用实现

#### 5.1 抽象工厂

```java
public interface Factory {

    //定义一个创建对象的接口，如何创建对象由其子类（具体工厂）实现。
    <T extends Product> T createProduct(Class<T> clazz);
    
}
```



#### 5.2 具体工厂实现

```java
public class ConcreteFactory implements Factory{
    @Override
    @SuppressWarnings("unchecked")
    public <T extends Product> T createProduct(Class<T> clazz) {
        Product product = null;
        try {
            product = (Product)clazz.newInstance();
        } catch (InstantiationException | IllegalAccessException e) {
            e.printStackTrace();
        }

        return (T)product;
    }
}
```



#### 5.3 抽象产品

```java
public interface Product {
    //定义了产品类的共性
    void doSomething();
}
```



#### 5.4 具体产品实现

```java
public class ConcreteProduct implements Product{
    @Override
    public void doSomething() {
        System.out.println("doSomething!");
    }
}
```



#### 5.5 Client端调用

```java
public class Main {

    public static void main(String[] args) {
        Factory factory = new ConcreteFactory();
        Product product = factory.createProduct(ConcreteProduct.class);
        
        product.doSomething();
    }
    
}
```

运行程序最终打印：doSomething!



### 六、工厂方法模式的扩展

#### 扩展一：简单工厂方法模式（静态工厂方法模式）

有些时候工厂类十分简单，而且不太会发生变化，这时我们就不需要在使用工厂的时候去把工厂实例new出来，并且了解到该工厂不太容易改变，我们就不需要抽象工厂类了，直接定义一个工厂类，该工厂类提供一个静态的工厂方法用于生产对象，这样我们在使用工厂的时候直接使用工厂类提供的静态方法就可以了，而不需要去创建该工厂实例。

![](https://pic002.cnblogs.com/images/2012/369936/2012011721470279.png)

代码只需要做简单修改：抽象产品角色和具体产品角色与通用工厂方法模式一致，只需要去掉抽象工厂角色，工厂角色提供一个static的工厂方法。

该变形有个严重的缺点：工厂类难以扩展。

```java
public class ConcreteFactory {
    @SuppressWarnings("unchecked")
    public static <T extends Product> T createProduct(Class<T> clazz) {
        Product product = null;
        try {
            product = (Product)clazz.newInstance();
        } catch (InstantiationException | IllegalAccessException e) {
            e.printStackTrace();
        }

        return (T)product;
    }
}
```



#### 扩展二：升级为多工厂

考虑到这样一个情况，有多种产品。如果我们按照以前的方式设计工厂方法模式，则会定义如下一个巨大的工厂方法：

```java
public <T exetends Product> T createProduct(Class<T> clazz,parameters){

　　　　... 

　　　　if(clazz == ConcreteProductA.class)

　　　　{...} 

　　　　else if(clazz == ConcreteProductB.class)

　　　　{...} 

　　　　else if(clazz == ConcreteProductC.class)

　　　　{...} 

　　　　... 

}
```

光看函数体就头疼，这种情况下产品种类越多这个函数就越臃肿（因为函数体还要有判断），可读性越差。为了解决这样的情况，工厂方法模式出现了多工厂的变形。将这个巨大的函数体拆解成多个小的函数体（多工厂的目的）。

![](http://image.easyblog.top/2012011722393863.png)

代码修改：抽象产品类和具体产品类没有什么变化，抽象工厂依然定义一个生产对象的接口供子类实现，子类按照自己的需求分别实现生产对象接口，比如工厂A对应生产产品A，工厂B对应生产产品B......

具体工厂：

ConcreteFactoryA，ConcreteFactoryB....类似：

```java
public class ConcreteFactoryA implements Factory{
    //对象属性的默认值，用于填充创建对象的属性
    private static final String DEFAULT_VLAVE1 = "default_value1";
    private static final String DEFAULT_VLAVE2 = "default_value2";
    
    @Override
    public <T extends Product> T createProduct(Class<T> clazz){
        T product =null;
        
        try {
            //获取创建该对象的构造函数。这里也可以使用默认构造函数，然后使用set方法完成初始化。
            Constructor<T> constructor = clazz.getDeclaredConstructor(new Class<?>[]{String.class,String.class});
            product = constructor.newInstance(new Object[]{DEFAULT_VLAVE1,DEFAULT_VLAVE2});
        } catch (Exception e) {
            e.printStackTrace();
        }
        
        return product;
    }
}
```



场景类中调用：

```java
public class Client {
    public static void main(String[] args) {
        /*要创建ProductA，先获取工厂A，然后调用创建对象接口创建对象A*/
        Factory factoryA = new ConcreteFactoryA();
        Product productA = factoryA.createProduct(ProductA.class);
        
        productA.doSomething();
    }
}
```



通过这样的变形，将一个臃肿的工厂分成了许多小的工厂，代码的条理性也比较清晰了，这样的变形让工厂的职责也变得清晰起来。但该变形有一个缺点：扩展起来比较麻烦，例如要增加一个ProductD，那也要增加一个对应的工厂，因为必须要维持产品与工厂的对应关系。一般在使用该变形时，需要一个协调类封装子工厂，对高层提供统一的接口，避免调用者直接与各子工厂交流。



#### 扩展三：代替单例模式

单例模式一般都没有接口或抽象类，不能面向接口编程，扩展性比较差等一些缺点，使用工厂方法模式可以一定程度上解决它自身的一些缺点。

![](http://image.easyblog.top/2012011814491066.png)

代码修改：具体产品类都应该将他们的构造方法设置为private的。每一个具体工厂持有负责的单例属性。

具体产品：

```java
public class SingletonA implements SingletonProduct {
    /*防止SingletonA在其他地方被实例化*/
    private SingletonA() {

    }

    @Override
    public void doSomething() {
        System.out.println("singletonA");
    }
}
```

具体工厂类：

```java
public class SingletonFactoryA implements SingletonFactory {

    private static SingletonA singletonA = null;

    static {
        Class<SingletonA> clazz = SingletonA.class;
        try {
            Constructor<SingletonA> constructor = clazz.getDeclaredConstructor();
            constructor.setAccessible(true);
            singletonA = constructor.newInstance();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    @Override
    public SingletonProduct getSingleProduct() {
        return singletonA;
    }
}
```

调用方式与多工厂变形类似，若单例产品不多可以使用一个静态工厂变形（看具体需求），以上变形的工厂模式代替单例模式，可以让原来的单例注重自己的业务逻辑，不再需要负责单例的实现，单例的实现由工厂实现，这样类的职责就更清晰了。并且单例都实现了共有接口，调用者只用关注接口就可以，不必与具体的单例产品交互，降低了耦合。



#### 扩展四：带缓存的工厂方法模式

如果遇到创建一个对象比较消耗资源的情况下（例如，硬盘访问、涉及多方面交互），可以使用带缓存的工厂方法模式来降低创建和销毁该类对象带来的复杂性。通过定义一个Map集合对象，容纳所有产生的对象，如果在Map中已经存在的对象，则直接取出。如果没有，则根据需要产生一个对象放入到Map中，以便下次使用。

带缓存的工厂：

```java
public class CacheFactory implements Factory {

    /*工厂缓存，用于存放已经生产出来的对象*/
    private static final Map<Class<? extends Product>, Product> PRODUCT_MAP = new ConcurrentHashMap<>();

    @Override
    @SuppressWarnings("unchecked")
    public <T extends Product> T createProduct(Class<T> clazz) {
        Product product = null;
        if (PRODUCT_MAP.containsKey(clazz)) {
            product = PRODUCT_MAP.get(clazz);
        } else {
            try {
                product = clazz.newInstance();
            } catch (Exception e) {
                e.printStackTrace();
            }

            PRODUCT_MAP.put(clazz, product);
        }
        return (T)product;
    }
}
```

