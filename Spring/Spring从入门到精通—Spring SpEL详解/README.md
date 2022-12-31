## Spring从入门到精通—Spring SpEL详解





### 一、SpEL概念

Spring 表达式语言（简称 ”SpEL”）是一种强大的表达式语言，支持在运行时查询和操作对象图。语言语法类似于 Unified EL，但提供了额外的功能，最值得注意的是方法调用和基本的字符串模板功能。

虽然 SpEL 是 Spring 产品组合中表达式评估的基础，但它不直接与 Spring 绑定，可以独立使用。

SpEL 表达式语言支持以下功能：

* （1）**基本表达式**： 字面量表达式、布尔和关系运算符、算数运算符、字符串连接及截取表达式、三目运算及Elivis表达式、正则表达式、括号优先级表达式；

* （2）**类相关表达式**： 类类型表达式、类实例化、instanceof表达式、变量定义及引用、赋值表达式、自定义函数、对象属性存取及安全导航表达式、对象方法调用、Bean引用；

* （3）**集合相关表达式**： 内联List、内联数组、集合，字典访问、列表，字典，数组修改、集合投影、集合选择；（不支持多维内联数组初始化；不支持内联字典定义）

* （4）**其他表达式**：模板表达式。



### 二、SpEL使用场景

#### 1、在注解中使用

常见的比如 @Value注解、spring-redis-cache 缓存中提供的 @CacheXxx系列注解.....以及我们可以在自定义注解中使用SpEL

```java
public class UserService{
  
  @Value("${config.host}")     // 在@Value注解中使用SpEL
  private String host;
  
  
  @CachePut(value="Manager",key="#request.getId()")   // 在CachePut注解中使用SpEL
  @DLock(key="user_lock",extKey={"#request.getId()"})  //自定义注解中使用SpEL
  public void registerUser(CreateUserRequest request){
    //TODO
  }
  
}
```



#### 2、在XML配置中使用

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    
    
    <bean id="helloService" class="com.example.demo.service.HelloService">
        <!-- 同@Value,#{}内是表达式的值，可放在property或constructor-arg内 -->
        <property name="arg" value="#{'hello world'.toUpperCase()}">
    </bean>
    


</beans>
```



#### 3、在代码块中使用

```java
@Test
public void test(){
    //创建ExpressionParser解析表达式
    ExpressionParser parser = new SpelExpressionParser();
    Expression exp = parser.parseExpression("#表达式");
    //执行表达式，默认容器是spring本身的容器：ApplicationContext
    Object value = exp.getValue();
    System.out.println(value);
}
```



### 三、SpEL语法

#### 1.基本表达式

##### 1） 字面量表达式

SpEL支持的字面量包括：字符串、数字类型（int、long、float、double）、布尔类型、null类型。

<table><thead><tr><th>类型</th><th>示例</th></tr></thead><tbody><tr><td>字符串</td><td>String str1 = parser.parseExpression("'Hello World!'").getValue(String.class);</td></tr><tr><td>数字类型</td><td>int int1 = parser.parseExpression("1").getValue(Integer.class);<br> long long1 = parser.parseExpression("-1L").getValue(long.class);<br> float float1 = parser.parseExpression("1.1").getValue(Float.class);<br> double double1 = parser.parseExpression("1.1E+2").getValue(double.class);<br> int hex1 = parser.parseExpression("0xa").getValue(Integer.class);<br> long hex2 = parser.parseExpression("0xaL").getValue(long.class);</td></tr><tr><td>布尔类型</td><td>boolean true1 = parser.parseExpression("true").getValue(boolean.class);<br> boolean false1 = parser.parseExpression("false").getValue(boolean.class);</td></tr><tr><td>null类型</td><td>Object null1 = parser.parseExpression("null").getValue(Object.class);</td></tr></tbody></table>



##### 2）算数运算表达式

SpEL支持加(+)、减(-)、乘(*)、除(/)、求余（%）、幂（^）运算

<table><thead><tr><th>类型</th><th>示例</th></tr></thead><tbody><tr><td>加减乘除</td><td>int result1 = parser.parseExpression("1+2-3*4/2").getValue(Integer.class);//-3</td></tr><tr><td>求余</td><td>int result2 = parser.parseExpression("4%3").getValue(Integer.class);//1</td></tr><tr><td>幂运算</td><td>int result3 = parser.parseExpression("2^3").getValue(Integer.class);//8</td></tr></tbody></table>



##### 3）关系表达式

SpEL支持支持等于（==）、不等于(!=)、大于(>)、大于等于(>=)、小于(<)、小于等于(<=)，区间（between）运算。SpEL同样提供了等价的“EQ” 、“NE”、 “GT”、“GE”、 “LT” 、“LE”来表示等于、不等于、大于、大于等于、小于、小于等于，不区分大小写。



##### 4）逻辑表达式

SpEL支持且（and或者&&）、或(or或者||)、非(!或NOT) 逻辑表达式

| 类型 | 示例                                                         |
| ---- | ------------------------------------------------------------ |
| 且   | int result1 = parser.parseExpression("2>1 and 1==1").getValue(Integer.class);//true |
| 或   | int result1 = parser.parseExpression("2>1 or 1==1").getValue(Integer.class);//true |
| 非   | int result1 = parser.parseExpression("!(2>1 or 1==1)").getValue(Integer.class);//false |



##### 5）字符串连接及截取表达式