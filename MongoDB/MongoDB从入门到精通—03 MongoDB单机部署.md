## MongoDB从入门到精通—03 MongoDB单机部署



### 一、Windows系统中安装启动

#### 1.1 下载安装包

MongoDB 提供了可用于 32 位和 64 位系统的预编译二进制包，你可以从MongoDB官网下载安装，MongoDB 预编译二进制包下载地址： [MongoDB下载地址](https://www.mongodb.com/try/download/community?tck=docs_server)

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220412202142.png)

根据上图所示下载 zip 包。

 提示：版本的选择： 

MongoDB的版本命名规范如：x.y.z； 

y为奇数时表示当前版本为开发版，如：1.5.2、4.1.13；

 y为偶数时表示当前版本为稳定版，如：1.6.3、4.0.10； 

z是修正版本号，数字越大越好。 

详情：http://docs.mongodb.org/manual/release-notes/#release-version-numbers



#### 1.2 解压安装启动

将压缩包解压到一个目录中。 在解压目录中，手动建立一个目录用于存放数据文件，如 `data/db`



##### 方式1：命令行参数方式启动服务

在 bin 目录中打开命令行提示符，输入如下命令：

```shell
mongod --dbpath=..\data
```

执行上述命令之后会看到mongoDB打印日志，最后如果看到mongoDB监听在27017端口，则证明mongoDB已经启动成功了

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220412205802.png)

从在启动信息中可以看到，mongoDB的默认端口是27017，如果我们想改变默认的启动端口，可以通过`--port`来指定端口。 



##### 方式2：配置文件方式启动服务

为了方便我们每次启动，可以将安装目录的bin目录设置到环境变量的path中， bin 目录下是一些常用命令，比如 mongod 启动服务用的， mongo 客户端连接服务用的。

在解压目录中新建 config 文件夹，该文件夹中新建配置文件 `mongod.conf` ，内如参考如下：

```yml
systemLog:
  destination: file
  #The path of the log file to which mongod or mongos should send all diagnostic logging information
  path: "D:/mongodb/mongod.log"
  logAppend: true
storage:
  journal:
    enabled: true
  #The directory where the mongod instance stores its data.Default Value is "/data/db".
  dbPath: "D:/mongodb/data"
net:
  #bindIp: 127.0.0.1
  port: 27017
setParameter:
  enableLocalhostAuthBypass: false
```

更多详细配置项内容可以参考官方文档：https://docs.mongodb.com/manual/reference/configuration-options/

配置完成之后同样在mongoDB的bin目录下执行如下如下命令：

```shell
mongod -f ..\conf\mongod.conf
```

![QQ截图20220412211117](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220412211117.png)

启动之后就可以在mongo的安装目录下看到产生的日志文件，同样如果看到mongoDB监听在配置文件中配置指定的端口，则证明mongoDB已经在启动成功了



【**注意**】

1）配置文件中如果使用双引号，比如路径地址，自动会将双引号的内容转义。如果不转义，则会报错：`error-parsing-yaml-config-file-yaml-cpp-error-at-line-3-column-15-unknown-escape-character-d`

2）配置文件中不能以Tab分割字段

#### 1.3 Shell连接(mongo命令)

在命令提示符输入以下shell命令即可完成登陆

```shell
mongo
或
mongo --host=127.0.0.1 --port=27017
```

查看已经有的数据库

```shell
show databases
```

退出mongodb

```shell
exit
```

更多参数可以通过帮助命令 `mongo --help` 查看



#### 1.4 Compass-图形化界面客户端

到MongoDB官网下载 MongoDB Compass， 地址：https://www.mongodb.com/download-center/v2/compass?initial=true

如果是下载安装版，则按照步骤安装；如果是下载加压缩版，直接解压，执行里面的 `MongoDBCompassCommunity.exe` 文件即可。

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220412211831.png)

 在打开的界面中，输入主机地址、端口等相关信息，点击连接：

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220412212038.png)



连接成功后的界面：

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220412212201.png)



### 二、Linux系统中安装启动

#### 2.1 安装Linux依赖包

安装前我们需要安装各个 Linux 平台依赖包。

**Red Hat/CentOS：**

