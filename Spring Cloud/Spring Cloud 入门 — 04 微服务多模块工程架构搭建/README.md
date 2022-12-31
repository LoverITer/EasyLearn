## Spring Cloud —  微服务多模块工程架构搭建



### 一、新建父工程

（1）新建Maven父工程

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210904201436.png)



（2）填写父工程名称，比如 ` springcloud-learning`

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210904201625.png)



（3）选择3.6版本以上的Maven

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210904201736.png)



（4）检查并设置字符编码为UTF-8

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210904202624.png)



（5）设置注解生效激活

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210904202741.png)



（6）选择JDK版本为1.8

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210904202903.png)



（7）删除父工程src目录，只保留pom文件

![QQ截图20210904204237](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210904204237.png)



（8）定义父工程POM

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>top.easyblog.prometheus</groupId>
    <artifactId>springcloud-learning</artifactId>
    <version>1.0-SNAPSHOT</version>
    <modules>
        <module>cloud-provider-payment8001</module>
        <module>cloud-consumer-order80</module>
    </modules>
    <packaging>pom</packaging>

    <!--统一管理jar包版本号-->
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <spring.boot.version>2.2.2.RELEASE</spring.boot.version>
        <spring.cloud.version>Hoxton.SR1</spring.cloud.version>
        <spring.cloud.alibaba.version>2.1.0.RELEASE</spring.cloud.alibaba.version>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <junit.version>4.12</junit.version>
        <log4j.version>1.2.17</log4j.version>
        <lombok.version>1.16.18</lombok.version>
        <mysql.version>8.0.15</mysql.version>
        <druid.version>1.1.16</druid.version>
        <mybatis.spring.boot.version>1.3.0</mybatis.spring.boot.version>
        <fastjson.version>1.2.62</fastjson.version>
    </properties>

    <!--定义整个项目的版本依赖，在这里面引入的jar包并不会被实际导入jar包到项目，
    可以达到锁定版本+自工程应用父工程依赖不写groupId 和 version-->
    <dependencyManagement>
        <dependencies>
            <!--spring boot 2.2.2-->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring.boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!--H版 spring cloud-->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring.cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!--spring cloud alibaba-->
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>${spring.cloud.alibaba.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>

            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>${mysql.version}</version>
            </dependency>
            <dependency>
                <groupId>com.alibaba</groupId>
                <artifactId>druid</artifactId>
                <version>${druid.version}</version>
            </dependency>
            <dependency>
                <groupId>org.mybatis.spring.boot</groupId>
                <artifactId>mybatis-spring-boot-starter</artifactId>
                <version>${mybatis.spring.boot.version}</version>
            </dependency>
            <dependency>
                <groupId>junit</groupId>
                <artifactId>junit</artifactId>
                <version>${junit.version}</version>
            </dependency>
            <dependency>
                <groupId>log4j</groupId>
                <artifactId>log4j</artifactId>
                <version>${log4j.version}</version>
            </dependency>
            <dependency>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
                <version>${lombok.version}</version>
                <optional>true</optional>
            </dependency>
            <dependency>
                <groupId>com.alibaba</groupId>
                <artifactId>fastjson</artifactId>
                <version>${fastjson.version}</version>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <pluginManagement><!-- lock down plugins versions to avoid using Maven defaults (may be moved to parent pom) -->
            <plugins>
                <plugin>
                    <artifactId>maven-clean-plugin</artifactId>
                    <version>3.1.0</version>
                </plugin>
                <plugin>
                    <artifactId>maven-site-plugin</artifactId>
                    <version>3.7.1</version>
                </plugin>
                <plugin>
                    <artifactId>maven-project-info-reports-plugin</artifactId>
                    <version>3.0.0</version>
                </plugin>
            </plugins>
        </pluginManagement>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-site-plugin</artifactId>
                <configuration>
                    <locales>en,fr</locales>
                </configuration>
            </plugin>
        </plugins>
    </build>

    <reporting>
        <plugins>
            <plugin>
                <artifactId>maven-project-info-reports-plugin</artifactId>
            </plugin>
        </plugins>
    </reporting>
</project>
```



导入依赖之后父工程创建工作只剩下最后一步执行mvn:install将父工程发布到仓库方便子工程继承。当看到控制台出现如下图所示 `BUILD SUCCESS` 时，父工程的构建工作就大功告成了！

![QQ截图20210904205521](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210904205521.png)

### 二、Rest微服务工程构建 

本小节我们来搭建一个paymen 支付单服务提供者模块`cloud-provider-payment8001` 并对外提供两个接口：插入支付流水接口 和 查询支付流水接口。

#### 2.1 微服务提供者支付Module模块构建

（1）New Module

![QQ截图20210904210021](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210904210021.png)



![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210905102640.png)



（2）修改POM，导入依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>springcloud-learning</artifactId>
        <groupId>top.easyblog.prometheus</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>cloud-provider-payment8001</artifactId>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>1.1.10</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
        </dependency>
    </dependencies>

</project>
```



（3）写配置文件

```yml
server:
  port: 8001

debug: on
spring:
  application:
    name: cloud-payment-service
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://47.99.161.205:3306/springcloud-learning?autoReconnect=true&useSSL=false&characterEncoding=utf-8&zeroDateTimeBehavior=convertToNull&serverTimezone=Asia/Shanghai
    username: root
    password: 95162437hx$

mybatis:
  mapperLocations: classpath:mapper/*.xml
```



（4）编写主启动类 `PaymentApplication`

```java
package top.easyblog.prometheus;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.EnableAspectJAutoProxy;

/**
 * @author frank.huang
 * @since 2021/9/4 21:10
 */
@EnableAspectJAutoProxy
@SpringBootApplication
public class PaymentApplication {
    public static void main(String[] args) {
        SpringApplication.run(PaymentApplication.class,args);
    }
}
```



（5）编写业务类

（5.1）新建数据表payment

```sql
CREATE TABLE payment (
 `id` BIGINT(20) NOT NULL PRIMARY KEY AUTO_INCREMENT COMMENT ID,
 `serial` VARCHAR(200) DEFAULT ""
) ENGINE=INNODB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8
```

（5.2）新建实体类 Payment

```java
package top.easyblog.prometheus.dao.entity;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

/**
 * @author frank.huang
 * @since 2021/9/4 21:30
 */
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Payment {
    private Long id;
    private String serial;
}
```

（5.3）新建DAO接口以及对应的Mappper映射文件

DAO接口：PaymentMapper.java

```java
package top.easyblog.prometheus.dao.mapper;

import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Param;
import top.easyblog.prometheus.dao.entity.Payment;

import java.util.List;

/**
 * @author frank.huang
 * @since 2021/9/4 21:43
 */
@Mapper
public interface PaymentMapper {

    int create(Payment payment);

    Payment queryPaymentById(@Param("id") Long id);

    List<Payment> queryByCondition(@Param("payment") Payment payment);

}
```

Mappper映射文件：PaymentMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="top.easyblog.prometheus.dao.mapper.PaymentMapper">
    <resultMap id="BaseResultMap" type="top.easyblog.prometheus.dao.entity.Payment">
        <id column="id" property="id" jdbcType="BIGINT"></id>
        <id column="serial" property="serial" jdbcType="VARCHAR"></id>
    </resultMap>

    <insert id="create" parameterType="top.easyblog.prometheus.dao.entity.Payment" useGeneratedKeys="true"
            keyProperty="id">
        insert into payment(serial)
        values (#{serial});
    </insert>

    <select id="queryPaymentById" parameterType="Long" resultMap="BaseResultMap">
        select *
        from payment
        where id = #{id}
    </select>

    <select id="queryByCondition" parameterType="top.easyblog.prometheus.dao.entity.Payment" resultMap="BaseResultMap">
        select * from payment
        <where>
            <if test="payment.id!=null">
                id=#{payment.id}
            </if>
            <if test="payment.serial!=null">
                AND serial=#{payment.serial}
            </if>
        </where>
    </select>
</mapper>
```

（5.4）业务逻辑处理层新建实现类

面向接口编程，首先提供对应业务的接口

```java
package top.easyblog.prometheus.service;

import top.easyblog.prometheus.dao.entity.Payment;
import top.easyblog.prometheus.request.CreatePaymentRequest;
import top.easyblog.prometheus.request.QueryPaymentRequest;
import top.easyblog.prometheus.response.CommonResponse;

/**
 * @author frank.huang
 * @since 2021/9/4 22:03
 */
public interface PaymentService {

    long create(CreatePaymentRequest request);

    Payment queryByRequest(QueryPaymentRequest request);

}
```



实现类实现业务接口

```java
package top.easyblog.prometheus.service.impl;

import com.alibaba.fastjson.JSON;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.BeanUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.util.CollectionUtils;
import top.easyblog.prometheus.dao.entity.Payment;
import top.easyblog.prometheus.dao.mapper.PaymentMapper;
import top.easyblog.prometheus.request.CreatePaymentRequest;
import top.easyblog.prometheus.request.QueryPaymentRequest;
import top.easyblog.prometheus.response.BusinessException;
import top.easyblog.prometheus.response.PaymentResultCode;
import top.easyblog.prometheus.service.PaymentService;

import java.util.List;
import java.util.Objects;

/**
 * @author frank.huang
 * @since 2021/9/4 22:17
 */
@Service
@Slf4j
public class PaymentServiceImpl implements PaymentService {

    @Autowired
    private PaymentMapper paymentMapper;

    @Override
    public long create(CreatePaymentRequest request) {
        Payment payment = new Payment();
        BeanUtils.copyProperties(request, payment);
        paymentMapper.create(payment);
        log.info("[DB] insert into payment {}", JSON.toJSON(payment));
        //新增成功返回主键ID
        return payment.getId() == null ? -1 : payment.getId();
    }

    @Override
    public Payment queryByRequest(QueryPaymentRequest request) {
        if (Objects.isNull(request.getId()) && Objects.isNull(request.getSerial())) {
            throw new BusinessException(PaymentResultCode.EMPTY_REQUEST_PARAM);
        }
        Payment payment = new Payment();
        BeanUtils.copyProperties(request, payment);
        log.info("[DB] going to query payment: {}", request.getId());
        List<Payment> payments = paymentMapper.queryByCondition(payment);
        if (CollectionUtils.isEmpty(payments)) {
            return null;
        }
        return payments.get(0);
    }
}
```



（5.5）控制器层新增控制器PaymentController

```java
package top.easyblog.prometheus.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;
import top.easyblog.prometheus.annotation.ResponseWrapper;
import top.easyblog.prometheus.dao.entity.Payment;
import top.easyblog.prometheus.request.CreatePaymentRequest;
import top.easyblog.prometheus.request.QueryPaymentRequest;
import top.easyblog.prometheus.service.PaymentService;

import javax.validation.Valid;

/**
 * @author frank.huang
 * @since 2021/9/4 22:23
 */
@RestController
@RequestMapping("/v1")
public class PaymentController {

    @Autowired
    private PaymentService paymentService;

    @ResponseWrapper
    @PostMapping("/payment")
    public Object create(@Valid @RequestBody CreatePaymentRequest request){
        return paymentService.create(request);
    }

    @ResponseWrapper
    @GetMapping("/payment")
    public Payment query(QueryPaymentRequest request){
        return paymentService.queryByRequest(request);
    }

}
```



（5.6）启动服务，验证结果

在postman中对上面编写的两个接口进行简单测试，测试结果截图如下：

**新增Payment接口**：

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210905105046.png)

接口插入成功，并返回自增主键ID

**查询Payment接口**

根据返回的主键Id查询，成功获取到刚才插入的数据

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210905105117.png)



### 三、微服务消费者支付消费Module模块构建

在本节我们将新建一个消费者模块 `cloud-consumer-order80` ，通过使用RestTemplate对第二节中创建的服务接口进行消费。

（3.1）新建并配置Module基本环境

如法炮制 第二节 中步骤，依次进行【New Module】 -> 【导入Maven依赖】 -> 【新建启动类OrderConsumerApplication]】

**Maven依赖**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>springcloud-learning</artifactId>
        <groupId>top.easyblog.prometheus</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>cloud-consumer-order80</artifactId>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
        </dependency>
    </dependencies>
</project>
```



**启动类 OrderConsumerApplication**

```java
package top.easyblog.promethens;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.EnableAspectJAutoProxy;

/**
 * @author frank.huang
 * @since 2021/9/4 21:10
 */
@EnableAspectJAutoProxy
@SpringBootApplication
public class OrderConsumerApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrderConsumerApplication.class,args);
    }
}
```

（3.2）提供RestTemplate配置

RestTemplate 说明。。。。

```java
package top.easyblog.promethens.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

/**
 * @author frank.huang
 * @since 2021/9/5 0:57
 */
@Configuration
public class ApplicationContextConfig {


    @Bean
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }

}
```

（3.3）控制器层新建控制器OrderConsumerController

```java
package top.easyblog.promethens.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.client.RestTemplate;
import top.easyblog.promethens.annotation.ResponseWrapper;
import top.easyblog.promethens.entity.Payment;
import top.easyblog.promethens.request.CreatePaymentRequest;
import top.easyblog.promethens.request.QueryPaymentRequest;
import top.easyblog.promethens.response.CommonResponse;

import javax.validation.Valid;

/**
 * @author frank.huang
 * @since 2021/9/5 0:55
 */
@RestController
@RequestMapping("/consumer")
public class OrderConsumerController {

    public static final String PAYMENT_BASE_URL = "http://localhost:8001";

    @Autowired
   private RestTemplate restTemplate;


    @PostMapping("/payment")
    public CommonResponse<Long> create(@RequestBody @Valid CreatePaymentRequest request){
      return restTemplate.postForObject(String.format("%s/v1/payment",PAYMENT_BASE_URL),request,CommonResponse.class);
    }


    @GetMapping("/payment")
    public CommonResponse<Payment> get(QueryPaymentRequest request){
        return restTemplate.getForObject(String.format("%s/v1/payment?id=%d",PAYMENT_BASE_URL,request.getId(),request.getSerial()),CommonResponse.class);
    }

}
```

（3.4）启动服务，验证结果

本节我们想要达到的结果是通order-cousumer服务远程调用payment服务进行消费，需要达到的目标是可以正确插入以及查询到数据，如下图1、2 是测结果。

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210905020938.png)



![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210905023217.png)



需要注意的是，本小节服务在启动测试时需要保证第二节的支付服务 `cloud-provider-payment8001` 是处于已运行的状态。

然后，在启动本小节消费服务的时候IDEA 就会给我们提示 `Multiple Spring Boot run configurations were detected...`  这是多模块服务管理Services，我们按照提示点击 `Show run configurations in Services` 。

随后我们就可以看到如下图所示控制台出现，此后我们在cloud微服务上开发多模块服务的启动、停止、调试等都可以通过此控制台进行统一管理。

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210905013607.png)