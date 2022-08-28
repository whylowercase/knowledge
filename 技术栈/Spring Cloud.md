# Spring Cloud

## 微服务基础

随着传统单体架构应用随着项目规模的扩大，暴露出却来越多的问题，此时“微服务”架构的就应运而生了，

* 微服务把一个庞大的单体应用拆分为一个个的小型服务，比如我们原来的图书管理项目中，有登录、注册、添加、删除、搜索等功能，那么我们可以将这些功能单独做成一个个小型的SpringBoot项目，独立运行。
* 每个小型的微服务，都可以独立部署和升级，这样，就算整个系统崩溃，那么也只会影响一个服务的运行。
* 微服务之间使用HTTP进行数据交互，不再是单体应用内部交互了，虽然这样会显得更麻烦，但是带来的好处也是很直接的，甚至能突破语言限制，使用不同的编程语言进行微服务开发，只需要使用HTTP进行数据交互即可。
* 我们可以同时购买多台主机来分别部署这些微服务，这样，单机的压力就被分散到多台机器，并且每台机器的配置不一定需要太高，这样就能节省大量的成本，同时安全性也得到很大的保证。
* 甚至同一个微服务可以同时存在多个，这样当其中一个服务器出现问题时，其他服务器也在运行同样的微服务，这样就可以保证一个微服务的高可用。

## 什么是Spring Cloud

* 实现微服务不是仅仅需要简单的把项目进行拆分，还需要考虑对各个微服务进行管理、监控等，这样才能及时地寻找问题，因此微服务往往需要的是一整套解决方案，包括服务注册和发现、容灾处理、负载均衡、配置管理等。
* 微服务的架构不像单体架构方便维护，由于部署在多个服务器，所以在管理难度上是高于传统应用的
* 在分布式的环境下，单体应用的某些功能可能会变得麻烦

所以，为了更好地解决这些问题，SpringCloud正式登场。

SpringCloud是Spring提供的一套分布式解决方案，集合了一些大型互联网公司的开源产品，包括诸多组件，共同组成SpringCloud框架。并且，它利用Spring Boot的开发便利性巧妙地简化了分布式系统基础设施的开发，如服务发现注册、配置中心、消息总线、负载均衡、熔断机制、数据监控等，都可以用Spring Boot的开发风格做到一键启动和部署。

SpringCloud对分布式架构下的各个场景都有对应的组件来处理，比如Netfilx：

- Eureka  -  实现服务治理（服务注册与发现），我们可以对所有的微服务进行集中管理，包括他们的运行状态、信息等。
- Ribbon  -  为服务之间相互调用提供负载均衡算法（现在被SpringCloudLoadBalancer取代）。
- Hystrix  -  断路器，保护系统，控制故障范围。暂时可以跟家里电闸的保险丝类比，当触电危险发生时能够防止进一步的发展。
- Zuul   -     api网关，路由，负载均衡等多种作用，就像我们的路由器，可能有很多个设备都连接了路由器，但是数据包要转发给谁则是由路由器在进行（已经被SpringCloudGateway取代）
- Config  -  配置管理，可以实现配置文件集中管理

## Eureka注册中心

### 1.服务间调用

比如图书借阅服务中需要获取借阅信息中的详细信息，需要去访问除了借阅表之外的用户表和图书表，形成一个生产者和消费者的关系

需要用到`RestTemplate`来进行：

```java
@Service
public class BorrowServiceImpl implements BorrowService{

    @Resource
    BorrowMapper mapper;

    @Override
    public UserBorrowDetail getUserBorrowDetailByUid(int uid) {
        List<Borrow> borrow = mapper.getBorrowsByUid(uid);
        //RestTemplate支持多种方式的远程调用
        RestTemplate template = new RestTemplate();
        //这里通过调用getForObject来请求其他服务，并将结果自动进行封装
        //获取User信息
        User user = template.getForObject("http://localhost:8082/user/"+uid, User.class);
        //获取每一本书的详细信息
        List<Book> bookList = borrow
                .stream()
                .map(b -> template.getForObject("http://localhost:8080/book/"+b.getBid(), Book.class))
                .collect(Collectors.toList());
        return new UserBorrowDetail(user, bookList);
    }
}
```

### 2.服务注册与发现