```
sudo yum install libcurl openssl
```

**Ubuntu 18.04 LTS ("Bionic")/Debian 10 "Buster"：**

```
sudo apt-get install libcurl4 openssl
```

**Ubuntu 16.04 LTS ("Xenial")/Debian 9 "Stretch"：**

```
sudo apt-get install libcurl3 openssl
```



#### 2.2 下载并安装(解压)MongoDB

MongoDB 提供了 linux 各个发行版本 64 位的安装包，你可以在官网下载安装包。[MongoDB 源码下载地址](https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel70-5.0.7.tgz)

![](http://image.easyblog.top/0D72BC20-1D77-437E-972C-286EB5EFB183.jpg)

![](http://image.easyblog.top/558D36F2-01AF-49C3-BA07-F2728B216C87.jpg)

这里我们选择 tgz 下载，下载完安装包，并解压 **tgz**（以下演示的是 64 位 Linux上的安装）

```shell
wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-ubuntu1604-4.2.8.tgz    # 下载
tar -zxvf mongodb-linux-x86_64-ubuntu1604-4.2.8.tgz    # 解压
mkdir /usr/local/mongodb
mv mongodb-linux-x86_64-ubuntu1604-4.2.8.tgz/*  /usr/local/mongodb/  # 将解压包拷贝到指定目录
```

MongoDB 的可执行文件位于 bin 目录下，所以可以将其添加到 **PATH** 路径中：

```shell
MONGODB_HOME=/usr/local/mongodb
export PATH=${MONGODB_HOME}/bin:$PATH
```



#### 2.3 创建数据库目录

默认情况下 MongoDB 启动后会初始化以下两个目录：

- 数据存储目录：/var/lib/mongodb
- 日志文件目录：/var/log/mongodb

我们在启动前可以先创建这两个目录并设置当前用户有读写权限：

```shell
sudo mkdir -p /var/lib/mongo
sudo mkdir -p /var/log/mongodb
sudo chown `whoami` /var/lib/mongo     # 设置权限
sudo chown `whoami` /var/log/mongodb   # 设置权限
```



#### 2.4 提供配置文件

在Liunx在我们也可以使用命令参数的形式启动，但是一般Linux都是作为服务器需要稳定提供服务的，所以我们这里重点演示如何使用配置文件的方式启动：

（1）在MongoDB安装目录下席间conf目录，并在目录下提供一下配置项：

```yml
systemLog:
  #MongoDB发送所有日志输出的目标指定为文件
  # #The path of the log file to which mongod or mongos should send all diagnostic logging information
  destination: file
  #mongod或mongos应向其发送所有诊断日志记录信息的日志文件的路径
  path: "/var/log/mongodb/mongodb.log"
  #当mongos或mongod实例重新启动时，mongos或mongod会将新条目附加到现有日志文件的末尾。
  logAppend: true
storage:
  #mongod实例存储其数据的目录。storage.dbPath设置仅适用于mongod。
  #The directory where the mongod instance stores its data.Default Value is "/data/db".
   dbPath: "/var/lib/mongo"
journal:
  #启用或禁用持久性日志以确保数据文件保持有效和可恢复。
  enabled: true
processManagement:
  #启用在后台运行mongos或mongod进程的守护进程模式。
  fork: true
net:
  #服务实例绑定的IP，默认是localhost  后面 172.17.15.109 是服务器的内网地址，棘具体通过ifconfig命令查看
  bindIp: localhost,172.17.15.109
  port: 27017
```

配置完成之后我们执行命令：`mongod -f /usr/local/mongodb/conf/mongod.conf`  执行命令后如果看到如下输出则表示启动成功：

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220412221357.png)

此时mongoDB就会以守护进程的方式在后台运行，我们可以执行命令 `cat /var/log/mongodb/mongodb.log` 查看日志，可以在日志中看到mongoDB已经监听在配置文件指定的端口和IP上了。

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220412220909.png)



#### 2.5 连接MongoDB

远程连接我们可以直接选择使用 Compass

![QQ截图20220412222351](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220412222351.png)



**提示：如果使用的是远程云服务器，注意配置安全组放开27017端口(或者你在配置中指定的端口)以及打开防火墙**

