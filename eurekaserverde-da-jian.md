##### application.yml

```
eureka:
  server:
    # 该配置可以移除这种自我保护机制，防止失效的服务也被一直访问 (Spring Cloud默认该配置是 true)
    enable-self-preservation: false
    # 每隔3s扫描服务列表，移除失效服务
    response-cache-update-interval-ms: 3000
    response-cache-auto-expiration-in-seconds: 180
    # 该配置可以修改检查失效服务的时间，每隔10s检查失效服务，并移除列表 (Spring Cloud默认该配置是 60s)
    eviction-interval-timer-in-ms: 10000
  instance:
    hostname: 127.0.0.1
    status-page-url-path: /actuator/info #eureka注册中心的url link
    health-check-url-path: /actuator/health #健康检查的url
    # 该配置指示eureka服务器在接收到最后一个心跳之后等待的时间，然后才能从列表中删除此实例 (Spring Cloud默认该配置是 90s)
    lease-expiration-duration-in-seconds: 15
    # 该配置指示eureka客户端需要向eureka服务器发送心跳的频率  (Spring Cloud默认该配置是 30s)
    lease-renewal-interval-in-seconds: 5
    prefer-ip-address: true
  client:                            #eureka默认情况下,把自己当做客户端来注册自己,所以我们要禁用它,本地测试时都是关闭此两项
    registerWithEureka: false        #表示是否将自己注册到Eureka Server上，默认为true
    fetchRegistry: false             #表示是否从Eureka Server上获取注册信息，默认为true
    # registerWithEureka关闭后，defaultZone没有配置的必要。如果打开，即使配置为本机一样报错。也就是说defaultZone任何时候都没有配置为localhost的必要。
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
server:
  port: 8761
spring:
  application:
    name: eurekaServer

```

#### 

**另可以看到我们pom文件**

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.3.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.zxy</groupId>
    <artifactId>myspringcloud</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>myspringcloud-parent</name>
    <description>myspringcloud-parent</description>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.SR1</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
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

##### 引入时对准版本和复制需要的包即可

![](/assets/简易搭建eurekaServer_pom.png)

##### 此处使用的版本是2.1.3 。

##### 1.x版本跟2.x版本的区别是eureka的包是没有starter-netflix的。

##### 1.x使用↓

```
<artifactId>spring-cloud-eureka-server</artifactId>
```

##### 注意需要在启动类加上注解@EnableEurekaServer，才可开启eureka服务

##### ![](/assets/EurekaServer启动类注解.png)

##### 启动，在浏览器输入localhost:8761，可以看到EurekaServer已经启动成功。

##### "No instances available" 表示无client注册