我们需要一个集中管理微服务的平台，所以Eureka就应运而生了，Eureka能够自动注册并发现微服务，然后对服务的状态、信息进行集中管理，这样当我们需要获取其他服务的信息时，只需要向Eureka进行查询就可以了。

### 3.注册中心高可用

如果我们搭建的Eureka server崩溃了，那不是所有需要用到服务发现的微服务就全部<s>木大</s>了,所以，搭建Eureka集群，存在多个Eureka服务器，是有必要的。



## LoadBalancer负载均衡

在2020之前使用Ribbon的，但之后SpringCloud移除了Ribbon,采用了自己编写的loadBalancer替代

### 负载均衡

`@LoadBalanced`服务端会在发起请求时执行这些拦截器，在实行负载均衡的时候，回向eureka发起请求，选择一个可用的对应服务，然后会返回此服务的主机地址

### 自定义负载均衡策略

LoadBalancer默认提供了两种负载均衡策略：

* RandomLoadBalancer  -  随机分配策略
* **(默认)** RoundRobinLoadBalancer  -  轮询分配策略

### OpenFeign实现负载均衡

openfeign与resttemplate一样，也是http客户端请求工具，但使用更加便捷

在引入依赖后，需要在主启动类中添加`@EnableFeignClients`的注解

```java
@SpringBootApplication
@EnableFeignClients
public class BorrowApplication {
    public static void main(String[] args) {
        SpringApplication.run(BorrowApplication.class, args);
    }
}
```

此时有调用其他服务的接口的需求，创建对应服务的接口类

```java
@FeignClient("userservice")
public interface UserClient {

  	//路径保证和其他微服务提供的一致即可
    @RequestMapping("/user/{uid}")
    User getUserById(@PathVariable("uid") int uid);  //参数和返回值也保持一致
}
```

之后就可以直接注入使用了

```java
@Resource
UserClient userClient;

@Override
public UserBorrowDetail getUserBorrowDetailByUid(int uid) {
    List<Borrow> borrow = mapper.getBorrowsByUid(uid);
    
    User user = userClient.getUserById(uid);
    //这里不用再写IP，直接写服务名称bookservice
    List<Book> bookList = borrow
            .stream()
            .map(b -> template.getForObject("http://bookservice/book/"+b.getBid(), Book.class))
            .collect(Collectors.toList());
    return new UserBorrowDetail(user, bookList);
}
```

## Hystrix服务熔断

微服务之间相互调用，如果一项出现了故障，可能会导致所有服务全线崩溃。所以SC(Spring Cloud下同)提供了Hystrix熔断器组件

### 服务降级

服务降级不会直接返回错误，而是可以提供一个补救措施，正常响应给请求者，需要主启动类`@EnableHystrix`开启，在Controller层中提供一个备选方案，当服务出现异常时，返回备选方案

```java
@RestController
public class BorrowController {

    @Resource
    BorrowService service;

    @HystrixCommand(fallbackMethod = "onError")    //使用@HystrixCommand来指定备选方案
    @RequestMapping("/borrow/{uid}")
    UserBorrowDetail findUserBorrows(@PathVariable("uid") int uid){
        return service.getUserBorrowDetailByUid(uid);
    }
		
  	//备选方案，这里直接返回空列表了
  	//注意参数和返回值要和上面的一致
    UserBorrowDetail onError(int uid){
        return new UserBorrowDetail(null, Collections.emptyList());
    }
}
```

### 服务熔断

是对应雪崩效应的一种微服务链路保护机制，当检测出链路的某个微服务长时间不可用时，会进行服务降级，进而熔断，快速返回错误的信息

### OpenFeign实现降级

创建一个实现类，对原有的接口方法进行替代方案实现：

```java
@Component   //注意，需要将其注册为Bean，Feign才能自动注入
public class UserFallbackClient implements UserClient{
    @Override
    public User getUserById(int uid) {   //这里我们自行对其进行实现，并返回我们的替代方案
        User user = new User();
        user.setName("我是替代方案");
        return user;
    }
}
```

实现完成后，我们只需要在原有的接口中指定失败替代实现即可：

```java
//fallback参数指定为我们刚刚编写的实现类
@FeignClient(value = "userservice", fallback = UserFallbackClient.class)
public interface UserClient {

    @RequestMapping("/user/{uid}")
    User getUserById(@PathVariable("uid") int uid);
}
```

现在去掉`BorrowController`的`@HystrixCommand`注解和备选方法：

