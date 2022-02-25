### Vue实战—手把手教你用vue-cli搭建vue项目

> 本篇主要是介绍如何使用vue-cli来快速搭建vue项目，继续阅读本文其中前提是在你的机器上node和npm已经安装好，文章结尾将会简单提到一个简单的例子。使用vue-cli搭建项目最开始我也是看网上的教程一步步搭下来，所以其中的一些步骤说法为了表达正确会进行一定参考。



### 一、 项目使用技术、框架介绍

vue-cli 是一个官方发布 vue.js 项目脚手架，使用 vue-cli 可以快速创建 vue 项目，步骤很简单，输入几个命令之后就会生成整个项目，里面包括了webpack、ESLint、babel很多配置等等，省了很多事。根据小高之前给的要求以及结合DSS项目中的经验，我们今天搭建的项目结构组成为：

**Vue+ ESLint + webpack +ES6**

* **Vue:** 主要框架

* **ESLint:** 帮助我们检查Javascript编程时的语法错误，这样在一个项目中多人开发，能达到一致的语法

* **Webpack:** 设置代理、插件和loader处理各种文件和相关功能、打包等功能。整个项目中核心配置

* **ES6:** 紧跟时代潮流，使用ES6语法，利用babel处理。



### 二、 安装vue-cli

1. 打开cmd ,敲入命令：

   ```shell
   cnpm install --global vue-cli
   ```

   ![](http://image.easyblog.top/2257137-dfaa4df79a0feca3.png)

   

2. 安装好之后打开 `C:\Users\Administrator\AppData\Roaming\npm`  可以看到这个文件夹下已经有相关的脚本文件，如图：
   ![img](https://i.imgur.com/8WIGUcx.png)

可以看到以上文件就表示vue-cli正常安装成功，接下来就可以正式创建vue-cli工程项目了。

### 三、 创建项目

根据官方的文档说明，使用vue-cli有两种可以创建项目的方式：一种是使用纯命令，另一种可以使用vue-cli提供的UI界面，具体使用哪种见仁见智，但本文将会对两种方式分别做介绍和演示。

#### 1、使用命令行创建vue项目

**（1）在工作目录下运行以下命令来创建一个新项目**

```shell
vue init webpack vue-test
```

如果在创建过程中遇到如下图片中所示的问题，请参考博友的这篇文章解决：[vue-cli Failed to download repo vuejs-templates/webpack解决办法](https://blog.csdn.net/qq_45731083/article/details/114921816?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_title~default-4.queryctrv2&spm=1001.2101.3001.4242.3&utm_relevant_index=7)

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220225215432.png)



**（2）合理选择创建过程中的选项参数**

在创建过程中有几个选项参数这里需要着重说明一下：

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220225221944.png)

* **“Project name”：**这个是项目名称，默认是输入时的那个名称，想改的话直接输入修改，也可以直接回车

* **“Install vue-router”：**是否需要vue-router，这里默认选择使用，这样生成好的项目就会有相关的路由配置文件

* **“Use ESLint to lint your code”：**是否使用ESLint，刚才说了我们这个项目需要使用所以也是直接回车，默认使用，这样会生成相关的ESLint配置

* **“Setup unit tests with Karma + Moch?”：**是否安装单元测试。由于我们现在还没有单元测试，所以这里选择的是”N”，而不是直接回车哦

* **“Setup e2e tests with Nightwatch”：**是否安装e2e测试，这里我也同样选择的是“N”

这几个配置选择yes 或者 no 对于我们项目最大的影响就是，如果选择了yes 则生成的项目会自动有相关的配置，有一些loader我们就要配套下载。所以如果我们确定不用的话最好不要yes，要么下一步要下很多没有用的loader。选择好之后等到vue-cli帮我们自动安装依赖



**（3）项目创建完成，初始化项目**

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220225221650.png)

当看到上图所示界面时表示vue项目已经创建成功，此时我们就可以进入项目目录并输入`npm install`命令执行项目初始化：

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220225222910.png)

**（4）启动项目**

此时我们就可以在命令行再次输入命令 `npm run dev` 启动项目：

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220225222629.png)

访问 [http://localhost:8080](http://localhost:8080) 就可以可以看到如下Web界面就表明vue项目基本骨架已经搭建好啦，接下就愉快的coding吧~~

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220225222835.png)

#### 2、使用图形化界面创建vue项目

**（1）输入命令，启动图形化界面**

vue-cli图形化界面是一中更简单地创建和维护vue项目的方式，要使用vue-cli图形化界面也非常容易，只需要在工作目录的命令行输入如下命令：

```java
vue ui
```

上述命令会打开一个浏览器窗口，并以图形化界面将你引导至项目创建的流程。

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220225223045.png)

上图是之前创建的一个vue项目的管理界面，创建新的vue项目跳转到[Vue 项目管理器](http://localhost:8001/project/select) 界面，点击【创建】以创建一个新项目：

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220225223546.png)



**（2）填写项目路径**

填写项目文件路径 并 填写一下git初始化信息，其余选项默认，之后点击【下一步】

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220225231525.png)

**（3）选择预设配置**

由于我之前已经使用过vue-cli图形化界面创建过项目并保存了一个预设，这里我就直接选择之前的预设了，如果你是第一次使用，你可以选择【手动配置项目】，并在项目创建成功的保存你的配置为预设，这样在下次创建项目的时候就可以直接使用预设配置了。

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220225224749.png)

随后，点击【创建项目】开始项目真正的创建：

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220225231626.png)

创建成功之后会自动跳转到此项目的管理界面：

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220225231750.png)





**（4）安装依赖和插件**

在管理界面侧边栏可以看到 【插件】和【依赖】两个Tab，我们可以在对应界面搜索并安装项目所需插件或依赖，比如下图所示安装 HTTP client `axios` ：

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220225232405.png)



**（5）启动项目**

使用图形化界面创建的项目可以使用 `npm run dev` 来启动，但是没必要，因为图形化界面提供了管理vue项目的一整套方案：在项目图形化管理界面侧边栏选择【任务】->【serve】然后点击按钮【运行】以启动项目，启动成功后如下图所示右侧红框会有一个绿色对钩。

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220225232749.png)

成功启动后的管理界面：

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220225233201.png)

### 参考

* [vue-cli官方文档](https://cli.vuejs.org/zh/guide/creating-a-project.html)

* [手把手教你用vue-cli搭建vue项目](https://www.cnblogs.com/liaoanran/p/8042893.html)

