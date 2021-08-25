##  Jenkins — 01 Jenkins快速入门



## 一、CI/CD相关概念

 **CI**（Continuous integration，中文意思是**持续集成**）是一种软件开发时间。**持续集成的目的，就是让产品可以快速迭代，同时还能保持高质量**。它的核心措施是，代码集成到主干 之前，必须通过自动化测试。只要有一个测试用例失败，就不能集成。

 通过持续集成， 团队可以快速的从一个功能到另一个功能，简而言之，敏捷软件开发很大一部分都要归功于持续集成。持续集成强调开发人员提交了新代码之后，立刻进行构建、（单元）测试。根据测试结果，我们可以确定新代码和原有代码能否正确地集成在一起。可以借助下图对CI加以理解。

<img src="https://upload-images.jianshu.io/upload_images/6464255-1b6e3bfdbece1492.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1000" alt="img" style="zoom:70%;" />

根据持续集成的设计，代码从提交到生产，整个过程有以下几步。 

1. **提交**  流程的第一步，是开发者向代码仓库提交代码。所有后面的步骤都始于本地代码的一次提交 （commit）。

2.  **测试（第一轮）** 代码仓库对commit操作配置了钩子（hook），只要提交代码或者合并进主干，就会跑自动化测试。
3.  **构建**  通过第一轮测试，代码就可以合并进主干，就算可以交付了。 交付后，就先进行构建（build），再进入第二轮测试。所谓构建，指的是将源码转换为可以运行的实 际代码，比如安装依赖，配置各种资源（样式表、JS脚本、图片）等等。
4.  **测试（第二轮）** 构建完成，就要进行第二轮测试。如果第一轮已经涵盖了所有测试内容，第二轮可以省略，当然，这时 构建步骤也要移到第一轮测试前面。
5.  **部署**  过了第二轮测试，当前代码就是一个可以直接部署的版本（artifact）。将这个版本的所有文件打包（ tar filename.tar * ）存档，发到生产服务器。 
6. **回滚**  一旦当前版本发生问题，就要回滚到上一个版本的构建结果。最简单的做法就是修改一下符号链接，指向上一个版本的目录。



**持续集成的好处**

1. 降低风险，由于持续集成不断去构建，编译和测试，可以很早期发现问题，所以修复的代价就少；

2. 对系统健康持续检查，减少发布风险带来的问题； 
3. 减少重复性工作； 
4. 持续部署，提供可部署单元包；
5. 持续交付可供使用的版本； 
6. 增强团队信心；



**CD**（Continuous Delivery， 中文意思**持续交付**）是在持续集成的基础上，将集成后的代码部署到更贴近真实运行环境(类生产环境)中。比如，我们完成单元测试后，可以把代码部署到连接数据库的Staging环境中更多的测试。如果代码没有问题，可以继续手动部署到生产环境。如下图所示反应的是CI/CD 的大概工作模式。

<img src="https://upload-images.jianshu.io/upload_images/6464255-ba088ec7257062c0.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1000" alt="img" style="zoom:60%;" />





## 二、Jenkins 介绍

![img](https://image.easyblog.top/u=4181632889,1695848186&fm=26&fmt=auto&gp=0.jpg)

Jenkins是一个开源的、界面操作友好的持续集成(CI)工具，起源于Hudson（Hudson是商用的），**主要用于持续、自动的构建/测试软件项目、监控外部任务的运行**。**Jenkins用Java语言编写，可在Tomcat等流行的servlet容器中运行，也可独立运行**。通常与常见的的版本控制工具SVN、GIT以及构建工具Maven、Ant、Gradle结合使用。官网： http://jenkins-ci.org/。

**Jenkins的特征**： 

* 开源的Java语言开发持续集成工具，支持持续集成，持续部署。 
* 易于安装部署配置：可通过yum安装,或下载war包以及通过docker容器等快速实现安装部署，可方便web界面配置管理。
*  消息通知及测试报告：集成RSS/E-mail通过RSS发布构建结果或当构建完成时通过e-mail通知，生成JUnit/TestNG测试报告。
* 分布式构建：支持Jenkins能够让多台计算机一起构建/测试。 
* 文件识别：Jenkins能够跟踪哪次构建生成哪些jar，哪次构建使用哪个版本的jar等。 
* 丰富的插件支持：支持扩展插件，你可以开发适合自己团队使用的工具，如git，svn，maven， docker等。





## 三、Jenkins 安装和持续集成环境的配置

### 3.1 安装Jenkins

首先前往Jenkins官网下载稳定版本的package，地址：https://www.jenkins.io/download/

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210824230025.png)

