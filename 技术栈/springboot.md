# 什么是Spring？
Spring是重量级企业开发框架EJB的替代品，为企业级Java开发提供了一种相对简单的方式，通过*依赖注入*和*面向切面编程*，用简单的java对象实现了EJB的功能。*虽然Spring的组价代码是轻量级的，但他的配置却是重量级的*。
# 为什么要有SpringBoot？
旨在简化Spring开发（减少配置文件，开箱即用）
# SpringBoot的主要优点
1. 开发基于Spring的应用程序很容易
2. 开发所需的时间明显减少，提升了整体的生产力
3. 不需要编写大量样板代码、xml配置和注释
4. 与Spring生态集成
5. 遵循“固执己见的默认配置”，以减少开发工作
6. 提供嵌入式HTTP服务器，如Tomcat和Jetty，可以轻松开发和测试web应用程序
7. 提供命令行接口（CLI）工具，用于开发和测试SpringBoot应用
8. 提供多种插件
# Spring-Boot-Starters
是一系列依赖关系的集合，因为他的存在，项目的依赖之间的关系对我们来说变得简单
```

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

```
# 内嵌Servlet容器
springboot支持多种嵌入式Servlet容器：如tomcat、jetty、undertow。默认开启tomcat，如果想要使用jetty，则需要修改pom文件
```
<!--从Web启动器依赖中排除Tomcat-->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
	<exclusions>
		<exclusion>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-tomcat</artifactId>
		</exclusion>
	</exclusions>
</dependency>
<!--添加Jetty依赖-->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-jetty</artifactId>
</dependency>
```

# @SpringBootApplication
`@SpringBootApplicaiton`是`@Configuration`、`@EnableAutoConfiguration`、`@ComponentScan`注解的集合。
- `@EnableAutoConfiguration`：启用 SpringBoot 的自动配置机制  
- `@ComponentScan`： 扫描被@Component (@Service,@Controller)注解的 bean，注解默认会扫描该类所在的包下所有的类。  
- `@Configuration`：允许在上下文中注册额外的 bean 或导入其他配置类

# 常用注解
## Spring Bean相关：
- `@AutoWired`:自动导入对象到类中。
- `@RestController`:是`@Controller`和`@ResponseBody`的合计，表示这是个控制器bean，并且将函数的返回值直接填入HTTP响应体中，是REST风格的控制器。
- `@Component`：通用的朱姐，可标注任意类为`Spring`组件。如果不知道一个bean是属于哪个层，可以使用`@Component`。
- `@Repository`:对应持久层DAO层，主要用于数据库相关操作，`@Mapping`类似，需要主启动类`@MapperScan`扫描该接口。
- `@Service`:对应服务层，设计复杂的逻辑。
- `@Controller`：controller层。
- `@RestController`: 是`@Controller`和`@ResponseBody`的合集,表示这是个控制器 bean,并且是将函数的返回值直接填入 HTTP 响应体中,是 REST 风格的控制器。
- `@Scope`:声明SpringBean的作用域。singleton：唯一bean实例，默认单例；prototype：每次请求都会创建一个新的bean实例；request：每一次http请求都会产生一个新的bean，仅在当前httprequest内有效；session：每一个httpsession会产生一个新的bean，仅在当前httpsession内有效。
- 
## 常见HTTP请求类型
- `@GetMapping`:get请求，请求从服务器获取特定资源
- `@PostMapping`：post请求，资源的接受者，不具有幂等性，在服务器上创建一个新的资源
- `@PutMapping`：put请求，用于创建或更新资源，多次put请求与第一次相同，具有幂等性
- `@DeleteMapping`：delete请求，从服务器删除特定的资源
## 前后端传值
- `@RequestParam`以及`@PathVairable`:前者用于获取查询参数，后者用于获取路径参数
- `@RequestBody`:用于读取Request请求，接收到数据之后会自动将数据绑定到Java对象上去
```
@PostMapping("/sign-up")
public ResponseEntity signUp(@RequestBody @Valid UserRegisterRequest userRegisterRequest) {
  userService.save(userRegisterRequest);
  return ResponseEntity.ok().build();
}
```
一个请求方法只能有一个`@RequestBody`，但可以有多个`@RequestParam`和`@PathVariable`

# yml和properties
yml配置的方式更加直观清晰，简单明了，有层次感

