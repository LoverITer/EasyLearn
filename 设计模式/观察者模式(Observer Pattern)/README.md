### 观察者模式(Observer Pattern)



在软件系统设计中经常有这样的需求：当一个对象发生改变，某些与它相关的对象也要随之做出相应的变化。

* MVC 模式中的模型与视图的关系，当模型的数据发生变化，视图也应该跟随变化，这其中模型是『被观察者』，视图是『观察者』
* 气象站可以将每天预测到的温度、湿度、气压等以公告的形式发布给各种第三方网站，如果天气数据有更新，要能够实时的通知给第三方，这里的气象局就是『被观察者』，第三方网站就是『观察者』
* 微信公众号，如果一个用户订阅了某个公众号，那么便会收到公众号发来的消息，那么，公众号就是『被观察者』，而用户就是『观察者』

观察者模式是使用频率较高的设计模式之一。观察者模式包含观察目标和观察者两类对象，一个目标可以有任意数目的与之相依赖的观察者，一旦观察目标的状态发生改变，所有的观察者都将得到通知。





### 一、观察者模式的定义

**观察者模式(Observer Pattern)**： 定义对象间一种一对多的依赖关系，使得当一个对象改变状态时，所有依赖于它的对象都会得到通知并自动更新。

观察者模式的别名包括发布-订阅（Publish/Subscribe）模式、模型-视图（Model/View）模式、源-监听器（Source/Listener）模式或从属者（Dependents）模式，它是**对象行为型模式**。



### 二、观察者模式的优点

#### 优点

- 降低了被观察者（目标）与观察者之间的耦合关系，两者之间是抽象耦合关系，两者都可以独立扩展
- 观察者模式是一种常用的触发机制，它形成一条触发链，依次对各个观察者的方法进行处理
- 支持广播通信
- 符合“开闭原则”的要求

#### 缺点

- 观察者与被观察者之间的依赖关系并没有完全解除，而且有可能出现循环引用
- 观察者与被观察者时间构成一条触发链条，当链条上观察者较多时，性能问题是比较令人担忧的



### 三、观察者模式的应用用场景

#### **JDK中的观察者模式**

观察者模式在 Java 语言中的地位非常重要。在 JDK 的 java.util 包中，提供了 `Observable` 类以及 `Observer` 接口，它们构成了 JDK 对观察者模式的支持。

#### **Spring事件机制中的观察者模式**

Spring框架中大量使用了观察者模式来维护和管理Bean，基于JDK事件机制，Spring定义了四个角色：

1. **事件：ApplicationEvent** 是所有事件对象的父类。ApplicationEvent 继承自 jdk 的 `EventObject`, 所有的事件都需要继承 ApplicationEvent, 并且通过 source 得到事件源。

   > Spring 为我们提供了很多内置事件，`ContextRefreshedEvent`、`ContextStartedEvent`、 `ContextStoppedEvent`、`ContextClosedEvent`、`RequestHandledEvent`。

2. **事件监听：ApplicationListener**，也就是观察者，继承自 jdk 的 `EventListener`，该类中只有一个方法 onApplicationEvent。当监听的事件发生后该方法会被执行。

3. **事件源：ApplicationContext**，`ApplicationContext` 是 Spring 中的核心容器，在事件监听中 ApplicationContext 可以作为事件的发布者，也就是事件源。因为 ApplicationContext 继承自 ApplicationEventPublisher。在 `ApplicationEventPublisher` 中定义了事件发布的方法：`publishEvent(Object event)`

4. **事件管理：ApplicationEventMulticaster**，用于事件监听器的注册和事件的广播。监听器的注册就是通过它来实现的，它的作用是把 Applicationcontext 发布的 Event 广播给它的监听器列表。

#### MVC架构中的观察模式

MVC(Modew-View-Controller)架构中也应用了观察者模式，其中模型（Model）可以对应观察者模式中的观察目标，而视图（View）对应于观察者，控制器（Controller）就是中介者模式的应用



### 四、 观察者模式的通用类图

<img src="http://image.easyblog.top/168222880705741192aaa-eac0-4b32-bc01-241d68a076db.png" style="zoom:50%;" />

* **`Subject`**：目标对象（即被观察者）抽象类，类中有一个用来存放观察者对象的Vector容器（之所以使用Vector而不使用List，是因为多线程操作时，Vector在是安全的，而List则是不安全的），这个Vector容器是被观察者类的核心，另外还有三个方法：`subscribe`方法是向这个容器中添加观察者对象；`unsubscribe`方法是从容器中移除观察者对象；`notifyObserver`方法是依次调用观察者对象的对应方法。这个角色可以是接口，也可以是抽象类或者具体的类，因为很多情况下会与其他的模式混用，所以使用抽象类的情况比较多。
* **`Observer`**：观察者抽象类，观察者角色一般是一个接口，它只有一个`update`方法，在被观察者状态发生变化时，这个方法就会被触发调用。
* **`ConcreteSubject`**：具体被观察者对象，实际开发中真正实现业务逻辑的具体角色
* **`ConcreteObserver1`、`ConcreteObserver2`**：观察者接口的具体实现，在这个角色中，将定义被观察者对象状态发生变化时所要处理的逻辑。



### 五、观察者模式的通用实现

基于上面类图，我们简单实现一下观察者模式：

#### 定义观察者接口

```java
public interface Observer {
    // 更新观察者数据
    void update();
}
```



#### 定义观察者具体实现类

```java
public class ConcreteObserver1 implements Observer{
    @Override
    public void update() {
        System.out.println("观察者1收到更新信息，开始更新观察者1的数据....");
    }
}

public class ConcreteObserver2 implements Observer{
    @Override
    public void update() {
        System.out.println("观察者2收到更新信息，开始更新观察者1的数据....");
    }
}
```



#### 定义被观察者抽象类

```java
public abstract class Subject {

    //观察者集合(关注此事件的对象)
    private final Vector<Observer> observers;
  
    //标志被观察者事件是否发生 
    protected boolean isChange; 

    public Subject() {
        this.observers = new Vector<>();
        this.isChange= false;
    }

    public void subscribe(Observer observer) {
        this.observers.add(observer);
    }


    public void unsubscribe(Observer observer) {
        this.observers.remove(observer);
    }

    protected void notifyObserver() {
        if (!isChange) return;
        this.observers.forEach(observer -> {
            // 遍历观察者列表，更新观察者数据
            observer.update();
        });
    }

    public abstract void doProcess();

}
```



#### 定义观察者具体实现

```java
public class ConcreteSubject extends Subject{

    @Override
    public void doProcess() {
        System.out.println("被观察者对象事件发生");
        this.isChange = true;
        this.notifyObserver();
    }
}
```



基本角色准备完成，下面调用测试一下：

```java
public class Client {
    public static void main(String[] args) {

        ConcreteObserver1 concreteObserver1 = new ConcreteObserver1();
        ConcreteObserver2 concreteObserver2 = new ConcreteObserver2();

        //观察者监听目标事件
        ConcreteSubject concreteSubject = new ConcreteSubject();
        concreteSubject.subscribe(concreteObserver1);
        concreteSubject.subscribe(concreteObserver2);
        
        // 目前事件发生
        concreteSubject.doProcess();
    }
}
```

执行结果：

<img src="http://image.easyblog.top/16822317111345c033176-8078-4f70-b634-8dca04c84a63.png" style="zoom:50%;" />

