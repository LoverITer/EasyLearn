##  如何通过IDEA快速解决JAR冲突

本文转载自：https://blog.csdn.net/Evan_Leung/article/details/117402966



相信很多同学在过去做项目都遇到过Jar冲突的问题，在本地环境没问题，一旦部署到测试或生产环境突然就启动报错，报类似`ClassNotFound的Exception`。为什么会产生Jar包冲突?

作为 Java 开发人员，我们可能会使用 Maven 维护许多应用程序以进行依赖项管理。这些应用程序需要不时升级以保持最新状态并添加新功能或安全更新。

由于某些依赖项之间的冲突，这个简单的任务 - 更新依赖项的版本，很容易变成一场噩梦。解决这些依赖冲突可能需要很多时间。

### 直接依赖与传递依赖

Maven 中有两种类型的依赖项：**直接依赖项** 和 **传递依赖项**

**直接依赖项**： 部分中明确包含在我们的项目对象模型 ( pom.xml) 文件中的依赖项\<dependencies>。可以使用\<dependency>标签添加它们。以下是添加到pom.xml文件中的日志库示例：

```xml
<dependencies>
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>1.2.17</version>
    </dependency>
</dependencies>
```


**传递依赖项**：我们作为依赖项包含在项目中的项目，如上面的日志库，可以在pom.xml文件中声明自己的依赖项。然后将这些依赖项视为对我们项目的传递性依赖项。当 Maven 拉取一个直接依赖时，它也会拉取它的传递依赖。

现在我们对 Maven 中的不同依赖类型有了一个概述，让我们详细看看 Maven 如何处理项目中的传递依赖。

作为示例，我们将查看 Spring 框架中的两个依赖项：spring-context和spring-security-web.

在pom.xml文件中我们将它们添加为直接依赖项，特意选择了两个不同的版本号：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.3.5</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-web</artifactId>
        <version>5.4.5</version>
    </dependency>
</dependencies>
```



### 使用依赖树可视化版本冲突

不了解传递依赖的人会认为使用这个依赖声明只会拉取两个 JAR 文件。幸运的是，Maven 提供了一个命令，它将向我们展示与这两个依赖项相关的确切内容。

我们可以使用以下命令列出所有依赖项，包括可传递的依赖项：

```shell
mvn dependency:tree -Dverbose=true
```

我们使用此命令的详细模式，以便 Maven 告诉我们选择一个版本的依赖项而不是另一个版本的原因。

结果是这样：

```shell
+- org.springframework:spring-context:jar:5.3.5:compile
|  +- org.springframework:spring-aop:jar:5.3.5:compile
|  |  +- (org.springframework:spring-beans:jar:5.3.5:compile - omitted for duplicate)
|  |  \- (org.springframework:spring-core:jar:5.3.5:compile - omitted for duplicate)
|  +- org.springframework:spring-beans:jar:5.3.5:compile
|  |  \- (org.springframework:spring-core:jar:5.3.5:compile - omitted for duplicate)
...
   +- (org.springframework:spring-expression:jar:5.2.13.RELEASE:compile - omitted for conflict with 5.3.5)
   \- org.springframework:spring-web:jar:5.2.13.RELEASE:compile
      +- (org.springframework:spring-beans:jar:5.2.13.RELEASE:compile - omitted for conflict with 5.3.5)
      \- (org.springframework:spring-core:jar:5.2.13.RELEASE:compile - omitted for conflict with 5.3.5)
