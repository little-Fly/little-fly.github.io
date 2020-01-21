# springboot 学习笔记

## 1、环境准备：

> 开发工具 idea
>
> jdk1.8
>
> mysql 8.0
>
> apache-maven-3.5.4

## 2、使用idea快速创建一个springboot项目

在idea新建一个project，选择Spring Initializr，这样我们就可以直接联网创建一个springboot项目

![idea创建项目](study/1.PNG)

![idea创建项目](study/2.PNG)

这里我们都选择默认的，项目基于maven管理，并在打包的时候打包为jar包，项目完成之后可以直接丢到服务器，执行命令 java -jar xxxx.jar就可以把我们的项目跑起来。

点击下一步，springboot会让我们选择我们所需要的依赖，比如下面几个，为了我们项目先运行起来，我们只选择第一个

> Web -> Spring Web: 可以提供Rest服务  
>
> Developer Tools -> Lombok: 这是一个可以省去getter、setter、构造方法、tostring方法等的一个依赖，使用注解就可以实现。
>
> SQL -> MySQL Driver: 我们使用mysql，所以我们这个依赖帮我们提供mysql驱动()
>
> SQL -> MyBatis Framework: 数据库持久层框架

这样，我们就完成了springboot项目的创建。项目结构如下：

![idea创建项目](study/3.PNG)

- DemoApplication是我们springboot项目的程序入口，由注解@SpringBootApplication标识，我们可以在这里直接运行这个项目

- application.properties文件是进行springboot项目配置的地方，比如配置访问端口、日志、等等都在这里配置

springboot默认支持两种配置文件，一种是我们看到的application.properties，另一种就是application.yml。也就是说我们可以用application.yml来替换这个文件。我们删掉application.properties，新建application.yml文件。

文件建好后，我们进行第一个配置，配置我们rest的访问端口为8081（默认为8080）

![idea创建项目](study/4.PNG)

## 3、编写第一个rest接口 helloworld

1、创建controller类

在生成的代码路径下新建controller包，然后再controller包下新建我们的第一个Controller类FirstController.java

@RestController标识是一个rest服务类，@RequestMapping指定访问路径

```java
package com.example.demo.controller;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class FirstController {
    @RequestMapping("helloworld")
    public String helloworld() {
        return "helloworld";
    }
}
```
创建完成后我们回到DemoApplication入口类点击运行，项目启动成功如下：

![idea创建项目](study/5.PNG)

然后我们访问我们的第一个helloworld接口，打开浏览器访问 http://localhost:8081/helloworld

![idea创建项目](study/6.PNG)

## 4、增加与数据库交互实现用户管理

1、 添加**mysql**与**mybatis**依赖,这里我们尝试mybatis-plus

```xml

<dependency>
   <groupId>mysql</groupId>
   <artifactId>mysql-connector-java</artifactId>
   <scope>runtime</scope>
</dependency>

<dependency>
   <groupId>com.baomidou</groupId>
   <artifactId>mybatis-plus-boot-starter</artifactId>
   <version>2.3</version>
</dependency>

```

2、 添加完依赖进行一些基础配置，打开application.yml，我们提前装好mysql 8.0版本，并新建一个数据库test

```xml

<!--配置数据源-->
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/test?characterEncoding=utf8&useSSL=false&serverTimezone=UTC&rewriteBatchedStatements=true
    username: xxxx
    password: xxxxx

mybatis:
  type-aliases-package: com.springboot.demo.model
  mapper-locations: classpath:mybatis/*.xml

```

- type-aliases-package： 扫描我们的自定义POJO
- mapper-locations： 扫描我们的mapper文件

3、新建UserCrotroller、User、UserMapper、UserMapper.xml类

```java
import lombok.Data;

@Data
public class User {
    private int id;

    private String name;

    private String birth;
}
```

```java
@RestController
public class UserController {

    @Autowired
    private UserMapper userMapper;

    @RequestMapping("/queryAll")
    public List<User> queryAll() {
        List<User> userList = userMapper.queryAll();
        return userList;
    }
}
```

```java
public interface UserMapper{

    List<User> queryAll();
}
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.demo.mapper.UserMapper">
    <resultMap id="resultMap" type="com.example.demo.model.User">
        <result property="id" column="id"/>
        <result property="name" column="name"/>
        <result property="birth" column="birth"/>
    </resultMap>

    <select id="queryAll" resultMap="resultMap">
        select * from user
    </select>
</mapper>
```

4、 启动类增加对mapper的扫描注解

```java
@SpringBootApplication
@MapperScan(basePackages = "com.example.demo.mapper")
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

}

```
最后结构如下所示
![](4-1)

5、 启动项目，浏览器访问localhost:8080/queryAll

## 5、查询分页与log配置

1、 application.yml添加Log配置,定义文件名、输出路径、日志级别

```xml
logging:
  file:
    name: springboot.log
    path: D:\springboot
  pattern:
    level: info
```

2、 使用mybatis plus来进行查询分页

mybatis plus里有一个Page类可以帮助我们实现分页，只需要将Page参数放到我们的查询方法

```java
public interface UserMapper{

   List<User> queryAll(Page page);
}
```
可以直接使用@Slf4j来使用log

```java
@RestController
@Slf4j
public class UserController {

    @Autowired
    private UserMapper userMapper;

    @RequestMapping("/queryAll")
    public List<User> queryAll(@RequestParam int pageNum, @RequestParam int pageSize) {
		Page page = new Page(pageNum, pageSize)
        List<User> userList = userMapper.queryAll(page);
		log.info("userList size = {}", userList.size());
        return userList;
    }
}
```

