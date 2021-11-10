## Spring IOC源码阅读（一）IOC基本概念

`Main.java`

```java
import org.example.bean.Student;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;


public class Main {

    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("application.xml");
        Student student = (Student) context.getBean("student");
        System.out.println(student.toString());
    }

}
```

`application.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

   <!--
    @Data
    public class Student {
      private String name;
      private String subject;
      private Integer age;
    }-->
    <bean id="student" class="org.example.bean.Student">
        <property name="name" value="zhangsan"/>
        <property name="age" value="20"/>
        <property name="subject" value="Java"/>
    </bean>

</beans>
```



这是一段使用Spring管理Bean再经典不过的代码了，在上面的代码中，Spring会自动加载定义在 `resource` 目录下的 `application.xml` 配置文件，并按照配置中的定义初始化Bean并放入IOC容器中管理。在程序员的角度，这一过程中我们并没有使用 `new` 关键字来主动初始化 `Student` 这个对象，但是我们却可以通过Spring容器直接获取到 `Studnet` 对象并使用，这也就是IOC机制的核心。 



### **IOC 定义**

>  **所谓 IOC ，就是由 Spring IOC 容器来负责对象的生命周期和对象之间的关系**。IOC 是 Spring 的核心，也可以称为 Spring 容器。Spring 通过 IOC 容器来管理对象的实例化和初始化，以及对象从创建到销毁的整个生命周期。这个过程，不需要我们手动使用 new 关键字创建对象。由 IOC 容器管理的对象称为 Spring Bean，Spring Bean 就是 Java 对象，和使用 new 关键字创建的对象没有区别。



下图为 `ClassPathXmlApplicationContext` 的类继承体系结构，虽然只有一部分，但是它基本上包含了 IOC 体系中大部分的核心类和接口。

![](http://image.easyblog.top/202105092052551332.png)

下面我们就针对这个图进行简单的拆分和补充说明。

### **Resource体系**

Resource，对资源的抽象，它的每一个实现类都代表了一种资源的访问策略，如ClasspathResource 、 URLResource ，FileSystemResource 等。

![](http://image.easyblog.top/202105092051253023.png)

有了资源，就应该有资源加载，Spring 利用 ResourceLoader 来进行统一资源加载，类图如下:

![](http://image.easyblog.top/202105092051256374.png)

### **BeanFactory 体系**

BeanFactory 是一个非常纯粹的 bean 容器，它是 IOC 必备的数据结构，其中 BeanDefinition 是它的基本结构，它内部维护着一个 BeanDefinition Map ，并可根据 BeanDefinition 的描述进行 bean 的创建和管理。

![](http://image.easyblog.top/202105092051277805.png)

BeanFacoty 有三个直接子接口： `ListableBeanFactory`、`HierarchicalBeanFactory` 和 `AutowireCapableBeanFactory`，`DefaultListableBeanFactory` 为最终默认实现，它实现了所有接口。

### **Beandefinition 体系**

BeanDefinition 用来描述 Spring 中的 Bean 对象。

<img src="http://image.easyblog.top/202105092051280866.png" style="zoom:67%;" />

### **BeandefinitionReader体系**

BeanDefinitionReader 的作用是读取 Spring 的配置文件的内容，并将其转换成 Ioc 容器内部的数据结构：BeanDefinition。

![](http://image.easyblog.top/202105092051283977.png)

### **ApplicationContext体系**

 Spring 容器，也叫做应用上下文，与我们应用息息相关，它继承 BeanFactory，所以它是 BeanFactory 的扩展升级版，如果BeanFactory 是屌丝的话，那么 ApplicationContext 则是名副其实的高富帅。由于 ApplicationContext 的结构就决定了它与 BeanFactory 的不同，其主要区别有：

1. **继承 MessageSource，提供国际化的标准访问策略**。
2. **继承 ApplicationEventPublisher ，提供强大的事件机制**。
3. **扩展 ResourceLoader，可以用来加载多个 Resource，可以灵活访问不同的资源**。
4. **对 Web 应用的支持**。

上面五个体系可以说是 Spring IoC 中最核心的部分，Spring IOC源码系列后续也是针对这五个部分进行源码分析。其实 IoC 咋一看还是挺简单的，无非就是将配置文件（暂且认为是 xml 文件）进行解析（分析 xml 谁不会啊），然后放到一个 Map 里面就差不多了，初看有道理，其实要面临的问题还是有很多的。

