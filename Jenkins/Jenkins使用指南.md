## 一、jenkins是什么？

Jenkins是一个开源的、界面操作友好的持续集成(CI)工具，起源于Hudson（Hudson是商用的），**主要用于持续、自动的构建/测试软件项目、监控外部任务的运行**。**Jenkins用Java语言编写，可在Tomcat等流行的servlet容器中运行，也可独立运行**。通常与常见的的版本控制工具SVN、GIT以及构建工具Maven、Ant、Gradle结合使用。





## 二、CI/CD是什么？

 **CI**(Continuous integration，中文意思是**持续集成**)是一种软件开发时间。持续集成强调开发人员提交了新代码之后，立刻进行构建、（单元）测试。根据测试结果，我们可以确定新代码和原有代码能否正确地集成在一起。可以借助下图对CI加以理解。

<img src="https://upload-images.jianshu.io/upload_images/6464255-1b6e3bfdbece1492.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1000" alt="img" style="zoom:70%;" />

**CD**(Continuous Delivery， 中文意思**持续交付**)是在持续集成的基础上，将集成后的代码部署到更贴近真实运行环境(类生产环境)中。比如，我们完成单元测试后，可以把代码部署到连接数据库的Staging环境中更多的测试。如果代码没有问题，可以继续手动部署到生产环境。下图反应的是CI/CD 的大概工作模式。

<img src="https://upload-images.jianshu.io/upload_images/6464255-ba088ec7257062c0.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1000" alt="img" style="zoom:60%;" />



## 三、Jenkins 下载&安装&启动

### 安装Jenkins

到Jenkins官网下载稳定版本的package，地址：https://www.jenkins.io/download/



<img src="img/%E6%88%AA%E5%B1%8F2021-08-18%20%E4%B8%8B%E5%8D%885.51.00.png" alt="截屏2021-08-18 下午5.51.00" style="zoom:40%;" />



参考官网的安装教程，执行以下命令进行安装：

```shell
# yum 源导入
wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
# 导入密钥
rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
# 安装
yum install -y jenkins
```



### 开放端口

```shell
# 端口可以在 /etc/sysconfig/jenkins 文件中修改，默认为 8080
firewall-cmd --add-port=8080/tcp --permanent
firewall-cmd --reload
```





### 安装配置JDK环境

```shell
JAVA_HOME=/usr/local/jdk1.8.0_261
JRE_HOME=${JAVA_HOME}/jre
CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
PATH=${PATH}:$JAVA_HOME/bin:$JRE_HOME/bin:/home/lib
export JAVA_HOME JRE_HOME CLASSPATH PATH LD_LIBRARY_PATH
```



### 启动 Jenkins 并设置开机启动

首先需要修改jenkins启动脚本，执行`vim /etc/init.d/jenkins` 命令将上面我们配置的Java路径配置到jenkins的candidates参数中

```java
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

设置Jenkins开机启动

```shell
#重载服务（由于前面修改了 Jenkins 启动脚本）
systemctl daemon-reload
 
#启动 Jenkins 服务
systemctl start jenkins
 
#将 Jenkins 服务设置为开机启动
#由于 Jenkins 不是 Native Service，所以需要用 chkconfig 命令而不是 systemctl 命令
/sbin/chkconfig jenkins on
```

### **Jenkins 初始化**

之后默认情况下jenkins会在8080端口启动，我们输入服务器ip:8080 查看启动的Jenkins，首次使用会看到如下初始化界面

<img src="img/%E6%88%AA%E5%B1%8F2021-08-18%20%E4%B8%8B%E5%8D%887.20.13.png" style="width:95%;" />

此时我们需要使用命令：`cat /var/lib/jenkins/secrets/initialAdminPassword` 查看密码，在 Jenkins 管理页输入来解锁，之后选择默认配置安装插件即可：

<img src="img/%E6%88%AA%E5%B1%8F2021-08-18%20%E4%B8%8B%E5%8D%887.23.04.png" alt="截屏2021-08-18 下午7.23.04" style="width:90%;" />

注意⚠️	首次启动可能会遇到页面一直显示"Please wait while Jenkins is getting ready to work ..."，编辑 `/var/lib/jenkins/hudson.model.UpdateCenter.xml` 文件，将"https://updates.jenkins.io/update-center.json" 修改为 "http://mirror.xmission.com/jenkins/updates/update-center.json" 即可。



## 三、使用Jenkins进行Java代码(单元)测试、打包。

安装好，能成功访问，紧接着就进行自动化构建项目配置。



首先我们还需要安装两个工具Maven和Git：

安装git的目的是在自动化部署前实时从git远程仓库中拉取最新的代码，linux中可以使用命令 `yum -y install git`进行下载， 需要注意的是在安装Git的时候，一定要使用命令  `ssh-keygen -t rsa -C "youremail@abc.com"`  生成密钥，以便Jenkins可以全自动拉取代码，不然每次都输入密码那就无法首先自动化部署了。

 安装maven的目的是通过项目中的pom.xml文件自动解决项目依赖问题，构建项目。linux中通过wget+下载链接下载maven的zip包然后解压即可。配置maven环境变量：

```shell
vim /etc/profile
 
//在这个文件末尾加上
export MAVEN_HOME=/usr/local/maven3.4.5
export PATH=$MAVEN_HOME/bin:$PATH
 
//保存后在命令行输入,启动配置
. /etc/profile
```