# 常用读取配置文件的方法
```
wuhan2020: 2020年初武汉爆发了新型冠状病毒，疫情严重，但是，我相信一切都会过去！武汉加油！中国加油！

my-profile:
  name: Guide哥
  email: koushuangbwcx@163.com

library:
  location: 湖北武汉加油中国加油
  books:
    - name: 天才基本法
      description: 二十二岁的林朝夕在父亲确诊阿尔茨海默病这天，得知自己暗恋多年的校园男神裴之即将出国深造的消息——对方考取的学校，恰是父亲当年为她放弃的那所。
    - name: 时间的秩序
      description: 为什么我们记得过去，而非未来？时间“流逝”意味着什么？是我们存在于时间之内，还是时间存在于我们之中？卡洛·罗韦利用诗意的文字，邀请我们思考这一亘古难题——时间的本质。
    - name: 了不起的我
      description: 如何养成一个新习惯？如何让心智变得更成熟？如何拥有高质量的关系？ 如何走出人生的艰难时刻？
```
## 通过`@value`读取比较简单的配置信息
```
@Value("${wuhan2020}")
String wuhan2020;
```
## 通过`@ConfigurationProperties`读取并于bean绑定
```
@Component
@ConfigurationProperties(prefix = "library")
@Setter
@Getter
@ToString
class LibraryProperties {
    private String location;
    private List<Book> books;

    @Setter
    @Getter
    @ToString
    static class Book {
        String name;
        String description;
    }
}
```
之后就可以将其注入类中使用
```
@SpringBootApplication
public class ReadConfigPropertiesApplication implements InitializingBean {

    private final LibraryProperties library;

    public ReadConfigPropertiesApplication(LibraryProperties library) {
        this.library = library;
    }

    public static void main(String[] args) {
        SpringApplication.run(ReadConfigPropertiesApplication.class, args);
    }

    @Override
    public void afterPropertiesSet() {
        System.out.println(library.getLocation());
        System.out.println(library.getBooks());    }
}
```
### `@PropertySource`读取指定的properties文件
```
@Component
@PropertySource("classpath:website.properties")
@Getter
@Setter
class WebSite {
    @Value("${url}")
    private String url;
}
```
使用
```
@Autowired
private WebSite webSite;

System.out.println(webSite.getUrl());//https://javaguide.cn/
```
# SpringBoot加载配置文件的优先级
![图片](assets/IMG_10.png)

# Spring Boot监控系统实际运行状况
引入SpringbootActuator
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

可以通过get方法访问`/health`接口，获取应用程序的健康指标

# SpringBoot如何做请求参数校验
依赖`spring-boot-starter-web`
## JSR提供的
![图片](assets/IMG_11.png)
```
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Person {

    @NotNull(message = "classId 不能为空")
    private String classId;

    @Size(max = 33)
    @NotNull(message = "name 不能为空")
    private String name;

    @Pattern(regexp = "((^Man$|^Woman$|^UGM$))", message = "sex 值不在可选范围")
    @NotNull(message = "sex 不能为空")
    private String sex;

    @Email(message = "email 格式不正确")
    @NotNull(message = "email 不能为空")
    private String email;

}
```
## 验证请求体（RequestBody）
在需要验证的参数前加上`@Valid`注解，如果验证失败，则抛出`MethodArgumentNotValidException`。默认将其转换为HTTP Status 400
```
@RestController
@RequestMapping("/api")
public class PersonController {

    @PostMapping("/person")
    public ResponseEntity<Person> getPerson(@RequestBody @Valid Person person) {
        return ResponseEntity.ok().body(person);
    }
}
```
## 验证请求参数（PathVariables和RequestParameters）
```
@RestController
@RequestMapping("/api")
@Validated
public class PersonController {

    @GetMapping("/person/{id}")
    public ResponseEntity<Integer> getPersonByID(@Valid @PathVariable("id") @Max(value = 5,message = "超过 id 的范围了") Integer id) {
        return ResponseEntity.ok().body(id);
    }

    @PutMapping("/person")
    public ResponseEntity<String> getPersonByName(@Valid @RequestParam("name") @Size(max = 6,message = "超过 name 的范围了") String name) {
        return ResponseEntity.ok().body(name);
    }
}
```

# 处理全局异常
`@ControllerAdvice`和`@ExceptionHandler`

