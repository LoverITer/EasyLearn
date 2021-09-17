##  Jenkins — 创建流水线（Pipeline）作业



## 一、什么是流水线？

首先，了解 Jenkins 本身并不是流水线这一点很有帮助。只是创建一个新的 Jenkins 作业并不能构建一条流水线。可以把 Jenkins 看做一个遥控器，在这里点击按钮即可。当你点击按钮时会发生什么取决于遥控器要控制的内容。Jenkins 为其他应用程序 API、软件库、构建工具等提供了一种插入 Jenkins 的方法，它可以执行并自动化任务。Jenkins 本身不执行任何功能，但是随着其它工具的插入而变得越来越强大。

流水线是一个单独的概念，指的是按顺序连接在一起的事件或作业组：

> “ *流水线(pipeline)*”是可以执行的一系列事件或作业。

理解流水线的最简单方法是可视化一系列阶段，如下所示：

![](https://pic2.zhimg.com/80/v2-83827703885624ce359a59ab639183e1_1440w.jpg)



在这里，你应该看到两个熟悉的概念： **阶段(Stage)**和 **步骤(Step)**。

- **阶段**：一个包含一系列步骤的块。阶段块可以命名为任何名称；它用于可视化流水线过程。
- **步骤**：表明要做什么的任务。步骤定义在阶段块内。

Jenkins 流水线以**脚本**的形式提供，通常称为 “Jenkinsfile”。下面这是一个简单的 Jenkins 流水线文件的示例：

```shell
// Example of Jenkins pipeline script
pipeline {
  stages {
    stage("Build") {
      steps {
          // Just print a Hello, Pipeline to the console
          echo "Hello, Pipeline!"
          // Compile a Java file. This requires JDKconfiguration from Jenkins
          javac HelloWorld.java
          // Execute the compiled Java binary called HelloWorld. This requires JDK configuration from Jenkins
          java HelloWorld
          // Executes the Apache Maven commands, clean then package. This requires Apache Maven configuration from Jenkins
          mvn clean package ./HelloPackage
          // List the files in current directory path by executing a default shell command
          sh "ls -ltr"
      }
    }
   // And next stages if you want to define further...
  } // End of stages
} // End of pipeline
```

从示例脚本很容易看到 Jenkins 流水线的结构。如下图所示是Jenkins完整的流水线脚本结构示意图：

<img src="https://upload-images.jianshu.io/upload_images/2916037-4c95c6cfaf3a2bb1.png?imageMogr2/auto-orient/strip|imageView2/2/w/359" style="width:25%;" />



#### 1.1 agent

- 指定整个流水线或者一个特定的阶段在哪里运行
  - agent any：该流水线或阶段可以运行在任何一个定义好的代理节点上
  - agent none：在顶端时，代表不设置一个全局的代理节点，同时也代表，如有必要，需要为单个阶段指定一个代理节点
  - agent { label "\<label>" }：表明该流水线或阶段可以运行在任何一个具有<label>标签的代理节点上
  - agent {docker '\<image>'}：docker节点，这个在后续学习中继续补充

#### 1.2 environment

- 用于设置环境变量，可定义在stage或pipeline部分
- 在流水线代码顶部定义的环境，将使流水线所有的步骤都可以访问变量
- 在一个阶段中定义的环境，只能在这个阶段范围内访问变量

**示例**

```groovy
pipeline{
    environment{
         FIRSTNAME = "san"
         LASTNAME = "zhang"
         USERNAME = "${LASTNAME}${FIRSTNAME}"
    }
    agent any
    stages{
        stage('build'){
            steps{
                echo "我的名字${USERNAME}"
            }
        }
    }
}
```

#### 1.3 tools

- 可定义在pipeline或stage部分
- 会自动下载下载并安装我们指定的工具，并将其加入PATH变量中
- 在全局工具配置(Global Tool Configuration)界面配置工具版本，必须是选择了自动安装

![](https://upload-images.jianshu.io/upload_images/2916037-eabc7cca6b376944.png?imageMogr2/auto-orient/strip|imageView2/2/w/442)

**示例**

```groovy
pipeline{
    agent any
    tools{
        maven 'maven3-6'
    }
    stages{
        stage('build'){
            steps{
                sh 'mvn clean test install'
                echo "tools命令自动安装maven"
            }
        }
    }
}
```

#### 1.4 options

- 用于配置jenkins pipeline本身的选项
- 指定一些属性和值，这些预定义的选项可以应用到整个流水线
- 这些属性一般都可以在jenkins web表单进行基本的配置

#### 1.5 triggers

- 这些触发器并不适用于多分支流水线、Github组织或者Bitbuchet团队/项目等类型的任务
- 需要手动触发一次任务，让jenkins加载后，trigger指令才会生效

#### 1.6 parameters

> 参数化pipeline是指可以通过传参来决定pipeline的行为
>
> 参数化让写pipeline就像写函数，可重用

- 只能放在pipeline块下
- 三个参数
  - defaultValue：默认值
  - description：参数的描述信息
  - name：参数名

#### 1.7 stages

stages是pipeline中最主要的组成部分，一个pipeline中只允许有一个stages，Jenkins将会按照stages中描述的顺序从上往下的执行。

#### 1.8 stage

阶段，一个stages中可以有多个stage，每个stage都是一个标志性的操作过程，比如：拉取代码、编译、运行单元测试、部署等，在编写jenkinsfile的时候，每个stage所代表的阶段一定要写明

#### 1.9 steps

步骤，steps是最基本的操作单元，可以打印一句话，也可以构建一个Docker镜像

#### 1.10 post

声明式流水线通过其[post部分](https://jenkins.io/doc/book/pipeline/syntax/#post)支持健壮的失败处理，允许声明多个不同的“post条件”。 例如: `always`, `unstable`, `success`, `failure`和`changed`，如下所示是post块内部支持的条件：

* `always` 
  不管管道或stage的运行完成状态如何，在post部分运行这些步骤。 
* `changed` 
  如果当前管道或阶段的运行与之前的运行有不同的完成状态，则只运行post中的步骤。 
* `fixed` 
  如果当前管道或阶段的运行成功，且前一次运行失败或不稳定，则只运行post中的步骤。 
* `regression` 
  如果当前管道或阶段的运行状态为失败、不稳定或中止，且前一次运行成功，则只运行post中的步骤。 
* `aborted` 
  只有在当前管道或阶段的运行有“中止”状态时才运行post中的步骤，通常是由于管道被手动中止。这通常用web UI中的灰色表示。 
* `failure` 
  只有在当前管道或阶段的运行有“失败”状态时才运行post中的步骤，通常在web UI中表示为红色。 
* `success` 
  只有当当前管道或阶段的运行具有“成功”状态时，才运行post中的步骤，通常在web UI中以蓝色或绿色表示。 
* `unstable` 
  如果当前管道或阶段的运行状态为“不稳定”状态，通常是由测试失败、代码违规等引起的，那么只在post中运行这些步骤。 
* `cleanup` 
  不管管道或阶段的状态如何，在所有其他的post条件被评估之后，运行这个post条件下的步骤。

**示例**

```shell
pipeline {
    agent any
    stages {
        stage('Test') {
            steps {
                sh 'make check'
            }
        }
    }
    post {
        always {
            junit '**/target/*.xml'
        }
        failure {
            mail to: team@example.com, subject: 'The Pipeline failed :('
        }
    }
}
```



## 二、创建流水线

好。既然你已经了解了 Jenkins 流水线是什么，我将向你展示如何创建和执行 Jenkins 流水线。在本教程的最后，你将建立一个 Jenkins 流水线，如下所示：

![](https://pic4.zhimg.com/80/v2-c234d567676377ff57901445107736e7_1440w.jpg)

### 2.1 创建一个流水线作业

在此步骤中，你可以选择并定义要创建的 Jenkins 作业类型。选择 “Pipeline” 并为其命名（例如，“TestPipeline”）。单击 “OK” 创建流水线作业。

![](img/%E6%88%AA%E5%B1%8F2021-08-25%20%E4%B8%8B%E5%8D%884.29.48.png)

你将看到一个 Jenkins 作业配置页面。向下滚动以找到 “Pipeline” 部分。有两种执行 Jenkins 流水线的方法。一种方法是在 Jenkins 上直接编写流水线脚本，另一种方法是从 SCM（源代码管理）中检索 Jenkins 文件。在接下来的两个步骤中，我们将体验这两种方式。



### 2.2 通过直接脚本配置并执行流水线作业

要使用直接脚本执行流水线，请首先复制下来我在gitee上托管的 [Jenkinsfile 示例](https://gitee.com/LoveITer/CICDPractice)的内容。选择 “Pipeline script” 作为 “Destination”，然后将该 Jenkinsfile 的内容粘贴到 “Script” 中。

注意 ⚠️  共有三个阶段：Build、Test 和 Deploy，它们是任意的，可以是任何一个。每个阶段中都有一些步骤；在此示例中，它们只是打印一些随机消息。

![](img/%E6%88%AA%E5%B1%8F2021-08-25%20%E4%B8%8B%E5%8D%884.44.37.png)

单击 “Save” 以保留更改，程序会自动将你带回到 “Job Overview” 页面。



要开始构建流水线的过程，需要单击 “Build Now”。如果一切正常，你将看到如下图所示流水线。

![](img/%E6%88%AA%E5%B1%8F2021-08-25%20%E4%B8%8B%E5%8D%884.47.29.png)

要查看流水线脚本构建的输出，请单击任何阶段，然后单击 “Log”。你会看到这样的消息。

![](img/%E6%88%AA%E5%B1%8F2021-08-25%20%E4%B8%8B%E5%8D%884.49.54.png)



### 2.3 通过 SCM 配置并执行流水线作业

现在，换个方式：在此步骤中，你将通过从源代码控制的 Gitee 中复制 Jenkinsfile 来部署相同的 Jenkins 作业。在同一个 [GitHub 存储库](https://gitee.com/LoveITer/CICDPractice)中，通过单击 “Clone or download” 并复制其 URL 来找到其存储库 URL。

之后回到Jenkins配置页面，滚动到 “Advanced Project Options” 设置，但这一次，从 “Destination” 下拉列表中选择 “Pipeline script from SCM” 选项。将 GitHub 存储库的 URL 粘贴到 “Repository URL” 中，然后在 “Script Path” 中键入 “Jenkinsfile”。 单击 “Save” 按钮保存。

<img src="img/%E6%88%AA%E5%B1%8F2021-08-25%20%E4%B8%8B%E5%8D%884.55.09.png" style="width:67%;" />

要构建流水线，回到 “Task Overview” 页面后，单击 “Build Now” 以再次执行作业。结果与之前相同，除了多了一个称为 “Declaration: Checkout SCM” 的阶段。

![](img/%E6%88%AA%E5%B1%8F2021-08-25%20%E4%B8%8B%E5%8D%884.57.47.png)



## 参考资料

* [1] [用 Jenkins 构建 CI/CD 流水线](https://zhuanlan.zhihu.com/p/90612874)
* [2] [jenkins声明式流水线](https://www.jianshu.com/p/d3ed400d3d4a)