3、启动项目，访问http://localhost:8080/queryAll?pageNum=1&pageSize=5

![](4-3)

## 6、配置redis

1、 pom文件添加redis依赖

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
```

2、 application.yml添加redis基础配置

```xml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/test?characterEncoding=utf8&useSSL=false&serverTimezone=UTC&rewriteBatchedStatements=true
    username: xxxxx
    password: xxxxx
  redis:
    host: localhost
    port: 6379
```

3、编写一个简单的redisUtils操作类

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.stereotype.Component;
import java.util.concurrent.TimeUnit;

@Component
public class RedisUtils {
    @Autowired
    private RedisTemplate<String, String> redisTemplate;

    public void setValue(String key, String value, long time) {
        redisTemplate.opsForValue().set(key, value , time, TimeUnit.SECONDS);
    }

    public String getValue(String key) {
        return redisTemplate.opsForValue().get(key);
    }
}
```

4、测试一下

```java
@SpringBootTest
public class RedisTest {

    @Autowired
    private RedisUtils redisUtils;

    @Test
    public void testRedis() {
        redisUtils.setValue("redisKey", "redisValue", 1000);
        System.out.println(redisUtils.getValue("redisKey"));
    }
}
```

## 7、使用AOP进行身份验证

为了系统安全，我们在客户端访问提供的接口之前，需要做身份验证，验证用户是否登录或者是否有权限进行某些操作，可以利用springAOP来实现，同理，springAOP还可以做日志管理等其它可以切面化功能。

#### 1、添加springAOP依赖

```xml
<dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

#### 2、增强类型（切入类型）

> @Before：该注解标注的方法在业务模块代码执行之前执行，其不能阻止业务模块的执行，除非抛出异常；
> @AfterReturning：该注解标注的方法在业务模块代码执行之后执行；
> @AfterThrowing：该注解标注的方法在业务模块抛出指定异常后执行；
> @After：该注解标注的方法在所有的Advice执行完成后执行，无论业务模块是否抛出异常，类似于finally的作用；
> @Around：该注解功能最为强大，其所标注的方法用于编写包裹业务模块执行的代码，其可以传入一个ProceedingJoinPoint用于调用业务模块的代码，无论是调用前逻辑还是调用后逻辑，都可以在该方法中编写，甚至其可以根据一定的条件而阻断业务模块的调用；

#### 3、切点表达式

##### 3.1 execution
>  由于Spring切面粒度最小是达到方法级别，而execution表达式可以用于明确指定方法返回类型，类名，方法名和参数名等与方法相关的部件，并且在Spring中，大部分需要使用AOP的业务场景也只需要达到方法级别即可，因而execution表达式的使用是最为广泛的。如下是execution表达式的语法：
> execution(modifiers-pattern? ret-type-pattern declaring-type-pattern?name-pattern(param-pattern) throws-pattern?)

这里问号表示当前项可以有也可以没有，其中各项的语义如下：

- modifiers-pattern：方法的可见性，如public，protected；
- ret-type-pattern：方法的返回值类型，如int，void等；
- declaring-type-pattern：方法所在类的全路径名，如com.spring.Aspect；
- name-pattern：方法名类型，如buisinessService()；
- param-pattern：方法的参数类型，如java.lang.String；
- throws-pattern：方法抛出的异常类型，如java.lang.Exception；

------------


##### 3.2 within
> within表达式的粒度为类，其参数为全路径的类名（可使用通配符），表示匹配当前表达式的所有类都将被当前方法环绕。如下是within表达式的语法：

*within(declaring-type-pattern)*

within表达式只能指定到类级别，如下示例表示匹配com.spring.service.BusinessObject中的所有方法：
*within(com.spring.service.BusinessObject)*

within表达式路径和类名都可以使用通配符进行匹配，比如如下表达式将匹配com.spring.service包下的所有类，不包括子包中的类：
*within(com.spring.service.*)*

如下表达式表示匹配com.spring.service包及子包下的所有类：
*within(com.spring.service..*)*

还有其他多种请参考https://www.cnblogs.com/zhangxufeng/p/9160869.html

####4、编写切面类来对所有对外接口进行登录校验

这里我们使用@Before("execution(public * com.example.demo.controller.*.*(..))") 切入cotroller包下的所有类的所有public方法，也就是在目标方法执行前先执行我们的@Before方法。方法中对请求接口的参数HttpServletRequest携带的请求头third_session进行校验，如果合法，则继续执行，否则返回401异常。

```java
@Aspect
@Component
@Slf4j
public class LoginAspect {

    @Before("execution(public * com.example.demo.controller.*.*(..))")
    public void beforeMethod(JoinPoint joinpoint) throws Exception {
        String methodName = joinpoint.getSignature().getName();
        Object[] args = joinpoint.getArgs();
        log.info("method {} begin...", methodName);
        if (args[0] instanceof HttpServletRequest) {
            HttpServletRequest httpServletRequest = (HttpServletRequest) args[0];
            String third_session = httpServletRequest.getHeader("third_session");
            //如果携带的session有效，则无需登录，否则抛出异常401，提示重新登录
            if ("true".equals(third_session)) {
                log.info("third_session is {}", third_session);
            } else {
                throw new ServerException();
            }
        }
    }
}
```

```java
@ResponseStatus(code = HttpStatus.UNAUTHORIZED , reason = "user not login")
public class ServerException extends Exception {
}
```












