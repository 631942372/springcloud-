### **使用idea创建项目**

![](/assets/简易搭建Eureka_1.png)

### 选择jdk版本1.8

![](/assets/简易搭建Eureka_2.png)

### 起项目名，next

![](/assets/简易搭建Eureka_3.png)

### 选择cloud discovery-eureka Server 这样创建pom时会自动加入Eureka包

![](/assets/简易搭建Eureka_4.png)

### 选择项目存放地址-next

![](/assets/Eureka项目结构.png)

### application创建后是properties属性，手动改为yml

### 将Eureka配置输入

```
server:
   port: 8761  #端口号

eureka:
   instance:   # eureak实例定义
       hostname: localhost     #ip
  #eureka默认情况下,把自己当做客户端来注册自己,所以我们要禁用它,本地测试时都是关闭此两项，集群时改为true
   client:
       registerWithEureka: false  #表示是否将自己注册到Eureka Server上，默认为true
       fetchRegistry: false       #表示是否从Eureka Server上获取注册信息，默认为true
       serviceUrl:
           defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```

#### ![](/assets/Eureka基础配置.png)

#### 之后还需自己在启动类加上注解@EnableEurekaServer

![](/assets/EurekaServer启动类注解.png)

#### 启动，在浏览器输入localhost:8761

![](/assets/EurekaServer启动成功.png)

#### 可以看到一个Eureka已经启动成功，"No instances available" 表示无client注册

另可以看到我们pom文件

![](/assets/简易搭建eurekaServer_pom.png)

此处使用的版本是2.1.3,1.x版本跟2.x版本的区别是eureka的包是没有starter-netflix的。
1.x使用


