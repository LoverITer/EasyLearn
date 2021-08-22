### Apache Kafka —安装和使用



### 步骤1：确保机器上已经安装jdk环境

希望你已经在你的机器上安装了java，你可以使用下面的命令验证它。

```
$ java -version
```

如果jdk在你的机器上成功安装，执行上面命令之后你可以看到已安装的Java的版本。如果没有看到版本或命令报错，那么请参考 [XXX](xxxx) 进行jdk环境的安装以及配置。



### 步骤2：下载Kafka

打开Apache Kafka官网 [http://kafka.apache.org/downloads](http://kafka.apache.org/downloads) 下载Kafka的**tgz**安装包

![截屏2021-07-21 下午1.51.36](https://image.easyblog.top/%E6%88%AA%E5%B1%8F2021-07-21%20%E4%B8%8B%E5%8D%881.51.36.png)

然后解压并进入kafka路径下，

```ps1con
> tar -xzf kafka_2.12-2.7.0.tgz
> cd kafka_2.12-2.7.0
```



### 步骤3：启动zookpeer

**ZooKeeper是一种分布式协调服务**，用于管理大型主机。在分布式环境中协调和管理服务是一个复杂的过程。ZooKeeper通过其简单的架构和API解决了这个问题。 ZooKeeper允许开发人员专注于核心应用程序逻辑，而不必担心应用程序的分布式特性。

ZooKeeper框架最初是在“Yahoo!”上构建的，用于以简单而稳健的方式访问他们的应用程序。后来，Apache ZooKeeper成为许多分布式框架使用的有组织服务的标准，比如**Kafka就是使用ZooKeeper作为协调框架**，在Kafka的安装包里已经包含了ZooKeeper，因此不需要在单独去下载ZooKeeper的安装包。

**启动服务**

使用Kafka之前需要先启动一个ZooKeeper服务，这里直接使用Kafka中提供的启动脚本即可。

```text
> nohup bin/zookeeper-server-start.sh config/zookeeper.properties &
```

**注意**：命令中 `nohup xxx  &` 的含义是让服务在后台启动并运行。 

### 步骤4：启动Kafka服务

启动ZooKeeper服务之后再启动Kafka服务

```text
> nohup bin/kafka-server-start.sh config/server.properties &
```

需要强调一下，`config/server.properties` 是Kafka的配置文件，可以用于配置监听的host、port、broker等。

默认的ZooKeeper连接服务为localhost:2181，

```text
# Zookeeper connection string (see zookeeper docs for details).
# This is a comma separated host:port pairs, each corresponding to a zk
# server. e.g. "127.0.0.1:3000,127.0.0.1:3001,127.0.0.1:3002".
# You can also append an optional chroot string to the urls to specify the
# root directory for all kafka znodes.
zookeeper.connect=localhost:2181
```

另外，producer和consumer的监听端口为9092，如果需要更改server的host和port端口可以通过修改 `config/server.properties` 进行配置。



**注意**：这里如果在启动的时候报错“Cannot allocate memory”，这是你的机器内存不足了。

![](https://image.easyblog.top/%E6%88%AA%E5%B1%8F2021-07-21%20%E4%B8%8B%E5%8D%885.09.54.png)



**解决方法**：

根据你的机器内存大小合理修改 `bin/kafka-server-start.sh` 文件中的 `KAFKA_HEAP_OPTS` 的值，但是也不能太小，设置的太小kafka也会无法启动起来。

![截屏2021-07-21 下午5.13.20](https://image.easyblog.top/%E6%88%AA%E5%B1%8F2021-07-21%20%E4%B8%8B%E5%8D%885.13.20.png)



### 步骤5：使用kafka发送消息

Kafka的使用的方式有很多种，在他的安装包里提供的有命令行方式的脚本，这里重要介绍一下如何使用。

**1. 创建Topic**

使用Kafka，我们首先需要创建一个Topic，这样后续消息生产者和消息消费者才能针对性的发送和消费数据，

```text
> bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
```

这样我们就**创建**了一个名为test的Topic。

我们也可以通过命令来查看我们已经创建的 Topic，

```text
> bin/kafka-topics.sh --list --zookeeper localhost:2181
test
```

**2. 发送消息**

前面介绍Kafka专业术语中已经阐述，Kafka使用过程中首先需要消息生产者发送消息，那么消费者才可以读取到消息。

启动一个终端A，执行下面命令，

```text
> bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
>hello world
>
```

当执行producer脚本后，会出现消息输入提示符，这是我们可以输入消息(数据)，然后它会发送到对应的服务器(Broker)。

**3. 消费消息**

现在管道中已经有了数据，接下来我就可以使用消费者去读取数据。

另外启动一个终端B，执行下面命令，

```text
> ./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
hello world
```

可以看到，消息消费者一直处于监听状态，每当在终端A输入一条消息，终端B也会更新一条消息。