```

我们从两个依赖开始，在这个输出中，我们发现 Maven 拉取了额外的依赖。这些额外的依赖只是传递性的。

我们可以看到树中有相同依赖项的不同版本。例如，有两个版本的spring-beans依赖项：5.2.13.RELEASE和5.3.5

Maven解决了此版本冲突，但是如何解决？什么是省略了重复和省略的冲突是什么意思？

#### Maven 如何解决版本冲突？

首先要知道的是，Maven 无法对版本进行排序：版本是任意字符串，可能不遵循严格的语义序列。例如，如果我们有两个版本1.2和 1.11，我们知道1.11在 之后1.2但字符串比较给出了1.11之前1.2。其他版本值可以是1.1-rc1 或 1.1-FINAL，这就是为什么按 Maven 对版本进行排序不是解决方案的原因。

这意味着 Maven 不知道哪个版本是新的或旧的，并且不能选择总是采用最新的版本。

其次，Maven采用树深度最近的传递依赖并且根据解析顺序中排第一位的版本去解决。为了理解这一点，让我们看一个例子：

我们从一个 POM 文件开始，它有一些具有传递依赖关系的依赖关系（简而言之，所有的依赖关系都用字母 D 表示）：

```shell
D1(v1) -> D11(v11) -> D12(v12) -> DT(v1.3)
D2(v2) -> DT(v1.2)
D3(v3) -> D31(v31) -> DT(v1.0)
D4(v4) -> DT(v1.5)
```

请注意，每个直接依赖项都会引入不同版本的DT依赖项。

Maven 将创建一个依赖树，并按照上面提到的标准，将选择一个依赖项DT：

![](https://img-blog.csdnimg.cn/20210530191501509.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0V2YW5fTGV1bmc=,size_16,color_FFFFFF,t_70#pic_center)


我们注意到解析顺序在选择DT依赖项方面发挥了重要作用，因为v1.2和v1.5具有相同的深度，但v1.2在解析顺序中排在第一位。所以即使v1.2不是 的最后一个版本DT，Maven 也选择了它来使用。

如果我们想v1.5在这种情况下使用 version ，我们可以简单地在我们的 POM 文件中添加D4之前的依赖项D2。在这种情况下，v1.5将首先按照解析顺序，Maven 将选择它。

因此，为了帮助我们理解上面的依赖树结果，Maven 为每个传递依赖指明了为什么它被省略：

* “省略重复” 意味着 **Maven 更喜欢另一个具有相同名称和版本的依赖项而不是这个依赖项（即另一个依赖项根据解析顺序和深度具有更高的优先级）**

* “因冲突而省略” 意味着 Maven 更喜欢另一个具有相同名称但版本不同的依赖项（即根据解析顺序和深度，具有不同版本的另一个依赖项具有更高的优先级）

现在我们很清楚 Maven 是如何解决传递依赖的。出于某种原因，有一天我们可能会选择一个特定版本的依赖项，并摆脱 Maven 为选择它所做的所有过程。为此，我们有两个选择: **覆盖传递依赖版本**  和  **使用直接依赖覆盖传递依赖版本**

#### 覆盖传递依赖版本

如果我们想自己解决依赖冲突，我们必须告诉 Maven 选择哪个版本。有两种方法可以做到这一点。

#### 使用直接依赖覆盖传递依赖版本

对于有子模块的项目，为了确保所有模块之间的兼容性和一致性，我们需要一种方法来在所有子模块之间提供相同版本的依赖项。为此，我们可以使用以下dependencyManagement部分：它为 Maven 提供了一个查找表，以帮助确定传递依赖项的选定版本并集中依赖项信息。

一个dependencyManagement 部分包含依赖项元素。每个依赖项都是 Maven 的查找参考，以确定要为传递（和直接）依赖项选择的版本。在本节中，依赖项的版本是强制性的。但是，在该dependencyManagement 部分之外，我们现在可以省略依赖项的版本，Maven 将从中提供的依赖项列表中选择正确版本的传递依赖项dependencyManagement。

需要注意的是，在 dependencyManagement 中定义一个依赖并不会直接被安装到项目的依赖树中，它仅用于查找引用。

理解使用的更好方法dependencyManagement是通过一个例子。让我们回到之前的 Spring 依赖示例。现在我们要玩spring-beans依赖。当我们执行命令时mvn dependency:tree，解析的版本spring-beans是5.3.5.

使用dependencyManagement我们可以覆盖这个版本并选择我们想要的版本。我们所要做的就是将以下内容添加到我们的 POM 文件中：

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-beans</artifactId>
            <version>5.2.13.RELEASE</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```


现在我们希望 Maven 解析version 5.2.13.RELEASE而不是5.3.5.

让我们再执行 `mvn dependency:tree` 一次命令。结果是：

```xml
+- org.springframework:spring-context:jar:5.3.5:compile
|  +- org.springframework:spring-aop:jar:5.3.5:compile
|  +- org.springframework:spring-beans:jar:5.2.13.RELEASE:compile
|  +- org.springframework:spring-core:jar:5.3.5:compile
|  |  \- org.springframework:spring-jcl:jar:5.3.5:compile
|  \- org.springframework:spring-expression:jar:5.3.5:compile
\- org.springframework.security:spring-security-web:jar:5.4.5:compile
   +- org.springframework.security:spring-security-core:jar:5.4.5:compile
   \- org.springframework:spring-web:jar:5.2.13.RELEASE:compile
```

在依赖关系树，我们找到了5.2.13.RELEASE版本spring-beans。这是我们希望 Maven 为每个spring-beans传递依赖解析的版本。

如果spring-beans是直接依赖，为了利用该dependencyManagement部分，我们将不再需要在添加依赖时设置版本：

```xml-dtd
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-beans</artifactId>
</dependency>
```

这样，Maven 将使用dependencyManagement部分中提供的信息解析版本。

### 通过IDEA快捷解决依赖冲突

#### 查找冲突

打开pom.xml,点击右键 【Diagrams】->【Show Dependencies】

![](img/%E6%88%AA%E5%B1%8F2022-03-29%20%E4%B8%8B%E5%8D%882.21.50.png)

IDEA如果发现Jar冲突，会用红色线高亮现实。在这里，我们可以发现pom.xml中有两个版本的scala-library包,分别是2.12.4和2.12.6

![截屏2022-03-29 下午1.32.24](img/%E6%88%AA%E5%B1%8F2022-03-29%20%E4%B8%8B%E5%8D%881.32.24.png)



#### 解决冲突

这时候我们只需要把其中一个包选中右键选择【Exclude】，IDEA 就会自动帮我们把对应的jar排除掉，这里我选择的是把低版本移除

![](img/1.png)

冲突Jar移除后，新的依赖图已经没有了红线，而且IDEA会自动在pom.xml文件添加对应排除项