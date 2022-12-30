# Golang语言环境配置





### 下载Go安装包

Go 语言支持以下系统：

- Linux
- FreeBSD
- Mac OS X（也称为 Darwin）
- Windows

安装包下载地址为：https://golang.org/dl/。

如果打不开可以使用这个地址：https://golang.google.cn/dl/。

![](http://image.easyblog.top/16723000973537c3097a5-8f9e-4931-9966-1730cdffa7b0.png)

选择稳定版本下载即可。

下载完成是一个安装文件，我们需要进行安装，同时需要注意的就是安装目录，因为事后还需要配置环境变量，下面是安装界面的图片

![](http://image.easyblog.top/1672300615932d2f0a0df-2273-444b-b139-76d14290c01d.png)



### 配置环境变量

接下来我们需要配置一下环境变量，将Go安装目录所在路径定义到环境变量中，让系统帮我们去找运行的执行程序，这样在任何目录下都可以执行go指令，需要配置的环境变量有：

| 环境变量 | 说明              |
| -------- | ----------------- |
| GOROOT   | 指定SDK的安装目录 |
| Path     | 添加SDK的/binmulu |
| GOPATH   | 工作目录          |

#### Windows下配置环境变量

首先我们需要打开我们的环境变量，然后添加上GOROOT

![i](https://gitee.com/moxi159753/LearningNotes/raw/master/Golang/Golang%E5%9F%BA%E7%A1%80/0_Go%E8%AF%AD%E8%A8%80%E7%9A%84%E5%AE%89%E8%A3%85/images/image-20200718151418230.png)

然后我们在PATH上添加我们的bin目录

![](https://gitee.com/moxi159753/LearningNotes/raw/master/Golang/Golang%E5%9F%BA%E7%A1%80/0_Go%E8%AF%AD%E8%A8%80%E7%9A%84%E5%AE%89%E8%A3%85/images/image-20200718151503318.png)



#### MAC/Linux下配置环境变量

MAC或Liunx上 go默认路径如下：

- 默认安装路径： **/usr/local/go**
- 默认编译启动文件：**/usr/local/go/bin/go**

接下来将路径配置到环境变量中：

1、**如果是bash**

> vim ~/.bash_profile

**如果是zshrc**

> vim ~/.zshrc

或者配置到/etc/profile中

2、**输入环境命令**

```she
#Settig PATH for Golang
export PATH=/usr/local/go/bin:$PATH
export GOROOT=/usr/local/go
```

3、刷新配置文件

```shell
source ~/.bash_profile
或
source ~/.zshrc
或
source /etc/profile
```





添加完成后，我们输入下面的命令，查看是否配置成功

```she
go version
```

![](http://image.easyblog.top/167230168275291e2aff2-38db-40ae-b129-bf8d03a96770.png)



###  hello world

配置好环境变量之后就可以来写第一个go程序了——使用go在命令行窗口输出“hello world”字符。

编写go-hello world程序：hello.go

```go
package main

import "fmt"


func main(){
   fmt.Println("hello wolrd!")
}
```

保存代码，我们执行go命令：`go run hello.go` 

![](http://image.easyblog.top/1672302230792d9290ab0-2dac-4dc8-a574-174e995d87da.png)

