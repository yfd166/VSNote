# Spring Cloud组件：搭建 Eureka 服务注册中心
## 简介
Eureka服务注册中心是netflix开源组织提供的一个服务高可用的解决方案，在前端时间一直在疯传的2.0开源流产的问题，其实并不影响我们的使用，netflix只不过是不再维护2.0分支的开源代码，所以做出了免责声明，不过对于我们使用者来说确实比较担心这一点，还有不少人更换服务注册中心，比如：zookeeper、consul。

当然对于Eureka 2.0 流产这件事情就当做一场闹剧来对待吧，因为SpringCloud.Finchley.SR1版本依赖的Eureka是1.9.3，根本不需要考虑到这一点了。
我们还是来关心我们的分布式微服务架构系统该怎么去设计。
## 构建项目
跟我们之前构建`SpringBoot`项目一样， 使用idea工具直接创建一个新的SpringBoot项目，在选择依赖的界面勾选Cloud Discovert -> Eureka Server依赖，创建完成后的pom.xml配置文件内容如下：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" 
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.4.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.example</groupId>
    <artifactId>demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>demo</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
        <!-- SpringCloud最新稳定版本 -->
        <spring-cloud.version>Greenwich.SR1</spring-cloud.version>
    </properties>

    <dependencies>
    <!--Netflix Eureka依赖-->
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
 <!--SpringCloud依赖版本管理-->
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

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```
我们在创建新的项目时，如果选择了相关SpringCloud的依赖，则会自动在pom.xml配置文件内添加SpringCloud最新稳定版本依赖配置。
spring-cloud-dependencies这个依赖是SpringCloud内所需要依赖的版本维护，在maven项目内如果被`<dependencyManagement>`内修饰的`<dependency>`，子项目或者本项目在使用时可以不用设置版本号，默认使用`<dependencyManagement>`下`<dependency>`内设置的版本信息。
正因如此，这也是为什么我们添加了spring-cloud-dependencies依赖后，在使用相关SpringCloud插件时可以不用添加version标签设置导入指定版本的依赖。
## Eureka Server 的配置
添加spring-cloud-starter-netflix-eureka-server依赖后，我们就来看看怎么开启Eureka Server服务。开启Eureka的注册中心服务端比较简单，只需要修改注意两个地方。
* 第一个地方是在入口类上添加启用Eureka Server的注解@EnableEurekaServer，如下所示：
    ```java
    @SpringBootApplication
    @EnableEurekaServer
    public class DemoApplication {

        public static void main(String[] args) {
            SpringApplication.run(DemoApplication.class, args);
        }

    }
    ```
* 第二个地方是application.yml/application.properties文件内添加配置基本信息，如下所示：
    ```yml
    # 服务名称
    spring:
        application:
            name: hengboy-spring-cloud-eureka
    # 服务端口号
    server:
        port: 8880

    #Eureka 相关配置
    eureka:
        client:
            service-url:
                defaultZone: http://localhost:${server.port}/eureka/
            # 是否从其他的服务中心同步服务列表
            fetch-registry: false
            # 是否把自己作为服务注册到其他服务注册中心
            register-with-eureka: false
    ```
* spring.application.name:服务名称
* server.port:服务端口号
* eureka.client.service-url.defaultZone:Eureka默认的服务地址空间信息配置
* eureka.client.fetch-registry:是否从其它服务注册中心同步服务列表(单节点不启用)
* eureka.client.register-with-eureka:是否将自己作为服务注册到其他Eureka服务注册中心(单节点不启用)
## 运行测试
上面的步骤我们已经把Eureka服务端所需要的依赖以及配置进行了集成，接下来我们来运行测试看下效果，Eureka给我们提供了一个漂亮的管理界面，这样我们就可以通过管理界面来查看注册的服务列表以及服务状态等信息。
> 测试步骤：
> 1. 通过Application方式进行启动Eureka Server
> 2. 在本地浏览器访问http://localhost:8880，8880端口号是我们在application.yml配置文件内设置的server.port的值。
> 3. 成功访问到Eureka Server管理界面        

界面如下所示：
![Eureka服务注册管理中心](/image/Eureka服务注册管理中心.png)
对于界面我们可以看到一些Eureka Server的健康数据以及基本信息，比如：
* server-uptime:已启动耗时
* current-memory-usage:当前占用内存总量
* Instances currently registered with Eureka:注册到该中心的服务列表
* ipddr:当前Eureka Server的IP地址，如果没有配置eureka.instance.ip-address那么这里使用默认的IP地址。     

···此处省略一万字

# SpringCloud组件：将服务提供者注册到Eureka服务中心
Eureka提供了Server当然也提供了Client。

本章构建的项目其实是一个Eureka Client，因为是向Eureka Server注册的服务，相对于Eureka Server来说相当于一个客户端的形式存在。

我们使用spring-cloud-starter-netflix-eureka-client可以快速的构建Eureka Client项目，简单的配置就可以完成Client与Server之间的通信以及绑定，下面我们来看下具体是怎么向Eureka Server注册服务。

## 构建项目
同样的是采用idea开发工具创建一个SpringBoot项目，在依赖选择界面对应的添加Web以及Eureka Discovery依赖，直接完成创建项目。
项目的pom.xml内容如下所示：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" 
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.4.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.example</groupId>
    <artifactId>test</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>test</name>
    <description>Demo project for Spring Boot</description>

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

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```
跟Eureka Server项目不同依赖的选择的地方是Client项目需要添加spring-cloud-starter-netflix-eureka-client，通过该依赖可以完成服务的注册以及服务之间的通信等。
> 添加spring-boot-starter-web依赖的目的是为了简单创建一个Controller请求示例，在后面章节我们需要用到该依赖。
## Eureka Client 的配置
Eureka Client的配置步骤与Eureka Server几乎是一致的，不过采用的注解不同以及配置信息有出入，同样是两步完成配置：
* 第一步入口类添加注解@EnableDiscoveryClient
我们在配置Client时通常会采用通用的客户端注解配置，也就是@EnableDiscoveryClient注解，当然如果服务注册中心确定采用的是Eureka也可以使用@EnableEurekaClient注解来完成配置，至于这两个的区别后续章节细讲。
    ```java
    @SpringBootApplication
    @EnableDiscoveryClient
    public class TestApplication {

    //Logger instance
    static Logger logger = LoggerFactory.getLogger(TestApplication.class);

        public static void main(String[] args) {
            SpringApplication.run(TestApplication.class, args);
            logger.info("「「「「「Eureka服务提供者启动完成.」」」」」");
        }

    }

    ```
