---
title: Spring笔记
date: 2021-07-08 16:07:09
tags: 学习
---

Idea调试步骤：

1、F8 单步调试，不进入函数内部

2、F7 单步调试，进入函数内部

3、Shift+F7 选择要进入的函数

4、Shift+F8 跳出函数

5、Alt+F9 运行到断点

6、Alt+F8 执行表达式查看结果

7、F9 继续执行，进入下一个断点或执行完程序

8、Ctrl+F8 设置/取消当前行断点

9、Ctrl+Shift+F8 查看断点

# SpringCloud笔记

springcloud服务启动需要建一个含有psvm方法类然后在其类名上加一个@SpringBootApplication注解，psvm方法中调用SpringApplication.run（类名.class,args）

```java
@SpringBootApplication
public class ApiGatewayApp {

    public static void main(String[] args) {
        SpringApplication.run(ApiGatewayApp.class, args);
    }
}
```

如何创建返回给前端的实体类？

![截屏2020-08-14 上午10.11.21](/typora-user-images/截屏2020-08-14 上午10.11.21.png)



![截屏2020-08-14 上午10.53.32](/typora-user-images/截屏2020-08-14 上午10.53.32.png)

namespace必须要=被@Mapper注释的层

parameterType一定要是我们传入的类型，resultMap(结果集映射)为传出类型

resultMap 的id为自己规定，type为实体类

resultMap标签里的标签，result标签的colum属性为数据库中存储的字段，property属性为实体类中的字段，也可以如图为id标签

useGeneratedKeys="true" keyProperty="id"在插入语句中自动生成主键并返回

![截屏2020-08-14 上午11.12.56](/typora-user-images/截屏2020-08-14 上午11.12.56.png)

Controller层要给前端返回东西，是我们编写的返回值类（返回状态码信息和数据）



服务注册与发现，服务注册有两个组件一个是服务中心，另一个是服务提供者

7001为服务中心，需要配置一个集群7002位两个服务中心

![image-20200817105052628](/typora-user-images/image-20200817105052628.png)

server下的代码意思是关闭自我保护机制，还可以设置时间。



注册中心变更为zookeeper服务节点是临时的。重新启动后流水号变更

![image-20200818101841019](/typora-user-images/image-20200818101841019.png)

上图为入住到zookeeper服务器端



Nginx和Ribbon负载均衡的不同是Nginx服务端负载均衡，而Ribbon是本地客户端负载均衡

集中式负载均衡：

即在服务的消费方和提供方之间使用独立的LB设施(可以是硬件,如F5,也可以是软件，如nginx), 由该设施负责把访问请求通过某种策略转发至服务的提供方;

进程式负载均衡：

将LB逻辑集成到消费方，消费方从服务注册中心获知有哪些地址可用,然后自己再从这些地址中选择出一个合适的服务器。

Ribbon就是负载均衡+RestTemplate

在引入eureka-client的时候就已经引入了netflix-ribbon所以不用单独引包

### 核心组件 IRule

###### IRule默认自带的负载规则

1. RoundRobinRule   轮询
2. RandomRule   随机
3. RetryRule    先按照RoundRobinRule的 策略获取服务，如果获取服务失败则在指定时间里进行重试，获取可用服务
4. WeightedResponseTimeRule 对RoundRobinRule的扩展，响应速度越快，实例选择权重越大 ，越容易被选择
5. BestAvailableRule    会先过滤掉由于多次访问故障而处于断路器 跳闸状态的服务，然后选择一个并发一个最小的服务
6. BestAvaibilityFilteringRule  先过滤掉故障实例，再选择并发量较小的实例
7. ZoneAvoidanceRule    默认规则，符合server所在区域的性能和server的可用性选择服务器

Ribbon下如果需要自定义配置负载规则，不能放在@ComponentScan所扫描的当前包下，以及子包下

所以要新建一个包之后，新建MySelfRule规则类

```java
public class MySelfRule {
    @Bean
    public IRule myRule(){
        return new RandomRule();//定义为随机
    }
}
```



还要在主启动类上加一个@RibbonClient

```java
// 选择要接收的服务和配置类
@RibbonClient(name = "CLOUD-PAYMENT-SERVICE",configuration = MySelfRule.class)
```



负载均衡算法：rest接口第几次请求数 % 服务器集群总数量=实际调用服务器位置下标，每次服务重启动后rest接口计数从1开始

```java
List<ServiceInstance>instances=discoveryClient.getlnstances("CLOUD-PATMENT-SERVICE")
```

* 如果有两台机器是集群
* 按照轮询算法原理：
* 当总请求数为1时：1%2=1对应下标位置为1，则服务地址为list[1] instances=...
* 当总请求数为2时：2%2=0对应下标位置为0，则服务地址为list[0] instances=......



## 服务降级

### 断路器Hystrix

复杂的分布式体系结构中的应用程序有数十个依赖关系，每个依赖关系在某些时候将不可避免的失败

### 是什么

Hystrix 是处理分布式系统的延迟和容错的开源库，保证一个依赖出现问题时不会导致整体服务失败，避免级联故障，以提高分布式系统弹性。
断路器本身是一种开关装置，当某个服务单元发生故障后，通过断路器的故障监控，向调用方返回一个符合预期的可处理的备选响应，而不是长时间的等待或抛出调用方法无法处理的异常 。

#### 服务降级

假设对方系统不可用了，你需要给我一个兜底的方式，不让客户端等待并立刻返回一个友好提示，fallback

哪些情况会导致服务降级

1. 程序运行异常
2. 超时
3. 服务熔断触发服务降级
4. 线程池/信号量打满

如何解决给服务设置一个峰值，峰值内可以正常运行，超过了，需要有兜底的方法处理，作服务降级fallback

![image-20200820170852225](/typora-user-images/image-20200820170852225.png)

```java
@Service
public class PaymentService {
    public String paymentInfo_OK(Integer id){
        return "线程池："+Thread.currentThread().getName()+"ok"+id;
    }

    // 设置超过 3 秒采用服务降级
    @HystrixCommand(fallbackMethod = "paymentInfo_TimeoutHandler",commandProperties = {
            @HystrixProperty(name="execution.isolation.thread.timeoutInMilliseconds",value = "3000")
    })
    public String paymentInfo_Timeout(Integer id){
        try {
            TimeUnit.SECONDS.sleep(5);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "线程池："+Thread.currentThread().getName()+"Timeout"+id;
    }
    // 异常后调用的方法
    public String paymentInfo_TimeoutHandler(Integer id){
        return "服务超时，调用服务降级成功";
    }
}
```

运行异常:

```java
    @HystrixCommand(fallbackMethod = "paymentInfo_TimeoutHandler",commandProperties = {
            @HystrixProperty(name="execution.isolation.thread.timeoutInMilliseconds",value = "3000")
    })
    public String paymentInfo_Timeout(Integer id){
        int a = 10/0;
//        try {
//            TimeUnit.SECONDS.sleep(5);
//        } catch (InterruptedException e) {
//            e.printStackTrace();
//        }
        return "线程池："+Thread.currentThread().getName()+"Timeout"+id;
    }
```

无论是超时异常还是运行异常都有兜底策略

消费者同理

全面服务降级：

存在问题：

1.每一个方法都需要配置一个降级方法

2.和业务代码在一起

1：没有配置过得找defaultFallback的，配置了的找自己精确打击的fallback

```java
@RestController
@Slf4j
// 1. 添加注解，标注全局服务降级方法
@DefaultProperties(defaultFallback = "paymentGlobalFallBack")
public class OrderController {
    @Resource
    private PaymentHystrixService service;

    // 3. 写 @HystrixCommand单不指定具体方法 
    @GetMapping("/ok/{id}")
    @HystrixCommand
    public String paymentInfo_OK(@PathVariable Integer id) {
        int a = 10/0;
        return service.paymentInfo_OK(id);
    }

    public String paymentInfo_TimeoutHandler(Integer id){
        return "80异常，降级处理";
    }

    // 2. 定义全局服务降级方法
    // 下面是全局 fallback
    public String paymentGlobalFallBack(){
        return "80：获取异常，调用方法为全局fallback";
    }
}
```

2：

找到注解 @FeignClient 对应的接口

```java
@Component
@FeignClient(value = "CLOUD-PROVIDER-HYSTRIX-PAYMENT",fallback = PaymentFallBackService.class)
public interface PaymentHystrixService {
  @GetMapping("")
  public String paymentInfo_OK(@PathVariable("")Integer id)
}
```



再写一个类实现该接口，对降级方法进行处理

![image-20200820181300278](/typora-user-images/image-20200820181300278.png)

#### 服务熔断

1. 类比保险丝达到最大服务访问时，直接拒绝访问，拉闸限电，然后调用服务降级的方法返回友好提示。
2. 服务降级->进而熔断->恢复调用链路(检测到该节点微服务调用响应正常后，恢复调用链路)

1.service

```java
    // 服务熔断
    @HystrixCommand(fallbackMethod = "paymentInfo_Circuit",commandProperties = {
            @HystrixProperty(name="circuitBreaker.enabled",value = "true"),//是否开启断路器
            @HystrixProperty(name="circuitBreaker.requestVolumeThreshold",value = "10"),// 请求次数默认为20次
            @HystrixProperty(name="circuitBreaker.sleepWindowInMilliseconds",value = "10000"),// 时间窗口期默认为20秒
            @HystrixProperty(name="circuitBreaker.errorThresholdPercentage",value = "60")// 失败率默认为百分之50
            // 加起来就是在10s内的10次请求中如果失败超过6次进入服务熔断
      			// 当满足阈值，且失败率达到，将会开启断路器进行熔断，开启的时候不会进行请求转发，一段时间后，断路器是半开状态，会让其中一个请求进行转发，如果成功断路器关闭，若失败则继续开启。
    })
    public String paymentCircuitBreaker(@PathVariable("id") Integer id){
        if (id<0){
            throw new RuntimeException("id 不能为负数");
        }
        String serialNumber = IdUtil.simpleUUID();

        return "调用成功："+serialNumber;
    }

    public String paymentInfo_Circuit(Integer id){
        return "id不能为负数："+id;
    }
```

2.controller

```java
    // 服务熔断
    @GetMapping("/circuit/{id}")
    public String paymentCircuitBreaker(@PathVariable("id") Integer id){
        String result = paymentService.paymentCircuitBreaker(id);
        log.info("************"+result);
        return result;
    }
```

3.结果

​	一直输入id为负规定时间内，失败次数占到百分之60则开启熔断机制，之后就算输入是正确的，也是错误页面，一段时间恢复正确

总结：

熔断类型有三种分别是

* 打开，请求不在进行调用当前服务，内部时钟设置为MTTR（平均故障处理时间），当打开时长达到所设置时钟则进入半熔断状态
* 关闭，熔断关闭不会对服务进行熔断
* 半开，部分请求根据规则调用服务，如果请求成功且符合规则，则认为当前服务恢复正常关闭熔断



#### 服务限流

1.秒杀高并发等操作，严禁一窝蜂过来拥挤，一秒N个有序进行。



## web界面图形化展示Dashboard

### 搭建

1. 建 moudle
   cloud-consumer-hystrix-dashboard9001
2. pom

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
</dependency>
```

3. yml
   只需要配置端口号就行
4. 启动类
   加注解@EnableHystrixDashboard
5. 测试
   http://localhost:9001/hystrix有页面即为成功

### 使用

###### 注意

1. 注意：依赖于actuator，要监控哪个接口，哪个接口必须有这个依赖
2. 业务模块需要添加bean

```java
    @Bean
    public ServletRegistrationBean getServlet(){
        HystrixMetricsStreamServlet streamServlet = new HystrixMetricsStreamServlet();
        ServletRegistrationBean registrationBean = new ServletRegistrationBean(streamServlet);
        registrationBean.setLoadOnStartup(1);
        registrationBean.addUrlMappings("/hystrix.stream");
        registrationBean.setName("HystrixMetricsStreamServlet");
        return registrationBean;
    }
```

###### 使用

<img src="/typora-user-images/dashboard.png">

1. 进行8001 的访问查看对应页面变化
2. 页面状态
   1. 七色
      对应不同状态
   2. 一圈
      对应访问量
   3. 一线
      访问趋势



## Gateway

传统的web框架，比如说Struts2，springMVC等都是基于ServletAPI与Servlet容器基础上运行的，但是在Servlet3.1之后有了异步非阻塞的支持，而WebFlux是一个典型的非阻塞异步的框架，它的核心是基于Reactor的相关API实现的。springcloud Gateway是基于WebFlux框架实现的，而webFlux框架底层则使用了高性能的Reactor模式通信框架Netty，是异步非阻塞的框架

### 官网

https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.2.2.RELEASE/reference/html/

### 结构

<img src="/typora-user-images/springcloud结构.png">

<img src="/typora-user-images/网关作用.png">

### 三大核心概念

1. Route（路由）
   网关的基本构建块。它由ID，目标URI，谓词集合和过滤器集合定义。如果断言为true，则匹配路由。
2. Predicate（断言）
   这是Java 8 Function谓词。输入类型是Spring FrameworkServerWebExchange。这使您可以匹配HTTP请求中的所有内容，例如标头或参数。
3. Filter（过滤器）
   这些是使用特定工厂构造的Spring FrameworkGatewayFilter实例。在这里，您可以在发送下游请求之前或之后修改请求和响应。

### 工作流程

<img src="/typora-user-images/gateway工作流程.png">

客户端向Spring Cloud Gateway发出请求。如果网关处理程序映射确定请求与路由匹配，则将其发送到网关Web处理程序。该处理程序通过特定于请求的过滤器链来运行请求。筛选器由虚线分隔的原因是，过滤器可以在发送代理请求之前（pre）和之后（post）运行逻辑，filter在pre类型的过滤器可以做参数校验，权限校验，流量监控，日志输出，协议转换等，在post类型的过滤器中可以做响应内容，响应头的修改，日志的输出，流量监控等有着非常重要的作用

### 建模块:cloud-gateway-gateway9527

1. pom，gateway也要注册到服务中心

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
<!-- 注意不要添加 web的依赖，与gateway里的web flux冲突，因为是网关服务 -->
```

2. yml

```yml
server:
	port: 9527
spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      routes: # 可以配置多个路由
        - id: payment_routh # 路由id，没有固定规则但要求唯一
          uri:  http://localhost:8001 # 匹配后提供服务的路由地址
          predicates:
            - Path=/payment/get/** # 路径相匹配的进行路由

        - id: payment_routh2 # 路由id，没有
          uri:  http://localhost:8001 # 匹配后提供服务的路由地址
          predicates:
            - Path=/payment/payment # 路径相匹配的进行路由
eureka:
	instance:
		hostname: cloud-gateway-service
	client: #服务提供者provideder注册进eureka服务列表内
		service-url:
			register-with-eureka: ture
			fetch-registry: true
			defaultZone: http://eureka7001.com:7001/eureka
```

3. 启动类

```java
@SpringBootApplication
@EnableEurekaClient
@EnableDiscoveryClient
```

### 路由映射

1. 9527中配置路由

```yml
spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      routes: # 可以配置多个路由
        - id: payment_routh # 路由id，没有固定规则但要求唯一
          uri:  http://localhost:8001 # 匹配后提供服务的路由地址
          predicates:
            - Path=/payment/get/** # 路径相匹配的进行路由

        - id: payment_routh2 # 路由id，没有
          uri:  http://localhost:8001 # 匹配后提供服务的路由地址
          predicates:
            - Path=/payment/payment # 路径相匹配的进行路由
```

2. 配置后可以通过以下路径访问8001中的信息
   http://localhost:9527/payment/get/31
   不再暴露8001的端口
3. 配置路由的另一种方法，9527注入 RouteLocator的Bean

```java
@Configuration
public class GateWayConfig {
    @Bean
    public RouteLocator routeLocator(RouteLocatorBuilder routeLocatorBuilder){
        RouteLocatorBuilder.Builder  routes = routeLocatorBuilder.routes();

        /*
        * 代表访问http://localhost:9527/guonei
        * 跳转到http://news.baidu.com/guonei
        * */
        routes.route("route1",
                r->r.path("/guonei")
                .uri("http://news.baidu.com/guonei")).build();
        return routes.build();
    }
}
```

有什么问题？地址写死了！

负载均衡由网关去实现默认情况下Gateway会根据注册中心注册的服务列表，以注册中心上的微服务名为路径创建动态路由进行转发，从而实现动态路由的功能

1. 9527yml

```yml
server:
  port: 9527
spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true # 1.开启从服务在注册中心动态创建路由的功能
      routes:
        - id: payment_routh
#          uri:  http://localhost:8001 # 匹配后提供服务的路由地址
          uri:  lb://cloud-payment-service # 2.输入服务名，lb代表负载均衡
          predicates:
            - Path=/payment/get/** 

        - id: payment_routh2 
#          uri:  http://localhost:8001 # 匹配后提供服务的路由地址
          uri:  lb://cloud-payment-service # 2.输入服务名，lb代表负载均衡
          predicates:
            - Path=/payment/create 
```

### 断言Predicate

https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.2.2.RELEASE/reference/html/#the-between-route-predicate-factory
全部在 yml的Predicate之下

1. After

```yml
# 在该时间之后可以使用
- After=2020-05-26T17:07:03.043+08:00[Asia/Shanghai]
```

获取当前时区的时间

```java
ZonedDateTime z = ZonedDateTime.now();// 默认时区
```

2. Before

```yml
# 之前
- Before=2017-01-20T17:42:47.789-07:00[America/Denver]
```

3. Between

```yml
# 之间
- Between=2017-01-20T17:42:47.789-07:00[America/Denver], 2017-01-21T17:42:47.789-07:00[America/Denver]
```

4. Cookie

```yml
# 查看有没有指定kv的cookie
- Cookie=username,myq
```

5. Header

```yml
# 请求头，跟cookie一样指定kv键值对
- Header=X-Request-Id,\d+
```

6. Host

```yml
# 
```

7. Method

```yml
# 
```

8. Path

```yml
# 
```

9. Query

```yml
# 
```

10. ReadBodyPredicateFactory

```yml
# 
```

11. RemoteAddr

```yml
# 
```

12. Weight

```yml
# 
```

13. CloudFoundryRouteService

```yml
# 
```

### filter过滤器

常用的在请求头上加一个值的过滤器是

AddRequestParameter=X-Request-Id,1024 # 过滤器工厂会在匹配的请求头加上一个请求头，名为X-Request-Id值为1024。

### 自定义过滤器

1. 实现接口GlobalFilter,Ordered
2. 能干嘛
   1. 全局日志记录
   2. 统一网关鉴权
3. 案例

```java
@Component
@Slf4j
public class MyLogFilter implements GlobalFilter, Ordered {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        // 判断有没有 uname 这个参数
        log.info("自定义全局日志过滤器");
      	// 通过exchange参数获取请求，获取请求参数，在获取请求参数中带uname的
        String uname = exchange.getRequest().getQueryParams().getFirst("uname");
        if (uname==null){
            log.info("用户名非法");
            exchange.getResponse().setStatusCode(HttpStatus.NOT_ACCEPTABLE);
            return exchange.getResponse().setComplete();
        }
        return chain.filter(exchange);
    }
    /*
    *     int HIGHEST_PRECEDENCE = -2147483648;
            int LOWEST_PRECEDENCE = 2147483647;
            * 加载过滤器顺序
            * 数字越小优先级越高
    * */
    @Override
    public int getOrder() {
        return 0;
    }
}
```