```java
@RestController
public class BorrowController {

    @Resource
    BorrowService service;

    @RequestMapping("/borrow/{uid}")
    UserBorrowDetail findUserBorrows(@PathVariable("uid") int uid){
        return service.getUserBorrowDetailByUid(uid);
    }
}
```

最后我们在配置文件中开启熔断支持：

```yaml
feign:
  circuitbreaker:
    enabled: true
```

### 监控页面部署



## Gateway 路由网关

使用路由机制，添加一层防护，让所有的请求全员通过路由来转发到各个微服务中，并且转发给多个相同的微服务实例也可以实现负载均衡

```yaml
spring:
  cloud:
    gateway:
    	# 配置路由，注意这里是个列表，每一项都包含了很多信息
      routes:
        - id: borrow-service   # 路由名称
          uri: lb://borrowservice  # 路由的地址，lb表示使用负载均衡到微服务，也可以使用http正常转发
          predicates: # 路由规则，断言什么请求会被路由
            - Path=/borrow/**  # 只要是访问的这个路径，一律都被路由到上面指定的服务
```

### 路由过滤器

比如我们现在希望在请求到达时，在请求头中添加一些信息再转发给我们的服务，那么这个时候就可以使用路由过滤器来完成，我们只需要对配置文件进行修改：

```yaml
spring:
  application:
    name: gateway
  cloud:
    gateway:
      routes:
      - id: borrow-service
        uri: lb://borrowservice
        predicates:
        - Path=/borrow/**
      # 继续添加新的路由配置，这里就以书籍管理服务为例
      # 注意-要对齐routes:
      - id: book-service
        uri: lb://bookservice
        predicates:
        - Path=/book/**
        filters:   # 添加过滤器
        - AddRequestHeader=Test, HelloWorld!
        # AddRequestHeader 就是添加请求头信息，其他工厂请查阅官网
        
```

#### Pre类型

在请求被转发到微服务之前，对请求进行拦截和修改，如参数校验、权限校验、流量监控、日志输出以及协议转换等

#### Post类型

在微服务处理完请求后，返回响应给网关。网关再次进行处理

## Config配置中心

> Spring Cloud Config 为分布式系统中的外部配置提供服务器端和客户端支持。使用 Config Server，您可以集中管理所有环境中应用程序的外部配置。

TODO



## 微服务CAP原则

* 一致性（C）：在分布式系统中的所有数据备份，在同一时刻都是同样的值（所有的节点无论何时访问都能拿到最新的值）
* 可用性（A）：系统中非故障节点收到的每个请求都必须得到响应（比如我们之前使用的服务降级和熔断，其实就是一种维持可用性的措施，虽然服务返回的是没有什么意义的数据，但是不至于用户的请求会被服务器忽略）
* 分区容错性（P）：一个分布式系统里面，节点之间组成的网络本来应该是连通的，然而可能因为一些故障（比如网络丢包等，这是很难避免的），使得有些节点之间不连通了，整个网络就分成了几块区域，数据就散布在了这些不连通的区域中（这样就可能出现某些被分区节点存放的数据访问失败，我们需要来容忍这些不可靠的情况）

### AC 可用性+一致性

要同时保证可用性和一致性，代表着某个节点数据更新之后，需要立即将结果通知给其他节点，并且要尽可能的快，这样才能及时响应保证可用性，这就对网络的稳定性要求非常高，但是实际情况下，网络很容易出现丢包等情况，并不是一个可靠的传输，如果需要避免这种问题，就只能将节点全部放在一起，但是这显然违背了分布式系统的概念，所以对于我们的分布式系统来说，很难接受。

### CP 一致性+分区容错性

为了保证一致性，那么就得将某个节点的最新数据发送给其他节点，并且需要等到所有节点都得到数据才能进行响应，同时有了分区容错性，那么代表我们可以容忍网络的不可靠问题，所以就算网络出现卡顿，那么也必须等待所有节点完成数据同步，才能进行响应，因此就会导致服务在一段时间内完全失效，所以可用性是无法得到保证的。

### AP 可用性+分区容错性

既然CP可能会导致一段时间内服务得不到任何响应，那么要保证可用性，就只能放弃节点之间数据的高度统一，也就是说可以在数据不统一的情况下，进行响应，因此就无法保证一致性了。虽然这样会导致拿不到最新的数据，但是只要数据同步操作在后台继续运行，一定能够在某一时刻完成所有节点数据的同步，那么就能实现**最终一致性**，所以AP实际上是最能接受的一种方案。