# SpringBoot中的定时任务
使用`@Scheduled`注解创建
```
@Component
public class ScheduledTasks {
    private static final Logger log = LoggerFactory.getLogger(ScheduledTasks.class);
    private static final SimpleDateFormat dateFormat = new SimpleDateFormat("HH:mm:ss");

    /**
     * fixedRate：固定速率执行。每5秒执行一次。
     */
    @Scheduled(fixedRate = 5000)
    public void reportCurrentTimeWithFixedRate() {
        log.info("Current Thread : {}", Thread.currentThread().getName());
        log.info("Fixed Rate Task : The time is now {}", dateFormat.format(new Date()));
    }
}
```
还需要再SpringBoot中启动类中加上`@EnableScheduling`注解。

# Spring中的事务
事务是逻辑上的一组操作，要么都执行，要么都不执行
## 事务的特性
-   **原子性（Atomicity）：** 一个事务（transaction）中的所有操作，或者全部完成，或者全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会被回滚（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。即，事务不可分割、不可约简。
-   **一致性（Consistency）：** 在事务开始之前和事务结束以后，数据库的完整性没有被破坏。这表示写入的资料必须完全符合所有的预设约束、触发器、级联回滚等。
-   **隔离性（Isolation）：** 数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。事务隔离分为不同级别，包括未提交读（Read uncommitted）、提交读（read committed）、可重复读（repeatable read）和串行化（Serializable）。
-   **持久性（Durability）:** 事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。
## Spring对事务的支持
基于数据库的引擎，只有`Innodb`引擎支持事务
MySQL如何保证原子性：在异常发生时，对已经执行的操作进行回滚，恢复机制是通过回滚日志（undo log）实现的，所有事务进行的修改都会先记录到这个回滚日志中，然后再执行相关的操作，遇到异常可以直接利用回滚日志中的信息将数据回滚到修改之前的样子即可
### Spring支持两种方式的事务管理
- 通过`TransactionTemplate`或者`TransactionManager`手动管理事务，实际应用中很少使用。
- 声明式事务管理：使用`@Transactional`注解进行事务管理
### `@Transactional`注解的使用
作用范围：
- 方法：该注解只能应用到public方法上
- 类：表明该注解对该类中鄋的public方法都生效
- 接口：不推荐使用
常用配置参数：
| 属性名 | 说明 |
| --- | --- |
| propagation | 事务的传播行为，默认值为 REQUIRED，可选的值在上面介绍过 |
| isolation | 事务的隔离级别，默认值采用 DEFAULT，可选的值在上面介绍过 |
| timeout | 事务的超时时间，默认值为-1（不会超时）。如果超过该时间限制但事务还没有完成，则自动回滚事务。 |
| readOnly | 指定事务是否为只读事务，默认值为 false。 |
| rollbackFor | 用于指定能够触发事务回滚的异常类型，并且可以指定多个异常类型。 |

# Spring中的设计模式
## 控制反转和依赖注入
IoC（inversion of control）是一种解耦的设计思想，主要目的是借助于spring中的ioc容器实现具有依赖关系的对象之间的解耦，从而降低代码之间的耦合度。当我们需要创建一个对象的时候，只需要配置好配置文件/注解即可，完全不用考虑对象是如何被创建出来的。
DI（dependecy inject）是实现控制反转的一种设计模式，依赖注入就是将实例变量传入到一个对象中去。
## 工厂设计模式
通过BeanFactory和ApplicationContext创建bean对象
-   `BeanFactory` ：延迟注入(使用到某个 bean 的时候才会注入),相比于`ApplicationContext` 来说会占用更少的内存，程序启动速度更快。
-   `ApplicationContext` ：容器启动的时候，不管你用没用到，一次性创建所有 bean 。`BeanFactory` 仅提供了最基本的依赖注入支持， `ApplicationContext` 扩展了 `BeanFactory` ,除了有`BeanFactory`的功能还有额外更多功能，所以一般开发人员使用 `ApplicationContext`会更多。
## 单例设计模式
减少创建对象所花费的时间，对系统内存的使用频率降低
通过`@Scope(value=“singleton”)`实现
## 代理设计模式
AOP面向切面编程，对公共事务封装，减少系统的重复代码，降低耦合，有利于未来的可拓展性和可维护性。
## 模版方法
一种行为设计模式，定义一个操作中的算法的骨架，将一些步骤延迟到子类中，使得子类可以不改变一个算法的结构即可重定义该算法的默写特定不走的实现方式。Spring 中 `jdbcTemplate`、`hibernateTemplate` 等以 Template 结尾的对数据库操作的类，它们就使用到了模板模式。
## 观察者模式
是一种对象行为型模式。Spring事件驱动模型
## 适配器模式
将一个接口转换成客户希望的另一个接口

