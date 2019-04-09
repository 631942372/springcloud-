#### **消费者pom**

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.4.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.zxy</groupId>
    <artifactId>consumer</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>consumer</name>
    <description>消费者</description>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.SR1</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

#### 消费者application.yml

```
server:
    port: 8701
spring:
    application:
        name: consumer # 服务别名
eureka:
    client:
        serviceUrl:
            defaultZone: http://localhost:8761/eureka/
    fetch-registry: true          #检索服务
    register-with-eureka: true    #将服务注册到eureka
```

#### 消费者启动类

```
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;

@SpringBootApplication
@EnableEurekaClient
public class ConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConsumerApplication.class, args);
    }

    //解决RestTemplate没找到问题
    @Bean
    @LoadBalanced    //开启ribbin负载均衡，就可以用别名调用
    public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }
}
```

消费者controller

```
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

@RestController
@RequestMapping("/consumer")
public class ConsumerController {
    @Autowired
    private RestTemplate restTemplate;  //springBoot提供组件,底层实际上是httpclient eureka默认整合负载均衡器

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
}
```

修改provider的端口，启动，调用[http://xxxxxx/consumer/callProvider](http://zhouxiayu:8701/consumer/callProvider)   可以看到端口轮询切换