# Spring Cloud Alibaba

基于Netflix的解决方案很多框架都已经停止维护了。如

* **注册中心：**Eureka（属于*Netflix*，2.x版本不再开源，1.x版本仍在更新）
* **服务调用：**Ribbon（属于*Netflix*，停止更新，已经彻底被移除）、SpringCloud Loadbalancer（属于*SpringCloud*官方，目前的默认方案）
* **服务降级：**Hystrix（属于*Netflix*，停止更新，已经彻底被移除）
* **路由网关：**Zuul（属于*Netflix*，停止更新，已经彻底被移除）、Gateway（属于*SpringCloud*官方，推荐方案）
* **配置中心：**Config（属于*SpringCloud*官方）

阿里巴巴给出一套全新的解决方案：`Spring Cloud Alibaba`

目前 Spring Cloud Alibaba 提供了如下功能:

1. **服务限流降级**：支持 WebServlet、WebFlux, OpenFeign、RestTemplate、Dubbo 限流降级功能的接入，可以在运行时通过控制台实时修改限流降级规则，还支持查看限流降级 Metrics 监控。
2. **服务注册与发现**：适配 Spring Cloud 服务注册与发现标准，默认集成了 Ribbon 的支持。
3. **分布式配置管理**：支持分布式系统中的外部化配置，配置更改时自动刷新。
4. **Rpc服务**：扩展 Spring Cloud 客户端 RestTemplate 和 OpenFeign，支持调用 Dubbo RPC 服务
5. **消息驱动能力**：基于 Spring Cloud Stream 为微服务应用构建消息驱动能力。
6. **分布式事务**：使用 @GlobalTransactional 注解， 高效并且对业务零侵入地解决分布式事务问题。
7. **阿里云对象存储**：阿里云提供的海量、安全、低成本、高可靠的云存储服务。支持在任何应用、任何时间、任何地点存储和访问任意类型的数据。
8. **分布式任务调度**：提供秒级、精准、高可靠、高可用的定时（基于 Cron 表达式）任务调度服务。同时提供分布式的任务执行模型，如网格任务。网格任务支持海量子任务均匀分配到所有 Worker（schedulerx-client）上执行。
9. **阿里云短信服务**：覆盖全球的短信服务，友好、高效、智能的互联化通讯能力，帮助企业迅速搭建客户触达通道。



## Nacos更加全能的注册中心

### 服务注册与发现

```yaml
spring:  
  cloud:
    nacos:
      discovery:
        # 配置Nacos注册中心地址
        server-addr: localhost:8848
```

通过启动Nacos服务端和将module注册到Nacos中，我们可以发现在nacos的服务列表中已经有我们的项目了，之后可以使用OpenFeign,进行服务发现的远程调用以及负载均衡

### 集群分区

在一个分布式应用中，相同服务的实例可能会在不同的机器、位置上启动，比如我们的用户管理服务，可能在成都有1台服务器部署、重庆有一台服务器部署，而这时，我们在成都的服务器上启动了借阅服务，那么如果我们的借阅服务现在要调用用户服务，就应该优先选择同一个区域的用户服务进行调用，这样会使得响应速度更快。

