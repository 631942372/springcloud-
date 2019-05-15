在高并发访问下,服务依赖的稳定性与否对系统的影响非常大,但是依赖有很多不可控问题:如网络连接缓慢，资源繁忙，暂时不可用，服务脱机等。高并发的依赖失败时如果没有隔离措施，当前应用服务就有被拖垮的风险。

常看到的举例有：

```
例如:一个依赖30个SOA服务的系统,每个服务99.99%可用。  
99.99%的30次方 ≈ 99.7%  
0.3% 意味着一亿次请求 会有 3,000,00次失败  
换算成时间大约每月有2个小时服务不稳定.  
随着服务依赖数量的变多，服务不稳定的概率会成指数性提高.  
```

因此出现了Hystrix这种处理依赖隔离的框架,同时也是可以帮我们做依赖服务的治理和监控。

限流、熔断、隔离、降级这四个概念是我们谈起微服务会经常谈到的概念，这里我们讨论的是 Hystrix 的实现方式。

**限流**

* 这里的限流与 Guava 的 RateLimiter 的限流差异比较大，一个是为了“保护自我”，一个是“保护下游”

* 当对服务进行限流时，超过的流量将直接 Fallback，即熔断。而 RateLimiter 关心的其实是“流量整形”，将不规整流量在一定速度内规整

**熔断**

* 当我的应用无法提供服务时，我要对上游请求熔断，避免上游把我压垮

* 当我的下游依赖成功率过低时，我要对下游请求熔断，避免下游把我拖垮

**降级**

* 降级与熔断紧密相关，熔断后业务如何表现，约定一个快速失败的 Fallback，即为服务降级

**隔离**

* 业务之间不可互相影响，不同业务需要有独立的运行空间

* 最彻底的，可以采用物理隔离，不同的机器部

* 次之，采用进程隔离，一个机器多个 Tomcat

* 次之，请求隔离

* 由于 Hystrix 框架所属的层级为代码层，所以实现的是请求隔离，线程池或信号量

具体的不做详尽介绍。



前面已经提过Spring Cloud默认已经为Feign整合了Hystrix

但是我们还需要开启Hystrix支持

第一步：首先确保启动类加上**@EnableFeignClients，** 之后开启Feign对Hystrix的支持，在properties文件中添加以下配置：

```
feign：
    hystrix:
        enabled:true
```

第二步：在上一篇Feign的基础上添加Hystrix（断路由）

```
@FeignClient("sc-provider",fallback = "ProviderFeignServiceHystrix.class")   //指定服务别名
public interface ProviderFeignService {

    //指定调用服务接口路径    @PathVariable  @RequestParam
    //当使用feign传参数的时候,需要加上@RequestParam注解,否则对方服务无法识别参数;
    @GetMapping(value = "/provider/helloFeign")//,method = RequestMethod.GET
    String HelloFeign(@RequestParam("message")String message);


    @GetMapping(value = "/provider/{message}")//,method = RequestMethod.GET
    String HelloPathFeign(@PathVariable("message")String message);
}
```

第三步：编写ProviderFeignServiceHystrix类

```
@Component
public class ProviderFeignServiceHystrix implement ProviderFeignService {
    @Override
    public String HelloFeign(String message){
        return "调用服务失败";
    }
}
```

@FeignClient标签的常用属性如下：

* name：指定FeignClient的名称，如果项目使用了Ribbon，name属性会作为微服务的名称，用于服务发现
* url: url一般用于调试，可以手动指定@FeignClient调用的地址
* decode404:当发生http 404错误时，如果该字段位true，会调用decoder进行解码，否则抛出FeignException
* configuration: Feign配置类，可以自定义Feign的Encoder、Decoder、LogLevel、Contract
* fallback: 定义容错的处理类，当调用远程接口失败或超时时，会调用对应接口的容错逻辑，fallback指定的类必须实现@FeignClient标记的接口
* fallbackFactory: 工厂类，用于生成fallback类示例，通过这个属性我们可以实现每个接口通用的容错逻辑，减少重复的代码
* path: 定义当前FeignClient的统一前缀



