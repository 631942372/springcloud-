> ###### 介绍完分布式配置中心，结合前面的文章。我们已经有了一个微服务的框架了，可以对外提供api接口服务了。但现在试想一下，在微服务框架中，每个对外服务都是独立部署的，对外的api或者服务地址都不是不尽相同的。对于内部而言，很简单，通过注册中心自动感知即可。但我们大部分情况下，服务都是提供给外部系统进行调用的，不可能同享一个注册中心。同时一般上内部的微服务都是在内网的，和外界是不连通的。而且，就算我们每个微服务对外开放，对于调用者而言，调用不同的服务的地址或者参数也是不尽相同的，这样就会造成消费者客户端的复杂性，同时想想，可能微服务可能是不同的技术栈实现的，有的是http、rpc或者websocket等等，也会进一步加大客户端的调用难度。所以，一般上都有会有个api网关，根据请求的url不同，路由到不同的服务上去，同时入口统一了，还能进行统一的身份鉴权、日志记录、分流等操作。接下来，我们就来了解今天要讲解的路由服务：zuul。

#### API网关是什么

> API网关可以提供一个单独且统一的API入口用于访问内部一个或多个API。简单来说嘛就是一个统一入口，比如现在的支付宝或者微信的相关api服务一样，都有一个统一的api地址，统一的请求参数，统一的鉴权。

#### 客户端和服务端直连的弊端

* 客户端会对此请求不同的微服务，增加客户端复杂性
* 存在跨域请求时，需要进行额外处理
* 认证服务，每个服务需要独立认证
* UI端和微服务耦合

#### 网关的优缺点

**优点**：

* 减少api请求次数
* 限流
* 缓存
* 统一认证
* 降低微服务的复杂度
* 支持混合通信协议\(前端只和api通信，其他的由网关调用\)
* ……

缺点：

* 网关需高可用，可能产生单点故障
* 管理复杂

### 网关的选择

现在市场上有很多的网关可供选择：

* Spring Cloud Zuul：本身基于`Netflix`开源的微服务网关,可以和`Eureka`,`Ribbon`,`Hystrix`等组件配合使用。
* Kong : 基于OpenResty的 API 网关服务和网关服务管理层。
* Spring Cloud Gateway：是由`spring`官方基于`Spring5.0`,`Spring Boot2.0`,`Project Reactor`等技术开发的网关，提供了一个构建在`Spring Ecosystem`之上的API网关，旨在提供一种简单而有效的途径来发送API，并向他们提供交叉关注点，例如：安全性，监控/指标和弹性。目的是为了替换`Spring Cloud Netfilx Zuul`的。

在`Spring cloud`体系中，一般上选择`zuul`或者`Gateway`。当然，也可以综合自己的业务复杂性，自研一套或者改在一套符合自身业务发展的api网关的，最简单做法是做个`聚合api服务`，通过`SpringBoot`构建对外的api接口，实现统一鉴权、参数校验、权限控制等功能，说白了就是一个`rest`服务。

### Zuul实践

创建工程：`spring-cloud-zuul`。**这里直接加入了注册中心进行服务化模式。**

0.加入pom依赖。

```
    <!-- zuul 依赖 -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
    </dependency>        
    <!-- eureka client 依赖 -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
```

**注意：这里的**`Eureka`**不是必须的。在没有注册中心的情况下，也是可以进行zuul使用的。**

1.配置文件，配置注册中心相关信息、路由规则等。

