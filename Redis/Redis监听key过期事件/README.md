### Redis监听key过期事件



Redis  2.8.0版本之后提供Keyspace Notifications功能，当我们将<key,value>键值对使用Redis缓存并设置缓存失效时间的时候，会触发Redis的键事件通知，客户端订阅这个通知，服务端将会把对应的通知事件发送给客户端，客户端收到通知，然后根据自己的不同业务进行处理。要注意的是：**由于Redis的发布订阅模式采用的是发送即忘的策略，当订阅的客户端断线时，会丢失所有在断线期间发送给他的事件通知。当你的程序需要一个可靠的事件通知时，Redis的键空间通知就不适合了。**



### 一、开启 Redis key 过期提醒

事件类型

键空间通知都会发送两种不同类型的事件消息：**keyspace 和 keyevent**。以 keyspace 为前缀的频道被称为键空间通知（key-space notification）， 而以 keyevent 为前缀的频道则被称为键事件通知（key-event notification）。

修改 redis 相关事件配置。找到 redis 配置文件 redis.conf，查看 `notify-keyspace-events` 配置项，如果没有，添加 `notify-keyspace-events Ex`，如果有值，则追加 Ex，相关参数说明如下：

| 字符 | 发送通知                                                     |
| :--- | :----------------------------------------------------------- |
| K    | 键空间通知，全部通知以 keyspace@ 为前缀，针对Key             |
| E    | 键事件通知，全部通知以 keyevent@ 为前缀，针对event           |
| *g*  | *DEL 、 EXPIRE 、 RENAME 等类型无关的通用命令的通知*         |
| $    | 字符串命令的通知                                             |
| l    | 列表命令的通知                                               |
| s    | 集合命令的通知                                               |
| h    | 哈希命令的通知                                               |
| z    | 有序集合命令的通知                                           |
| *x*  | *过时事件：每当有过时键被删除时发送*                         |
| *e*  | *驱逐(evict)事件：每当有键由于 maxmemory 政策而被删除时发送* |
| A    | 参数 g$lshzxe 的别名，至关因而All                            |



### 二、基于Spring Boot实现监听并处理过期消息

**（1）添加Redis 消息监听的配置**

```java
@Configuration
public class RedisListenerConfigurer {

    @Bean
    public RedisMessageListenerContainer listenerContainer(RedisConnectionFactory connectionFactory) {
        RedisMessageListenerContainer container = new RedisMessageListenerContainer();
        container.setConnectionFactory(connectionFactory);
        return container;
    }

}
```

**（2）添加Redis key过期事件的监听**

```java
@Slf4j
@Component
public class RedisKeyExpireListener extends KeyExpirationEventMessageListener {

    public RedisKeyExpireListener(RedisMessageListenerContainer listenerContainer) {
        super(listenerContainer);
    }

    /**
     * 处理key过期事件
     *
     * @param message
     * @param pattern
     */
    @Override
    public void onMessage(Message message, byte[] pattern) {
        String expiredKey = message.toString();
        log.info("redis key过期：{}", expiredKey);
    }
}
```