## Stream

屏蔽底层消息中间件的差异，降低切换成本，统一消息的编程模型。

应用程序通过 inputs 和 outputs 来与 Spring Cloud Stream 中的 binder 对象交互。

一般标准的MQ都会有如下步骤

1. 生产者和消费者之间靠消息媒介传递消息内容？Message
2. 消息必须走特定通道？MessageChannel
3. 消息通道里的消息如何被消费？
   消息通道MessageChannel的子接口SubscribableChanner，由MessageHandler消息处理器所订阅

像RabbitMQ有exchange，kafka有Topic和Partitions分区，你凭什么可以屏蔽底层差异？

我们现在需要有一种目的地的绑定器作为中间层，完美地实现了应用程序与消息中间件细节之间的隔离。通过向应用程序暴露统一的Channel通道，使得应用程序不需要再考虑各种不同的消息中间件实现。

遵循发布订阅模式

1. @Input
   注解标识输入通道，通过该输入通道接收到的消息进入应用程序
2. @Output
   注解标识输出通道，发布的消息将通过该通道离开应用程序
3. @StreamListener
   监听队列，用于消费者的队列的消息接收
4. @EnableBinding
   channel和消息中间件的exchange,或Topic绑定在一起

流程简述：

需要写一个消息请求体Req，设置字段值后调用service的统一消息处理方法，根据匹配不同的消息类型统一构造消息数据，构造取得接受者的数据，并存储到list中，调用发消息方法，参数有主播用户，消息类型，以及通过jackson框架将对象转为string，发消息方法如下

```java
boolean success = streamMessageSource.output().send(MessageBuilder.withPayload(ojm.json(message)).build());
```

streamMessageSource接口如下走MessageChannel通道中的send方法，其中参数是消息构造者模式调用withPayload（）消息体方法，参数为通过json框架转为string类型

```java
public interface SocketMessageSource {

    String OUTPUT = "socketMessageOutput";

    @Output(SocketMessageSource.OUTPUT)
    MessageChannel output();
}
```

消费者方法

```java
public interface SocketMessageSink {

    String INPUT = "socketMessageInput";

    @Input(SocketMessageSink.INPUT)
    SubscribableChannel input();
}
```

消费者监听进行消费的操作，决定最后是由webSocket还是Scoket，需要@StreamListener注解进行消息监听，参数是消费者

其中建立sessionID最终调用WebSocketSession中的sendMessage发送消息

```java
 /**
     * 消费消息.
     *
     * @param message
     */
    @StreamListener(SocketMessageSink.INPUT)
    public void onMessage(Message<StreamMessage> message) {
```

在消费者端配置消费者配置文件

```java
@Configuration
@EnableBinding({WsMessageSink.class, StreamMessageSink.class, SocketMessageSink.class})
public class StreamConfig {
}
```

在生产者端配置生产者配置文件

```java
/**
 * 消息流配置.
 *
 * @author Peter
 */
@Configuration
@EnableBinding({
        GiveGiftSource.class,
        GiveGiftSink.class,
        StreamerGiftSource.class,
        StreamMessageSource.class,
        SocketMessageSource.class
})
public class StreamConfig {
}
```

通过监听者监听消息，通过注解@StreamListener(监听源类.INPUT)

在监听者类收到消息后，**Kafka阶段完成Kafka阶段只在项目的服务间进行消息发送**，kafka消息发送到socket服务中，socket服务中使用socket协议再去往APP端进行消息发送，创建会话会生成一个sessionID会话ID，通过WebSocketSession接口中的sendMassage发送消息

同样如果webScoket接收到Kafka消息后也会通过webSocke服务向网页端发送消息

## springboot整合caffeine缓存

第一步：引入依赖
这里不知道为什么很多版本的依赖包用不了

```xml
 <dependency>
     <groupId>com.github.ben-manes.caffeine</groupId>
     <artifactId>caffeine</artifactId>
     <version>2.5.5</version>
 </dependency>
```

第二部：配置相关文档：
有两种方法：

1.application.yml配置文件中配置：
优点：简单
缺点：无法针对每个cache配置不同的参数，比如过期时长、最大容量

2.配置类中配置
优点：可以针对每个cache配置不同的参数，比如过期时长、最大容量
缺点：要写一点代码

1. 在yml文件中

```yml
spring:
  cache:
    type: caffeine
    cache-names:
    - getPersonById
    - name2
    caffeine:
      spec: maximumSize=500,expireAfterWrite=5s
```

1. 在启动类中加入@EnableCaching
2. 在对应类中加入对应的缓存方法：

```java
@CacheConfig(cacheNames = "emp")
@Service
public class EmployService {

    @Autowired
    EmploeeMapper emploeeMapper;

    @Cacheable(cacheNames = {"emp"})
    public Employee getEmpById(Integer id){
        System.out.println("查询" + id + "号员工");
        Employee employee = emploeeMapper.getEmpById(id);
        System.out.print(employee);
        return  employee;
    }
}
```

运行后：可以在debug控制台中看到CaffeineCacheConfig被引用了

具体了解如下：https://www.codetd.com/article/6646484

## springsecurity@PreAuthorize @PostAuthorize @PreFilter @PostFilter 四者之间的区别？

 Spring Security中定义了四个支持使用表达式的注解，分别是@PreAuthorize、@PostAuthorize、@PreFilter和@PostFilter。其中前两者可以用来在方法调用前或者调用后进行权限检查，后两者可以用来对集合类型的参数或者返回值进行过滤。要使它们的定义能够对我们的方法的调用产生影响我们需要设置global-method-security元素的pre-post-annotations=”enabled”，默认为disabled。

  <security:global-method-security pre-post-annotations=*"disabled"*/>

###  使用@PreAuthorize和@PostAuthorize进行访问控制

@PreAuthorize可以用来控制一个方法是否能够被调用。

@Service

**public class** UserServiceImpl **implements** UserService {

@PreAuthorize("hasRole('ROLE_ADMIN')")

**public void** addUser(User user) {

System.*out*.println("addUser................" + user);

​	}

@PreAuthorize("hasRole('ROLE_USER') or hasRole('ROLE_ADMIN')")

**public** User find(**int** id) {

System.*out*.println("find user by id............." + id);

**return null**;

​	}

}

在上面的代码中我们定义了只有拥有角色ROLE_ADMIN的用户才能访问adduser()方法，而访问find()方法需要有ROLE_USER角色或ROLE_ADMIN角色。使用表达式时我们还可以在表达式中使用方法参数。

**public class** UserServiceImpl **implements** UserService {

/**

 * 限制只能查询Id小于10的用户

 */

 @PreAuthorize("#id<10")

**public** User find(**int** id) {

System.*out*.println("find user by id........." + id);

**return null**;

}

/**

\* 限制只能查询自己的信息

*/

  @PreAuthorize("principal.username.equals(#username)")

  **public** User find(String username) {

   System.*out*.println("find user by username......" + username);

   **return null**;

  }

  /**

   \* 限制只能新增用户名称为abc的用户

   */

  @PreAuthorize("#user.name.equals('abc')")

  **public void** add(User user) {

   System.*out*.println("addUser............" + user);

  }

}

  在上面代码中我们定义了调用find(int id)方法时，只允许参数id小于10的调用；调用find(String username)时只允许username为当前用户的用户名；定义了调用add()方法时只有当参数user的name为abc时才可以调用。

  有时候可能你会想在方法调用完之后进行权限检查，这种情况比较少，但是如果你有的话，Spring Security也为我们提供了支持，通过@PostAuthorize可以达到这一效果。使用@PostAuthorize时我们可以使用内置的表达式returnObject表示方法的返回值。

下面这一段示例代码：

@PostAuthorize("returnObject.id%2==0")

  **public** User find(**int** id) {

   User user = **new** User();

   user.setId(id);

   **return** user;

  }

​    上面这一段代码表示将在方法find()调用完成后进行权限检查，如果返回值的id是偶数则表示校验通过，否则表示校验失败，将抛出AccessDeniedException。需要注意的是@PostAuthorize是在方法调用完成后进行权限检查，它不能控制方法是否能被调用，只能在方法调用完成后检查权限决定是否要抛出AccessDeniedException。

### 使用@PreFilter和@PostFilter进行过滤