```
spring.application.name=zuul-server
server.port=8888

# 注册中心地址 -此为单机模式
eureka.client.service-url.defaultZone=http://127.0.0.1:1000/eureka
# 启用ip配置 这样在注册中心列表中看见的是以ip+端口呈现的
eureka.instance.prefer-ip-address=true
# 实例名称  最后呈现地址：ip:15678
eureka.instance.instance-id=${spring.cloud.client.ip-address}:${server.port}

## 路由规则
## 传统路由配置：不依赖服务发现。
## 所有以myapi开头的url路由至http://127.0.0.1:2000/下
## 如http://127.0.0.1:8888/myapi/hello --
>
 http://127.0.0.1:2000/hello
zuul.routes.myApi.path=/myapi/**
zuul.routes.myApi.url=http://127.0.0.1:2000

#forward模式 直接转发至zuul提供的rest服务
zuul.routes.myDemo.path=/myDemo/**
zuul.routes.myDemo.url=forward:/demo

## 服务发现模式
# 路由地址
zuul.routes.myEureka.path=/eureka/**
#为具体服务的名称
zuul.routes.myEureka.service-id=eureka-client
```

**友情提示：默认情况下：Zuul代理所有注册到EurekaServer的微服务，路由规则：**`http://ZUUL_HOST:ZUUL_PORT/微服务实例名(serverId)/**`**转发至serviceId对应的微服务。**

如：[http://127.0.0.1:8888/eureka-client/hello?name=oKong](http://127.0.0.1:8888/eureka-client/hello?name=oKong)最后就是转发至：[http://127.0.0.1:2000//hello?name=oKong](http://127.0.0.1:2000//hello?name=oKong)

2.启动类加入`@EnableZuulProxy`注解，声明一个Zuul代理。

```
/**
 * zuul 示例
 * 
 * @author oKong
 *
 */
@SpringBootApplication
@EnableZuulProxy
@EnableDiscoveryClient
@Slf4j
public class SpringCloudZuulApplication {
    public static void main(String[] args) throws Exception {
        SpringApplication.run(SpringCloudZuulApplication.class, args);
        log.info("spring-cloud-zuul启动!");
    }

}
```

3.编写一个控制类，测试`forward`功能。

```
/**
 * zuul 内部提供对外服务示例
 * @author oKong
 *
 */
@RestController
@RequestMapping("/demo")
public class DemoController {

    @GetMapping("/hello")
    public String hello(String name) {
        return "hi," + name + ",this is zuul api! ";
    }
}
```

### 路由规则说明

简单说明下关于路由规则的一些说明：

#### 传统路由配置：不依赖服务发现，如nginx

* 单例实例配置：通过`zuul.routes.<route>.path`和`zuul.routes.<route>.url`参数对的方式来配置

```
# 传统路由配置
zuul.routes.server-provide.path=/server-provide/**
zuul.routes.server-provide.url=http://127.0.0.1:1001/
```

* 多实例配置：通过一组  
  `zuul.routes.<route>.path`与`zuul.routes.<route>.serviceId`

  参数对的方式配置

```
# 多实例
zuul.routes.server-provide.path=/user-service/**
zuul.routes.server-provide.serviceId=user-service
ribbon.eureka.enabled=false
server-provide.ribbon.listOfServers=http://127.0.0.1:1001/,http://127.0.0.1:1001/
```

#### 服务路由配置：依赖服务发现，结合Eureka

* 默认规则：[http://ZUUL\_HOST:ZUUL\_PORT/微服务实例名\(serverId\)/\*\*](http://zuul_host:ZUUL_PORT/微服务实例名%28serverId%29/**)，转发至serviceId对应的微服务。

* 自定义路由规则：通过一组`zuul.routes.<route>.path`与`zuul.routes.<route>.serviceId`参数对的方式配置

```
# 自定义路由
zuul.routes.server-provide.path=/server-api/**
zuul.routes.server-provide.serviceId=server-provide
```

比如:

![](https://www.liangzl.com/editorImages/cawler/20181015104055_229.jpg)

**而且，要注意，这些过滤器是**`path`**进行最佳路径匹配的，所以，一般上在一些历史系统上，我们会在最后后面加上一个路径/\*\*的匹配规则，以保证历史api可以使用，做到最大兼容性,避免类似404的异常。**

```
zuul.routes.legacy.path=/**
```



