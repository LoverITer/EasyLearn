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

关于Spring的事件机制，推荐一篇网上大佬写的文章，质量非常高：https://segmentfault.com/a/1190000020967936

#### MVC架构中的观察模式

MVC(Modew-View-Controller)架构中也应用了观察者模式，其中模型（Model）可以对应观察者模式中的观察目标，而视图（View）对应于观察者，控制器（Controller）就是中介者模式的应用



### 四、 观察者模式的通用类图

<img src="http://image.easyblog.top/168222880705741192aaa-eac0-4b32-bc01-241d68a076db.png" style="zoom:50%;" />

* **`Subject`**：目标对象（即被观察者）抽象类，类中有一个用来存放观察者对象的Vector容器（之所以使用Vector而不使用List，是因为多线程操作时，Vector在是安全的，而List则是不安全的），这个Vector容器是被观察者类的核心，另外还有三个方法：`subscribe`方法是向这个容器中添加观察者对象；`unsubscribe`方法是从容器中移除观察者对象；`notifyObserver`方法是依次调用观察者对象的对应方法。这个角色可以是接口，也可以是抽象类或者具体的类，因为很多情况下会与其他的模式混用，所以使用抽象类的情况比较多。
* **`Observer`**：观察者抽象类，观察者角色一般是一个接口，它只有一个`update`方法，在被观察者状态发生变化时，这个方法就会被触发调用。
* **`ConcreteSubject`**：具体被观察者对象，实际开发中真正实现业务逻辑的具体角色
* **`ConcreteObserver1`、`ConcreteObserver2`**：观察者接口的具体实现，在这个角色中，将定义被观察者对象状态发生变化时所要处理的逻辑。



### 五、观察者模式的通用实现

在电商网站中， 商品时不时地会出现缺货情况。 可能会有客户对于缺货的特定商品表现出兴趣。 这一问题有三种解决方案：

1. 客户以一定的频率查看商品的可用性。
2. 电商网站向客户发送有库存的所有新商品。
3. 客户只订阅其感兴趣的特定商品， 商品可用时便会收到通知。 同时， 多名客户也可订阅同一款产品。

选项 3 是最具可行性的， 这其实就是观察者模式的思想。 观察者模式的主要组成部分有：

- 会在有任何事发生时发布事件的主体。
- 订阅了主体事件并会在事件发生时收到通知的观察者。



#### 定义观察者接口

```java
public interface Observer {
    // 更新观察者数据
    void update();
}
```



#### 定义具体用户观察者实现类

```java
@Data
public class Consumer implements Observer {


    private String email;

    public Consumer(String email) {
        this.email = email;
    }

    @Override
    public void update(ItemSubject itemSubject) {
        IphoneItem item = null;
        if (itemSubject instanceof IphoneItem) {
            item = (IphoneItem) itemSubject;
        }
        System.out.printf("Sending email to customer %s for item %s\n", this.getEmail(), Objects.requireNonNull(item, "Error").getName());
    }
}
```



#### 定义商品被关注抽象对象

```java
public abstract class ItemSubject {
    //观察者集合(关注此事件的对象)
    private final Vector<Observer> observers;

    public ItemSubject() {
        observers = new Vector<>();
    }

    public void subscribe(Observer observer) {
        this.observers.add(observer);
    }


    public void unsubscribe(Observer observer) {
        this.observers.remove(observer);
    }

    protected void notifyObserver() {
        this.observers.forEach(observer -> {
            // 遍历观察者列表，更新观察者数据
            observer.update(this);
        });
    }

}
```



#### 定义被关注商品具体实现

```java
@Data
public class IphoneItem extends ItemSubject {

    private String name;
    private boolean inStock;

    public IphoneItem() {
        super();
        this.name = "[iphone14 pro max 暗夜紫]";
    }

    public void updateAvailability() {
        System.out.printf("Item %s is now in stock\n", this.getName());
        this.inStock = true;
        this.notifyObserver();
    }
}
```



基本角色准备完成，下面调用测试一下：

```java
public class Client {

    public static void main(String[] args) {
        Consumer consumer1 = new Consumer("zhangsan@163.com");
        Consumer consumer2 = new Consumer("lsp@qq.com");

        // 客户对商品添加关注
        IphoneItem item = new IphoneItem();
        item.subscribe(consumer1);
        item.subscribe(consumer2);
        
        // 更新库存并通知用户
        item.updateAvailability();
    }
    
}
```

