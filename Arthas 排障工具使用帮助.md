## Arthas 排障工具使用帮助

在线上环境作为你是否也遇到过如下问题：

- 这个类从哪个 jar 包加载的？为什么会报各种类相关的 Exception？
- 我改的代码为什么没有执行到？难道是我没 commit？分支搞错了？
- 遇到问题无法在线上 debug，难道只能通过加日志再重新发布吗？
- 线上遇到某个用户的数据处理有问题，但线上同样无法 debug，线下无法重现！
- 是否有一个全局视角来查看系统的运行状况？
- 有什么办法可以监控到JVM的实时运行状态？
- 怎么快速定位应用的热点，生成火焰图？
- 怎样直接从JVM内查找某个类的实例？

当你遇到以上类似问题而束手无策时，**Arthas**可以帮助你解决。

Arthas支持JDK 6+，支持Linux/Mac/Windows，采用命令行交互模式，同时提供丰富的 Tab 自动补全功能，进一步方便进行问题的定位和诊断。



## 一、安装&启动

> 前提：Arthas是一个java的程序，运行前需保证机器上有运行中的java进程，不然Arthas无法启动。自己没有现成demo可以使用ailiyun提供的demo：https://arthas.aliyun.com/arthas-demo.jar

下载Arthas ：`curl -O https://arthas.aliyun.com/arthas-boot.jar`

![](http://image.easyblog.top/167101424590579b23465-2e95-49c2-bd94-486bb4e75169.png)



接下来执行命令：`java -jar arthas-boot.jar` 启动Arthas

![](http://image.easyblog.top/16711869650739323a6d9-9761-4eff-af3c-b3fa72bd8cf6.png)



## 二、常用命令介绍

### （1）help命令：获取到更多帮助信息

### （2）dashboard命令

dashboard 可以查看当前系统的实时数据面板，数据默认每5秒刷新一次，可以输入`q`+`回车` 或 `Ctrl+C` 退出

![](http://image.easyblog.top/16714189159946bf1a941-b38f-4395-9de5-c419c8760245.png)



### （3）thread命令

thread单独使用可以打印出当前第一页线程快照。thread 命令后面也可以跟上线程id可以打印对应线程的栈。arthas 支持管道符，我们可以使用这个能力查找出应用中已知名称的线程，例如：`thread 1 | grep ‘main’`  可以打印出main线程的栈。

**当没有参数时，显示第一页线程的信息**：

默认按照 CPU 增量时间降序排列，只显示第一页数据。

![](http://image.easyblog.top/1671419994101aa8e0a21-742a-4bc1-b10d-9acbc4023e4c.png)



**thread 命令后跟线程号**：

![](http://image.easyblog.top/16714201679505e4e470b-9bce-4620-b3c7-a430375b86fe.png)





### （4）sc命令

sc命令可以用于查找JVM中已经加载类，例如查看名称为MathGame的类：`sc -d *MathGame`

![](http://image.easyblog.top/167142050329920568ccb-024e-42c3-9b2e-1af379372bf2.png)





### （5）jad 命令

`jad 类全限定名` 可以用来反编译代码，当我们疑惑线上跑的代码是否是我们修改过的代码是可以使用此命令来实现查看，例如我们查看 MathGame的代码：`jad demo.MathGame`

![](http://image.easyblog.top/1671420774978999dbf44-604d-4a46-9c79-e20a011e3c41.png)



使用jad我们还可以直接查看某个类中的某段方法，例如查看String类里的substring方法：`jad java.lang.String substring`

![](http://image.easyblog.top/1671433937141f7c47f8a-1007-4392-9a3a-7bff14d9bf9a.png)



### （6）watch命令

watch命令可以用来查看方法的：`返回值`、`抛出异常`、`入参`，通过编写 OGNL 表达式进行对应变量的查看。

* watch基本语法：`watch 全类名 方法名 [观察表达式] [option] ` 
* 观察表达式是由OGNL表达式组成，默认值是`{params,returnObj,target}` ，分别表示观察方法的`入参`、`返回值` 和 `this对象`
* 选项参数 option ：watch 命令定义了 4 个观察事件点，即 `-b` 函数调用前，`-e` 函数异常后，`-s` 函数返回后，`-f`函数结束后。4 个观察事件点 `-b`、`-e`、`-s` 默认关闭，`-f` 默认打开，当指定观察点被打开后，在相应事件点会对观察表达式进行求值并输出
* 在 watch 命令的结果里，会打印出`location`信息。`location`有三种可能值：`AtEnter`，`AtExit`，`AtExceptionExit`。对应函数入口，函数正常 return，函数抛出异常。
* watch 提供了 `-n` 命令用于控制观察的次数，例如 `-n 2` 就是对某个方法总共观察两次

如下是查看MathGame方法的入参和返回值的示例：`watch demo.MathGame primeFactors "{params,returnObj}" -x 2 -b -s -e`

![](http://image.easyblog.top/1671435573951782159a1-6de9-4ce5-90e0-835933ba9c4e.png)

命令执行后会一直对方法进行采样，如果要退出采样可以输入`q`+`回车` 或 `Ctrl+C` 退出

### （7）heapdump命令

heapdump命令类似java中的jmp命令，可以对堆内存生成快照。常用方式：`heapdump 文件保存路径`

```sh
[arthas@11844]$ heapdump /tmp/demo.hprof
Dumping heap to /tmp/demo.hprof ...
Heap dump file created
```



### （8）jvm命令

jvm命令可以查看当前JVM的各项概览信息：jvm运行信息、类加载信息、垃圾回收器信息、内存信息、线程等信息

![](http://image.easyblog.top/1672037962267f3b48f97-dc74-4655-9100-230202ea2b8b.png)



### （9）vmoption命令

vmoption命令可以用来查看&更新VM诊断相关参数。

* 查看所有option

![](http://image.easyblog.top/167203832868999a64172-5657-48a5-919f-da92dbe75bcc.png)

也可以使用 `vmoption 选项` 查看指定option的开启状态

* 更新执行option

语法：`vmoption 选项 选项值`

![](http://image.easyblog.top/16720384913834ffaaac5-bc80-41df-ace3-690767e5c376.png)