* 第二步application.yml配置文件添加配置信息
我比较喜欢ymal这种配置风格，所以删除了创建项目时创建的application.properties配置文件，自行创建了application.yml，因为层级的原因可以更清晰明了的看清配置，配置内容如下所示：
    ```yml
    # 服务名称
    spring:
        application:
            name: hengboy-spring-cloud-eureka-provider

    # 服务提供者端口号
    server:
        port: 8881

    # 配置Eureka Server 信息
    eureka:
        client:
            service-url:
                defaultZone: http://localhost:8880/eureka/
    ```
* spring.application.name：配置服务的名称
* server.port：服务端口号
* eureka.client.service-url：配置Eureka Server服务注册中心地址

## 运行测试
我们已经完成了Eureka Client的相关配置信息，接下来我们按照下面的步骤进行执行测试。
> 1. 启动服务注册中心Eureka Server
> 2. 启动本章项目
> 3. 查看控制台日志输出信息
> 4. 查看服务注册中心管理界面服务列表
运行过程中本章项目控制台输出内容如下所示：
```
DiscoveryClient_HENGBOY-SPRING-CLOUD-EUREKA-PROVIDER/DESKTOP-U9PLQP4:hengboy-spring-cloud-eureka-provider:8881: registering service...
DiscoveryClient_HENGBOY-SPRING-CLOUD-EUREKA-PROVIDER/DESKTOP-U9PLQP4:hengboy-spring-cloud-eureka-provider:8881 - registration status: 204
```
可以看到控制台打印了向我们配置的服务注册中心进行registering service，既然控制台并没有给我抛出相关的异常信息，那么我们猜想是不是Eureka Server服务注册中心的服务列表已经存在了该条记录了呢？
## 查看Eureka Server 服务列表
我们带着这个疑问打开Eureka Server管理界面地址：http://localhost:8880。
![EurekaClient](/image/EurekaClient.png)
在管理界面我们可以看到本章的服务已经注册到了Eureka Server服务注册中心，而且是UP状态也就是正常运行状态。

在服务注册的过程中，SpringCloud Eureka为每一个服务节点都提供默认且唯一的实例编号(InstanceId)
* 实例编号默认值：
`${spring.cloud.client.ipAddress}:${spring.application.name}:${spring.application.instance_id:${server.port}}`
* 本章服务注册时的实例编号：`DESKTOP-U9PLQP4:hengboy-spring-cloud-eureka-provider:8881`
> 可以自定义实例编号，但是需要保证唯一性