执行结果：

```txt
Item [iphone14 pro max 暗夜紫] is now in stock
Sending email to customer zhangsan@163.com for item [iphone14 pro max 暗夜紫]
Sending email to customer lsp@qq.com for item [iphone14 pro max 暗夜紫]
```



### 六、Spring中观察者模式使用案例

现假设一个用户注册的案例场景：用户注册后，系统需要给用户发送邮件告知用户注册是否成功，需要给用户初始化积分，后续可能会添加其他的操作，如再发一条手机短信等，希望程序具有拓展性符合开闭原则.

如果不使用事件机制，那么我们写出来的代码将会是下面这个样子：

![image.png](https://segmentfault.com/img/bVbz71E)

相信这段代码是大多数Java开发者会第一直觉写出来的代码，这么写其实没有什么问题。但是这么写，实际上并不是特别符合隐含的设计需求（开闭规则）,假设增加更多的注册项`Service`，我们需要修改`register`方法,并让`UserService`注入对应的`Service`。而实际上`register`并不关心这些"额外"的操作，如何将这些代码抽取出去，这时可以考虑`Event`机制.



#### 定义用户注册事件

`ApplicationEvent`是由`Spring`提供的所有`Event`类的基类,这里为了简单只传递`name`

```java
import org.springframework.context.ApplicationEvent;

/**
 * @author frank.huang
 * @date 2023/04/23 16:20
 */
public class UserRegisterEvent extends ApplicationEvent {
    /**
     * Create a new ApplicationEvent.
     *
     * @param source the object on which the event initially occurred (never {@code null})
     */
    public UserRegisterEvent(Object source) {
        super(source);
    }
}
```



#### 定义用户注册服务(事件发布者)

服务交给`Spring`容器管理。`ApplicationEventPublishAware`是由`Spring`提供的用于`Service`注入`ApplicationEventPublisher`事件发布器的接口.使用这个接口,我们的`Service`就拥有发布事件的能力了

用户注册后,不再是显示调用其他的业务`Service`,而是发布一个用户注册事件

```java
@Slf4j
@Service
public class UserService implements ApplicationEventPublisherAware {


    private ApplicationEventPublisher applicationEventPublisher;


    @Override
    public void setApplicationEventPublisher(@NonNull ApplicationEventPublisher applicationEventPublisher) {
        this.applicationEventPublisher = applicationEventPublisher;
    }
    
    /**
     * 用户注册
     *
     * @param name
     */
    public void register(String name) {
        System.out.printf("用户 %s 开始注册\n", name);
        applicationEventPublisher.publishEvent(new UserRegisterEvent(name));
        System.out.printf("用户 %s 注册完成\n", name);
    }
}
```



#### 定义邮件服务,积分服务,其他服务(事件订阅者)

事件订阅者的服务同样需要托管于`Spring`容器，`ApplicationListener<E extends ApplicationEvent>`接口是`Spring`提供的事件订阅者必须实现的接口,我们一般把`Service`关心的事件作为泛型传入。事件处理:`ApplicationEvent#getSource`拿到事件的具体内容,本例中为`name`.

```java
@Service
public class EmailService implements ApplicationListener<UserRegisterEvent> {
    
    @Override
    public void onApplicationEvent(UserRegisterEvent event) {
        System.out.printf("用户 %s 注册成功，发送邮件....\n",event.getSource());
    }
}


@Service
public class ScoreService implements ApplicationListener<UserRegisterEvent> {
    
    @Override
    public void onApplicationEvent(UserRegisterEvent event) {
        System.out.printf("用户 %s 注册成功，初始化积分....\n",event.getSource());
    }
}

@Service
public class OtherService implements ApplicationListener<UserRegisterEvent> {
    
    @Override
    public void onApplicationEvent(UserRegisterEvent event) {
        System.out.printf("用户 %s 注册成功，初始其他....\n",event.getSource());
    }
}
```



启动服务，调用register方法，入参 zhangsan 运行结果：

```text
用户 zhangsan 开始注册
用户 zhangsan 注册成功，发送邮件....
用户 zhangsan 注册成功，初始化积分....
用户 zhangsan 注册完成
```