#### （1）使用yum安装Jenkins

本教程在CentOS 7 服务器上Jenkins进行安装，参考官网的安装教程，执行以下命令进行安装：

```shell
# yum 源导入
wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
# 导入密钥
rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
# 安装
yum install -y jenkins
```

#### （2）检查并开放系统端口

Jenkins默认的启动的端口号是8080，如果系统8080端口已经被别的服务使用了，那么可以使用命令 `vim /etc/sysconfig/jenkins` 编辑Jenkins配置文件对其启动端口进行修改。

```shell
# 端口可以在 /etc/sysconfig/jenkins 文件中修改，默认为 8080
firewall-cmd --add-port=8080/tcp --permanent
firewall-cmd --reload
```

#### （3）安装配置JDK环境

Jenkins是使用java开发的，那么他想运行起来肯定需要java环境，因此我们需要首先配置一下jdk环境，输入 `vim /etc/profile` 命令添加jdk安装目录，如下配置示例

```shell
JAVA_HOME=/usr/local/jdk1.8.0_261
JRE_HOME=${JAVA_HOME}/jre
CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
PATH=${PATH}:$JAVA_HOME/bin:$JRE_HOME/bin:/home/lib
export JAVA_HOME JRE_HOME CLASSPATH PATH LD_LIBRARY_PATH
```

之后需要修改jenkins启动脚本，执行`vim /etc/init.d/jenkins` 命令将上面我们配置的Java路径配置到jenkins的candidates参数中，Jenkins在启动的时候会挨个扫描candidates中配置的路径，直到扫描到jdk环境，如果扫描没有发现jdk环境，那么启动就会报错。

```shell
candidates="
/etc/alternatives/java
/usr/lib/jvm/java-1.8.0/bin/java
/usr/lib/jvm/jre-1.8.0/bin/java
/usr/lib/jvm/java-11.0/bin/java
/usr/lib/jvm/jre-11.0/bin/java
/usr/lib/jvm/java-11-openjdk-amd64
/usr/bin/java
/usr/local/jdk1.8.0_261/bin/java
"
```



#### （4）启动 Jenkins 并设置开机启动

配置好jdk环境之后其实就可以启动Jenkins服务了，但是为了以后Jenkins可以随着机器开机自动启动，需要做以下配置：

```shell
#重载服务（由于前面修改了 Jenkins 启动脚本）
systemctl daemon-reload
 
#启动 Jenkins 服务，执行命令之后没有报错就说明启动成功了，另外还可以使用jps命令查看运行中的jenkins线程
systemctl start jenkins
 
#将 Jenkins 服务设置为开机启动
#由于 Jenkins 不是 Native Service，所以需要用 chkconfig 命令而不是 systemctl 命令
/sbin/chkconfig jenkins on
```



<center><img src="img/%E6%88%AA%E5%B1%8F2021-08-25%20%E4%B8%8A%E5%8D%8810.08.17.png" alt="启动成功的Jenkins" style="zoom:90%;" /><br/>启动成功的Jenkins</center>



### 3.2 Jenkins 初始化

#### 3.2.1 获取Jenkins解锁密码

之后默认情况下jenkins会在8080端口启动，我的服务ip地址是47.99.161.205 ，因此我们在浏览器地址栏输入：http://47.99.161.205:8080 访问启动的Jenkins，首次使用会看到如下初始化界面：

