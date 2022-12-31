## Gradle 快速入门： 安装和配置



## 一、Gradle简介

Gradle是源于Apache Ant和Apache Maven概念的项目自动化构建开源工具，它使用一种基于Groovy的特定领域语言(DSL)来声明项目设置，抛弃了基于XML的各种繁琐配置面向Java应用为主。当前其支持的语言暂时有Java、Groovy、Kotlin和Scala。

Gradle是一个基于JVM的构建工具，是一款通用灵活的构建工具，支持maven， Ivy仓库，支持传递性依赖管理，而不需要远程仓库或者是pom.xml和ivy.xml配置文件，基于Groovy，build脚本使用Groovy编写。



## 二、下载 & 安装

## 第一种安装方式

### 1）官网下载

官方网站：https://gradle.org/install/#manually

官方提供以下 2 种压缩包可供下载

- binary-only (如果不需要源码、文档，选择下载这个压缩包就够了)
- complete (包含文档及源码)

### 2） 解压

以下操作以 gradle 4.9 版本为例

#### 2-1. Linux & macOS 用户

将下载的压缩包解压到任意你想要存放的位置，如：

```shell
> mkdir /opt/gradle
> unzip -d /opt/gradle gradle-4.9-bin.zip
> ls /opt/gradle/gradle-4.9

LICENSE  NOTICE  bin  getting-started.html  init.d  lib  media
```

#### 2-2. Windows 用户

将下载的压缩包解压到任意你想要存放的位置，如：

> 在 c 盘新建一个 Gradle 目录
>
> 将下载回来的压缩包解压到 c:\Gradle 下
>
> 最终路径格式 c:\Gradle\gradle-4.9



## 第二种安装方式

通过包管理软件快速安装 gradle

### 1. macOS 用户

#### 通过 [Homebrew](http://brew.sh/) 安装

```shell
brew install gradle
```

### 2. Windows 用户

#### 2-1. 通过 [Scoop](http://scoop.sh/) 安装

一款灵感来自 Homebrew 的 Windows 系统包管理软件

```shell
scoop install gradle
```

#### 2-2. 通过 [Chocolatey](https://chocolatey.org/) 安装

另一款 Windows 系统包管理软件

```shell
choco install gradle
```

### 3. Linux 用户

#### 3-1. 通过 [SDKMAN!](http://sdkman.io/) 安装

```shell
skd install gradle
```



## 三、 配置 Gradle

解压压缩包即可完成安装，接下来需要对Gradle进行一些必要的配置。

#### 3.1 配置环境变量

（1）新建变量

* 变量名：GRADLE_HOME

* 变量值：解压到的目录

　　　　　　![img](https://img2018.cnblogs.com/blog/1463514/201909/1463514-20190904133702208-490021808.png)

（2）新建变量

* 变量名：GRADLE_USER_HOME

* 变量值：自定义Gradle仓库目录或者Maven的仓库目录

　　　　　　![img](https://img2018.cnblogs.com/i-beta/1463514/201911/1463514-20191107112830439-540725559.png)

（3） 添加变量

* 变量名：Path

* 变量值：%GRADLE_HOME%\bin;

　　　　　　![img](https://img2018.cnblogs.com/blog/1463514/201909/1463514-20190904134044671-1441312871.png)

配置完成之后，在命令行中输入 `gradle -v` 可以看到gradle打印版本信息就证明配置成功。

<img src="http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220221213946.png" style="zoom:67%;" />

#### 3.2 配置Gradle仓库源

在Gradle安装目录下的 init.d 文件夹下，新建一个 init.gradle 文件，里面填写以下配置。

```groovy
allprojects {
    repositories {
        maven { url 'file:///D:/gradle-7.4-all/gradle-7.4/resposity'}   //配置本地仓库地址
        mavenLocal()
        maven { name "Alibaba" ; url "https://maven.aliyun.com/repository/public" }
        maven { name "Bstek" ; url "http://nexus.bsdn.org/content/groups/public/" }
        mavenCentral()
    }

    buildscript {
        repositories {
            maven { name "Alibaba" ; url 'https://maven.aliyun.com/repository/public' }
            maven { name "Bstek" ; url 'http://nexus.bsdn.org/content/groups/public/' }
            maven { name "M2" ; url 'https://plugins.gradle.org/m2/' }
        }
    }
}
```

其中，repositories 中配置的是获取 jar 包的仓库顺序。先是本地的 Maven 仓库路径；接着的 mavenLocal() 是获取 Maven 本地仓库的路径，应该是和第一条一样，但是不冲突；第三条和第四条是从国内和国外的网络上仓库获取；最后的 mavenCentral() 是从Apache提供的中央仓库获取 jar 包。

#### 3.3 在IDEA中配置Gradle

在IDEA的Setting里打开【Build, Execution, Deployment】--> 【Build Tools】-->【Gradle】

勾选 【Use local Gradle distribution】，在 Gradle home 中选择安装的Gradle的路径。

如果在变量和配置文件中设置了Gradle的仓库路径，在 Service directory path 中就会自动填写地址，如果想改的话可以手动修改。