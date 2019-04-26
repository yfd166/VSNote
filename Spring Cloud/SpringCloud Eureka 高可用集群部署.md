# SpringCloud组件：Eureka高可用集群部署
我们在之前文档中的SpringCloud组件：搭建Eureka服务注册中心学习到了单个服务注册中心的创建，不过单模式的部署方式在实战中确实不太提倡，因为有很多种原因可能会导致服务注册中心宕机，如果宕机就会有一些灾难性的问题出现，所以保证服务注册中心处于活着运行状态显得尤为重要！！
使用idea开发工具创建一个SpringBoot项目，添加Eureka Server依赖即可，pom文件与前文的eureka-server相同。
## 启用Eureka Server
在入口类XxxApplication上添加@EnableEurekaServer注解来启用Eureka Server服务以及实例化一些依赖，修改如下所示：
```java
@SpringBootApplication
@EnableEurekaServer
public class SpringCloudEurekaHighApplication {
    //....
}
```
## Eureka服务配置
依赖已经添加完成，接下来我们就需要在application.yml内编写相关配置信息，因为测试环境都在我们本机，有两种方式可以模拟测试同时运行：
* 创建两个不同的项目
* 使用一个项目进行根据spring.profiles.active设置运行不同环境
为了方便演示，我们使用的第一种方式
## 环境配置
我们在项目1的src/main/resources目录下创建名为application.yml的配置文件，在该配置文件内添加如下配置：
```yml
# 服务名称
spring:
  application:
    name: master1-eureka-server
# 服务端口号
server:
  port: 8880

#Eureka 相关配置
eureka:
  client:
    service-url:
      defaultZone: http://node2:8890/eureka/
    # 是否从其他的服务中心同步服务列表
    fetch-registry: true
    # 是否把自己作为服务注册到其他服务注册中心
    register-with-eureka: true
  instance:
    hostname: node1
    instance-id: ${eureka.instance.hostname}:${server.port}:@project.version@
  server:
    peer-node-connect-timeout-ms: 1000

```
继续在项目2的src/main/resources下创建一个名为application.yml的配置文件，内容如下所示：
```yml
# 服务名称
spring:
  application:
    name: master2-eureka-server
# 服务端口号
server:
  port: 8890

#Eureka 相关配置
eureka:
  client:
    service-url:
      defaultZone: http://node1:8880/eureka/
    # 是否从其他的服务中心同步服务列表
    fetch-registry: true
    # 是否把自己作为服务注册到其他服务注册中心
    register-with-eureka: true
  instance:
    #配置通过主机名方式注册
    hostname: node2
    #配置实例编号
    instance-id: ${eureka.instance.hostname}:${server.port}:@project.version@
  #集群节点超时时间
  server:
    peer-node-connect-timeout-ms: 1000
```
## 主机名设置
* Mac或者Linux配置方式
如果你使用的是osx系统。可以找到/etc/hosts文件并添加如下内容：
```shell
127.0.0.1       node1
127.0.0.1       node2
```
一般情况下配置完成后就会生效，如果你的配置并没有生效，你可以尝试重启。
* Windows配置方式
如果你使用的是windows系统，你可以修改C:\Windows\System32\drivers\etc\hosts文件，添加内容与Mac方式一致。
## Eureka Sever相互注册
* 项目1 application.yml
eureka.client.service-url.defaultZone这个配置参数的值，配置的是http://node2:8890/eureka/，那这里的node2是什么呢？其实一看应该可以明白，这是们在hosts文件内配置的hostname，而端口号我们配置的则是8890，根据hostname以及port我们可以看出，环境node1注册到了node2上。
* 项目2 application-node2.yml
在node2环境内配置eureka.client.service-url.defaultZone是指向的http://node1:8880/eureka/，同样node2注册到了node1上。
> 通过这种相互注册的方式牢靠的把两个服务注册中心绑定在了一块。
## 运行测试
1. 运行两个项目
2. 访问http://node1:8880查看node1环境的Eureka管理中心
![集群相互注册效果图](/image/Eureka集群相互注册效果.png)
3. 访问http://node2:8890查看node2环境的Eureka管理中心
![集群相互注册效果图](/image/Eureka集群相互注册效果2.png)

# 将服务提供者注册到Eureka集群
更改服务提供者application.yml文件
```yml
······
eureka:
  client:
    service-url:
      defaultZone: http://node1:8880/eureka/,http://node2:8890/eureka/
······
```
# SpringCloud组件：Eureka服务注册中心的失效剔除与自我保护机制
Eureka作为一个成熟的服务注册中心当然也有合理的内部维护服务节点的机制，比如我们本节将要讲解到的服务下线、失效剔除、自我保护，也正是因为内部有这种维护机制才让Eureka更健壮、更稳定。
## 服务下线
迭代更新、终止访问某一个或者多个服务节点时，我们在正常关闭服务节点的情况下，Eureka Client会通过PUT请求方式调用Eureka Server的REST访问节点/eureka/apps/{appID}/{instanceID}/status?value=DOWN请求地址，告知Eureka Server我要下线了，Eureka Server收到请求后会将该服务实例的运行状态由UP修改为DOWN，这样我们在管理平台服务列表内看到的就是DOWN状态的服务实例。
## 失效剔除
Eureka Server在启动完成后会创建一个定时器每隔60秒检查一次服务健康状况，如果其中一个服务节点超过90秒未检查到心跳，那么Eureka Server会自动从服务实例列表内将该服务剔除。
> 由于非正常关闭不会执行主动下线动作，所以才会出现失效剔除机制，该机制主要是应对非正常关闭服务的情况，如：内存溢出、杀死进程、服务器宕机等非正常流程关闭服务节点时。
## 自我保护
Eureka Server的自我保护机制会检查最近15分钟内所有Eureka Client正常心跳的占比，如果低于85%就会被触发。
我们如果在Eureka Server的管理界面发现如下的红色内容，就说明已经触发了自我保护机制
> EMERGENCY! EUREKA MAY BE INCORRECTLY CLAIMING INSTANCES ARE UP WHEN THEY'RE NOT. RENEWALS ARE LESSER THAN THRESHOLD AND HENCE THE INSTANCES ARE NOT BEING EXPIRED JUST TO BE SAFE.    

当触发自我保护机制后Eureka Server就会锁定服务列表，不让服务列表内的服务过期，不过这样我们在访问服务时，得到的服务很有可能是已经失效的实例，如果是这样我们就会无法访问到期望的资源，会导致服务调用失败，所以这时我们就需要有对应的容错机制、熔断机制，我们在接下来的文章内会详细讲解这块知识点。

我们的服务如果是采用的公网IP地址，出现自我保护机制的几率就会大大增加，所以这时更要我们部署多个相同InstanId的服务或者建立一套完整的熔断机制解决方案。
## 自我保护开关
如果在本地测试环境，建议关掉自我保护机制，这样方便我们进行测试，也更准备的保证了服务实例的有效性！！！
> 关闭自我保护只需要修改application.yml配置文件内参数eureka.server.enable-self-preservation将值设置为false即可。