![](img/%E6%88%AA%E5%B1%8F2021-08-25%20%E4%B8%8A%E5%8D%8810.11.13.png)

根据页面提示，我们需要执行命令：`cat /var/lib/jenkins/secrets/initialAdminPassword` 查看管理员密码，将此密码复制到输入框解锁 Jenkins 。

#### 3.2.2 安装插件

之后跳转到安装插件页面，选择默认的选项让jenkins安装插件，这个过程如果网络好的话非常的快。如果网络不好进场卡住，可暂时先跳过该步骤。

![](img/%E6%88%AA%E5%B1%8F2021-08-25%20%E4%B8%8A%E5%8D%8810.14.19.png)

如下图所示是Jenkins正在安装推荐的插件的页面截图，这个过程正常情况下会持续3~5分钟左右。

![](img/%E6%88%AA%E5%B1%8F2021-08-25%20%E4%B8%8A%E5%8D%8810.15.33.png)



#### 3.2.3 添加一个管理员账户，并登录 Jenkins

插件安装完成之后会再次跳转到另一个页面，Jenkins会要求我们创建一个管理员用户，这一步可以直接点击下面的 `使用admin账户继续` 跳过这一步。

![](img/%E6%88%AA%E5%B1%8F2021-08-25%20%E4%B8%8A%E5%8D%8810.18.06.png)



#### 3.2.4 实例配置

实例配置一般保持默认值就好了，配置项的含义在Jenkins页面下面灰色字体也有解释。截止这里，是Jenkins首次启动需要初始化配置的信息，点击按钮 `保存并完成` Jenkins初始化工作就完成了。

![](img/%E6%88%AA%E5%B1%8F2021-08-25%20%E4%B8%8A%E5%8D%8810.21.30.png)



### 3.3 系统全局安全配置

进入Jenkins管理后台，为了防止忘记用户名密码丢失带来无法登录到 Jenkins 后台的问题，首先需要在 Jenkins `【系统管理】->【全局安全设置】` 里打开 `允许用户注册` 选项 。当然，这一步不设置也可以，但是前面设置的用户名和密码一定要妥善保管！

<img src="https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210824233144.png" alt="QQ截图20210824233144" style="width:70%;" />

### 3.4 系统配置

Jenkins要想正常工作就必须在 `【系统管理】-> 【全局工具配置】` 中配置jdk、Maven以及git在服务器上的安装路径，以便Jenkins可以执行他们的命令进行操作。下图是配置示意图。

**Maven\ Gradle配置**

Maven需要配置两项东西，一个是 `settings.xml` 文件的路径

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210822165457.png)

往下拉，拉到页面最下边另一个需要配置的是 Maven_HOME 的路径

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210822165538.png)



如果项目使用是gradle，那么也可以配置一下gradle：

![](img/27612-20160118223316093-2139077993.png)

**JDK 配置**

JDK配置一下别名以及 JAVA_HOME 的路径就可以了

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210822165511.png)



**Git 配置**

Git也不用过多配置，一般只用配置一下别名就好了

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210822165522.png)







**启动脚本配置**

如果部署过程中，还需要运行shell脚本，以及通过scp向远程机器上传文件，则需要配置下面这些：

![](img/27612-20160118223348140-1485606417.png)

注意 ⚠️  上图中红色标1的地方，填写的是本机的ssh私钥证书，红色标2的地方为证书的访问密码，如果未设置证书密码，直接留空，Root Repository Path为远程服务器的上传根目录，hostname即为远程服务器的机器名称（或IP）.



此外，还有一个强大的publish over ssh，不仅能向远程服务器上传文件，还能远程执行服务器上的Shell脚本，可参考下面配置：

![](img/27612-20160118223403484-785001627.png)



和SCP的参数设置类似，不过这个插件多出了一个Test Configuration的按钮，如果参数正确，点击该按钮后，会尝试连接到远程服务器，成功的话，会显示Success.
