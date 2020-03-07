---
layout:     post
title:      "Dubbo构建示例"
subtitle:   "基于当前适合版本，牛刀小试。"
date:       2020-03-07
author:     "morece"
header-img: "img/in-post/2020.03/07/dubbo.png"
tags:
    - Dubbo
    - 微服务
---

## Dubbo构建示例

### 一、说明

- 基于[alibaba-dubbo](https://github.com/alibaba/spring-cloud-alibaba/tree/master/spring-cloud-alibaba-examples/spring-cloud-alibaba-dubbo-examples)最新支持版本示例
  - [spring-cloud-alibaba-dependencies](https://github.com/alibaba/spring-cloud-alibaba/blob/master/pom.xml) `2.2.0`
  - 同上spring boot 版本`2.2.0`
  - 同上dubbo版本 `2.7.4.1`

- 启动[Nacos](https://github.com/alibaba/nacos/releases)`1.2.0` 

  - 192.168.31.219:8848

- 实现服务消费方hello-apache-dubbo-consumer的外部化配置文件

- 实现服务提供方/消费方通信告诉序列化（Kryo）

  - dubbo-serialization-kryo 版本使用apache 而非 alibaba版本

  - Kryo同步版本`2.7.4.1`



### 二、创建项目工程

**创建hello-apache-dubbo项目POM工程**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.0.RELEASE</version>
        <relativePath/>
    </parent>

    <groupId>com.my</groupId>
    <artifactId>hello-apache-dubbo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>hello-apache-dubbo</name>
    <packaging>pom</packaging>

    <modules>
        <module>hello-apache-dubbo-dependencies</module>
        <module>hello-apache-dubbo-api</module>
        <module>hello-apache-dubbo-provider</module>
        <module>hello-apache-dubbo-consumer</module>
    </modules>

    <licenses>
        <license>
            <name>Apache 2.0</name>
            <url>https://www.apache.org/licenses/LICENSE-2.0.txt</url>
        </license>
    </licenses>

    <properties>
        <java.version>1.8</java.version>
        <maven.compiler.source>${java.version}</maven.compiler.source>
        <maven.compiler.target>${java.version}</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <developers>
        <developer>
            <id>zhanghaitao</id>
            <name>Haitao Zhang</name>
            <email>2636461000@qq.com</email>
        </developer>
    </developers>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>com.my</groupId>
                <artifactId>hello-apache-dubbo-dependencies</artifactId>
                <version>${project.version}</version>
                <type>pom</type>
               <scope>import</scope>
           </dependency>
        </dependencies>
    </dependencyManagement>

    <profiles>
        <profile>
            <id>default</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            <properties>
                <spring-javaformat.version>0.0.12</spring-javaformat.version>
            </properties>
            <build>
                <plugins>
                    <plugin>
                        <groupId>io.spring.javaformat</groupId>
                        <artifactId>spring-javaformat-maven-plugin</artifactId>
                        <version>${spring-javaformat.version}</version>
                    </plugin>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-surefire-plugin</artifactId>
                        <configuration>
                            <includes>
                                <include>**/*Tests.java</include>
                            </includes>
                            <excludes>
                                <exclude>**/Abstract*.java</exclude>
                            </excludes>
                            <systemPropertyVariables>
                                <java.security.egd>file:/dev/./urandom</java.security.egd>
                                <java.awt.headless>true</java.awt.headless>
                            </systemPropertyVariables>
                        </configuration>
                    </plugin>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-enforcer-plugin</artifactId>
                        <executions>
                            <execution>
                                <id>enforce-rules</id>
                                <goals>
                                    <goal>enforce</goal>
                                </goals>
                                <configuration>
                                    <rules>
                                        <bannedDependencies>
                                            <excludes>
                                                <exclude>commons-logging:*:*</exclude>
                                            </excludes>
                                            <searchTransitive>true</searchTransitive>
                                        </bannedDependencies>
                                    </rules>
                                    <fail>true</fail>
                                </configuration>
                            </execution>
                        </executions>
                    </plugin>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-install-plugin</artifactId>
                        <configuration>
                            <skip>true</skip>
                        </configuration>
                    </plugin>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-javadoc-plugin</artifactId>
                        <configuration>
                            <skip>true</skip>
                        </configuration>
                        <inherited>true</inherited>
                    </plugin>
                </plugins>
            </build>
        </profile>
    </profiles>
    <pluginRepositories>
        <pluginRepository>
            <id>spring-milestone</id>
            <name>Spring Milestone</name>
            <url>https://repo.spring.io/milestone</url>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </pluginRepository>
        <pluginRepository>
            <id>spring-snapshot</id>
            <name>Spring Snapshot</name>
            <url>https://repo.spring.io/snapshot</url>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
        </pluginRepository>
    </pluginRepositories>
</project>
```

### 三、创建共用依赖

**创建hello-apache-dubbo-dependencies共用依赖POM工程**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.my</groupId>
        <artifactId>hello-apache-dubbo</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>

    <artifactId>hello-apache-dubbo-dependencies</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>pom</packaging>

    <licenses>
        <license>
            <name>Apache 2.0</name>
            <url>https://www.apache.org/licenses/LICENSE-2.0.txt</url>
        </license>
    </licenses>
    <developers>
        <developer>
            <id>zhanghaitao</id>
            <name>Haitao Zhang</name>
            <email>2636461000@qq.com</email>
        </developer>
    </developers>

    <properties>
        <alibaba-spring-cloud.version>2.2.0.RELEASE</alibaba-spring-cloud.version>
      	<dubbo-kryo.version>2.7.4.1</dubbo-kryo.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>${alibaba-spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>org.apache.dubbo</groupId>
                <artifactId>dubbo-serialization-kryo</artifactId>
                <version>${dubbo-kryo.version}</version>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

### 四、定义Dubbo服务接口

**1、创建hello-apache-dubbo-api服务JAR接口**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.my</groupId>
        <artifactId>hello-apache-dubbo</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>

    <artifactId>hello-apache-dubbo-api</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>

    <licenses>
        <license>
            <name>Apache 2.0</name>
            <url>https://www.apache.org/licenses/LICENSE-2.0.txt</url>
        </license>
    </licenses>
    <developers>
        <developer>
            <id>zhanghaitao</id>
            <name>Haitao Zhang</name>
            <email>2636461000@qq.com</email>
        </developer>
    </developers>
</project>
```

**2、创建服务共用接口EchoService**

```java
package com.my.apache.dubbo.api;

public interface EchoService {
    String echo(String string);
}

```

### 五、实现Dubbo服务提供方

**1、创建hello-apache-dubbo-provider服务提供方JAR工程**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.my</groupId>
        <artifactId>hello-apache-dubbo</artifactId>
        <version>0.0.1-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>

    <artifactId>hello-apache-dubbo-provider</artifactId>
    <packaging>jar</packaging>

    <licenses>
        <license>
            <name>Apache 2.0</name>
            <url>https://www.apache.org/licenses/LICENSE-2.0.txt</url>
        </license>
    </licenses>
    <developers>
        <developer>
            <id>zhanghaitao</id>
            <name>Haitao Zhang</name>
            <email>2636461000@qq.com</email>
        </developer>
    </developers>

    <dependencies>
        <dependency>
            <groupId>com.my</groupId>
            <artifactId>hello-apache-dubbo-api</artifactId>
            <version>${project.version}</version>
        </dependency>

        <!-- Spring Boot dependencies -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-actuator</artifactId>
        </dependency>

        <!-- Dubbo Spring Cloud Starter -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-dubbo</artifactId>
        </dependency>

        <!-- Spring Cloud Nacos Service Discovery -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-nacos-discovery</artifactId>
        </dependency>

        <!-- kryo 高速序列化 -->
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-serialization-kryo</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <mainClass>com.my.apache.dubbo.provider.ProviderApplication</mainClass>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

**2、创建服务入口ProviderApplication**

```java
package com.my.apache.dubbo.provider;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class ProviderApplication {

    public static void main(String[] args) {
        SpringApplication.run(DubboProviderApplication.class,args);
    }
}
```

**3、创建服务提供方实现EchoServiceImpl**

```java
package com.my.apache.dubbo.provider.service;

import com.my.apache.dubbo.api.EchoService;
import org.apache.dubbo.config.annotation.Service;
import org.springframework.beans.factory.annotation.Value;

@Service(version = "1.0.0")
public class EchoServiceImpl implements EchoService {

    @Value("${dubbo.protocol.port}")
    private String port;

    @Override
    public String echo(String string) {
        return "zht :" + port + " : " + string;
    }
}

```

**4、创建配置文件bootstrap.yaml**

```yaml
dubbo:
  # 修改dubbo的负载均衡策略为轮询
  provider:
    loadbalance: roundrobin
  scan:
    base-packages: com.my.apache.dubbo.provider.service
  protocol:
    name: dubbo
    port: -1
    serialization: kryo
  registry:
    address: spring-cloud://localhost

spring:
  application:
    name: dubbo-provider
  main:
    allow-bean-definition-overriding: true
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.31.219:8848
```

### 六、实现Dubbo服务消费方

**1、创建hello-apache-dubbo-consumer服务消费方工程**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.my</groupId>
        <artifactId>hello-apache-dubbo</artifactId>
        <version>0.0.1-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>

    <artifactId>hello-apache-dubbo-consumer</artifactId>
    <packaging>jar</packaging>

    <licenses>
        <license>
            <name>Apache 2.0</name>
            <url>https://www.apache.org/licenses/LICENSE-2.0.txt</url>
        </license>
    </licenses>
    <developers>
        <developer>
            <id>zhanghaitao</id>
            <name>Haitao Zhang</name>
            <email>2636461000@qq.com</email>
        </developer>
    </developers>

    <dependencies>
        <dependency>
            <groupId>com.my</groupId>
            <artifactId>hello-apache-dubbo-api</artifactId>
            <version>${project.version}</version>
        </dependency>

        <!-- Spring Boot dependencies -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-actuator</artifactId>
        </dependency>

        <!-- Dubbo Spring Cloud Starter -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-dubbo</artifactId>
        </dependency>

        <!-- Spring Cloud Nacos Service Discovery -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-nacos-discovery</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-nacos-config</artifactId>
        </dependency>

        <!-- kryo 高速序列化 -->
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-serialization-kryo</artifactId>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <mainClass>com.my.apache.dubbo.consumer.ConsumerApplication</mainClass>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

**2、创建服务入口ConsumerApplication**

```java
package com.my.apache.dubbo.consumer;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class ConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConsumerApplication.class, args);
    }
}
```

**3、创建服务消费实现EchoController**

```java
package com.my.apache.dubbo.consumer.controller;

import com.my.apache.dubbo.provider.api.EchoService;
import org.apache.dubbo.config.annotation.Reference;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.context.config.annotation.RefreshScope;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

@RefreshScope
@RestController
public class EchoController {

    @Reference(version = "1.0.0")
    private EchoService echoService;
 		@Value("${user.name}")
    private String username;

    @GetMapping(value = "/echo/{string}")
    public String echo(@PathVariable String string) {
        return echoService.echo(string) + " --  " + username;
    }
}
```

**4、创建Nacos外置配置文件bootstrap.properties**

```properties
spring.application.name=dubbo-consumer-config
spring.cloud.nacos.server-addr=192.168.31.219:8848
spring.cloud.nacos.config.file-extension=yaml
```

**5、Nacos创建配置文件dubbo-consumer-config.yaml**

```yaml
spring:
  # Spring 应用名称，用于 Spring Cloud 服务注册和发现
  # 该值在 Dubbo Spring Cloud 加持下被视作 dubbo.application.name
  # 无需再显示地配置 dubbo.application.name
  application:
    name: dubbo-consumer
  # Spring Boot 2.1 需要设定  
  main:
    allow-bean-definition-overriding: true
  # Nacos注册中心配置  
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.31.219:8848
dubbo:
  # 指定 Dubbo 服务实现类的扫描基准包
  scan:
    base-packages: com.my.apache.dubbo.consumer.controller
  # Dubbo 服务暴露的协议配置，其中子属性 name 为协议名称，port 为协议端口（ -1 表示自增端口，从 20880 开始）
  protocol:
    name: dubbo
    port: -1
    # 使用kryo高速序列化
    serialization: kryo
  # 挂载到 当前配置的Spring Cloud注册中心    
  registry:
    address: spring-cloud://localhost
  # 订阅服务提供方的应用名称的列表，若需订阅多应用，使用 "," 分割。 不推荐使用默认值为 "*"，它将订阅所有应用。
  cloud:
    subscribed-services: dubbo-provider
    
server:
  port: 9090
  
endpoints:
  dubbo:
    enabled: true
management:
  health:
    dubbo:
      status:
        defaults: memory
        extras: threadpool
  endpoints:
    web:
      exposure:
        include: "*"
        
user:
  name: '测试动态配置变动'        
```

### 七、Dubbo服务提供方负载均衡

#### 配置负载均衡

- 修改 `dubbo-provider` 项目的负载均衡策略，默认的负载均衡策略是 **随机**，我们修改为 **轮循**，可配置的值分别是：`random（随机）`，`roundrobin（轮询）`，`leastactive（最少活跃数）`，`consistenthash（一致性Hash）`

```yaml
dubbo:
  provider:
    loadbalance: roundrobin
```

- 修改 `dubbo-provider` 的协议端口为 20880 和 20881，并启动多个实例，IDEA 中依次点击 **Run** -> **Edit Configurations** 并勾选 **Allow parallel run** 以允许 IDEA 多实例运行项目

![img](http://www.qfdmy.com/wp-content/uploads/2019/08/9b0a0ba30182ae3.png)

- Nacos Server 控制台可以看到 `dubbo-provider` 有 2 个实例

![img](http://www.qfdmy.com/wp-content/uploads/2019/08/6b78be78da8dc0a.png)

### 八、Dubbo服务消费方外部化配置文件

略~

> 参考: [千峰达摩院](http://www.qfdmy.com)