​    使用@PreFilter和@PostFilter可以对集合类型的参数或返回值进行过滤。使用@PreFilter和@PostFilter时，Spring Security将移除使对应表达式的结果为false的元素。

  @PostFilter("filterObject.id%2==0"）

 **public** List<User> findAll() {

   List<User> userList = **new** ArrayList<User>();

   User user;

   **for** (**int** i=0; i<10; i++) {

​     user = **new** User();

​     user.setId(i);

​     userList.add(user);

   }

   **return** userList;

  }

​    上述代码表示将对返回结果中id不为偶数的user进行移除。filterObject是使用@PreFilter和@PostFilter时的一个内置表达式，表示集合中的当前对象。当@PreFilter标注的方法拥有多个集合类型的参数时，需要通过@PreFilter的filterTarget属性指定当前@PreFilter是针对哪个参数进行过滤的。

如下面代码就通过filterTarget指定了当前@PreFilter是用来过滤参数ids的。

```java
   @PreFilter(filterTarget="ids", value="filterObject%2==0")
   public void delete(List<Integer> ids, List<String> usernames) {
      ...
   }
```

### JAVA中解决跨域问题

https://blog.csdn.net/DOTilld/article/details/108710295

```yml
gateway:
  globalcors:
    cors-configurations:
      '[/**]':
        allowCredentials: true
        allowedOrigins: "*" # 允许跨域的源(网站域名/ip)，设置*为全部
        allowedHeaders: "*" # 允许跨域请求里的head字段，设置*为全部
        allowedMethods: "*" # 允许跨域的method， 默认为GET和OPTIONS，设置*为全部
```

# 工作知识记录:

@JsonProperty 此注解用于属性上，作用是把该属性的名称序列化为另外一个名称，如把trueName属性序列化为name

1.前端传参数过来的时候，使用这个注解，可以获取到前端与注解中同名的属性

2.后端处理好结果后，返回给前端的属性名也不以实体类属性名为准，而以注解中的属性名为准

@RequestBody的作用其实是将json格式的数据转为java对象。

前台向后台传递json格式的数据，后台会自动匹配到实体对象中，当然实体对象中的属性名称要一样。

在postman测试中必须要带一个X-Request-Client

```java
// LocalTime类型可以使用SecondOfDay得出这个时间的秒数，例如01:30:00 得出 4800秒
```



RestTemplate提供了多种便携访问远程http服务的方法，是一种简单便捷的访问restful服务模板类，是spring提供的用于访问rest服务的客户端模板工具集

使用restTemplate访问restful接口非常的简单无脑粗暴。

（url，requestMap，ResponseBean.class）这三个参数分别是

rest请求地址，请求参数或是对象，http响应转换成对象类型。

如果想用一个List的类型去接在restTemplate里该怎样接？

```java
new ParameterizedTypeReference<List<类>>()// 变为引用类型可以拿List去接，对应resttemplate
```

```java
new TypeReference<List<类>>() {}// 通过objectMapper反序列化为一个List
```



因为main方法并没有被spring管理，所以在main方法中使用被spring管理的方法会造成空指针异常



例如通过template访问API

```java
方法外定义：
  private final static String AUTO_NUMBER = "/autonumber/auto?num={num}&key={key}";
  private final static String POLL_QUERY = "/poll/query.do";

String paramJSON = ojm.json(param);

    @Value("${kuaidi100.key}")
    private String key;

    @Value("${kuaidi100.customer}")
    private String customer;

    @Value("${kuaidi100.domain}")
    private String domain;

// 签名
String signValue = String.format("%s%s%s", paramJSON, key, customer);
HashFunction md5 = Hashing.md5();
String sign = md5.hashString(signValue, Charsets.UTF_8).toString().toUpperCase();

MultiValueMap<String, String> map = new LinkedMultiValueMap<>();// 应用在form表单一个key存储多个value
map.add("customer", customer);
map.add("sign", sign);
map.add("param", paramJSON);

// 采用form表单提交的方式
HttpHeaders headers = new HttpHeaders();
HttpEntity<MultiValueMap<String, String>> httpEntity = new HttpEntity(map, headers);// 设置两个参数（请求体，请求头）
ResponseEntity<String> responseEntity = restTemplate.exchange(domain + POLL_QUERY, POST, httpEntity, String.class);
String resBody = responseEntity.getBody();
log.info("[快递100] 物流查询 响应 - {}", resBody);

// 将String格式的JSON字符串转换为对象
PollQueryRes res = ojm.object(resBody, PollQueryRes.class);


识别快递公司编码
String resBody = restTemplate.getForObject(domain + AUTO_NUMBER, String.class, num, key);
log.info("[快递100] 单号识别 响应 - {}", resBody);
List<AutoNumRes> res = ojm.object(resBody, new TypeReference<List<AutoNumRes>>() {
});
```



@Bean的意思是通过注解的方式去依赖注入

https://www.jianshu.com/p/f303d51cc79d

@Autowired，和@Resource是自动装配

详解：https://www.cnblogs.com/zhuwoyao88/p/6596295.html



SQL中LIMIT 6 限定查六条

ORDER BY RAND()随机排序



通过分隔符分割字符串并转为数组或集合存储起来

```java
List<String> contentList = Arrays.asList(req.getContent().split("\n"));
// 还有一个方法如下
Lists.asList()
```



在对一个字段进行批次处理的时候可以使用三木运算符

对于在别的分支上首先要提到本地仓库再去切换别的分支



使用docker-compose运行文件时，通过此条代码可以选择运行环境

java -jar -Dspring.profile=uat

docker如何配置debug

* 选择启动栏编辑，找到Remote添加进来（！非子路径下的Remote）

![image-20210301184827444](/typora-user-images/image-20210301184827444.png)

```yaml
  dw-service:
    image: adoptopenjdk/openjdk8:latest
    restart: always
    hostname: dw-service
    volumes:
      - ./business-center/dw-service/target:/deploy:ro
    ports:
      - 5008:5008 # 别忘了配置端口号
    depends_on:
      - redis
      - zoo-1
      - kafka-1
      - es-1
    command: "java -Xms128m -Xmx128m \
      -Duser.timezone=GMT+08 -Dfile.encoding=utf-8 \
      -server -XX:+HeapDumpOnOutOfMemoryError \
      -Dserver.port=10008 \
      -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5008 # 图上语句加到此
      -jar /deploy/dw-service-0.0.1-SNAPSHOT.jar"
```

.plusDays()工具类，意思是加多少天



"<![CDATA[]]>"为转义字符，<=的意思被转成<

```sql
<![CDATA[
UPDATE cs_user
SET group_id = #{groupID},
    pkg_id = #{packageID},
    pkg_nm = #{packageName},
    days = #{days},
    open_time = expire_time,
    expire_time = #{expireTime}
WHERE client_id = #{clientID}
AND user_nm = #{userName}
AND (expire_time IS NULL OR expire_time <= #{expireTime})
]]>
```

返回空数据！

```java
PagingApiRes res = PagingApi.ok(Lists.newArrayList(), 0);
res.setTotalPage(Paging.totalPage(req, res));
res.setPage(req.getPage());
return res;
```



### Elasticsearch 

通过倒排索引完全过滤掉不相干的数据提高效率，分片的底层就是倒排索引

分词器：

最少切分ik_smart（只切分一次）例如"中国共产党"————>"中国共产党"

最细粒度划分ik_max_word（我理解为最多切分）穷尽可能性，在字典中有很多可能性，如果自己的词被拆开了可以把词加在字典中

例如"中国共产党"————>"中国"，"共产"，"党"，"国共"，"共产"



编写自己的配置文件注入到扩展字典即可，kuang.dic为自己编写的配置文件，里边是文字

![image-20201016142741601](/typora-user-images/image-20201016142741601.png)

我们需要自己配置分词就在自己定义的dic文件中进行配置即可

重启es

#### REST风格操作

![image-20201130100723661](/typora-user-images/image-20201130100723661.png)

关于索引的基本操作

创建一个索引！

```json
PUT /索引名/类型名/文档id
{
  "name":"yuanqing",
  "age":3
}
```

创建索引规则

```json
PUT /test2
{
  "mappings":{
    "properties":{
      "name":{
        "type":"text"
      },
      "age":{
        "type":"long"
      },
      "birthday":{
        "type":"text"
      }
    }
  }
}
```

获取这个规则！可以通过GET请求获取具体的信息

```json
GET /索引名
```

查看默认的信息

```json
GET _cat/health 可以获得es当前很多信息
GET _cat/indices?v
```

如果自己的文档字段没有指定，那么es就会给我们默认配置字段类型

```json
PUT /test3/_doc/1                 _doc是默认的
{	
  "name":"kuangshenshuo",如果自己的文档没有指定，那么es就会给我们默认配置字段类型为keywords，不可分割
  "age":13,
  "birth":"1998-08-11"
}

修改的方法

1.直接覆盖
修改的操作版本号会增加,但如果漏掉一个字段那么就没了
PUT /test3/_doc/1                 _doc是默认的
{	
  "name":"mengyuanqing",
  "age":13,
  "birth":"1998-08-11"
}

2.
如下修改操作，版本号也会增加
POST /test3/_doc/1/_update                 _doc是默认的
{	
  "doc":{
    "name":"法外狂徒"
  }
}

GET test3

删除索引：
DELETE 索引                    删除索引根据请求判断删除索引还是文档（后面跟上id会删除库中其中一个）
```

关于文档的基本操作

```json
GET 索引名/类型/id
就能获取到数据
```

更新数据
Post _update，推荐这种更新方式，只更新第一个数据当中的name，灵活性更高

![image-20201130103612951](/typora-user-images/image-20201130103612951.png)

可以根据默认的映射规则，产生基本的查询

```json
GET 索引/字段（type）/_search?q=name:要搜索的的字段（条件）

_search?q=name:蒙哥

```

查询结果有_score字段，分值越高证明匹配度越高

如果搜索条件模糊搜不到是为什么？

查看索引规则，有可能是因为name字段类型是keyword，通过分词器就会不生效导致搜不到 

#### 复杂查询

排序，分页，高亮，模糊查询，精准查询

![image-20201130151628163](/typora-user-images/image-20201130151628163.png)

![image-20201130151815583](/typora-user-images/image-20201130151815583.png)

Hits是一个对象对应着我们的java操作es，的对象操作

![image-20201130152126987](/typora-user-images/image-20201130152126987.png)

查询指定字段

![image-20201130152333227](/typora-user-images/image-20201130152333227.png)

相当于mysql中select*————>select 字段，字段

用java操作es，所有方法和对象就是这里面的key！

* 排序

![image-20201130152835759](/typora-user-images/image-20201130152835759.png)

* 分页

![image-20201130153050111](/typora-user-images/image-20201130153050111.png)

数据下标还是从0开始的，和学的所有数据结构是一样的

* 布尔值查询
  
  * must（and），所有的条件都要符合 where id=1 and name=xxx
  
  * ```java
    {
        "query":{
            "bool":{
                "must":[
                    {
                        "match":{
                        "user_nm":"13841131700"
                        }
                    },
                    {
                        "match":{
                        "before_amt":"54609"
                        }
                    }
                ]
            }
        }
    }
    ```

![image-20201130153443372](/typora-user-images/image-20201130153443372.png)

* * shuould（or），所有的条件都要符合 where id=1 or name=xxx

![image-20201130153757172](/typora-user-images/image-20201130153757172.png)

* * 查询年龄不是三岁的must_not

![image-20201130154029970](/typora-user-images/image-20201130154029970.png)

* * 使用filter进行数据过滤

![image-20201130154352658](/typora-user-images/image-20201130154352658.png)

* gt 大于

* lt 小于

* gte 大于等于

* lte 小于等于

![image-20201130154559140](/typora-user-images/image-20201130154559140.png)

* 匹配多个条件

![image-20201201102744015](/typora-user-images/image-20201201102744015.png)

* 精确查询

term 查询是直接通过倒排索引指定的词条进行精确查找的！

* 关于分词

term，直接精确查询

match，会使用分词器解析！（先分析文档，然后在通过分析的文档进行查询）

**两个类型 text keyword**

![image-20201201110836694](/typora-user-images/image-20201201110836694.png)

而不是keywords发现被拆分了

![image-20201201111022218](/typora-user-images/image-20201201111022218.png)

从而得出结论

![image-20201201111320782](/typora-user-images/image-20201201111320782.png)

精确查询多个值

![image-20201201123053683](/typora-user-images/image-20201201123053683.png)

* 高亮查询

![image-20201201123903725](/typora-user-images/image-20201201123903725.png)

自定义高亮条件：

![image-20201201124107645](/typora-user-images/image-20201201124107645.png)

<beans id=方法名，class=返回值类型>现在用@Bean注入到spring

创建索引的本质,初始化一个请求对象，设置索引名，对象规则，通过JSON框架将对象转化为JSON数据放入ElasticSearch中，设置ID

```java
            IndexRequest req = new IndexRequest()
                    .index(CASH.name(clientID, datetime))
                    .source(ojm.json(cash), JSON)
                    .id(cash.getCashID());
```

查询索引记录

```java
            SearchRequest searchReq = new SearchRequest()
                    .indices(CASH.alias())
                    .source(source);

            SearchResponse searchRes = esClient.client()
                    .search(searchReq, RequestOptions.DEFAULT);
```

修改索引记录,注意这里传入参数为doc

```java
            UpdateRequest updateRequest = new UpdateRequest().doc(ojm.json(source), JSON).index("xxxx").id("1");
            UpdateResponse response = esClient.client().update(updateRequest,RequestOptions.DEFAULT);
```

删除索引记录

```java
            DeleteRequest deleteRequest = new DeleteRequest().index("xxx").id("1");
            deleteRequest.timeout("1");
            DeleteResponse deleteResponse = esClient.client().delete(deleteRequest,RequestOptions.DEFAULT);
            SearchResponse searchRes = esClient.client()
                    .search(searchReq, RequestOptions.DEFAULT);
```

批量插入数据，批量更新和删除就在批处理修改对应请求即可

![image-20201202105139903](/typora-user-images/image-20201202105139903.png)

精确查询

```java
QueryBuilders.termQuery("name","yuanqing");
QueryBuilders.matchAllQuery();// 匹配所有查询
TermQueryBuilder termQueryBuilder = termQuery("name", "yuanqing");
HighLightBuilder // 构建高亮
MatchAllQueryBuilder // 匹配所有构造器
```

构造搜索条件

```java
// 其中cashListQuery是匹配所有的查询方法，条件构造
SearchSourceBuilder source = new SearchSourceBuilder()
  .query(cashListQuery(clientID, req))
  .sort("create_time", DESC);
if (req.getPageSize() > 0) {
  source.from(req.getOffset()).size(req.getPageSize());
}
// 将上述精确查询构造器也可以放入.query中当做条件查询例如：

  TermQueryBuilder termQueryBuilder = termQuery("name", "yuanqing");
SearchSourceBuilder source = new SearchSourceBuilder()
  .query(termQueryBuilder)
  .sort("create_time", DESC);

// 发送请求，并通过SearchResponse接到请求后的结果
  SearchRequest searchReq = new SearchRequest()
  .indices(CASH.alias())
  .source(source);
// 收到结果
SearchResponse searchRes = esClient.client()
  .search(searchReq, RequestOptions.DEFAULT);
// 我们查询到的结果全部在hit中
  SearchHits hits = searchRes.getHits();
List<Cash> items = Stream.of(hits.getHits())
  			// 将EScash实体转为cash实体
  .map(hit -> cash(ojm.object(hit.getSourceAsString(), EsCash.class)))
  .collect(toList());
int count = Long.valueOf(hits.getTotalHits().value).intValue();
```

最后奉上全部代码

```java
package io.cs.user.service;

import com.google.common.collect.Lists;
import io.cs.common.exception.ServiceException;
import io.cs.common.exception.SystemException;
import io.cs.common.model.Paging;
import io.cs.common.model.PagingApi;
import io.cs.common.model.PagingApiRes;
import io.cs.es.client.EsClient;
import io.cs.user.model.Cash;
import io.cs.user.model.CashListReq;
import io.cs.user.model.EsCash;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.lang.StringUtils;
import org.apache.commons.lang.exception.ExceptionUtils;
import org.elasticsearch.ElasticsearchException;
import org.elasticsearch.action.ActionListener;
import org.elasticsearch.action.index.IndexRequest;
import org.elasticsearch.action.index.IndexResponse;
import org.elasticsearch.action.search.SearchRequest;
import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.index.query.BoolQueryBuilder;
import org.elasticsearch.index.query.QueryBuilder;
import org.elasticsearch.rest.RestStatus;
import org.elasticsearch.search.SearchHits;
import org.elasticsearch.search.builder.SearchSourceBuilder;
import org.springframework.stereotype.Service;

import javax.annotation.Resource;
import java.time.Instant;
import java.time.LocalDateTime;
import java.util.List;
import java.util.stream.Stream;

import static io.cs.common.enumeration.EsIndex.CASH;
import static io.cs.common.util.Client.clientID;
import static io.cs.common.util.DateTimeUtil.OFFSET;
import static java.util.stream.Collectors.toList;
import static org.elasticsearch.common.xcontent.XContentType.JSON;
import static org.elasticsearch.index.query.QueryBuilders.*;
import static org.elasticsearch.search.sort.SortOrder.DESC;

/**
 * 余额记录服务
 *
 * @author doinb
 */
@Service
@Slf4j
public class EsCashService {

    @Resource
    private OjmService ojm;

    @Resource
    private EsClient esClient;

    /**
     * 创建流水
     *
     * @param cash
     */
    public void createEsCash(EsCash cash, ActionListener<IndexResponse> listener) {
        try {
            String clientID = cash.getClientID();
            Instant instant = Instant.ofEpochMilli(cash.getCreateTime());
            LocalDateTime datetime = LocalDateTime.ofInstant(instant, OFFSET);

            IndexRequest req = new IndexRequest()
                    .index(CASH.name(clientID, datetime))
                    .source(ojm.json(cash), JSON)
                    .id(cash.getCashID());

            esClient.client().indexAsync(req, RequestOptions.DEFAULT, listener);
        } catch (Exception e) {
            log.error("createEsCash error - {}, {}", ExceptionUtils.getStackTrace(e), cash);
        }
    }

    /**
     * 查询流水日志
     *
     * @param req
     * @return
     */
    public PagingApiRes getEsCashList(CashListReq req) {
        try {
            String clientID = clientID();

            SearchSourceBuilder source = new SearchSourceBuilder()
                    .query(cashListQuery(clientID, req))
                    .sort("create_time", DESC);
            if (req.getPageSize() > 0) {
                source.from(req.getOffset()).size(req.getPageSize());
            }

            SearchRequest searchReq = new SearchRequest()
                    .indices(CASH.alias())
                    .source(source);

            SearchResponse searchRes = esClient.client()
                    .search(searchReq, RequestOptions.DEFAULT);
            if (searchRes.status() == RestStatus.OK) {
                log.info("查询流水花费{}", searchRes.getTook().toString());
                SearchHits hits = searchRes.getHits();
                List<Cash> items = Stream.of(hits.getHits())
                        .map(hit -> cash(ojm.object(hit.getSourceAsString(), EsCash.class)))
                        .collect(toList());
                int count = Long.valueOf(hits.getTotalHits().value).intValue();

                PagingApiRes res = PagingApi.ok(items, count);
                res.setPage(req.getPage());
                res.setTotalPage(Paging.totalPage(req, res));
                return res;
            }
            throw new ServiceException(20001, "查询流水出错");
        } catch (ServiceException e) {
            throw e;
        } catch (ElasticsearchException e) {
            if (e.status() == RestStatus.NOT_FOUND) {
                PagingApiRes res = PagingApi.ok(Lists.newArrayList(), 0);
                res.setPage(1);
                res.setTotalPage(1);
                return res;
            }
            throw e;
        } catch (Exception e) {
            log.error("getEsCashList error - {}, {}", ExceptionUtils.getStackTrace(e), req);
            throw new SystemException(10001, "查询流水出错");
        }
    }

    /**
     * cashListQuery
     *
     * @param clientID
     * @param req
     * @return
     */
    private QueryBuilder cashListQuery(String clientID, CashListReq req) {
        LocalDateTime datetime = LocalDateTime.now(OFFSET);
        LocalDateTime createTimeFrom = req.getCreateTimeFrom() != null ? req.getCreateTimeFrom() : datetime.minusMonths(3);
        LocalDateTime createTimeTo = req.getCreateTimeTo() != null ? req.getCreateTimeTo() : datetime;

        BoolQueryBuilder query = boolQuery()
                .must(matchQuery("client_id", clientID))
                .must(rangeQuery("create_time")
                        .gte(createTimeFrom.toInstant(OFFSET).toEpochMilli())
                        .lte(createTimeTo.toInstant(OFFSET).toEpochMilli()));
        if (StringUtils.isNotEmpty(req.getCashType())) {
            query.must(matchQuery("cash_type", req.getCashType()));
        }
        if (StringUtils.isNotEmpty(req.getUserID())) {
            query.must(matchQuery("user_id", req.getUserID()));
        }
        if (StringUtils.isNotEmpty(req.getUserName())) {
            query.must(matchQuery("user_nm", req.getUserName()));
        }
        if (StringUtils.isNotEmpty(req.getPfTypeCode())) {
            query.must(matchQuery("pf_type_cd", req.getPfTypeCode()));
        }
        if (StringUtils.isNotEmpty(req.getGameTypeCode())) {
            query.must(matchQuery("game_type_cd", req.getGameTypeCode()));
        }
        return query;
    }

    /**
     * cash
     *
     * @param esCash
     * @return
     */
    private Cash cash(EsCash esCash) {
        Instant instant = Instant.ofEpochMilli(esCash.getCreateTime());
        LocalDateTime createTime = LocalDateTime.ofInstant(instant, OFFSET);
        Cash cash = new Cash();
        cash.setCashID(esCash.getCashID());
        cash.setCashType(esCash.getCashType());
        cash.setCashAmount(esCash.getCashAmount());
        cash.setBeforeAmount(esCash.getBeforeAmount());
        cash.setAfterAmount(esCash.getAfterAmount());
        cash.setUserID(esCash.getUserID());
        cash.setUserName(esCash.getUserName());
        cash.setPfTypeCode(esCash.getPfTypeCode());
        cash.setPfTypeName(esCash.getPfTypeName());
        cash.setGameTypeCode(esCash.getGameTypeCode());
        cash.setGameTypeName(esCash.getGameTypeName());
        cash.setRemark(esCash.getRemark());
        cash.setCreateTime(createTime);
        return cash;
    }
}

```

#### Elasticsearch之索引别名 alias

我们为索引my_index创建一个别名my_index_alias,这样我们对my_index_alias的操作就像对my_index的操作一样

```bash
POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "my_index",
        "alias": "my_index_alias"
      }
    }
  ]
}
```

别名不仅仅可以关联一个索引，它能聚合多个索引

我们为索引my_index_1 和 my_index_2 创建一个别名my_index_alias，这样对my_index_alias的操作(仅限读操作)，会操作my_index_1和my_index_2，类似于聚合了my_index_1和my_index_2.我们是不能对my_index_alias进行写操作，当有多个索引时alias，不能区分到底操作哪一个

```bash
POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "my_index_1",
        "alias": "my_index_alias"
      }
    },
    {
      "add": {
        "index": "my_index_2",
        "alias": "my_index_alias"
      }
    }
  ]
}

GET /my_index_alias/_search
{
}

实例如下
POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "my_index",
        "alias": "my_index__teamA_alias",
        "filter":{
            "term":{
                "team":"teamA"
            }
        }
      }
    },
    {
      "add": {
        "index": "my_index",
        "alias": "my_index__teamB_alias",
        "filter":{
            "term":{
                "team":"teamB"
            }
        }
      }
    },
    {
      "add": {
        "index": "my_index",
        "alias": "my_index__team_alias"
      }
    }
  ]
}

GET /my_index__teamA_alias/_search 只能看到teamA的数据
GET /my_index__teamB_alias/_search 只能看到teamB的数据
GET /my_index__team_alias/_search 既能看到teamA的，也能看到teamB的数据
```

https://www.cnblogs.com/rainwang/p/6062650.html

类型：

* 字符串
* text，keyword（如果类型是keyword则不会被分词器分词）
* 数值类型
* long，integer，short，byte，double，float，scaled，float
* 日期类型
* data

前端设置好时间表星期几的几点到几点播只有数字例如1是星期一，例如周一的11点到12点要进行播放，允许隔天设置，如周二的下午8点到周三的凌晨两点

代码如下：

```java
    /**
     * 批处理执行串流计划
     */
    public void executeStreamPlan() {
        try {
            Map<String, StreamPlanSchedule> map = getScheduleMap(LocalDateTime.now(OFFSET));
            for (Map.Entry<String, StreamPlanSchedule> entry : map.entrySet()) {
                try {
                    String key = entry.getKey();
                    String[] keys = key.split("_");
                    StreamPlanSchedule schedule = entry.getValue();
                    if (schedule != null) {
                      	// 在时间表中说明需要播放则调用开播
                        openStream(keys[0], keys[1], schedule.getRelayURL());
                    } else {
                      	// 不在时间表内说明需要关播
                        closeStream(keys[0], keys[1]);
                    }
                } catch (Exception e) {
                    log.error("executeStreamPlan error - {}, {}", ExceptionUtils.getStackTrace(e), entry);
                }
            }
        } catch (Exception e) {
            log.error("executeStreamPlan error - {}", ExceptionUtils.getStackTrace(e));
            throw new SystemException(10004, "处理执行串流计划出错");
        }
    }
    /**
     * 获取串流主播当前时间应当执行的串流任务.
     *
     * @param datetime
     * @return
     */
    private Map<String, StreamPlanSchedule> getScheduleMap(LocalDateTime datetime) {
        try {
            Map<String, StreamPlanSchedule> scheduleMap = Maps.newHashMap();
						// 去查询某个星期几的时间表，方法中的内容是如果结束时间小于开始时间则结束时间加一天
            List<StreamPlanSchedule> schedules = getSchedulesByDate(datetime.toLocalDate());
          	// 查询当前周几的前一天的时间表，方法内容同上
            schedules.addAll(getSchedulesByDate(datetime.toLocalDate().minusDays(1)));

            Map<String, List<StreamPlanSchedule>> map = Maps.newHashMap();
            for (StreamPlanSchedule schedule : schedules) {
                String key = String.format("%s_%s", schedule.getClientID(), schedule.getStreamerID());
                if (!map.containsKey(key)) {
                    map.put(key, Lists.newArrayList());
                }
                map.get(key).add(schedule);
            }
            for (Map.Entry<String, List<StreamPlanSchedule>> entry : map.entrySet()) {
                // 当前时间是否在开始时间和结束时间内
                StreamPlanSchedule schedule = entry.getValue().stream().filter(e -> !datetime.isBefore(e.getBeginDateTime()) && !datetime.isAfter(e.getEndDateTime())).findFirst().orElse(null);
                scheduleMap.put(entry.getKey(), schedule);
            }
            return scheduleMap;
        } catch (Exception e) {
            log.error("getScheduleMap error - {}, {}", ExceptionUtils.getStackTrace(e), datetime);
            throw new SystemException(10004, "获取串流任务出错");
        }
    }

```



# 填坑记：

出现该错误的原因：

HttpMessageConversionException: JSON mapping problem: io.cs.common.model.PagingApiRes[\"payload\"]->java.util.ArrayList[1]->io.cs.user.model.ClientDomain[\"domainName\"]; nested exception is com.fasterxml.jackson.databind.JsonMappingException: (was java.lang.NullPointerException) (through reference chain: io.cs.common.model.PagingApiRes[\"payload\"]->java.util.ArrayList[1]->io.cs.user.model.ClientDomain[\"domainName\"])

解决方法：

在实体中调用这个方法时，数据库的domainType字段肯定有为空的数据！！！

看看是什么原因造成了这个空的数据，或者将数据库中的数据都弄成有数据！

```java
public String getDomainName(){
    DomainType domainType = DomainType.instanceOf(this.domainType);
    return domainType == null ? null : domainType.getName();
}
```

如果priority=#{priority}，是int类型你没有给这个字段传参数那么这个字段会被赋值为0

SQL替换函数REPLACE（字段，之前的，要更新的）

```xml
<update id="batchUpdateDomain">
    UPDATE cs_movie
    SET cover = REPLACE(cover,#{beforeDomain},#{afterDomain}),
        video_url = REPLACE(video_url,#{beforeDomain},#{afterDomain})
</update>
```

```sql
        <!--如果有主键冲突则忽略-->
        INSERT IGNORE INTO oc_open_code (
            client_id,
            provider_id,
            lottery_id,
            issue_no,
            `code`,
            create_time
        ) VALUES (
            #{clientID},
            #{providerID},
            #{lotteryID},
            #{issueNO},
            #{code},
            NOW()
        ) ON DUPLICATE KEY UPDATE
            provider_id =#{providerID},
            provider_id = IF(#{openTime} >= open_time, #{providerID}, provider_id),通过条件去判断是否赋值
            `code` = #{code},
            create_time = NOW()
```

线程中断的三种方式https://www.cnblogs.com/liyutian/p/10196044.html



在进行插入和更新操作时，如果数据库为NOT NULL 并有默认值，那么插入不用管，但更新操作必须要照顾到全部字段，通过三木运算符去判断如果为空则自己设置默认值，或者提前查一下拿对象去更新，只更换对线的某些字段

查询时映射的什么对象，就拿什么对象去接，否则会报错



数据库报0值错误

* 数据库的类型是DATETIME类型

  Caused by: com.mysql.cj.exceptions.DataReadException: Zero date value prohibited

因为序列化的时候不能序列日期为0000



修改数据库密码出错可使用如下语句:

```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY '用户名';
```

association和collection有什么区别？

使用association要保证select=的语句，只能查出一个数据，或是一个对象，也就是一对一，"property="写到实体中拿对象或者拿基本类型去接

collectionselect=中语句，会查询出多个数据，或是多个对象，也就是一对多，通过"property="映射的字段需要写到实体中，拿List<对象>或者拿<基本类型>去接

column中要写select语句的入参，左边是实体=右边是数据库字段

```xml
<!--Mybatis-->
字段操作：
IF(f.user_id IS NULL, 0, 1) AS is_follow
一对一映射操作：
<association property="giftNo" column="username=user_nm,statsDate=stats_date,clientID=client_id,streamerID=streamer_id"select="io.cs.report.repository.StreamerGiftRepository.selectGiftNo"/>
一对多映射操作：
<collection property="articleCategoryList" column="clientID=client_id,articleID=article_id"select="io.cs.content.repository.ArticleCategoryRepository.selectList"/>
```

JAVA判空处理

```java
            if (amount.compareTo(ZERO) <= 0) {
                throw new ServiceException(20001, "提现金额无效");
            }
            if (!Lists.newArrayList(1, 2).contains(payType)) {
                throw new ServiceException(20001, "请选择一个账户类型");
            }

```

group_concat函数详解

https://blog.csdn.net/u012620150/article/details/81945004

时间处理

```java
当天
LocalDate now = LocalDate.now();
这周的第一天
LocalDate firstWeekDay = now.minusDays(date.getDayOfWeek().getValue() - 1);
这月的第一天
LocalDate firstMonthDay = now.with(TemporalAdjusters.firstDayOfMonth());
LocalDate firstMonthDay = now.withDayOfMonth(1);// 老板是这么用的
// 当天变为String类型
String dayInterval = date.toString();
// 当月第一天转为String类型
String firstMountDayInterval = firstMonthDay.format(DateTimeFormatter.ofPattern(DateTimeUtil.DATE_FORMATTER));
// 当周第一天转为String类型
String firstWeekDayInterval = firstWeekDay.format(DateTimeFormatter.ofPattern(DateTimeUtil.DATE_FORMATTER));
// 如果只传月份我该怎么处理
  String firstDay = month + "-01 00:00:00";
            // 开始时间转化
            LocalDateTime startTime = DateTimeUtil.parse(firstDay);
            // 结束时间转化
            LocalDateTime endTime = startTime.with(TemporalAdjusters.lastDayOfMonth());
```

踢走token

```java
            ClientInfo clientInfo = Client.info();
            clientInfo.setClientType(ClientType.APP.name());
            if (i > 0) {
                accessTokenService.deleteToken(clientInfo, user.getUserName());
            }
```

查询时通过聚合函数分组

```xml
SELECT COUNT(DISTINCT user_nm)
```

计算机网络概述：https://blog.csdn.net/gatieme/article/details/50989257

小数转String拼百分比

```java
1. ROUND_DOWN
BigDecimal b = new BigDecimal("2.225667").setScale(2, BigDecimal.ROUND_DOWN);
System.out.println(b);//2.22 直接去掉多余的位数
2. ROUND_UP
BigDecimal c = new BigDecimal("2.224667").setScale(2, BigDecimal.ROUND_UP);
System.out.println(c);//2.23 跟上面相反，进位处理

						BigDecimal s = BigDecimal.valueOf(successCount).divide(BigDecimal.valueOf(totalCount), 2, BigDecimal.ROUND_DOWN);
						// 创建格式化器
						/**
             * NumberFormat.getNumberInstance();数字格式化
             * NumberFormat.getCurrencyInstance();货币格式化
             * NumberFormat.getPercentInstance();百分比格式化
						 */
            NumberFormat percent = NumberFormat.getPercentInstance();
						// 设置小数部分允许最大位数
            percent.setMaximumFractionDigits(2);
            return percent.format(s);
```

MyBatis天坑

choose when中如果条件是字符串应该用单引号括起来，字符串时双引号

```xml
            <choose>
                <when test='dateInterval == "D"'>
                    AND stats_date = #{dateFrom}
                </when>
            </choose>
```

判断是哪个端

```java
            String clientType = clientType();
 if (clientType.equals(ClientType.ADMIN.name())){}
```

selectCount里写分页会报错，不能写下面这个

```xml
        <if test="pageSize != 0">
            LIMIT #{offset}, #{pageSize}
        </if>
```

```sql
        -- 连表查询时，a连b on后的条件相当于select b WHERE b.client_id = a中查出来的clien_id AND 
        FROM gp_game_cate a
        left JOIN gp_game_cate b
        ON b.client_id = a.client_id
        AND b.cate_id = a.parent_cate_id
        WHERE a.client_id = "test"
        AND a.cate_id = "AG"
```

peek用来修改数据,map用来转换数据类型

@transactional注解使用

https://developer.ibm.com/zh/articles/j-master-spring-transactional-use/

lambda在实际中的运用1：

```java
// 将lotteries中的数据构造一个map，键为lotteryID，而值为lotteries中对应的对象
Map<String, LotteryInfo> lotteryInfoMap = lotteries.stream().collect(toMap(LotteryInfo::getLotteryID,identity()));
// 通过map转变类型，通过get()方法将stream中的gameID拿出来放到上面构建的map如果有值则可以get到value否则不get，将get到的value在重新赋值到lotteries中，返回
lotteries = streamGameList.stream().map(e -> lotteryInfoMap.get(e.getGameID())).filter(Objects::nonNull).collect(toList());
```

# MySQL

索引是帮助MySQL高效获取排好序的数据的数据结构

SHOW GLOBAL STATUS LIKE 'Innodb_page_size' 可以查到MYSQL默认给磁盘存储页分配了16384，可以放16kb大小的数据

BIGINT=8个字节高度为3的索引一共能放1170 * 1170 * 16 = 两万多万 个索引

存储引擎是修饰数据库表的

MyISAM .frm存储表结构 .MYD存储表数据 .MYI存储表索引，条件查找时先判断是否有索引先去MYI定位索引元素，叶子节点存的是索引所在行的磁盘文件地址，根据地址到MYD文件通过地址定位到数据。

Innodb .frm存储表结构 .idb存储表数据和表索引合并存储，叶子节点放的是索引所在行的其他列数据。 

所以Innodb是聚簇索引（叶子节点包含了完整的数据记录），而MyISAM是非聚簇索引（索引文件和数据文件是分离的）

聚簇索引要快一点。

Innodb的非主键索引，叶子节点放的是主键，如果没有主键MYSQL会帮你找一个主键，依次去找到一个列所有元素都不重复的列，如果没有这样的列MYSQL会维护一个隐藏列（例如Row_id）来维护你整张表的所有数据

为什么要使用整型自增主键，因为B+树中间有很多比大小的比对，整形数据比大小比较快，UUID还要转化成ASICC码，而且UUID存储空间比整形要大

HASH索引：

B树和B+树的区别：

B树叶子节点没有指针，没办法支撑范围查找，B+树的非叶子节点只放索引，不存储数据，而B树每个节点都不冗余索引而且存储数据，树的分叉越多，其实高度越低，IO次数越少

自增的话永远会往叶子节点的右边去追加，如果主键不是自增的话，会造成每次插入数据时造成B+树分裂，自平衡

B+树为了支持范围查找，会将数据排好序尽管数据并不是有序的，所以自增效率更高

联合索引怎么拍好序：

逐个比较先比较第一个字段，第一个字段相同去比较第二个字段。

索引的优势：

- 类似大学图书馆建书目索引，提高数据检索效率，降低数据库的IO成本
- 通过索引列对数据进行排序，降低数据排序成本，降低了CPU的消耗

索引的劣势

- 实际上索引也是一张表，该表保存了主键和索引字段，并指向实体表的记录,所以索引列也是要占用空间的
- 虽然索引大大提高了查询速度，同时却会降低更新表的速度,如果对表INSERT,UPDATE和DELETE。
  因为更新表时，MySQL不仅要不存数据，还要保存一下索引文件每次更新添加了索引列的字段，
  都会调整因为更新所带来的键值变化后的索引信息
- 索引只是提高效率的一个因素，如果你的MySQL有大数据量的表，就需要花时间研究建立优秀的索引，或优化查询语句

索引的原理

任何一种数据结构都不是凭空产生的，一定会有它的背景和使用场景，我们现在总结一下，我们需要这种数据结构能够做些什么，其实很简单，那就是：每次查找数据时把磁盘IO次数控制在一个很小的数量级，最好是常数数量级。那么我们就想到如果一个高度可控的多路搜索树是否能满足需求呢？就这样，b+树应运而生（B+树是通过二叉查找树，再由平衡二叉树，B树演化而来）。

[![img](https://img2018.cnblogs.com/blog/1534894/201912/1534894-20191215142721830-1545447801.png)](https://img2018.cnblogs.com/blog/1534894/201912/1534894-20191215142721830-1545447801.png)

如上图，是一颗b+树，关于b+树的定义可以参见[B+树](http://zh.wikipedia.org/wiki/B%2B树)，这里只说一些重点，浅蓝色的块我们称之为一个磁盘块，可以看到每个磁盘块包含几个数据项（深蓝色所示）和指针（黄色所示），如磁盘块1包含数据项17和35，包含指针P1、P2、P3，P1表示小于17的磁盘块，P2表示在17和35之间的磁盘块，P3表示大于35的磁盘块。**真实的数据存在于叶子节点以及索引所在行的磁盘文件地址（或是索引所在行的其他列）即3、5、9、10、13、15、28、29、36、60、75、79、90、99。非叶子节点不存储真实的数据，只存储指引搜索方向的数据项，如17、35并不真实存在于数据表中。**

**b+树的查找过程**

如图所示，如果要查找数据项29，那么首先会把磁盘块1由磁盘加载到内存，此时发生一次IO，在内存中用二分查找确定29在17和35之间，锁定磁盘块1的P2指针，内存时间因为非常短（相比磁盘的IO）可以忽略不计，通过磁盘块1的P2指针的磁盘地址把磁盘块3由磁盘加载到内存，发生第二次IO，29在26和30之间，锁定磁盘块3的P2指针，通过指针加载磁盘块8到内存，发生第三次IO，同时内存中做二分查找找到29，结束查询，总计三次IO。真实的情况是，3层的b+树可以表示上百万的数据，如果上百万的数据查找只需要三次IO，性能提高将是巨大的，如果没有索引，每个数据项都要发生一次IO，那么总共需要百万次的IO，显然成本非常非常高。

**b+树叶子节点为什么要有序？**

从上图可以看出来， 所有的数据都在叶子节点，且每一个叶子节点都带有指向相邻节点的指针，形成了一个有序的链表。 这么做的原因也很简单， 是为了范围查询。 比如说`select * from Table where id > 1 and id < 100;` 当找到1后，只需顺着节点和指针顺序遍历就可以一次性访问到所有数据节点，极大提到了区间查询效率。

**b+树的性质**

1. **索引字段要尽量的小**：通过上面的分析，我们知道IO次数取决于b+数的高度h，假设当前数据表的数据为N，每个磁盘块的数据项的数量是m，则有h=㏒(m+1)N，当数据量N一定的情况下，m越大，h越小；而m = 磁盘块的大小 / 数据项的大小，磁盘块的大小也就是一个数据页的大小，是固定的，如果数据项占的空间越小，数据项的数量越多，树的高度越低。这就是为什么每个数据项，即索引字段要尽量的小，比如int占4字节，要比bigint8字节少一半。这也是为什么b+树要求把真实的数据放到叶子节点而不是内层节点，一旦放到内层节点，磁盘块的数据项会大幅度下降，导致树增高。当数据项等于1时将会退化成线性表。
2. **索引的最左匹配特性**：当b+树的数据项是复合的数据结构，比如(name,age,sex)的时候，b+树是按照从左到右的顺序来建立搜索树的，比如当(张三,20,F)这样的数据来检索的时候，b+树会优先比较name来确定下一步的所搜方向，如果name相同再依次比较age和sex，最后得到检索的数据；但当(20,F)这样的没有name的数据来的时候，b+树就不知道下一步该查哪个节点，因为建立搜索树的时候name就是第一个比较因子，必须要先根据name来搜索才能知道下一步去哪里查询。比如当(张三,F)这样的数据来检索时，b+树可以用name来指定搜索方向，但下一个字段age的缺失，所以只能把名字等于张三的数据都找到，然后再匹配性别是F的数据了， 这个是非常重要的性质，即索引的最左匹配特性。
3. **查询效率稳定:** 数据全都存在叶子节点上，在数据库中，b+树的高度又一般都在24层，这也就是说查找某一个键值的行记录时最多只需要2到4次IO，因为当前一般的机械硬盘每秒至少可以做100次IO，24次的IO意味着查询时间只需要0.02~0.04秒, 所以随机访问时，io此时都是一样的，时间也都差不多。
4. **排序能力强:** b+tree的叶子节点形成了一个有序双向链表, 在范围查询的时候，会沿着链表扫描数据，不会再返回上一层节点。

mysql中常用索引：

**普通索引**: 最基本的索引，它没有任何限制。

**唯一索引：**

- 唯一索引: 与前面的普通索引类似，不同的就是：索引列的值必须唯一，但允许有空值。如果是组合索引，则列值的组合必须唯一。
- 主键索引： 特殊的唯一索引，一个表只能有一个主键，不允许有空值, 一般是在建表的时候同时创建主键索引。

**联合索引：** 多个字段上创建的索引，只有在查询条件中使用了创建索引时的第一个字段，索引才会被使用，使用组合索引时遵循最左前缀集合。上述三种索引都支持使用联合索引，及联合主键索引，联合唯一索引，联合普通索引。

创建删除索引

```mysql
Copy# 方法一：创建表时
CREATE TABLE 表名 (
     字段名1  数据类型 [完整性约束条件…],
     字段名2  数据类型 [完整性约束条件…],
     [UNIQUE | FULLTEXT | SPATIAL ]   INDEX | KEY
     [索引名]  (字段名[(长度)]  [ASC |DESC]) 
);

# 方法二：CREATE在已存在的表上创建索引
CREATE  [UNIQUE | FULLTEXT | SPATIAL ]  INDEX  索引名 
      ON 表名 (字段名[(长度)]  [ASC |DESC]) ;

# 方法三：ALTER TABLE在已存在的表上创建索引
ALTER TABLE 表名 ADD  [UNIQUE | FULLTEXT | SPATIAL ] INDEX
     索引名 (字段名[(长度)]  [ASC |DESC]) ;
                             
# 删除索引：
DROP INDEX [索引名] ON 表名字;

# 查看索引：
SHOW INDEX FROM table_name\G
```

使用索引的情况

- 主键自动建立唯一索引
- 频繁作为查询的条件的字段应该创建索引
- 查询中与其他表关联的字段，外键关系建立索引
- 单间/组合索引的选择问题（在高并发下倾向创建组合索引）
- 查询中排序的字段，排序字段若通过索引去访问将大大提高排序的速度
- 查询中统计或者分组字段

不使用索引的情况：

- 表记录太少
- 经常增删改的表,频繁更新的字段不适合创建索引(因为每次更新不单单是更新了记录还会更新索引，加重IO负担)
- Where条件里用不到的字段不创建索引
- 数据重复且分布平均的表字段，因此应该只为经常查询和经常排序的数据列建立索引。
  注意，如果某个数据列包含许多重复的内容，为它建立索引就没有太大的实际效果。

## 读写锁

在处理并发或者写时，可以通过实现一个由两种类型的锁组成的锁系统来解决问题。这两种类型的锁通常被称为**共享锁（shared lock）**和**排他锁（exclusive lock）**，也叫**读锁（read lock）**和**写锁（write lock）**。

- 读锁：是共享的，也就是相互不阻塞的。多个客户端在同一时刻可以同时读取同一个资源，而互不干扰。
- 写锁：是排他的，也就是一个写锁会阻塞其他的写锁和读锁，确保在给定的时间里，只有一个用户能执行写入，并防止其他用户读取正在写入的同一资源。

## 锁粒度

一种提高共享资源并发性的方式就是让锁定对象更有选择性。尽量只锁定需要修改的部分数据，而不是所有的资源。更理想的方式是：只对会修改的数据片进行精确的锁定。下面是两种最重要的锁策略：**表锁和行级锁。**

### 表锁

**表锁（table lock）是MySQL中最基本的锁策略，并且是开销最小的策略：它会锁定整张表。**

在特定场景表锁可能有良好的性能。另外，**写锁也比读锁有更高的优先级**，因此一个写锁请求可能会被插入到读锁列表的前面，反之读锁则不能插入到写锁的前面。

尽管存储引擎可以管理自己的锁，MySQL本身还是会使用各种有效的表锁来实现不同的目的。例如服务器会为ALTER TABLE之类的语句使用表锁，而忽略存储引擎的锁机制。

### 行级锁

行级锁可以最大程度的支持并发，同时开销也最大。

行级锁只在存储引擎层实现，而MySQL服务层没有实现。

## 事务

**事务就是一组原子性的SQL查询，或者说一个独立的工作单元。**如果数据库引擎能够成功的对数据库应用该组查询的全部语句，那么就执行该组查询。如果其中有任何一条语句因为崩溃或其他原因无法执行，那么所有的语句都不会执行。也就是说，**事务内的语句，要么全部执行成功，要么全部执行失败。**

可以同`START TRANSACTION`开始一个事务，然后要么用`COMMIT`提交事务将修改的数据持久保留，要么使用`ROLLBACK`撤销所有的修改。

**数据库的特性有四条，简称为ACID，分别是：原子性（atomicity）、一致性（consistency）、隔离性（isolated）和持久性（durability）。**

- **原子性（atomicity）**：**一个事务必须被视为一个不可分割的最小工作单元**，整个事务操作要么全部提交成功，要么全部失败回滚，对于一个事务来说，不可能只执行其中一部分操作，这就是事务的原子性。
- **一致性（consistency）**：**数据库总是从一个一致性的状态转换到另一个一致性的状态。**
- **隔离性（isolated）**：**通常来说，一个事务所做的修改在最终提交以前，对其他事务是不可见的。**这里的“通常来说”与隔离级别有关。
- **持久性（durability）**：**一旦事务提交，则其所做的修改就会永久保存到数据库中。**此时即使系统崩溃，修改的数据也不会丢失。

### 隔离级别

较低的隔离级别通常可以执行更高的并发，系统的开销也更低

SQL的隔离级别：

READ UNCOMMITTED（未提交读，脏读）

* 事务中的修改，即使没有提交，对其他事务也是可见的，事务可以读取未提交的数据，也被称为"脏读"

READ COMMITTED（提交读）**由数据引起的**

* 大多数的数据库系统默认隔离级别都是READ COMMITTED（但mysql并不是）一个事务从开始直到提交之前，所做的任何修改对其他事务都是不可见的。这个级别有时候也叫做"不可重复读"，因为两次执行同样的查询，可能会得到不一样的结果

REPEATABLE READ （可重复读）**幻读是由插入和删除’行‘引起的**

* 解决了脏读的问题，该级别保证了在同一事务中多次读取同一记录的结果是一致的，但是理论上，可重复读还是无法解决另外一个幻读（Phantom READ），所谓幻读：指的是当某个事物在读取某个范围内的记录，另外一个事务又在该范围内插入了新的记录，当之前事物再次读取该范围的记录时，会产生幻行（Phantom Row）InnoDB和XtraDB存储引擎通过多版本并发控制（MVCC）解决了幻读的问题，**可重复读是MySQL的默认事务隔离级别**

SERIALIZABLE（可串行化）

* 这是最高的隔离级别，他通过强制事务串行执行，可以避免幻读问题，简单来说SERIALIZABLE会在读取的每一行数据都加上锁，所以会导致大量的超时和锁争用问题。实际应用中也很少用到这个隔离级别，只有在非常需要确保数据一致性而且可以接受没有并发的情况下，才考虑采用该级别

![image-20201221183440796](/typora-user-images/image-20201221183440796.png)

### 死锁

是指两个或多个事务在同一资源互相占用，并请求锁定对方占用的资源，从而导致恶性循环的现象。当多个事务试图以不同的顺序锁定资源时，就可能会产生死锁。多个事务同时锁定同一资源时，也会产生死锁。

InnoDB目前处理死锁的方法是：将持有最少行级排它锁的事务进行回滚。

### 事务日志

事务日志可以帮助提高事务效率。使用事务日志，存储引擎在修改表的数据时只需要修改其内存拷贝，再把修改行为记录到持久在硬盘上的事务日志中，而不用每次都将修改的数据本身持久到磁盘。事务日志采用的方式是追加的方式，因此写日志操作是磁盘上一小块区域内的顺序IO，而不像随机IO需要在磁盘多个地方移动磁头，所以采用事务日志的方式相对来说要快得多。事务日志持久后，内存中被修改的数据在后台可以慢慢的刷回磁盘。通常称之为预写式日志。

需要修改两次磁盘

​    1.先写入日志文件中

​		写入日志文件中的仅仅是操作过程,而不是操作数据本身,所以速度比写数据库文件速度要快很多.

​	2.然后再写入数据库文件中

​		写入数据库文件的操作是重做事务日志中已提交的事务操作的记录.

### 显式锁定

显式锁定：MySQL也支持LOCK TABLES和UNLOCK TABLES语句，这是在服务器层实现的，但它们不能代替事务处理，如果应用到事务，还是应该选择事务型存储引擎。 （建议：除了在事务中禁用了AUTOCOMMIT时可以使用LOCK TABLE之外，其他任何时候都不要显式地执行LOCK TABLES，不管使用的是什么存储引擎，因为LOCK TABLE和事务之间相互影响时，问题会变得非常复杂）

InnoDB也支持通过特定的语句进行显式锁定，这些语句不属于SQL规范，如： SELECT … LOCK IN SHARE MODE SELECT … FOR UPDATE

上述两条语句详解

https://blog.csdn.net/cug_jiang126com/article/details/50544728

数据库性能分析工具

https://www.php.cn/mysql-tutorials-357655.html

https://cloud.tencent.com/developer/article/1582640

### 多版本并发控制

MySQL的大多数事务型存储引擎实现的都不是简单的行级锁，基于提升并发性能的考虑，它们一般都同时实现了**多版本并发控制（MVCC）**。可以认为MVCC是行级锁的一个变种，但是它在很多情况下避免了加锁操作，因此开销更低。虽然实现机制有可能不同，但大都**实现了非阻塞的读操作，写操作也只锁定必要的行。**

**MVCC的实现是通过保存数据在某个时间点的快照来实现的。**也就是说，不管需要执行多长时间，每个事务看到的数据都是一致的。根据事务开始的时间不同，每个事务对同一张表，同一时刻看到的数据可能是不一样的。

**InnoDB的MVCC是通过在每行记录后面保存两个隐藏的列来实现的。这两个列，一个保存了行的创建时间，一个保存行的过期时间（或删除时间）。当然存储的不是实际的时间值，而是系统版本号。每开始一个新的事务，系统版本号都会递增。事务开始时刻的系统版本号会作为事务的版本号，用来和查询到的每行记录的版本号进行比较。**

REPEATABLE READ隔离级别下MVCC是如何操作的

**SELECT**：InnoDB会根据以下两个条件检查每行记录：

> - **InnoDB只查找早于当前事务版本的数据行**（即行的系统版本号小于或等于事务的系统版本号），这样可以确保读取的行要么在事务开始前已经存在，要么是事务自身插入或者修改过的。
> - **行的删除版本要么未定义，要么大于当前事务版本号。**这样可以确保读取的行在事务开始之前未被删除。
>
> 只有符合上述两个条件的记录，才能作为查询结果。

**INSERT**：InnoDB为新插入的每一行保存当前系统版本号作为行版本号。

**DELETE**：InnoDB为删除的每一行保存当前系统版本号作为行删除标识。

**UPDATE**：**InnoDB为插入一行新记录，**保存当前系统版本号作为行版本号，**同时保存当前系统版本号到原来的行作为行删除标识。**

**保存这两个额外系统版本号，优点就是使大多数读操作都可以不用加锁，读数据操作很简单，性能很好，并且也能保证只会读取到符合标准的行。缺点就是每行记录都需要额外的存储空间，需要做更多的行检查工作，以及一些额外的维护工作。**

**MVCC只在 REPEATABLE READ（可重复读）和READ COMMITTED（提交读）两个隔离级别下工作**，因为READ UNCOMMITTED总是读取最新的数据行，而SERIALIZABLE则会对所有读取的行加锁。

## MySQL的存储引擎

可以使用`SHOW TABLE STATUS` 命令显示表的相关信息。例如：

```sql
mysql> show table status like 'user' \G
*************************** 1. row ***************************
           Name: user	//表名
         Engine: InnoDB	//表引擎
        Version: 10	//版本
     Row_format: Dynamic//行的格式。Dynamic的行长度是可变的，一般包含可变长度的字段，如VARCHAR
			    //Fixed的行长度则是固定的，只包含固定长度的列，如CHAR。Compressed的行只在压缩表中存在。
           Rows: 5	//表中的行数，MyISAM和其他一些存储引擎是精确值，但InnoDB是估计值。
 Avg_row_length: 59	//平均每行包含的字节数
    Data_length: 16384	//表数据的大小（字节）
Max_data_length: 0	//表的最大数据容量，该值与存储引擎关
   Index_length: 0	//索引的大小
      Data_free: 0	//对于MyISAM表，表示已分配但目前没有使用的空间
 Auto_increment: 6	//下一个AUTO_INCREMENT的值
    Create_time: 2018-11-19 20:58:21	//表的创建时间
    Update_time: NULL	//表数据的最后修改时间
     Check_time: NULL	//使用CHECK TABLE命令或者myisamchk工具最后一次检查表的时间
      Collation: utf8_general_ci	//表的默认字符集和字符列排序规则
       Checksum: NULL	//如果启用，保存的是整个表的实时校验和
 Create_options:	//创建表时指定的其他选项
        Comment:	//建表时的备注
1 row in set (0.00 sec)
```

## InnoDB存储引擎

InnoDB是MySQL的默认事务型引擎，也是最重要、使用最广泛的存储引擎。它被设计用来处理大量的短期事务，短期事务大部分情况是正常提交的，很少会回滚。InnoDB的性能和自动崩溃恢复特性，使得它在非事务型存储的需求中也很流行。

InnoDB的数据存储在表空间中，表空间是由InnoDB管理的一个黑盒子，有一系列的数据文件组成，InnoDB可以将每个表的数据和索引存放在单独的文件中。

InnoDB采用MVCC来支持高并发，并且实现了四个标准的隔离级别，默认为REPATABLE READ，并且通过**间隙锁（next-key locking）**策略防止幻读的出现。间隙锁使得InnoDB不仅仅锁定查询涉及的行，还会对索引中的间隙进行锁定，以防止幻影行的插入。

InnoDB表是基于聚簇索引建立的，聚簇索引对主键查询有很高的性能，不过它的二级索引中必须包含主键列，所以如果主键列很大的话，其他的所有索引都会很大。因此若表上的索引较多的话，主键应当尽可能的小。InnoDB是平台独立的，可以将数据和索引文件在平台之间拷贝。

InnoDB内部做了很多优化，包括从磁盘读取数据时采用的可预测性预读，能够自动在内存中创建hash索引以及加速读操作的自适应哈希索引，以及能够加速插入操作的插入缓冲区等。

## MyISAM存储引擎

MyISAM是MySQL5.1之前的默认存储引擎。MyISAM提供了大量的特性，包括全文索引、压缩、空间函数等，但MyISAM不支持事务和行级锁，而且有一个**最大的缺陷就是崩溃后无法安全恢复。**

### 存储

MyISAM会将表存储在两个文件中：**数据文件和索引文件**，分别以.MYD和.MYI为扩展名。MyISAM表可以包含动态或者静态（长度固定）行。MySQL会根据表的定义来决定采用何种行格式。MyISAM表可以存储的行记录数，一般**受限于可用的磁盘空间，或者操作系统中单个文件的最大尺寸。**

在MySQL5.0中，MyISAM表如果是可变行，则默认配置只能处理256TB的数据，可以通过修改表单`MAX_ROWS`和`AVG_ROW_lENGTH`选项的值来实现，两者相乘就是表可能达到的最大大小。修改这两个参数会导致重建整个表和表的所有索引，这可能需要很长的时间才能完成。

### MyISAM特性

- **加锁与并发**：**MyISAM对整张表加锁**，而不是针对行。**读取时会对需要读到的表加共享锁，写入时则对表加排他锁。但是在表有读取查询的同时，也可以往表中插入新的记录（也被称为并发插入）**。
- **修复**：对于MyISAM表，**MySQL可以手工或者自动检查和修复操作。**执行表的修复可能导致一些数据的丢失，而且修复操作是非常慢的。可以通过`CHECK TABLE mytable`检查表的错误，如果有错误可以通过执行`REPAIR TABLE mytable`来修复，或者使用myisamchk命令行工具也可以。
- **索引特性**：对于MyISAM表，即使是BLOB和TEXT等长字段，也可以基于其前500个字符创建索引。MyISAM也支持全文索引，这是一种基于分词创建的索引。
- **延迟更新索引键（Delayed Key Write）**：如果在创建MyISAM表时指定了`DELAY_KEY_WRITE`选项，在每次修改执行完时，不会立刻将修改的索引数据写入磁盘，而是会写到内存中的**键缓冲区**，只有在清理键缓冲区或者关闭表的时候才会将对应的索引块写入到磁盘。这种方式可以极大的提升写入性能。

### MyISAM压缩表

如果表在创建并导入以后，不会再进行修改操作，那么这样得表更适合采用MyISAM压缩表。

可以使用myisampack对MyISAM表进行压缩。**压缩表是不能进行修改的（除非先将表解压、修改数据、然后再次压缩）**。压缩表可以极大的减少磁盘空间占用，减少磁盘I/O，从而提升查询性能。压缩表也支持索引，但索引也是只读的。

### MyIASM性能

MyISAM引擎设计简单，数据以紧密格式存储，默认在写场景下的性能很好。MyISAM有一些服务器级别的性能扩展限制，比如对索引键缓冲区（key cache）的Mutex锁，MariaDB基于段（segment）的索引键缓冲区机制来避免该问题。但MyISAM最典型的性能问题还是表锁的问题，如果你发现所有的查询都长期处于Locked状态，那么毫无疑问就是表锁的问题。

## 选择合适的引擎

对于如何选择存储引擎，可以简单的归纳为一句话：**除非需要用到某些InnoDB不具备的特性，并且没有其他办法可以代替，否则都应该优先选择InnoDB引擎**。

**除非万不得已，否则建议不要混合使用多种存储引擎，佛祖额可能带来一系列复杂的问题，以及一些潜在的bug和边界问题。**

如果需要不同的存储引擎，应先考虑以下几个因素：

- 事务：如果应用到事务，那么InnoDB是目前最稳定并且经过沿着轨道选择。如果不需要事务，并且主要是SELECT和INSERT操作，那么MyISAM是不错的选择。
- 备份：如果需要在线热备份，那么选择InnoDB就是基本的要求。
- 崩溃恢复：建议选择InnoDB，因为拥有自动恢复功能。
- 特有的特性：有些应用可能依赖一些存储引擎所独有的特性或者优化，比如聚簇索引的优化，应该综合各种情况考虑，选择满足特殊情况下最优的引擎。

## 转换表的引擎

转换表的引擎有三种方法：

- **ALTER TABLE**：将表从一个引擎修改为另一个引擎最简单的办法是使用`ALTER TABLE`语句。例如：`ALTER TABLE mytable ENGINE = InnoDB`。**这种语法是用于任何存储引擎，但是有一个缺点：需要执行很长时间。**MySQL会按照行将数据从原表复制到一张新的表，在复制期间可能会消耗系统所有的I/O能力，同时原表会加上锁。**同时如果转换表的存储引擎，将会失去和引擎相关的所有特性.**

- **导出与导入**：可以使用mysqldump工具将数据导出到文件，然后修改文件中`CREATE TABLE`语句的存储引擎选项。

- 创建与查询

  ：这种方法不需要导出整个表，而是先创建一个新的存储引擎的表，然后利用INSERT–SELECT语法来导数据。

  ```
  CREATE TABLE innodb_table LIKE myisam_table;
  ALTER TABLE innodb_table ENGINE = InnoDB;
  INSERT INTO innodb_table (SELECT * FROM myisam_table);
  ```

当然如果数据量很大的情况下，可以采用分批导入的方法。











# spring揭秘

创建对象以前通过程序员自己去new现在交由spring创建对象

简单来说你需要出门（被注入）首先需要穿衣服，衣服（依赖对象）通常情况下你需要自己去穿衣服，现在这个情况下你只需要一个眼神，你的另一半回到衣柜取出衣服给你穿上你就可以出门了。

在IoC的场景中，二者之间通过IoC Service Provider（例子中你的另一半）来打交道，素有的被注入对象和依赖对象现在由IoC Service Provider统一管理。被注入对象需要什么，直接跟IoC Service Provider招呼一声，后者就会把相应的被依赖对象注入到被注入对象中。

## 依赖注入的方式

**IoC有三种依赖注入的方式，即构造方法注入、setter方法注入以及接口注入。**

### 构造方法注入

构造方法注入，就是**被注入对象可以通过在其构造方法中声明依赖对象的参数列表，让外部（通常是Ioc容器）知道它需要哪些依赖对象**，例子如下：

```java
public FXNewsProvider(IFXNewsListener newsListner,IFXNewsPersister newsPersister{  
	this.newsListener = newsListner;  
	this.newPersistener = newsPersister; 
}
```

IoC Service Provider会检查被注入对象的构造方法，取得它所需要的依赖对象列表，进而为其注入相应的对象。**同一个对象是不可能被构造两次的，因此，被注入对象的构造乃至整个生命周期，应该是由IoC Service Provider来管理的。**

### setter方法注入

对应Java Bean对象来说，通过setter方法，可以更改相应的对象属性；通过getter方法，可以获得相应属性的状态。所以当前对象只要为其依赖对象所对应的属性添加setter方法，就可以**通过setter方法将相对应的依赖对象设置到被注入对象中。**。例子如下：

```java
public class FXNewsProvider {  

	private IFXNewsListener  newsListener;  
	private IFXNewsPersister newPersistener;    
	
	public IFXNewsListener getNewsListener() {   
		return newsListener;  
	}  
	public void setNewsListener(IFXNewsListener newsListener) {   
		this.newsListener = newsListener;  
	}  
	public IFXNewsPersister getNewPersistener() {   
		return newPersistener;  
	}  
	public void setNewPersistener(IFXNewsPersister newPersistener) {   
		this.newPersistener = newPersistener;  
	} 
}
```

这样外界就可以通过调用setNewsListener和setNewPersistener方法为FXNewsProvider对象注入所依赖的对象了。

**setter方法注入相对更宽松一些，可以在对象构造完成后在注入。**

### 接口注入

相对于前两种方式来说，接口注入就没有那么简单明了了。**被注入对象如果想要IoC Service Provider为其注入依赖对象，就必须实现某个接口。这个接口提供一个方法，用来为其注入依赖对象。**

![image-20210119153945247](/typora-user-images/image-20210119153945247.png)

如上图，FXNewsProvider为了让IoC Service Provider为其注入所依赖的IFXNewsListener，首先需要实现IFXNewsListenerCallable接口，这个接口会声明一个injectNewsListner方法（方法名随意），该方法的参数，就是所依赖对象的类型。这样，InjectionServiceContainer对象，即对应IoC Service Provider就可以通过这个接口方法将依赖对象注入到被注入对象FXNewsProvider中。

相对于前两种依赖注入方式，**接口注入比较死板和繁琐。如果需要注入依赖对象，被注入对象就必须声明和实现另外的接口。**

### 三种注入方式比较

- **接口注入**：从使用上来说，**接口注入是现在不提倡的一种方式，因为它强制被注入对象实现不必要的接口，带有侵入性。**而构造方法注入和setter方法注入则不需要如此。
- **构造方法注入**：这种注入方法的**优点就是，对象在构造完后，即进入就绪状态，可以马上使用。缺点就是，当依赖对象比较多的时候，构造方法的参数列表会比较长。而通过反射构造对象的时候，对相同类型的参数的处理会比较困难，维护和使用上也比较麻烦。而且在Java方法中，构造方法无法被继承，无法设置默认值。对于非必须的依赖处理，可能需要引入多个构造方法，而参数列表的变动可能造成维护上的不便。**
- **setter方法注入**：因为方法可以命名，所以setter方法注入在描述上要比构造方法注入好一些。另外**setter方法可以被继承，允许设置默认值，而且有良好IDE支持。缺点就是对象无法再构造完成马上进入就绪状态。**

## IoC Service Provider简介

Ioc Service Provider是一个抽象出来的概念，它可以指代任何将IoC场景中的业务对象绑定到一起的实现方式。它可以是一段代码，也可以是一组相关的类，甚至可以是比较通用的IoC框架或者IoC容器。

### Ioc Service Provider的职责

IoC Service Provider的职责相对来说比较简单，主要有两个：**业务对象的构建管理和业务对象间的依赖绑定。**

- **业务对象的构建管理**：在IoC场景中，业务对象无需关心所依赖的对象如何构建如何取得，但这部分工作始终需要有人来做，所以IoC Service Provider需要将对象的构建逻辑从客户端对象那里剥离出来，以免这部分逻辑污染业务对象的实现。
- **业务对象间的绑定**：Ioc Service Provider通过结合之前构建和管理的所有业务对象，以及各个业务对象间可以识别的依赖关系，**将这些对象所依赖的对象注入绑定，从而保证每个业务对象在使用的时候，可以处于就绪状态。**

### IoC Service Provider如何管理对象间的依赖关系

介绍归纳当前流行的IoC Service Provider产品使用的注册对象管理信息的方式主要有以下几种。

### 直接编码方式

当前大部分的IoC容器都应该支持直接编码方式，包括Spring。**在容器启动之前，我们就可以通过程序编码的方式将被注入对象和依赖对象注册到容器中，并明确它们相互之间的依赖注入关系。**下面代码演示了这样的一个过程：

```java
IoContainer container = ...;
container.register(FXNewsProvider.class,new FXNewsProvider());
container.register(IFXNewsListener.class,new DowJonesNewsListener());
... 

FXNewsProvider newsProvider = (FXNewsProvider)container.get(FXNewsProvider.class); 
newProvider.getAndPersistNews();
```

通过为相应的类指定对应的具体实例，可以告知IoC容器，当我们要这种类型的对象实例时，请将容器中注册的、对应的那个具体实例返回给我们。

如果是接口注入方式，可能伪代码要多一些，但是本质的道理上是一样的。

```java
IoContainer container = ...;
container.register(FXNewsProvider.class,new FXNewsProvider()); 
container.register(IFXNewsListener.class,new DowJonesNewsListener());
... 

container.bind(IFXNewsListenerCallable.class, container.get(IFXNewsListener.class));
... 

FXNewsProvider newsProvider = (FXNewsProvider)container.get(FXNewsProvider.class);  
newProvider.getAndPersistNews();
```

通过bind方法将“被注入对象”所依赖的对象，绑定为容器中注册过的IFXNewsListener类型的对象实例。

### 配置文件方式

这是一种较为普遍的依赖注入关系管理方式，像properties文件、XML文件等，都可以成为管理依赖注入关系的载体，其中最为常见的就是XML文件的方式。代码例子如下：

```xml
<bean id="newsProvider" class="..FXNewsProvider">
	<property name="newsListener">   
		<ref bean="djNewsListener"/>  
	</property>  
	<property name="newPersistener">   
		<ref bean="djNewsPersister"/>  
	</property> 
</bean> 
 
<bean id="djNewsListener"   class="..impl.DowJonesNewsListener"> </bean> 

<bean id="djNewsPersister"   class="..impl.DowJonesNewsPersister"> </bean>
```

最后，在代码中通过“newProvider”这个名字，即可从容器中取得已经组装好的FXNewsProvider并直接使用。

```java
... 
container.readConfigurationFiles(...); 
FXNewsProvider newsProvider = (FXNewsProvider)container.getBean("newsProvider"); 
newsProvider.getAndPersistNews();
```

### 元数据方式 略

## IoC容器之BeanFactory

Spring提供了两种容器类型：**BeanFactory**和**ApplicationContext**。

- **BeanFactory**：基础类型的Ioc容器，提供完整的IoC服务支持。**如果没有特殊指定，默认采用延迟初始化策略。即只有当客户端对象需要访问容器中的某个受管对象时，才对该受管对象进行初始化以及依赖注入操作。**所以相对来说，该容器启动初期速度较快，所需要的资源有限，因此BeanFactroy比较适合用于资源有限，并且功能要求不是很严格的场景。
- **ApplicationContext**：**ApplicationContext是在BeanFactory的基础上构建的**，是相对于高级的容器实现，因此ApplicationContext还提供了一些例如事件发布、国际化信息支持的高级特性。**ApplicationContext所管理的对象，在该容器启动之后，默认全部初始化并绑定完成。**所以相对BeanFactory来说，ApplicationContext要求更多的系统资源，同时启动时间也会长一些。因此ApplicationContext更适合于资源充足并且要求更多功能的场景中。

通过下图可以对BeanFactory和ApplicationContext之间的关系有一个更清晰的认知。

![image-20210119174606328](/typora-user-images/image-20210119174606328.png)

BeanFactory，顾名思义就是生产Bean的工厂，而Spring提倡使用POJO，所以可以把每个业务对象看作一个JavaBean对象。BeanFactory可以完成作为IoC Service Provider的所有职责，包括业务对象的注册和对象间依赖关系的绑定。

BeanFactory就像一个汽车生产厂。你从其他汽车零件厂商或者自己的零件生产部门取得汽车零件送入这个汽车生产厂，最后只需要从生产线的终点取得汽车成品就可以了。相似的，将应用程序所需的所有业务对象交给BeanFactory之后，剩下的就是直接从BeanFactory取得最终组装完成并且可用的对象。至于这个最终业务对象如何组装，你不需要担心，BeanFactory会帮你搞定。

## 拥有BeanFactory之后的生活

确切的说，拥有BeanFactory之后的生活没有太大的变化，有变化的也在只是一些拉拉扯扯的事情，客观一点就是对象之间依赖关系的解决方式改变了。之前系统业务对象需要自己去“拉”所依赖的业务对象，有了BeanFactory之类的IoC之后，需要依赖什么，让BeanFactory为我们推过来就行啦。以一个FX新闻系统的例子来说，FX新闻应用设计和实现框架代码如下：

```java
1-设计FXNewsProvider类用于普遍的新闻处理 
public class FXNewsProvider {  ... }   
 
2-设计IFXNewsListener接口抽象各个新闻社不同的新闻获取方式，并给出相应实现类 
public interface IFXNewsListener {  ... } 
以及 
public class DowJonesNewsListener implements IFXNewsListener{  ... } 


3-设计IFXNewsPersister接口抽象不同数据访问方式，并实现相应的实现类 
public interface IFXNewsPersister {  ...  }   
以及 
public class DowJonesNewsPersister implements IFXNewsPersister {  ... }
```

BeanFactory会说，这些都让我来干吧！此时BeanFactory说这些事情让他来做，但是它并不知道该怎么做，所以就需要你来教它怎么做。通常情况下，会使用XML文件配置的方式来教它，也就是告诉BeanFactory如何来注册并管理各个业务对象之间的依赖关系。下面是BeanFactory的XML配置方式实现业务对象间的依赖关系的一个例子代码：

```xml
<beans>   
	<bean id="djNewsProvider" class="..FXNewsProvider">   
		<constructor-arg index="0">    
			<ref bean="djNewsListener"/>   
		</constructor-arg>    
		<constructor-arg index="1">    
			<ref bean="djNewsPersister"/>   
		</constructor-arg>   
	</bean>  
	... 
</beans>
```

在BeanFactory出现之前，我们通常会直接在应用程序的入口类的main方法中，自己动手实例化相应的对象并调用，如以下代码：

```java
FXNewsProvider newsProvider = new FXNewsProvider();  
newsProvider.getAndPersistNews();
```

不过现在有了BeanFactory，通过xml文件这张“图纸”告诉BeanFactory怎么做之后，让BeanFactory为我们生产一个FXNewsProvider，如以下代码所示：

```java
BeanFactory container = new XmlBeanFactory(new ClassPathResource("配置文件路径")); 
FXNewsProvider newsProvider = (FXNewsProvider)container.getBean("djNewsProvider"); 
newsProvider.getAndPersistNews();

/*或者如以下代码所示*/ 
ApplicationContext container = new ClassPathXmlApplicationContext("配置文件路径"); 
FXNewsProvider newsProvider = (FXNewsProvider)container.getBean("djNewsProvider"); 
newsProvider.getAndPersistNews(); 

/*也可以如以下代码所示*/
ApplicationContext container = new FileSystemXmlApplicationContext("配置文件路径"); 
FXNewsProvider newsProvider = (FXNewsProvider)container.getBean("djNewsProvider"); 
newsProvider.getAndPersistNewimage-20210119174606328s();
```

## BeanFactory的对象注册与依赖绑定方式

为了让BeanFactory能够明确管理各个业务对象以及业务对象之间的依赖绑定关系，同样需要需要某种途径来记录和管理这些信息。与上一章的IoC Service Provider提到的三种方式，BeanFactory几乎支持所有这些方式。

### 直接编码方式

先看一下使用直接编码方式FX新闻系统相关类是如何注册并绑定的：

```java
public static void main(String[] args)
{
 DefaultListableBeanFactory beanRegistry = new DefaultListableBeanFactory();
 BeanFactory container = (BeanFactory)bindViaCode(beanRegistry);
 FXNewsProvider newsProvider = ➥
 (FXNewsProvider)container.getBean("djNewsProvider");
 newsProvider.getAndPersistNews();
}
public static BeanFactory bindViaCode(BeanDefinitionRegistry registry)
{
 AbstractBeanDefinition newsProvider = ➥
 new RootBeanDefinition(FXNewsProvider.class,true);
 AbstractBeanDefinition newsListener = ➥
 new RootBeanDefinition(DowJonesNewsListener.class,true);
 AbstractBeanDefinition newsPersister = ➥
 new RootBeanDefinition(DowJonesNewsPersister.class,true);
 // 将bean定义注册到容器中
 registry.registerBeanDefinition("djNewsProvider", newsProvider);
 registry.registerBeanDefinition("djListener", newsListener);
 registry.registerBeanDefinition("djPersister", newsPersister);
 // 指定依赖关系
 // 1. 可以通过构造方法注入方式
 ConstructorArgumentValues argValues = new ConstructorArgumentValues();
 argValues.addIndexedArgumentValue(0, newsListener);
 argValues.addIndexedArgumentValue(1, newsPersister);
 newsProvider.setConstructorArgumentValues(argValues);
 // 2. 或者通过setter方法注入方式
 MutablePropertyValues propertyValues = new MutablePropertyValues(); 
 propertyValues.addPropertyValue(new ropertyValue("newsListener",newsListener)); 
 propertyValues.addPropertyValue(new PropertyValue("newPersistener",newsPersister)); 
 newsProvider.setPropertyValues(propertyValues); 
 // 绑定完成
 return (BeanFactory)registry; 
}
```

​	BeanFactory只是一个接口，我们最终需要一个该接口的实现来进行实际的Bean的管理，DefaultListableBeanFactory就是这么一个比较通用的BeanFactory实现类。DefaultListableBeanFactory除了间接地实现了BeanFactory接口，还实现了BeanDefinitionRegistry接口，该接口才是在BeanFactory的实现中担当Bean注册管理的角色。基本上，BeanFactory接口只定义如何访问容器内管理的Bean的方法，各个BeanFactory的具体实现类负责具体Bean的注册以及管理工作。

​	BeanDefinitionRegistry接口定义抽象了Bean的注册逻辑。通常情况下，具体的BeanFactory实现类会实现这个接口来管理Bean的注册。它们之间的关系如图

![image-20210120120152235](/typora-user-images/image-20210120120152235.png)

​	打个比方说，BeanDefinitionRegistry就像图书馆的书架，所有的书是放在书架上的。虽然你还书或者借书都是跟图书馆（也就是BeanFactory，或许BookFactory可能更好些）打交道，但书架才是图书馆存放各类图书的地方。所以，书架相对于图书馆来说，就是它的“BookDefinitionRegistry”。每一个受管的对象，在容器中都会有一个BeanDefinition的实例（instance）与之相对应，该BeanDefinition的实例负责保存对象的所有必要信息，包括其对应的对象的class类型、是否是抽象类、构造方法参数以及其他属性等。当客户端向BeanFactory请求相应对象的时候，BeanFactory会通过这些信息为客户端返回一个完备可用的对象实例。RootBeanDefinition和ChildBeanDefinition是BeanDefinition的两个主要实现类。

现在，我们再来看这段绑定代码，应该就有“柳暗花明”的感觉了。

* 在 main 方法中，首先构造一个 DefaultListableBeanFactory 作为 BeanDefinitionRegistry，然后将其交给bindViaCode方法进行具体的对象注册和相关依赖管理，然后通过bindViaCode返回的BeanFactory取得需要的对象，最后执行相应逻辑。在我们的实例里，当然就是取得FXNewsProvider进行新闻的处理。
* 在bindViaCode方法中，首先针对相应的业务对象构造与其相对应的BeanDefinition，使用了RootBeanDefinition 作为 BeanDefinition 的实现类。构造完成后，将这些BeanDefinition注册到通过方法参数传进来的BeanDefinitionRegistry中。之后，因为我们的FXNewsProvider是采用的构造方法注入，所以，需要通过ConstructorArgumentValues为其注入相关依赖。在这里为了同时说明setter方法注入，也同时展示了在Spring中如何使用代码实现setter方法注入。如果要运行这段代码，需要把setter方法注入部分的4行代码注释掉。最后，以BeanFactory的形式返回已经注册并绑定了所有相关业务对象的BeanDefinitionRegistry实例。

总结一下就是

1. 在main方法中，首先构造一个DefaultListableBeanFactory作为BeanDefinitionRegistry，然后将其交给bindViaCode方式进行具体的对象注册和相关逻辑管理，然后就可以通过该方法返回的BeanFactory取得需要的对象。
2. 在bindViaCode方法中，首先针对相应的业务对象构造与其相对应的BeanDefinition，使用了RootBeanDefinition作为BeanDefinition的实现类。构造完成后，将这些BeanDefinite注册到通过方法参数传进来的BeanDefinitionRegistry中。
3. 之后我们可以通过构造方法，或者setter方法，为其注入相关依赖。最后以BeanFactory的形式返回已经注册并绑定了所有相关业务对象的BeanDefinitionRegistry实例。

### 外部配置文件方式

#### Properties配置格式的加载

​	Spring的IoC容器支持两种配置文件格式：Properties文件格式和XML文件格式。当然，如果你愿意也可以引入自己的文件格式，前提是真的需要。采用外部配置文件时，Spring的IoC容器有一个统一的处理方式。通常情况下，需要根据不同的外部配置文件格式，给出相应的BeanDefinitionReader实现类，由BeanDefinitionReader的相应实现类负责将相应的配置文件内容读取并映射到BeanDefinition，然后将映射后的BeanDefinition注册到一个BeanDefinitionRegistry，之后，BeanDefinitionRegistry即完成Bean的注册和加载。

​	当然，大部分工作，包括解析文件格式、装配BeanDefinition之类的工作，都是由BeanDefinitionReader的相应实现类来做的，BeanDefinitionRegistry只不过负责保管而已。整个过程类似于如下

代码：

```java
BeanDefinitionRegistry beanRegistry = <某个BeanDefinitionRegistry实现类，通常为➥
DefaultListableBeanFactory>; 
BeanDefinitionReader beanDefinitionReader = new BeanDefinitionReaderImpl(beanRegistry); 
beanDefinitionReader.loadBeanDefinitions("配置文件路径"); 
// 现在我们就取得了一个可用的BeanDefinitionRegistry实例
```

​	Spring提供了org.springframework.beans.factory.support.PropertiesBeanDefinitionReader类用于Properties格式配置文件的加载，所以，我们不用自己去实现BeanDefinitionReader，只要根据该类的读取规则，提供相应的配置文件即可。

```java
djNewsProvider.(class)=..FXNewsProvider 
djListener.(class)=..impl.DowJonesNewsListener 
djPersister.(class)=..impl.DowJonesNewsPersister 
# ----------通过构造方法注入的时候------------- 
djNewsProvider.$0(ref)=djListener 
djNewsProvider.$1(ref)=djPersister 
# ----------通过setter方法注入的时候---------  
djNewsProvider.newsListener(ref)=djListener  
djNewsProvider.newPersistener(ref)=djPersister
```

这些内容都是特定于Spring的PropertiesBeanDefinitionReader的，下面是简单的语法介绍：

* djNewsProvider作为beanName，后面通过.(class)表明对应的实现类是什么，实际上使用djNewsProvider.class=...的形式也是可以的,但是不提倡
* 通过在表示beanName的名称后添加.$[number]后缀的形式，来表示当前beanName对应的对象需要通过构造方法注入的方式注入相应依赖对象.$0和.$1后面的（ref）用来表示所依赖的是引用对象，而不是普通的类型。如果不加(ref)，PropertiesBeanDefinitionReader会将djListener和djPersister作为简单的String类型进行注入，异常自然不可避免啦。
* setter方法注入与构造方法注入最大的区别就是，它不使用数字顺序来指定注入的位置，而使用相应的属性名称来指定注入，同样也不要忘掉（ref）

当这些对象之间的注册和依赖注入信息都表达清楚之后，就可以将其加载到BeanFactory而付诸使用了。而这个加载过程实际上也就像我们之前总体上所阐述的那样

加载Properties配置的BeanFactory的使用

```java
public static void main(String[] args) 
{ 
 DefaultListableBeanFactory beanRegistry = new DefaultListableBeanFactory(); 
 BeanFactory container = (BeanFactory)bindViaPropertiesFile(beanRegistry); 
 FXNewsProvider newsProvider = (FXNewsProvider)container.getBean("djNewsProvider");
 newsProvider.getAndPersistNews();
}
public static BeanFactory bindViaPropertiesFile(BeanDefinitionRegistry registry)
{
PropertiesBeanDefinitionReader reader = new PropertiesBeanDefinitionReader(registry);
reader.loadBeanDefinitions("classpath:../../binding-config.properties");
return (BeanFactory)registry;
}
```

#### XML配置格式的加载

XML配置格式是Spring支持最完整，功能最强大，也是使用最频繁的表达方式。例子如下：

```xml
<?xml version="1.0" encoding="UTF-8"?> 
<!DOCTYPE beans PUBLIC "-//SPRING//DTD BEAN//EN" "http://www.springframework.org/dtd/spring-beans.dtd"> 
<beans>
	<bean id="djNewsProvider" class="..FXNewsProvider">
		<constructor-arg index="0">
			<ref bean="djNewsListener"/>
		</constructor-arg>
		<constructor-arg index="1">
			<ref bean="djNewsPersister"/>
		</constructor-arg>
	</bean>
	<bean id="djNewsListener" class="..impl.DowJonesNewsListener">
	</bean>
	<bean id="djNewsPersister" class="..impl.DowJonesNewsPersister">
	</bean>
</beans>
```

相对应的加载XML配置文件的BeanFactory的使用演示如下：

```java
public static void main(String[] args)  { 
	DefaultListableBeanFactory beanRegistry = new DefaultListableBeanFactory();  
	BeanFactory container = (BeanFactory)bindViaXMLFile(beanRegistry);  
	FXNewsProvider newsProvider = (FXNewsProvider)container.getBean("djNewsProvider"); 
	newsProvider.getAndPersistNews(); 
}
public static BeanFactory bindViaXMLFile(BeanDefinitionRegistry registry)  {  
	XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(registry);  
	reader.loadBeanDefinitions("classpath:../news-config.xml"); 
	return (BeanFactory)registry;  
	// 或者直接  
	//return new XmlBeanFactory(new ClassPathResource("../news-config.xml")); 
}
```

​	与为Properties配置文件格式提供PropertiesBeanDefinitionReader相对应，Spring同样为XML格式的配置文件提供了现成的BeanDefinitionReader实现，即XmlBeanDefinitionReader。XmlBeanDefinitionReader负责读取Spring指定格式的XML配置文件并解析，之后将解析后的文件内容映射到相应的BeanDefinition，并加载到相应的BeanDefinitionRegistry中（在这里是DefaultListableBeanFactory）。这时，整个BeanFactory就可以放给客户端使用了。除了提供XmlBeanDefinitionReader用于XML格式配置文件的加载，Spring还在DefaultListableBeanFactory的基础上构建了简化XML格式配置加载的XmlBeanFactory实现。从以上代码最后注释掉的一行，你可以看到使用了XmlBeanFactory之后，完成XML的加载和BeanFactory的初始化是多么简单。

#### 注解方式

​	注解是Java5之后才引入的，所以相对来说也更加简洁方便一些，使用注解的方式为FXNewsProvider注入所需要的依赖，现在可以使用@Autowired以及@Component对相关类进行标记。下面是FXNews使用注解标注后的情况。

```java
@Component 
public class FXNewsProvider  {  
	@Autowired  
	private IFXNewsListener  newsListener;  
	@Autowired  
	private IFXNewsPersister newPersistener;
  
	public FXNewsProvider(IFXNewsListener newsListner,IFXNewsPersister newsPersister)  { 
		this.newsListener   = newsListner;   
		this.newPersistener = newsPersister;  
	}  
	... 
} 
@Component 
public class DowJonesNewsListener implements IFXNewsListener  {  ... } 
@Component 
public class DowJonesNewsPersister implements IFXNewsPersister  {  ... }
```

**@Autowired是这里的主角，它的存在将告知Spring容器需要为当前对象注入哪些依赖对象。而@Component则是配合Spring中的classpath-scanning功能使用。现在我们只要再向Spring的配置文件中增加一个“触发器”，使用@Autowired和@Component标注的类就能获得依赖对象的注入了。**下面是在配置文件中配置的方法：

```xml
<?xml version="1.0" encoding="UTF-8"?> 

<beans xmlns="http://www.springframework.org/schema/beans" 
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
	xmlns:context="http://www.springframework.org/schema/context" 
	xmlns:tx="http://www.springframework.org/schema/tx" 
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
	http://www.springframework.org/schema/beans/spring-beans-2.5.xsd 
	http://www.springframework.org/schema/context 
	http://www.springframework.org/schema/context/spring-context-2.5.xsd 
	http://www.springframework.org/schema/tx 
	http://www.springframework.org/schema/tx/spring-tx-2.5.xsd"> 
	
	<context:component-scan base-package="cn.spring21.project.base.package"/> 
</beans>
```

**context:component-scan会到指定的包下面扫描有@Component的类，如果找到，则将他们添加到容器进行管理，并根据它们所标注的@Autowired为这些类注入符合条件的依赖对象。**所以在做完上面的工作之后，就可以使用了：

```java
public static void main(String[] args) {  
	ApplicationContext ctx = new ClassPathXmlApplicationContext("配置文件路径");  
	FXNewsProvider newsProvider = (FXNewsProvider)container.getBean("FXNewsProvider"); 
	newsProvider.getAndPersistNews(); 
}
```

**<beans>和<bean>**

所有使用 XML 文件进行配置信息加载的 Spring IoC 容器，包括 BeanFactory 和 ApplicationContext 的所有XML相应实现，都使用统一的XML格式。在Spring 2.0版本之前，这种格式由Spring提供的DTD规定，也就是说，所有的Spring容器加载的XML配置文件的头部，都需要以下形式的DOCTYPE声明：

```xml
<?xml version="1.0" encoding="UTF-8"?>  
<!DOCTYPE beans PUBLIC "-//SPRING//DTD BEAN//EN" ➥ "http://www.springframework.org/dtd/spring-beans.dtd"> 
<beans>  ... </beans> 
```

之后也可以使用XSD的方式进行声明

```xml
<beans xmlns="http://www.springframework.org/schema/beans" 
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
xmlns:util="http://www.springframework.org/schema/util" 
xmlns:jee="http://www.springframework.org/schema/jee"  
xmlns:lang="http://www.springframework.org/schema/lang"  
xmlns:aop="http://www.springframework.org/schema/aop"  
xmlns:tx="http://www.springframework.org/schema/tx"  
xsi:schemaLocation="http://www.springframework.org/schema/beans  
http://www.springframework.org/schema/beans/spring-beans-2.0.xsd  
http://www.springframework.org/schema/util 
http://www.springframework.org/schema/util/spring-util-2.0.xsd  
http://www.springframework.org/schema/jee 
http://www.springframework.org/schema/jee/spring-jee-2.0.xsd 
http://www.springframework.org/schema/lang 
http://www.springframework.org/schema/lang/spring-lang-2.0.xsd 
http://www.springframework.org/schema/aop 
http://www.springframework.org/schema/aop/spring-aop-2.0.xsd  
http://www.springframework.org/schema/tx 
http://www.springframework.org/schema/tx/spring-tx-2.0.xsd">   
</beans>
```

所有注册到容器的业务对象，在Spring中称之为Bean。所以，每一个对象在XML的映射中也自然而然的对应一个<bean>。既然容器最终可以管理所有的业务对象，那么XML中把这些叫做<bean>的元素组织起来，就叫做<beans>。

### <beans>之唯我独尊

<beans>是XML配置文件中最顶层的元素，它下面可以包含0或者1个<description>和多个<bean>以及<import>或者<alias>。

![image-20210127165609676](/typora-user-images/image-20210127165609676.png)

<beans>作为所有的“统帅”，它拥有相应的属性对所辖的<bean>进行统一的默认行为设置，包括如下几个：

- **default-lazy-init**:其值可以指定为true或者false，默认值为false。用来标志是否对所有的<bean>进行延迟初始化，但是指定为true时并不一定会延迟初始化，如果某个非延迟的bean依赖于该bean，那么该bean就不会延迟初始化。
- **default-autowire**：可以取值为no-不采取任何形式的自动绑定，完全依赖于手工明确配置、byName-与XML文件中声明的bean定义的beanName的值进行匹配、byType-会根据当前bean定义类型，按类型匹配、constructor-同样是按类型绑定，但是是匹配构造放的参数类型，而不是实例属性的类型，以及autodetect-byTye和constructor的结合体，如果是无参构造方法则是由byType，否则使用constructor模式。默认为no。如果使用自动绑定的话，用来标志全体bean使用哪一种默认绑定方式。
- **default-dependency-check**：可以取值为none-不检查、objects-只对对象引用类型依赖进行检查、simple-对简单属性类型以及相关的collection进行依赖检查，对象引用类型的依赖除外，以及all-simple和objects的结合，默认为none，即不做检查。
- **default-init-method**：如果所管辖的所有<bean>都有同样名称的初始化方法，可以在这里统一指定这个初始化方法名，而不用每一个单独指定。
- **default-destroy-method**：与default-init-method相对应，如果所管辖的所有<bean>都有同样名称的销毁方法，可以在这里统一指定这个销毁方法名，而不用每一个单独指定。

### <description>、<import>和<alias>

这几个元素通常情况下不是必须的，这里只是了解一下。

- **<description>**：在配置文件中指定一些描述性的信息。
- **<import>**：可以在主要的配置文件中通过这个标签元素对其所依赖的配置文件引用，例如在A文件中<import resource=”B.xml”>
- **<alias>**：为某些<bean>起一些别名，通常情况下是为了减少输入。

### id属性

通常，每个注册到容器的对象都需要一个唯一标志来将其与同处一室的bean兄弟们区分开来，通过id属性来指定当前注册对象的beanName是什么。实际上并非任何情况下都需要指定每个<bean>的id，有些情况下可以省略，会在后面提到。

除了使用id来指定<bean>在容器中的标志，还可以使用name属性来指定<bean>的别名。name比id灵活之处在于，name可以使用id不能使用的一些字符，比如、。而且还可以通过逗号、空格或者冒号分割指定多个name。name的作用和alias的作用基本相同。

```xml
<bean id="djNewsListener" 

name="/news/djNewsListener,dowJonesNewsListener" 

class="..impl.DowJonesNewsListener">

</bean>

<alias name="djNewsListener" alias="/news/djNewsListener"/> 

<alias name="djNewsListener" alias="dowJonesNewsListener"/> 
```

### class属性

每个注册到容器的对象都需要通过<bean>元素的class属性指定其类型，否则，容器可不知道这个对象到底是何方神圣。在大部分情况下该属性都是必须的，仅在少数情况下不需要指定，后面会提到。

### xml中的继承

```xml
<bean id="newsProviderTemplate" abstract="true">  
	<property name="newPersistener">    
		<ref bean="djNewsPersister"/>  
	</property> 
</bean>  

<bean id="superNewsProvider" parent="newsProviderTemplate"   class="..FXNewsProvider">  
	<property name="newsListener">   
		<ref bean="djNewsListener"/>  
	</property> 
</bean>
 
<bean id="subNewsProvider" parent="newsProviderTemplate"   class="..SpecificFXNewsProvider">  
	<property name="newsListener">   
		<ref bean="specificNewsListener"/>   
	</property> 
< /bean>
```

​	我们在声明subNewsProvider的时候，使用了parent属性，将其值指定为superNewsProvider，这样就继承了superNewsProvider定义的默认值，只需要将特定的属性进行更改，而不要全部又重新定义一遍。

​	parent属性还可以与abstract属性结合使用，达到将相应bean定义模板化的目的。newsProviderTemplate的bean定义通过abstract属性声明为true，说明这个bean定义不需要实例化，**这就是之前提到的可以不指定class属性的少数场景之一。**该bean只是配置一个模板，不对应任何对象，superNewsProvider和subNewsProvider通过parent指向这个模板定义，就拥有了该模板定义的所有属性配置。

​	**容器在初始化对象实例的时候，不会关注abstract的bean。如果你不想容器在初始化对象实例的时候，那么可以将其abstract属性赋值为true，以避免容器将其实例化。同样对于ApplicationContext也是如此。**



## bean的scope（作用域）

cope用来声明容器中的对象所应该处的限定场景或者说该对象的存活时间，即容器在对象进入相应的scope之前，生产并装配这些对象，在该对象不在处于这些scope的限定之后，容器通常会销毁这些对象。

Spring最初提供了两种bean的scope类型：singleton和prototype，但是2.0发布之后，又引入了另外三种scope类型，即request、session和globa session类型，对应这三种类型只能在Web应用中使用。

### singleton

**标记为拥有singleton scope的对象定义，在Spring的IoC容器中只存在一个实例，所有对该对象的引用将共享这个实例，也就是说它与IoC容器的寿命“几乎”相同。**

**不要将这里的singleton与设计模式中的singleton相混淆。二者的语义是不同的：标记为singleton的bean是由容器来保证这种类型的bean在同一个容器中只存在一个共享实例；而Singleton模式则是保证在同一个Classloader中只存在一个这种类型的实例。**

可以从两个方面看singleton的bean所具有的特性：

- **对象实例数量**：singleton的bean定义，在一个容器中只存在一个共享实例，所有对该类型bean的依赖都引用这一单一实例。
- **对象存活时间**：singleton类型bean定义，从容器启动，到它第一次被请求而实例化开始，只要容器不销毁或者退出，该类型bean的单一实例就会一直存活。

**通常情况下，如果你不指定bean的scope，singleton便是容器默认的scope。**

![image-20210128190632061](/typora-user-images/image-20210128190632061.png)

### prototype

**针对声明为拥有prototype scope的bean定义，容器在接到该类型对象的请求的时候，会每次都重新生成一个新的对象实例给请求方。虽然这种类型的对象的实例化以及属性设置等工作都是由容器负责的，但是只要准备完毕，并且对象实例返回给请求方之后，容器就不再拥有当前返回对象的引用，请求方需要自己负责当前返回对象的后继生命周期的管理工具，包括销毁。**

所以对于那些请求不能共享使用的对象类型，应该将其bean定义的scope设置为prototype，这样可以用它来保存一些用户的状态信息。

![image-20210128190809990](/typora-user-images/image-20210128190809990.png)

你用以下形式来指定某个bean定义的scope为prototype类型，效果是一样的：

```xml
<!-- DTD --> 
<bean id="mockObject1" class="...MockBusinessObject" singleton="false"/> 
<!-- XSD --> 
<bean id="mockObject1" class="...MockBusinessObject" scope="prototype"/>
```

### request

**Spring容器，即XmlWebApplicationContext会为每一个HTTP请求创建一个全新的RequestProcessor对象供当前请求使用，当请求结束后，该对象实例的生命周期即告结束。**

### session

Spring容器会为每个独立的session创建属于它们自己的全新的UserPreferences对象实例。与request相比，除了可能有更长的存活时间，其它方面真是没什么区别。

### global session

global session只应用于基于porlet的Web应用程序中才有意义，在普通的基于servlet的Web应用中使用了这个类型的scope，容器会将其作为普通的session类型对待。

### 自定义scope

默认的singleton和prototype是硬编码到代码中的，而request、session和globa session，包括自定义scope类型，都实现了Scope接口，该接口定义如下：

```java
public interface Scope {    
	Object get(String name, ObjectFactory objectFactory); 
	
	Object remove(String name);  

	void registerDestructionCallback(String name, Runnable callback); 
 
	String getConversationId(); 
}
```

要实现自己的scope类型，首先要给出一个Scope接口的实现类，并非4个方法都是必须的，**但get和remove方法必须实现。**下面是一个例子：

```java
public class ThreadScope implements Scope { 
	
	private final ThreadLocal threadScope = new ThreadLocal() {       
		protected Object initialValue() {         
			return new HashMap();       
		}      
	};      

	public Object get(String name, ObjectFactory objectFactory) {     
		Map scope = (Map) threadScope.get();     
		Object object = scope.get(name);     
		if(object==null) {       
			object = objectFactory.getObject();        
			scope.put(name, object);     
		}     

		return object;    
	} 
   
	public Object remove(String name) {     
		Map scope = (Map) threadScope.get();     
		return scope.remove(name);   
	} 
	
	public void registerDestructionCallback(String name, Runnable callback) {   
	}   
	... 
}
```

接下来就是把这个新定义的scope注册到容器中，才能供相应的bean使用。我们有两种方式可以注册。

一种就是我们可以使用ConfigurableBeanFactory的以下方法去注册：`void registerScope(String scopeName, Scope scope)`，其中参数scopeName就是使用的bean定义可以指定的名称，参数scope就是我们提供的scope实现类实例。注册例子如下：

```java
Scope threadScope = new ThreadScope(); 
beanFactory.registerScope("thread",threadScope);
```

另一种方式就是可以在xml文件中配置CustomScopeConfigurer来注册scope。例如：

```xml
<bean class="org.springframework.beans.factory.config.CustomScopeConfigurer">  
	<property name="scopes">   
		<map>    
			<entry key="thread" value="com.foo.ThreadScope"/>   
		</map>  
	</property> 
</bean>
```

接下来就可以使用这个自定义的scope了，例如：

```xml
<bean id="beanName" class="..." scope="thread">  
	<aop:scoped-proxy/> 
</bean>
```

## 工厂方法与FactoryBean

工厂方法（Factory Method）模式，提供一个工厂类来实例化具体的接口实现类，这样主体对象只需要依赖工厂类，具体使用的实现类有变更的话，只是变更工厂类，而主体对象不需要做任何变动，这样就避免了接口与实现类的耦合性。

针对这种方式，Spring的IoC容器同样提供了对应的集成支持，我们要做的，只是将工厂类所返回的具体的接口实现类注入给主体对象。

### 静态工厂方法

假设现在有一个接口BarInterface，为了向使用该接口的客户端对象屏蔽以后可能对BarInterface实现类的变动，因此还提供了一个静态的工厂方法实现类StaticBarInterfaceFactory，代码如下：

```java
public class StaticBarInterfaceFactory {  
	public static BarInterface getInstance()  {  
		return new BarInterfaceImpl();  
	} 
}
```

为了将该静态方法类返回的实现注入Foo，我们使用以下方式进行配置：

```xml
<bean id="foo" class="...Foo">  
	<property name="barInterface">   
		<ref bean="bar"/>  
	</property> 
</bean> 
 
<bean id="bar" class="...StaticBarInterfaceFactory" factory-method="getInstance"/>
```

**class指定静态方法工厂类，factory-method指定工厂方法名称，然后，容器调用该静态方法工厂类的指定工厂方法，并返回方法调用后的结果，即BarInterfaceImpl的实例。**

某些时候，有的工厂类的工厂方法需要参数来返回相应实例，可以通过<constructor-arg>来指定工厂方法需要的参数：

```java
public class StaticBarInterfaceFactory {  
	public static BarInterface getInstance(Foobar foobar)  {   
		return new BarInterfaceImpl(foobar);  
	} 
}
```

```xml
<bean id="foo" class="...Foo">  
	<property name="barInterface">   
		<ref bean="bar"/>  
	</property> 
</bean> 
 
<bean id="bar" class="...StaticBarInterfaceFactory" factory-method="getInstance">  
	<constructor-arg>   
		<ref bean="foobar"/>  
	</constructor-arg> 
</bean> 
 
<bean id="foobar" class="...FooBar"/>
```

**唯一需要注意的是，针对静态工厂方法实现类的bean定义，使用<constructor-arg>传入的是工厂方法的参数，而不是静态工厂方法的构造方法的参数，况且，静态工厂方法实现类也没有提供显式的构造方法。**

### 非静态工厂方法

针对非静态方法的调用方式也很简单，只是需要稍微该变一下代码即可：

```java
public class NonStaticBarInterfaceFactory  {  
	public BarInterface getInstance()  {   
		return new BarInterfaceImpl();   
	}  
	... 
}
```

```xml
<bean id="foo" class="...Foo">  
	<property name="barInterface">   
		<ref bean="bar"/>  
	</property>  
</bean> 
 
<bean id="barFactory" class="...NonStaticBarInterfaceFactory"/>   

< bean id="bar" factory-bean="barFactory" factory-method="getInstance"/>
```

**现在的最主要不同就在于是使用factory-bean属性来指定工厂方法所在的工厂类实例，而不是通过class属性来指定工厂方法所在类的类型。**如果需要参数的话，配置方法同静态工厂方法一样。

### FactoryBean

当某些对象的实例化过程因为繁琐，使用XML配置过于复杂时，就可以考虑通过代码的方式来完成这个实例化过程，FactoryBean接口就是为此而生的。FactoryBean接口只定义了三个方法，如下：

```java
public interface FactoryBean { 
	Object getObject() throws Exception;  
	Class getObjectType();  
	boolean isSingleton(); 
}
```

- **getObject()方法会返回该FactoryBean生产的对象实例，我们需要实现该方法以给出自己的对象实例化逻辑；**
- **getObjectType()方法仅返回getObject()方法所返回的对象的类型，如果预先无法确定，则返回null；**
- **isSingleton()方法返回结果用于表明是否为单例实例。**

例子：得到第二天的日期

```java
public class NextDayDateFactoryBean implements FactoryBean { 
 
	public Object getObject() throws Exception {   
		return new DateTime().plusDays(1);  
	} 
 
	public Class getObjectType() {   
		return DateTime.class;  
	} 

	public boolean isSingleton() {   
		return false;  
	} 
}
```

接下来只需要将其注册到容器中即可：

```xml
<bean id="nextDayDateDisplayer" class="...NextDayDateDisplayer">  
	<property name="dateOfNextDay">   
		<ref bean="nextDayDate"/>  
	</property> 
</bean>         

<bean id="nextDayDate" class="...NextDayDateFactoryBean"> </bean>
```

接下来就是最重要的地方，来看看NextDayDateDisplayer的定义：

```java
public class NextDayDateDisplayer {  
	private DateTime dateOfNextDay;  
	// 相应的setter方法  
	// ... 
}
```

**NextDayDateDisplayer所声明的依赖dateOfNextDay的类型为DateTime，而不是NextDayDateFactoryBean，也就是说FactoryBean类型的bean定义，通过正常id引用，容器返回的时FactoryBean所生产的对象类型，而非其本身。当然如果一定要取得FactoryBean本身的话，可以通过bean定义的id之前加前缀“&”来达到目的。**

## 方法注入以及方法替换

在引出标题内容之前，需要先提一下有关bean的scope的prototype属性的陷阱，我们知道，拥有prototype类型scope的bean，在请求方法每次向容器请求该类型对象的时候，容器都会返回一个全新的该对象实例，下面看看这个情况：

```java
public class MockNewsPersister implements IFXNewsPersister {  
	private FXNewsBean newsBean;    
	public void persistNews(FXNewsBean bean) {   
		persistNewes();  
	}  
	public void persistNews()  {   
		System.out.println("persist bean:"+getNewsBean());  
	}  
	public FXNewsBean getNewsBean() {  
		return newsBean;  
	} 
	public void setNewsBean(FXNewsBean newsBean) {   
	this.newsBean = newsBean;  
	} 
}
```

配置为：

```xml
<bean id="newsBean" class="..domain.FXNewsBean" singleton="false">
</bean>

<bean id="mockPersister" class="..impl.MockNewsPersister">
	<property name="newsBean">
		<ref bean="newsBean"/>
	</property>
</bean>
```

当我们多次调用MockNewsPersister的persistNews时看看结果：

```java
BeanFactory container = new XmlBeanFactory(new ClassPathResource("..")); 
MockNewsPersister persister = (MockNewsPersister)container.getBean("mockPersister"); 
persister.persistNews(); persister.persistNews(); 

输出： 
persist bean:..domain.FXNewsBean@1662dc8 
persist bean:..domain.FXNewsBean@1662dc8
```

从输出看对象实例是相同的，而这与我们的初衷是相悖的。问题实际上不出在FXNewsBean的scope类型是否是prototype的，而是出在实例的取得方式上面。**虽然FXNewsBean拥有prototype类型的scope，但当容器将一个FXNewsBean的实例注入MockNewsPersister之后，MockNewsPersister就会一直持有这个FXNewsBean实例的引用。虽然每次输出都调用了getNewBean()方法并返回一个FXNewsBean的实例，但实际上每次返回的都是MockNewsPersister持有的容器第一次注入的实例。这就是问题之所在**。

**换句话说，第一个实例注入后，MockNewsPersister再也没有重新向容器申请新的实例，所以容器也不会重新为其注入新的FXNewsBean类型的实例。**

下面就是介绍解决这个问题的方法了。

### 方法注入

Spring容器提出了一种叫做方法注入（Method Injection）的方式，可以帮助我们解决上述问题。我们所要做的很简单，只要让getNewsBean方法声明符合规定的格式，并在配置文件中通知容器，当该方法被调用的时候，每次返回指定类型的对象实例即可。

**若要使用方法注入的方式，该方法必须能够被子类实现或者覆写，因为容器会为我们要进行方法注入的对象使用Cglib动态生成一个子类实现，从而代替当前对象，**既然我们的getNewsBean()方法已经满足以上方法声明要求，就只需要正确配置该类了：

```xml
<bean id="newsBean" class="..domain.FXNewsBean" singleton="false"> 
</bean> 

<bean id="mockPersister" class="..impl.MockNewsPersister">  
	<lookup-method name="getNewsBean" bean="newsBean"/> 
</bean>
```

**通过<lookup-method>的name属性指定需要注入的方法名，bean属性指定需要注入的对象，当getNewsBean方法被调用的时候，容器可以每次返回一个新的FXNewsBean类型的实例。所以这个时候再次检查执行结果，输出的实例引用就是不同的了。**

### 殊途同归

除了使用方法注入来达到“每次调用都让容器返回新的对象实例”的目的，还可以使用其他方式达到相同的目的：

* **使用BeanFactoryAware**：我们知道，即使没有方法注入，只要在实现getNewsBean()方法的时候，能够保证每次调用BeanFactory的getBean(“newsBean”)，就同样可以每次都取得新的FXNewsBean对象实例。BeanFactoryAware接口就可以帮我们完成这一步，其定义如下：

```java
public interface BeanFactoryAware {   
	void setBeanFactory(BeanFactory beanFactory) throws BeansException; 
}
```

Spring框架提供了一个BeanFactoryAware接口，容器在实例化实现了该接口的bean定义的过程中，会自动将容器本身注入该bean。这样，该bean就持有了它所处的BeanFactory的引用。实现BeanFactoryAware接口和配置的代码如下：

```java
public class MockNewsPersister implements IFXNewsPersister,BeanFactoryAware {
	private BeanFactory beanFactory;
  
	public void setBeanFactory(BeanFactory bf) throws BeansException {
		this.beanFactory = bf;
	}
	public void persistNews(FXNewsBean bean) {
		persistNews();
	}
	public void persistNews() {
		System.out.println("persist bean:"+getNewsBean());
	}
	public FXNewsBean getNewsBean() {
		return beanFactory.getBean("newsBean");
	}
}
```

```xml
<bean id="newsBean" class="..domain.FXNewsBean" singleton="false"> 
</bean> 

<bean id="mockPersister" class="..impl.MockNewsPersister"> 
</bean>
```

* **使用ObjectFatoryCreatingFatoryBean**：实际上ObjectFatoryCreatingFatoryBean实现了BeanFactoryAware接口，它返回的ObjectFactory实例这是特定于与Spring容器进行交互的一个实现而已。**使用它的好处就是，隔离了客户端对象对BeanFactory的直接引用。**

```java
public class MockNewsPersister implements IFXNewsPersister {  
	private ObjectFactory newsBeanFactory;     
	
	public void persistNews(FXNewsBean bean) {   
		persistNews();  
	}  
	public void persistNews()  {   
		System.out.println("persist bean:"+getNewsBean());  
	}  
	public FXNewsBean getNewsBean() {   
		return newsBeanFactory.getObject();  
	}  
	public void setNewsBeanFactory(ObjectFactory newsBeanFactory) {   
		this.newsBeanFactory = newsBeanFactory;  
	} 
}
```

```xml
<bean id="newsBean" class="..domain.FXNewsBean" singleton="false"> 
</bean> 

<bean id="newsBeanFactory" class="org.springframework.beans.factory.config.ObjectFactoryCreatingFactoryBean">  
	<property name="targetBeanName">   
		<idref bean="newsBean"/>  
	</property> 
</bean> 

<bean id="mockPersister" class="..impl.MockNewsPersister"> 
	<property name="newsBeanFactory">   
		<ref bean="newsBeanFactory"/>  
	</property> 
</bean>
```

### 方法替换

基本上可以认为，方法替换可以帮助我们实现简单的方法拦截功能。假设某天我看FXNewsProvider不爽，想替换掉它的getAndPersistNews方法默认逻辑，这时我就可以用方法替换将它的原有逻辑给替换掉。

替换的方法就是实现一个叫做MethodReplacer接口，简单的实现例子如下：

````java
public class FXNewsProviderMethodReplacer implements MethodReplacer { 
	private static final transient Log logger = LogFactory.getLog(FXNewsProviderMethodReplacer.class);    

	public Object reimplement(Object target, Method method, Object[] args) throws Throwable {   
		logger.info("before executing method["+method.getName()+ "] on Object["+target.getClass().getName()+"].");      
		System.out.println("sorry,We will do nothing this time.");    
		logger.info("end of executing method["+method.getName()+ "] on Object["+target.getClass().getName()+"].");   
		return null;   
	} 
}
````

接下来把它配置到xml文件中就可以了：

```xml
<bean id="djNewsProvider" class="..FXNewsProvider">  
	<constructor-arg index="0">   
		<ref bean="djNewsListener"/>  
	</constructor-arg>  
	<constructor-arg index="1">   
		<ref bean="djNewsPersister"/>  
	</constructor-arg>  
	
	<replaced-method name="getAndPersistNews" replacer="providerReplacer">  
	</replaced-method> 
</bean> 
 
<bean id="providerReplacer" class="..FXNewsProviderMethodReplacer"> </bean>
```

## 容器加载过程

IOC容器工作框图如下：

![image-20210204173225185](/typora-user-images/image-20210204173225185.png)

### 两大阶段

IoC容器所起的作用就如上图所展示的那样，**他会以某种方式加载Configuration Metadata（通常为XML配置文件），然后根据这些信息绑定整个系统对象，最终组装成一个可用的基于轻量级容器的应用系统。**

Spring的IoC容器实现以上功能的过程，基本上分为两个阶段：**容器启动阶段**和**Bean的实例化阶段**。

![image-20210204173534555](/typora-user-images/image-20210204173534555.png)

### 容器启动阶段

**容器启动伊始，首先会通过某种途径加载Configuration MetaData。除了代码方式比较直接，大部分情况下都会依赖于工具类BeanDefinitionReader对加载的配置文件进行解析和分析，并将分析后的信息编组为相应的BeanDefinition，最后把这些保存了bean定义必要信息的BeanDefinition，注册到相应的BeanDefinitionRegistry，**这样启动阶段就完成了。

![image-20210204173747427](/typora-user-images/image-20210204173747427.png)

### Bean实例化阶段

经过第一阶段，现在所有的bean定义信息都通过BeanDefinition的方式注册到了BeanDefinitionRegistry中。当某个请求方通过容器的getBean方法明确地请求某个对象，或者因依赖关系容器需要隐式地调用getBean方法时，就会触发第二阶段的活动。

该阶段，容器会首先检查所请求的对象之前是否已经初始化。如果没有，则会根据注册的BeanDefinition所提供的信息实例化被请求对象，并为其注入依赖。如果该对象实现了某些回调接口，也会根据回调接口的要求来装配它。当该对象装配完毕之后，容器会立即将其返回请求方使用。如果说第一阶段只是根据图纸装配生产线的话，那么第二阶段就是使用装配好的生产线来生产具体的产品了。

## 插手两大阶段

**Spring提供了一种叫做BeanFactoryPostProcessor的容器扩展机制，该机制允许我们在容器实例化相应对象之前，对注册到容器的BeanDefinition所保存的信息做相应的修改。这就相当于在容器第一阶段结束之后，第二阶段开始之前，加入一道工序，让我们对最终的BeanDefinition做一些额外的操作，**比如修改其中bean定义的某些属性，为bean定义增加其他信息等。

如果自定义实现BeanFactoryPostProcessor，通常实现接口BeanFactoryPostProcessor就可以了，如果有多个自定义实现的类，则实现Ordered接口来规定它们的顺序。

但是Spring为我们提供了三种常用的已经实现好的BeanFactoryPostProcessor实现类，它们分别是 PropertyPlaceholderConfigurer、PropertyOverrideConfigurer和CustomEditorConfigure。在介绍这三种实现类之前，先需要了解它们的配置方法是什么。

手动装配BeanFactory使用的BeanFactoryPostProcessor

```java
// 声明将被后处理的BeanFactory实例
ConfigurableListableBeanFactory beanFactory = new XmlBeanFactory(new ClassPathResource("...")); 
// 声明要使用的BeanFactoryPostProcessor 
PropertyPlaceholderConfigurer propertyPostProcessor = new PropertyPlaceholderConfigurer(); 
propertyPostProcessor.setLocation(new ClassPathResource("...")); 
// 执行后处理操作
propertyPostProcessor.postProcessBeanFactory(beanFactory);
```

很简单，只需要在XML配置文件中加入以下代码即可：

通过ApplicationContext使用BeanFactoryPostProcessor

```xml
... 
<beans>  
	<bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer"> 
		<property name="locations">    
			<list>     
				<value>conf/jdbc.properties</value>     
				<value>conf/mail.properties</value>    
			</list>   
		</property>  
	</bean>   
	... 
</beans>
```

### PropertyPlaceholderConfigurer

​	通常情况下，我们不想将类似于系统管理相关的信息同业务对象相关的配置信息混杂到XML配置文件中，以免部署或者维护期间因为改动繁杂的XML配置文件而出现问题。我们会将一些数据库连接信息、邮件服务器等相关信息单独配置到一个properties文件中，这样，如果因系统资源变动的话，只需要关注这些简单properties配置文件即可。

​	PropertyPlaceholderConfigurer允许我们在XML配置文件中使用占位符（PlaceHolder），并将这些占位符所代表的资源单独配置到简单的properties文件中来加载。

使用了占位符的数据源配置

```xml
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"destroy-method="close"> 
 <property name="url"> 
 <value>${jdbc.url}</value> 
 </property> 
 <property name="driverClassName"> 
 <value>${jdbc.driver}</value> 
 </property> 
 <property name="username"> 
 <value>${jdbc.username}</value> 
 </property> 
 <property name="password"> 
 <value>${jdbc.password}</value> 
 </property> 
 <property name="testOnBorrow"> 
 <value>true</value> 
 </property> 
 <property name="testOnReturn"> 
 <value>true</value> 
 </property> 
 <property name="testWhileIdle"> 
 <value>true</value> 
 </property> 
 <property name="minEvictableIdleTimeMillis"> 
 <value>180000</value> 
 </property> 
 <property name="timeBetweenEvictionRunsMillis"> 
 <value>360000</value> 
 </property> 
 <property name="validationQuery"> 
 <value>SELECT 1</value> 
 </property> 
 <property name="maxActive"> 
 <value>100</value> 
 </property> 
</bean>
```

对应的properties文件内容则为：

```properties
jdbc.url=jdbc:mysql://server/MAIN?useUnicode=true&characterEncoding=ms932&failOverReadOnly=false&roundRobinLoadBalance=true 
jdbc.driver=com.mysql.jdbc.Driver 
jdbc.username=your username
jdbc.password=your password
```

​	当BeanFactory在第一阶段加载完成所有配置信息时，BeanFactory中保存的对象的属性信息还只是以占位符的形式存在，如${jdbc.url}、${jdbc.driver}。当PropertyPlaceholderConfigurer作为BeanFactoryPostProcessor被应用时，它会使用properties配置文件中的配置信息来替换相应BeanDefinition中占位符所表示的属性值。这样，当进入容器实现的第二阶段实例化bean时，bean定义中的属性值就是最终替换完成的了。PropertyPlaceholderConfigurer不单会从其配置的properties文件中加载配置项，同时还会检查Java的System类中的Properties。

### PropertyOverrideConfigurer

可以通过PropertyOverrideConfigurer对容器中配置的任何你想处理的bean定义的property信息进行覆盖替换。比如之前的dataSource定义中，maxActive的值为100，如果我们觉得不合适，那么可以通过PropertyOverrideConfigurer将其覆盖为200：dataSource.maxActive=200。

如果要对容器中的某些bean定义的property信息进行覆盖，我们需要按照如下规则提供一个PropertyOverrideConfigurer使用的配置文件：

beanName.propertyName=value 

也就是说，properties文件中的键是以XML中配置的bean定义的beanName为标志开始的（通常就是id指定的值），后面跟着相应被覆盖的property的名称，比如上面的maxActive。下面是针对dataSource定义给出的PropertyOverrideConfigurer的propeties文件配置信息：

```properties
#pool-adjustment.properties 

dataSource.minEvictableIdleTimeMillis=1000 

dataSource.maxActive=50 
```

这样，当按照如下代码，将PropertyOverrideConfigurer加载到容器之后，dataSource原来定义的默认值就会被pool-adjustment.properties文件中的信息所覆盖：

```xml
<bean class="org.springframework.beans.factory.config.PropertyOverrideConfigurer"> 

 <property name="location" value="pool-adjustment.properties"/> 

</bean> 
```

pool-adjustment.properties中没有提供的配置项将继续使用原来XML配置中的默认值。当容器中配置的多个PropertyOverrideConfigurer对同一个bean定义的同一个property值进行处理的时候，最后一个将会生效。

### CustomEditorConfigure

**因为我们配置XML文件都是String类型，即容器从XML格式的文件中读取的都是字符串形式，最终应用程序却是由各种类型的对象所构成的。要想完成这种由字符串到具体对象的转换，都需要这种转换规则相关的信息，而CustomEditorConfigure就是帮我们传达类似信息的。**

## bean的生命周期

接下来就是第二大阶段了，即bean实例化阶段的实现逻辑。这里还需要再提一次BeanFactory和ApplicationContext对于bean加载的区别：

- **BeanFactory对象实例化默认采用延迟初始化**。
- **ApplicationContext启动之后会实例化所有的bean定义**。

当调用getBean()方法时，内部发现该bean定义之前还没有被实例化之后，会通过createBean()方法来进行具体的对象实例化，实例化过程如图所示：

![image-20210218152110747](/typora-user-images/image-20210218152110747.png)

下面就来详细看看其中的步骤。

### Bean的实例化与BeanWrapper

容器在内部实现的时候，采用“策略模式”来决定采用何种方式初始化bean实例。通常，可以通过反射或者CGLIB动态字节码生成来初始化相应的bean实例或者动态生成其子类。

#### 第一步

**容器只要根据相应的bean定义的BeanDefintion取得实例化信息，结合CglibSubclassingInstantiationStrategy通过反射的方式，以及不同的bean定义类型，就可以返回实例化完成的对象实例。但是，返回的不是构造完成的对象实例，而是以BeanWrapper对构造完成的对象实例进行包括，返回相应的BeanWrapper实例。**

#### 第二步

**第一步结束后返回BeanWrapper实例而不是原先的对象实例，就是为了第二步“设置对象属性”。**使用BeanWrapper对bean实例操作很方便，可以免去直接使用Java反射API操作对象实力的反锁，看一段代码就知道Spring容器内部时如何设置对象属性的了：

```java
Object provider = Class.forName("package.name.FXNewsProvider").newInstance(); 
Object listener = Class.forName("package.name.DowJonesNewsListener").newInstance(); 
Object persister = Class.forName("package.name.DowJonesNewsPersister").newInstance();  

BeanWrapper newsProvider = new BeanWrapperImpl(provider); 
newsProvider.setPropertyValue("newsListener", listener); 
newsProvider.setPropertyValue("newPersistener", persister);
```

### 各色的Aware接口

**当对象实例化完成后并且相关属性以及依赖设置完成之后，Spring容器会检查当前对象实例是否实现了一系列的以Aware命名结尾的接口定义，如果是，则将这些Aware接口定义中规定的依赖注入给当前对象实例。**下面介绍几个重要的Aware接口：

- **ApplicationContextAware**: 获得ApplicationContext对象,可以用来获取所有Bean definition的名字。
- **BeanFactoryAware**:获得BeanFactory对象，可以用来检测Bean的作用域。
- **BeanNameAware**:获得Bean在配置文件中定义的名字。
- **ResourceLoaderAware**:获得ResourceLoader对象，可以获得classpath中某个文件。
- **ServletContextAware**:在一个MVC应用中可以获取ServletContext对象，可以读取context中的参数。
- **ServletConfigAware**： 在一个MVC应用中可以获取ServletConfig对象，可以读取config中的参数。

### BeanPostProcessor

BeanPostProcessor会处理容器内所有符合条件的实例化后的对象实例，该接口声明了两个方法，分别在不同的时机执行：

```java
public interface BeanPostProcessor  {  
	Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException; 
	Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException; 
}
```

**postProcessBeforeInitialization()方法是上图中BeanPostProcessor前置处理这一步将会执行的方法，postProcessAfterInitialization()则是对应图中BeanPostProcessor后置处理那一步将会执行的方法。**BeanPostProcessor的两个方法中都传入了原来的对象实例的引用，这为我们操作扩展容器的对象实例化过程中的行为提供了极大的便利。

### InitializingBean和init-method

InitializingBean是容器内部广泛使用的一个对象生命周期表示接口，其定义如下：

```java
public interface InitializingBean {   
	void afterPropertiesSet() throws Exception; 
}
```

**其作用在于，在对象实例化过程调用过“BeanPostProcessor”的前置处理之后，会接着检测当前对象是否实现了InitializingBean接口，如果是，则回到调用其afterPropertiesSet()方法进一步调整对象实例的状态。**

**通过init-method，系统中业务对象的自定义初始化操作可以以任何方式命名。而不再受制于InitializingBean的afterPropertiesSet()。**在自己定义init方法后，只需要在XML文件中简单配置一下就好了：

```xml
<beans>  
	<bean id="tradeDateCalculator" class="FXTradeDateCalculator" init-method="setupHolidays" > 
		<constructor-arg>    
			<ref bean="sqlMapClientTemplate"/>   
		</constructor-arg>  
	</bean> 
	<bean id="sqlMapClientTemplate" class="org.springframework.orm.ibatis.SqlMapClientTemplate">   
		...  
	</bean>    
	... 
< /beans>
```

这样FXTradeDateCalculator类中的setupHolidays方法就是会在该阶段执行的方法了。

### DisposableBean与destroy-method

**当所有的一切，该设置的设置，该注入的注入，该调用的调用之后，容器将检查singleton类型的bean实例，看其是否实现了DisposableBean接口。或者其对应的bean定义是否通过<bean>的destroy-method属性制定了自定义的对象销毁方法，如果是，就会为该实例注册一个用于对象销毁的回调，以及在这些singleton类型的对象实例销毁之前，执行销毁逻辑。**

**不过这些自定义的对象销毁逻辑，在对象实例初始化完成并注册了相关的回调方法之后，并不会马上执行。回调方法注册后，在使用完成后，只有在执行回调方法之后，才会执行销毁操作。**

对于BeanFactory注册回调方法的方式为：

```java
((ConfigurableListableBeanFactory)container).destroySingletons();
```

对于ApplicationContext注册回调方法的方式为：

```java
((AbstractApplicationContext)container).registerShutdownHook();
```

至此为止，bean就算走完了它在容器中“光荣”的一生。

