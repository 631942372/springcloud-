### **使用idea创建项目**

![](/assets/简易搭建Eureka_1.png)

### 选择jdk版本1.8

![](/assets/简易搭建Eureka_2.png)

### 起项目名，next

![](/assets/简易搭建Eureka_3.png)

###选择cloud discovery-eureka Server 这样创建pom时会自动加入Eureka包

![](/assets/简易搭建Eureka_4.png)

###选择项目存放地址-next



![](/assets/Eureka项目结构.png)



###application创建后是properties属性，手动改为yml

###将Eureka配置输入

#### ![](/assets/Eureka基础配置.png)

#### 之后还需自己在启动类加上注解@EnableEurekaServer
![](/assets/EurekaServer启动类注解.png)


#### 启动，在浏览器输入localhost:8761
![](/assets/EurekaServer启动成功.png)

#### 可以看到一个Eureka已经启动成功，"No instances available" 表示无client注册





