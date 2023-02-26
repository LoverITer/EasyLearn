## SpringBoot+MyBatis+AOP 实现多数据源动态自主切换



在实际开发中，我们一个项目可能会用到多个数据库，通常一个数据库对应一种数据，而且在大数据量的业务下通常都会有多个数据源的。最近恰好工作上遇到一个Spring Boot + Mybatis 框架的多数据源配置切换的业务，所以就研究了一下关于Springboot+ Mybatis 多数据源切换。



### 1. 多数据源解决方案

正常情况下，我们操作数据是通过配置一个DataSource数据源来连接数据库，然后绑定给SqlSessionFactory，然后通过Dao或Mapper指定SqlSessionFactory来操作数据库的。

![](https://image.easyblog.top/16750732887147a5e565d-6d5a-4345-ac5a-b9426ce08e3b.png)

而操作多数据源则更要复杂一点，可以通过如下两种方式来实现：

#### 普通的多数据源

多个DataSource数据源绑定多个SqlSessionFactory，每个数据源绑定一个SqlSessionFactory，然后通过Dao或Mapper指定SqlSessionFactory来操作数据库。

操作不同的数据源是通过在业务层调用对应的实现了不同数据源的方法来同时操作不同的数据源的。

![](https://image.easyblog.top/1675073445775722fe565-3040-4d2f-9628-5689f3492269.png)



#### 动态切换的数据源

方式一中，必须要使多个数据源之间完全的物理分离，如果存在一个用户表，几个数据库都有的情况，并且业务也类似，那写多套代码是冗余的，并且代码维护起来也更加困难，有没有更便捷的方式呢？

其实可以通过配置多个DataSource数据源到一个**DynamicDataSource**动态数据源上，动态数据源绑定一个SqlSessionFactory，除了中间多出一个动态数据源外，其他部分都是相同的。

![](https://image.easyblog.top/16750741110166edfab95-2f3b-4f25-8995-15953131c482.png)

Q：那么这种方式是怎么实现数据源的切换的呢？

A：通过在业务类或方法上添加一个数据源标识（注解），使用切面来监听这个标志，进而切换数据源，通过一个注解就可以更加灵活切换数据源。



### 2. 实现动态数据源切换

#### 2.1 禁用SpringBoot的自动配置数据源类

在SpringBoot中，程序会自动读取`src/main/resources/application.yml 或 application.properties`配置文件中的`spring.datasource.xxx`的数据源配置信息，如果我们需要配置多数据源的话，要先把这个自动读取数据源配置信息的类禁掉。禁掉该类的方法是：在SpringBoot的启动类中，使用`@SpringBootApplication`注解时把类`DataSourceAutoConfiguration.class`排除。

![](http://image.easyblog.top/16750745335678aa07aad-8746-4635-87b6-ce86b3ce306e.png)



#### 2.2 重写数据源配置读取类

禁掉自动读取数据源配置类之后，需要自己写读取不同数据源的配置信息代码了，要配置多少个数据源，就有多少个配置方法，并将数据源实例交给Spring容器管理，如下（**这里以MySQL 和 TiDB为例，数据库连接池使用Hikari**）：

```java
package com.example.config.db;

import com.google.common.collect.Maps;

import com.zaxxer.hikari.HikariConfig;
import com.zaxxer.hikari.HikariDataSource;

import org.apache.ibatis.session.AutoMappingBehavior;
import org.apache.ibatis.session.ExecutorType;
import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
import org.springframework.core.io.support.ResourcePatternResolver;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
import org.springframework.transaction.PlatformTransactionManager;

import java.util.Map;
import java.util.Properties;

import javax.sql.DataSource;


@Configuration
public class DataSourceConfig {

    // 配置主数据源
    @Value("${datasource.default:mysql_master}")
    private String defaultDataSource;

    // *********************Mysql************************
    @Value("${spring.datasource.mysql.url}")
    private String mysqlMasterURL;

    @Value("${spring.datasource.mysql.username}")
    private String mysqlMasterUsername;

    @Value("${spring.datasource.mysql.password}")
    private String mysqlMasterPassword;

    @Value("${spring.datasource.mysql.readonly:false}")
    private boolean mysqlMasterReadonly;

    // 连接数据库的超时时间，单位毫秒
    @Value("${spring.datasource.mysql.connection-timeout:30000}")
    private long mysqlMasterConnectionTimeout;

    // 最大空闲时间，非核心线程的空闲时间如果超过此阈值，则被线程池销毁掉, 单位毫秒
    @Value("${spring.datasource.mysql.idle-timeout:60000}")
    private long mysqlMasterIdleTimeout;

    // 最大生存时间，核心线程如果存活的时间超过此阈值，会被销毁, 单位毫秒
    @Value("${spring.datasource.mysql.max-lifetime:1800000}")
    private long mysqlMasterMaxLifetime;

    // 最大线程池容量
    @Value("${spring.datasource.mysql.maximum-pool-size:100}")
    private int mysqlMasterMaximumPoolSize;

    @Value("${spring.datasource.mysql.drive:com.mysql.cj.jdbc.Driver}")
    private String mysqlMasterDrive;

    // *********************TiDB************************
    @Value("${spring.datasource.tidb.url}")
    private String tidbMasterURL;

    @Value("${spring.datasource.tidb.username}")
    private String tidbMasterUsername;

    @Value("${spring.datasource.tidb.password}")
    private String tidbMasterPassword;

    @Value("${spring.datasource.tidb.readonly:false}")
    private boolean tidbMasterReadonly;

    // 连接数据库的超时时间，单位毫秒
    @Value("${spring.datasource.tidb.connection-timeout:30000}")
    private long tidbMasterConnectionTimeout;

    // 最大空闲时间，非核心线程的空闲时间如果超过此阈值，则被线程池销毁掉, 单位毫秒
    @Value("${spring.datasource.tidb.idle-timeout:60000}")
    private long tidbMasterIdleTimeout;

    // 最大生存时间，核心线程如果存活的时间超过此阈值，会被销毁, 单位毫秒
    @Value("${spring.datasource.tidb.max-lifetime:1800000}")
    private long tidbMasterMaxLifetime;

    // 最大线程池容量
    @Value("${spring.datasource.tidb.maximum-pool-size:100}")
    private int tidbMasterMaximumPoolSize;

    @Value("${spring.datasource.tidb.drive:com.mysql.cj.jdbc.Driver}")
    private String tidbMasterDrive;


    // 配置mysql主数据源
    @Bean(name = "mysqlMasterDataSource")
    DataSource mysqlMasterDataSource() {
        return new HikariDataSource(
                createHikariConfig(
                        mysqlMasterURL,
                        mysqlMasterUsername,
                        mysqlMasterPassword,
                        mysqlMasterReadonly,
                        mysqlMasterConnectionTimeout,
                        mysqlMasterIdleTimeout,
                        mysqlMasterMaxLifetime,
                        mysqlMasterMaximumPoolSize,
                        mysqlMasterDrive
                )
        );
    }


    // 配置TiDB主数据源
    @Bean(name = "tidbMasterDataSource")
    DataSource tidbMasterDataSource() {
        return new HikariDataSource(
                createHikariConfig(
                        tidbMasterURL,
                        tidbMasterUsername,
                        tidbMasterPassword,
                        tidbMasterReadonly,
                        tidbMasterConnectionTimeout,
                        tidbMasterIdleTimeout,
                        tidbMasterMaxLifetime,
                        tidbMasterMaximumPoolSize,
                        tidbMasterDrive
                )
        );
    }


    private HikariConfig createHikariConfig(String url, String username, String password, boolean readonly, long connectionTimeout, long idleTimeout,
                                            long maxLifeTime, int maxPoolSize, String driver) {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl(url);
        config.setUsername(username);
        config.setPassword(password);
        config.setReadOnly(readonly);
        config.setConnectionTimeout(connectionTimeout);
        config.setIdleTimeout(idleTimeout);
        config.setMaxLifetime(maxLifeTime);
        config.setMaximumPoolSize(maxPoolSize);
        config.setDriverClassName(driver);
        return config;
    }

}
```



#### 2.3 自定义DynamicDataSource实现数据源可切换

有了多个数据源之后我们需要一个地方来管理系统的数据源，可以使用枚举，也可以使用常量类，如下定义了MYSQL 和 TiDB 两种数据源枚举：

```java
package com.example.config.db;

// 数据源枚举
public class DataSourceDialect {
    public static final String MYSQL_MASTER = "mysql_master";
    public static final String TIDB_MASTER = "tidb_master";
}
```



SpringBoot中提供了一个类 `AbstractRoutingDataSource` 可以让用户实现数据源的动态切换，我们需要重写该类的 `determineCurrentLookupKey()` 方法，从数据源类型容器中获取当前线程的数据源类型。如下：

```java
package com.example.config.db;

import org.springframework.jdbc.datasource.lookup.AbstractRoutingDataSource;

import lombok.extern.slf4j.Slf4j;


@Slf4j
public class DynamicDataSource extends AbstractRoutingDataSource {

    @Override
    protected Object determineCurrentLookupKey() {
        log.info("Switch current datasource to {}", DataSourceSwitch.getDataSourceType());
        return DataSourceSwitch.getDataSourceType();
    }

}
```

其中 DataSourceSwith 是自定义的数据源切换类，它保存了当前线程下的应该使用的数据源名称，构建一个数据源类型容器，并提供了向其中设置、获取和清空数据源类型的方法，具体代码如下：

```java
package com.example.config.db;


public class DataSourceSwitch {

    /**
     * 保存数据源类型线程安全容器
     */
    private static final ThreadLocal<String> contextHolder = new ThreadLocal<>();

    /**
     * 获取数据源类型
     *
     * @return
     */
    public static String getDataSourceType() {
        return contextHolder.get();
    }

    /**
     * 设置数据源类型
     *
     * @param dataSourceType 数据源类型
     */
    public static void setDataSourceType(String dataSourceType) {
        contextHolder.set(dataSourceType);
    }

    /**
     * 清空数据源类型
     */
    public static void clearDataSourceType() {
        contextHolder.remove();
    }

}
```



#### 2.4 使用自定义DynamicDataSource重写SpringBoot的数据源配置

在DataSourceConfig类中增加方法，从DynamicDataSource中获取当前线程的Datasoure，实现数据源的切换：

```java
@Bean(name = "dataSource")
public DataSource dataSource() {
    DataSource mysqlMasterDataSource = mysqlMasterDataSource();
    DataSource tidbMasterDataSource = tidbMasterDataSource();

    DynamicDataSource dynamicDataSource = new DynamicDataSource();
    if (DataSourceDialect.MYSQL_MASTER.equalsIgnoreCase(defaultDataSource)) {
        dynamicDataSource.setDefaultTargetDataSource(mysqlMasterDataSource);
    } else if (DataSourceDialect.UDS_TIDB_MASTER.equalsIgnoreCase(defaultDataSource)) {
        dynamicDataSource.setDefaultTargetDataSource(tidbMasterDataSource);
    }

    Map<Object, Object> dataSources = Maps.newHashMap();
    dataSources.put(DataSourceDialect.MYSQL_MASTER, mysqlMasterDataSource);
    dataSources.put(DataSourceDialect.TIDB_MASTER, tidbMasterDataSource);
    dynamicDataSource.setTargetDataSources(dataSources);
    return dynamicDataSource;
}
```

根据动态数据源配置，创建sqlSessionFactory，具体代码如下：

```java
@Primary
@Bean(name = "sqlSessionFactory")
public SqlSessionFactory sqlSessionFactoryBean() throws Exception {
    Properties properties = new Properties();
    properties.setProperty("dialect", "mysql");

    ResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();

    SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
    sqlSessionFactoryBean.setDataSource(dataSource());
    sqlSessionFactoryBean.setConfiguration(sessionConfiguration());
    sqlSessionFactoryBean.setConfigurationProperties(properties);
    sqlSessionFactoryBean.setMapperLocations(resolver.getResources("classpath*:mapper/*.xml"));
    return sqlSessionFactoryBean.getObject();
}
```

根据动态数据源配置，创建transactionManager，具体代码如下：

```java
@Bean
public PlatformTransactionManager transactionManager() {
    DataSourceTransactionManager transactionManager = new DataSourceTransactionManager();
    transactionManager.setDataSource(dataSource());
    return transactionManager;
}
```

根据动态数据源配置，创建mybatis sessionConfiguration，具体代码如下：

```java
@Bean("sessionConfiguration")
public org.apache.ibatis.session.Configuration sessionConfiguration() {
    org.apache.ibatis.session.Configuration configuration = new org.apache.ibatis.session.Configuration();
    // 全局映射器启用缓存
    configuration.setCacheEnabled(true);
    // 查询时，关闭关联对象即时加载以提高性能
    configuration.setLazyLoadingEnabled(true);
    // 设置关联对象加载的形态，此处为按需加载字段(加载字段由SQL指 定)，不会加载关联表的所有字段，以提高性能
    configuration.setAggressiveLazyLoading(false);
    // 对于未知的SQL查询，允许返回不同的结果集以达到通用的效果
    configuration.setMultipleResultSetsEnabled(true);
    // 允许使用列标签代替列名
    configuration.setUseColumnLabel(true);
    // 给予被嵌套的resultMap以字段-属性的映射支持
    configuration.setAutoMappingBehavior(AutoMappingBehavior.FULL);
    // 对于批量更新操作缓存SQL以提高性能
    configuration.setDefaultExecutorType(ExecutorType.SIMPLE);
    // 数据库超过25000秒仍未响应则超时
    configuration.setDefaultStatementTimeout(25000);
    configuration.setMapUnderscoreToCamelCase(true);

    return configuration;
}
```



### 3. AOP+自定义注解 实现数据源动态切换

通过上面第2步我们配置了MySQL和TiDB两种可动态切换的数据源，我们只需要在业务代码中使用自定义的`DataSourceSwitch.setDataSourceType()` 方法设置业务员方法应该使用的数据源即可实现数据源的动态切换，可是这种方式略显麻烦且不够优雅，在AOP加持下，我们可以实现通过自定义注解实现数据源动态切换。

实现的基本原理也很简单：通过在业务类上标注自定义注解并指定数据源，然后AOP切面监听注解，如果被注解标注的类或方法被执行那么就按照注解中指定的数据源调用DataSourceSwitch的setDataSourceType方法实现数据源切换。

#### 3.1 自定义数据 `@DataSource`

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD, ElementType.TYPE})
public @interface DataSource {

    // 设置DataSource，可以设置一个默认的数据源来兜底
    String value() default DataSourceDialect.MYSQL_MASTER;

}
```



#### 3.2 在业务类上标注自定义数据源注解

在使用 MySQL 数据源的业务类上标注注解 `@DataSource(value = DataSourceDialect.MYSQL_MASTER)` , 由于注解默认值使用的就是MySQL数据源，所以可以缺省写为 `@DataSource` 

![](http://image.easyblog.top/1675132951502e73af71f-48ad-4593-948a-95d5e341a4b7.png)

在使用 TiDB 数据源的业务类上标注注解 `@DataSource(value = DataSourceDialect.TIDB_MASTER)`

![](http://image.easyblog.top/1675133092402c09f5bcf-8eb8-4788-9d5b-3eadddb1b71b.png)



#### 3.3 通过切面实现动态切换数据源

```java
package com.example.aspect;

import com.example.annotation.DataSource;
import com.example.config.db.DataSourceDialect;
import com.example.config.db.DataSourceSwitch;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.Signature;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;

import java.lang.reflect.Method;

import lombok.extern.slf4j.Slf4j;


@Slf4j
@Aspect
@Order(-1)
@Component
public class DynamicDataSourceAspect {


    @Around("@annotation(com.example.annotation.DataSource) || @within(com.example.annotation.DataSource)")
    public Object switchDatasource(ProceedingJoinPoint point) throws Throwable {
        Signature signature = point.getSignature();
        MethodSignature methodSignature = (MethodSignature) signature;
        Method agentMethod = methodSignature.getMethod();
        Method targetMethod = point.getTarget().getClass().getMethod(agentMethod.getName(), agentMethod.getParameterTypes());
        //获取要切换的数据源
        DataSource dataSource = targetMethod.getAnnotation(DataSource.class);
        if (dataSource != null) {
            DataSourceSwitch.setDataSourceType(dataSource.value());
        } else {
            // 获取类上的注解
            dataSource = point.getTarget().getClass().getAnnotation(UdsDataSource.class);
            if (dataSource == null) {
                DataSourceSwitch.setDataSourceType(DataSourceDialect.UDS_MYSQL_MASTER);
            } else {
                DataSourceSwitch.setDataSourceType(dataSource.value());
            }
        }
        log.info("切换数据源:{}", DataSourceSwitch.getDataSourceType());
        try {
            // 通过创建对象的形式才能保证正常,直接return point.proceed()则会导致该方法执行两次
            final Object proceed = point.proceed();
            return proceed;
        } finally {
            // 销毁数据源 在执行方法之后
            DataSourceSwitch.clearDataSourceType();
        }
    }


}
```



### 4. 验证结果

- 访问依赖MySQL数据源的接口 `/v1/user`

接口正常返回 & 日志打印了切换过程：

![](http://image.easyblog.top/16751354215182e262eff-c037-442f-81ea-58d572ede778.png)



- 访问依赖TiDB数据源的接口 `/v1/order/list`

接口正常返回 & 日志打印了切换过程：

![](http://image.easyblog.top/1675135432119801a7da2-b186-475d-b073-5b61ef54588c.png)





