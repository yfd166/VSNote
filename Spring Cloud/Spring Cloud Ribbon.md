# SpringCloud组件：Eureka服务消费者与Ribbon负载均衡
现在已经构建了服务注册中心和服务提供中心，下面就来构建服务消费者：
服务消费者主要完成：发现服务和消费服务。其中服务的发现主要由Eureka的客户端完成，而消费的任务由Ribbon完成。
Ribbon是一个基于HTTP和TCP的客户端负载均衡器，它可以通过客户端中配置ribbonServerList服务端列表去轮询访问以达到负载均衡的作用。当Ribbon和Eureka联合使用时，Ribbon的服务实例清单RibbonServerList会被DiscoveryEnabledNIWSServerList
重写，扩展成从Eureka注册中心获取服务端列表。同时它也会用NIWSDiscoveryPing来取代Ping，通过Eureka来确定服务端是否已经启动。
首先，启动eureka-server和eureka-client服务，为了实现Ribbon客户端负载均衡功能，启动两个不同端口的eureka-client.

## 构建项目
我们只需要创建一个服务节点项目即可，因为服务提供者也是消费者，然后将本项目注册到之前编写的服务注册中心。
我们使用idea开发工具创建一个空的Maven项目，删除src目录
#### 服务注册中心：Eureka-Server
在父工程下再创建一个SpringBoot工程，实现服务注册中心功能。
* pom
```xml
<properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.SR1</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```
* application.yml
```yml
spring:
  application:
    name: eureka-server
server:
  port: 8000
eureka:
  client:
    service-url:
      defaultZone: http://localhost:${server.port}/eureka/
    fetch-registry: false
    register-with-eureka: false
  server:
    enable-self-preservation: false
```
***`enable-self-preservation: false`关闭EurekaServer自我保护机制***
#### 服务提供者：Eureka-Client
在父工程下创建两个SpringBoot工程，实现服务提供者功能
* pom
两个Client的pom文件一致
```xml
<properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.SR1</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```
* application.yml
    Client1的application.yml
    ```yml
        spring:
          application:
            name: eureka-client
        server:
          port: 8001
        eureka:
          client:
            service-url:
              defaultZone: http://localhost:8000/eureka/
    ```
    Client2的application.yml
    ```yml
    spring:
      application:
        name: eureka-client
    server:
      port: 8002
    eureka:
      client:
        service-url:
          defaultZone: http://localhost:8000/eureka/
    ```
    两个Eureka-Client的`spring.application.name`必须一致，否则不能实现负载均衡
* IndexController
在XxxApplication同级目录下创建controller包，再创建IndexContrller，内容如下：
```java
package com.eureka.client1.controller;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/index")
public class IndexController {
    @RequestMapping("/test")
    public String logic() {
        return "Client1:8001 { Hello World }";
    }
}
```
`@RestController`：此控制器返回内容全部为字符串，相当于在每个方法上加了一个`@ResponseBody`
#### 服务消费者：Eureka-Consumer
在空Maven项目创建一个SpringBoot工程，过程与创建SpringBoot项目一致，对应的选择`spring-boot-starter-web`、`spring-cloud-starter-netflix-ribbon`、`spring-cloud-starter-netflix-eureka-client`三个依赖，pom.xml配置文件如下所示：
* pom
```xml
<!--省略部分POM内容-->
<properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.SR1</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
<!--省略-->
```
* application.yml
将application.properties改为application.yml，内容如下：
```yml
#服务名
spring:
  application:
    name: eureka-consumer
#端口号
server:
  port: 8003
#将此服务注册到那个EurekaServer
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8000/eureka/
```
***每个“:”字符后需要空一格***
* XxxApplicaiton
修改SpringBoot启动类，内容如下：
```java
package com.eurekaconsumer.eurekaconsumer;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;

@SpringBootApplication
@EnableDiscoveryClient
public class EurekaconsumerApplication {

    @Bean
    @LoadBalanced
    RestTemplate restTemplate() {
        return new RestTemplate();
    }

    public static void main(String[] args) {
        SpringApplication.run(EurekaconsumerApplication.class, args);
    }

}
```
* 创建IndexController
先在XxxApplication同级目录创建controller包，再创建IndexController类，内容如下：
```java
package com.eurekaconsumer.eurekaconsumer.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.client.discovery.DiscoveryClient;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

@RestController
public class IndexController {

    @Autowired
    RestTemplate restTemplate;

    @RequestMapping("/logic")
    public String index() {
        return restTemplate.getForEntity("http://EUREKA-CLIENT/index/test",String.class).getBody();
    }
}

```
### 运行测试
流程如下：
> 1. 启动服务注册中心
> 2. 启动所有Springboot子工程
> 3. 访问lcaolhost:8003/logic/

* Eureka-Server
![负载均衡](/image/负载均衡.png)
* 负载均衡效果图
![负载均衡效果1](/image/负载均衡效果.png)
![负载均衡效果2](/image/负载均衡效果2.png)
