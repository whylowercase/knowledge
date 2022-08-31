# Spring基础

## 什么是Spring框架？

Spring是一款开源的轻量级Java开发框架，一般来说Spring框架指的是Spring Framework，是很多模块的集合，比如Spring支持的IOC和AOP、对数据库的访问等。。。Spring是Java生态的一个杀手级的应用框架，核心就是IOC和AOP。

## Spring包含的模块有哪些？

![Spring5.x主要模块](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/github/javaguide/system-design/framework/spring/20200831175708.png)

- Core Container

核心模块，主要提供IOC的依赖注入功能，其他所有的模块都需要依赖此模块

- AOP

**spring-aspects** ：该模块为与 AspectJ 的集成提供支持。

**spring-aop** ：提供了面向切面的编程实现。

**spring-instrument** ：提供了为 JVM 添加代理（agent）的功能。 具体来讲，它为 Tomcat 提供了一个织入代理，能够为 Tomcat 传递类文 件，就像这些文件是被类加载器加载的一样。没有理解也没关系，这个模块的使用场景非常有限。

- Data Access

jdbc等

- Spring Web

对Web功能的实现提供基础支持，以及Webmvc的实现

- Messaging

主要是为Spring框架集成一些基础的报文传送应用

- Spring Test

JUnit等



## Spring、Spring MVC、Spring Boot

1. Spring指的是Spring Framework，包含多个模块
2. Spring MVC是Spring中的一个模块，主要是赋予Spring快速构建MVC框架的Web程序的能力，MVC指的是模型(Model)、试图(View)、控制器(Controller)
3. Spring Boot是快速开发的脚手架，简化了很多配置，做到真正的开箱即用



## Spring IoC

IoC控制反转是一种设计思想，并不是具体的技术体现。将原本在程序员收中的创建对象的控制权，交给Spring框架来管理。

将对象之间的相互依赖关系交给IoC容器来管理，并用IoC容器来完成对象的注入，很大程度上简化了应用的开发，把应用从复杂的依赖关系中解放出来。当我们需要创建一个对象的时候，只需要配置文件/注解即可，不用真正去考虑对象是怎么被创建出来的。

可以把IoC容器理解为一个Map，里面存放的是各种对象，之前我们都用xml文件来配置Bean，当到了SpringBoot时代，注解配置就慢慢开始流行了。



## Spring Bean

Bean指的就是被IoC容器管理的对象。