![image-20220326150024118](https://tva1.sinaimg.cn/large/e6c9d24ely1h0namonso5j21em0bcgnr.jpg)

```yaml
spring:
  application:
    name: borrowservice
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
        # 修改为重庆地区的集群
        cluster-name: Chongqing
        #设置权重
        weight: 0.5
    # 将loadbalancer的nacos支持开启，集成Nacos负载均衡
    loadbalancer:
      nacos:
        enabled: true
```

### 配置中心

在Nacos中配置文件，并发布

![image-20220326162108899](https://tva1.sinaimg.cn/large/e6c9d24ely1h0ncyoyq7ij21sw0u0n0o.jpg)

项目中导入依赖，在resources文件夹下添加bootstrap.yml文件：

```yaml
spring:
  application:
  	# 服务名称和配置文件保持一致
    name: borrowservice
  profiles:
  	# 环境也是和配置文件保持一致
    active: dev
  cloud:
    nacos:
      config:
      	# 配置文件后缀名
        file-extension: yml
        # 配置中心服务器地址，也就是Nacos地址
        server-addr: localhost:8848
```

### 命名空间

在nacos中添加命名空间

### 实现高可用

Nacos集群的建立



## Sentinel流量防卫兵

相比于Hystrix更加强大，从流量为切入点、熔断降级、系统负载保护等多个维度来保护服务的稳定性

启用Sentinel监控页面，将服务链接到Sentinel控制台

```yaml
spring:
  application:
    name: userservice
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
    sentinel:
      transport:
      	# 添加监控页面地址即可
        dashboard: localhost:8858
```

### 流量控制

当一段时间内的流量到达一定阈值的时候，新的请求将不再进行处理，这样不仅可以合理地应对高并发，同时也能在一定程度上保护服务器不受到外界的恶意攻击。

* 方案一：**快速拒绝**，既然不再接受新的请求，那么我们可以直接返回一个拒绝信息，告诉用户访问频率过高。
* 方案二：**预热**，依然基于方案一，但是由于某些情况下高并发请求是在某一时刻突然到来，我们可以缓慢地将阈值提高到指定阈值，形成一个缓冲保护。
* 方案三：**排队等待**，不接受新的请求，但是也不直接拒绝，而是进队列先等一下，如果规定时间内能够执行，那么就执行，要是超时就算了。

### 限流和异常处理

当出现异常时，直接执行我们的替代方法并返回

```java
@Override
@SentinelResource(value = "getBorrow", blockHandler = "blocked")   //指定blockHandler，也就是被限流之后的替代解决方案，这样就不会使用默认的抛出异常的形式了
public UserBorrowDetail getUserBorrowDetailByUid(int uid) {
    List<Borrow> borrow = mapper.getBorrowsByUid(uid);
    User user = userClient.getUserById(uid);
    List<Book> bookList = borrow
            .stream()
            .map(b -> bookClient.getBookById(b.getBid()))
            .collect(Collectors.toList());
    return new UserBorrowDetail(user, bookList);
}

//替代方案，注意参数和返回值需要保持一致，并且参数最后还需要额外添加一个BlockException
public UserBorrowDetail blocked(int uid, BlockException e) {
    return new UserBorrowDetail(null, Collections.emptyList());
}
```

### 热点参数限流

我们还可以对某一热点数据进行精准限流，比如在某一时刻，不同参数被携带访问的频率是不一样的：

* http://localhost:8301/test?a=10  访问100次
* http://localhost:8301/test?b=10  访问0次
* http://localhost:8301/test?c=10  访问3次

由于携带参数`a`的请求比较多，我们就可以只对携带参数`a`的请求进行限流。

这里我们创建一个新的测试请求映射：

```java
@RequestMapping("/test")
@SentinelResource("test")   //注意这里需要添加@SentinelResource才可以，用户资源名称就使用这里定义的资源名称
String findUserBorrows2(@RequestParam(value = "a", required = false) int a,
                        @RequestParam(value = "b", required = false) int b,
                        @RequestParam(value = "c",required = false) int c) {
    return "请求成功！a = "+a+", b = "+b+", c = "+c;
}
```

启动之后，我们在Sentinel里面进行热点配置：

![image-20220328145654180](https://tva1.sinaimg.cn/large/e6c9d24ely1h0plrnnjlqj22fh0u0aec.jpg)

然后开始访问我们的测试接口，可以看到在携带参数a时，当访问频率超过设定值，就会直接被限流，这里是直接在后台抛出异常

### 服务的熔断和降级

当下游服务因为某种原因变得不可用或是响应过慢的时候，上游服务为了保证自己整体服务的可用性，不再继续调用目标服务而是快速返回或是执行自己的替代方案，这就是服务降级

熔断策略：

1. 慢调用比例：如果出现那种半天都处理不完的调用，有可能就是服务出现故障，导致卡顿，这个选项是按照最大响应时间（RT）进行判定，如果一次请求的处理时间超过了指定的RT，那么就被判定为`慢调用`，在一个统计时长内，如果请求数目大于最小请求数目，并且被判定为`慢调用`的请求比例已经超过阈值，将触发熔断。经过熔断时长之后，将会进入到半开状态进行试探
2. 异常比例：这个与慢调用比例类似，不过这里判断的是出现异常的次数
3. **异常数：**这个和上面的唯一区别就是，只要达到指定的异常数量，就熔断



## Seata与分布式事务

TODO

