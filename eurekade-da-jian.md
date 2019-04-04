##### application.yml

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
#####引入时对准版本和复制需要的包即可	

![](/assets/简易搭建eurekaServer_pom.png)

##### 此处使用的版本是2.1.3 。

##### 1.x版本跟2.x版本的区别是eureka的包是没有starter-netflix的。

##### 1.x使用↓

```
<artifactId>spring-cloud-eureka-server</artifactId>
```

##### 注意需要在启动类加上注解@EnableEurekaServer，才可开启eureka服务

##### ![](/assets/EurekaServer启动类注解.png)

##### 启动，在浏览器输入localhost:8761，可以看到Eureka已经启动成功,"No instances available" 表示无client注册