![img](https://img-blog.csdnimg.cn/062b422bd7ac4d53afd28fb74b2bc94d.png)

1. `@Component`:通用注解，可以将任何类标注为Spring组件，当你不知道这个类应该属于哪一层时，可以使用。
2. `@Repository`:对应数据持久层（Dao）
3. `@Service`:对应服务层
4. `@Controller`：对应Spring MVC控制层

### @Component 和 @Bean 的区别是什么？

- `@Component` 注解作用于类，而`@Bean`注解作用于方法。
- `@Component`通常是通过类路径扫描来自动侦测以及自动装配到 Spring 容器中（我们可以使用 `@ComponentScan` 注解定义要扫描的路径从中找出标识了需要装配的类自动装配到 Spring 的 bean 容器中）。`@Bean` 注解通常是我们在标有该注解的方法中定义产生这个 bean,`@Bean`告诉了 Spring 这是某个类的实例，当我需要用它的时候还给我。
- `@Bean` 注解比 `@Component` 注解的自定义性更强，而且很多地方我们只能通过 `@Bean` 注解来注册 bean。比如当我们引用第三方库中的类需要装配到 `Spring`容器时，则只能通过 `@Bean`来实现。

### Bean的注入

- `@Autowired` 是 Spring 提供的注解，`@Resource` 是 JDK 提供的注解。
- `Autowired` 默认的注入方式为`byType`（根据类型进行匹配），`@Resource`默认注入方式为 `byName`（根据名称进行匹配）。
- 当一个接口存在多个实现类的情况下，`@Autowired` 和`@Resource`都需要通过名称才能正确匹配到对应的 Bean。`Autowired` 可以通过 `@Qualifier` 注解来显示指定名称，`@Resource`可以通过 `name` 属性来显示指定名称。

### Bean的作用域

- **singleton** : IoC 容器中只有唯一的 bean 实例。Spring 中的 bean 默认都是单例的，是对单例设计模式的应用。
- **prototype** : 每次获取都会创建一个新的 bean 实例。也就是说，连续 `getBean()` 两次，得到的是不同的 Bean 实例。
- **request** （仅 Web 应用可用）: 每一次 HTTP 请求都会产生一个新的 bean（请求 bean），该 bean 仅在当前 HTTP request 内有效。
- **session** （仅 Web 应用可用） : 每一次来自新 session 的 HTTP 请求都会产生一个新的 bean（会话 bean），该 bean 仅在当前 HTTP session 内有效。
- **application/global-session** （仅 Web 应用可用）： 每个 Web 应用在启动时创建一个 Bean（应用 Bean），，该 bean 仅在当前应用启动时间内有效。
- **websocket** （仅 Web 应用可用）：每一次 WebSocket 会话产生一个新的 bean。

### Bean的生命周期

TODO

## Spring AOP

面向切面编程，将与业务无关，确为业务模块提供方法的模块（比如事务处理、日资管理等）封装起来，减少了系统的重复代码，降低了模块之间的耦合度，提升了可拓展性和可维护性。

Spring AOP 就是基于动态代理的，如果要代理的对象，实现了某个接口，那么 Spring AOP 会使用 **JDK Proxy**，去创建代理对象，而对于没有实现接口的对象，就无法使用 JDK Proxy 去进行代理了，这时候 Spring AOP 会使用 **Cglib** 生成一个被代理对象的子类来作为代理，如下图所示：

![SpringAOPProcess](https://img-blog.csdnimg.cn/img_convert/230ae587a322d6e4d09510161987d346.jpeg)

### Spring AOP和AspectJ AOP有什么区别

Spring AOP属于运行时增强，而AspectJ是编译时增强。前者基于代理，后者基于字节码操作，虽然Spring AOP已经集成了AspectJ，但是AspectJ的功能更加强大，而Spring AOP更加简单便捷。

TODO



## Spring MVC

![img](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/java-guide-blog/image-20210809181452421.png)

### Spring MVC的核心组件

- **`DispatcherServlet`** ：**核心的中央处理器**，负责接收请求、分发，并给予客户端响应。
- **`HandlerMapping`** ：**处理器映射器**，根据 uri 去匹配查找能处理的 `Handler` ，并会将请求涉及到的拦截器和 `Handler` 一起封装。
- **`HandlerAdapter`** ：**处理器适配器**，根据 `HandlerMapping` 找到的 `Handler` ，适配执行对应的 `Handler`；
- **`Handler`** ：**请求处理器**，处理实际请求的处理器。
- **`ViewResolver`** ：**视图解析器**，根据 `Handler` 返回的逻辑视图 / 视图，解析并渲染真正的视图，并传递给 `DispatcherServlet` 响应客户端

### Spring MVC的工作流程

![img](https://img-blog.csdnimg.cn/img_convert/de6d2b213f112297298f3e223bf08f28.png)

1. 客户端（浏览器）发送请求， `DispatcherServlet`拦截请求。
2. `DispatcherServlet` 根据请求信息调用 `HandlerMapping` 。`HandlerMapping` 根据 uri 去匹配查找能处理的 `Handler`（也就是我们平常说的 `Controller` 控制器） ，并会将请求涉及到的拦截器和 `Handler` 一起封装。
3. `DispatcherServlet` 调用 `HandlerAdapter`适配执行 `Handler` 。
4. `Handler` 完成对用户请求的处理后，会返回一个 `ModelAndView` 对象给`DispatcherServlet`，`ModelAndView` 顾名思义，包含了数据模型以及相应的视图的信息。`Model` 是返回的数据对象，`View` 是个逻辑上的 `View`。
5. `ViewResolver` 会根据逻辑 `View` 查找实际的 `View`。
6. `DispaterServlet` 把返回的 `Model` 传给 `View`（视图渲染）。
7. 把 `View` 返回给请求者（浏览器）

