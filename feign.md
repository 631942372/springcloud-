#### ![](/assets/feign与restTemplate.png)

#### 加入feign相关依赖

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

#### 因为是在consumer调用provider的服务，因此我们在consumer项目启动类加上@EnableFeignClients开启feign



#### consumer项目创建feign接口

```
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.*;

@FeignClient("sc-provider")   //指定服务别名
public interface ProviderFeignService {

    //指定调用服务接口路径    @PathVariable  @RequestParam
    //当使用feign传参数的时候,需要加上@RequestParam注解,否则对方服务无法识别参数;
    @GetMapping(value = "/provider/helloFeign")//,method = RequestMethod.GET
    String HelloFeign(@RequestParam("message")String message);


    @GetMapping(value = "/provider/{message}")//,method = RequestMethod.GET
    String HelloPathFeign(@PathVariable("message")String message);
}

```

#### 新增provider项目controller调用feign接口

```

import com.zxy.consumer.feign.ProviderFeignService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

@RestController
@RequestMapping("/consumer")
public class ConsumerController {
    @Autowired
    private RestTemplate restTemplate;  //springBoot提供组件,底层实际上是httpclient eureka默认整合负载均衡器

    @Autowired
    private ProviderFeignService providerFeignService;

    @GetMapping("/hello")
    public String Hello(){
        return "Hello,I am consumer";
    }

    @GetMapping("/callProvider")
    public String call(){
        //两种方式，一种是服务别名调用，一种是直接调用
        //String str = restTemplate.getForObject("http://127.0.0.1:8801/provider/hello",String.class);
        String str = restTemplate.getForObject("http://sc-provider/provider/hello",String.class);  //需在启动类开启ribbin
        return "consumer call provider ,result: "+str;
    }

    @GetMapping("/helloFeign")
    public String helloFeign(){
        return providerFeignService.HelloFeign("provider over over!");
    }

    @GetMapping("/{message}")
    public String helloPathFeign(@PathVariable("message")String message){
        return providerFeignService.HelloPathFeign(message);
    }
}
```

#### 新增provider被调用接口

```
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/provider")
public class ProviderController {

    @Value("${server.port}")
    private String serverPort;

    @GetMapping("/hello")
    public String Hello(){
        return "Hello,I am provider,port： "+serverPort;
    }

    @GetMapping(value="/helloFeign")
    public String Hello(String message){
        return "Hello,I am provider Feign url,port： "+serverPort+" ,message: "+message;
    }

    @GetMapping(value="/{message}")
    public String HelloPath(@PathVariable("message")String message){
        return "HelloPath,I am provider Feign url,port： "+serverPort+" ,message: "+message;
    }
}
```

启动项目，部署集群，可以发现调用接口时，调用端口号在改变，说明feign整合的ribbin起了负载均衡作用

调用consumer接口 

[http://localhost:8701/consumer/helloFeign](http://localhost:8701/consumer/helloFeign)

[http://localhost:8701/consumer/11231](http://localhost:8701/consumer/11231)

