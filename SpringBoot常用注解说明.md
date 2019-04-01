# SpringBoot 常用注解说明
### @SpringBootApplication
```java
@SpringBootApplication
public class SpringBootApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringBootApplication.class);
    }
}
```
标志着这是一个SpringBoot入口文件，所有SpringBoot应用程序都依赖这注解启动
### @Configuration @Bean
```java
@Configuration
public class AppConfig{
    @Bean("user")
    public User initUser() {
        User user = new User();
        user.setId(1L);
        user.setUserName("user_name_1");
        user.setNote("note_1");
        return user;
    }
}
```
@Configuration 代表这是一个Java配置文件，Spring的容器会根据它来生成IOC容器去装配bean     
@Bean 代表将initUser方法返回的POJO装配到IoC容器中，而其name属性定义了这Bean的名称，如果没有配置name属性，则将方法名initUser作为Bean的名称存入Spring IoC容器中
### @Component @Value
```java
@Component("user")
public class User {

    @Value("1")
    private Long id;

    @Value("user_name_1")
    private String userName;

    @Value("note_1")
    private String note;

    /** setter and getter **/
}
```
@Component 表明这个类将被 Spring IoC 容器扫描装配，其中配置的“user”则是作为Bean的名称，当然也可以不配置这个字符串，那么IoC容器就会把类名第一个字母作为小写，其他不变作为Bean名称存入IoC容器中       
@Value 指定具体值，使得Spring IoC 给予对应的属性注入对应的值

### @ComponentScan
```java
@ComponentScan
public class AppConfig {}
```
使用 @ComponentScan 意味着它会进行扫描，但是它只会扫描类AppConfig所在的当前包及其子包
```java
@ComponentScan("com.springboot.demo.*")
```
以上方式可以实现扫描 com.springboot.demo 包下所有所有类，推荐使用这种方式扫描这种方式进行指定路径扫描
### @AutoWired
```java
@AutoWired
private Animal animal;
```
@AutoWired是我们在Spring中最常用的注解之一，它会根据属性的类型找到对应的Bean进行装配
### @Primary
```java
@Component
@Primary
public class cat implements Animal{}
```
告诉 Spring IoC 容器，当发现存在多个同样类型的Bean时，优先使用我注入
### @Qualifier
```java
@Component
public class BussinessPerson implements Person {

    @AutoWired
    @Qualifier("cat")
    private Animal animal;
}

@Component
@Qualifier("cat")
public class Cat implements Animal {}

@Component
@Qualifier("dog")
public class Dog implements Animal {}
```
当存在多个相同类型的Bean时，使用 @Qualifier 实现通过类型和名称找到Bean

