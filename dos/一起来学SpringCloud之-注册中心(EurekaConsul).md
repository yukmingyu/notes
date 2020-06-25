# 一起来学SpringCloud之-注册中心(EurekaConsul)

## - Spring Cloud简介

SpringCloud为开发者提供了在分布式系统中的一些通用的组件（如管理配置、服务发现、断路器、智能路由、微代理，控制总线，全局锁，决策竞选，分布式会话集群状态），使用Spring Cloud开发人员可以快速地完成实现这些模式的服务和应用程序。它们在任何分布式环境中都能很好地工作

## - Eureka

1. 是纯正的 servlet 应用，需构建成jar/war包部署
2. 使用了 Jersey 框架实现自身的 RESTful HTTP接口
3. peer之间的同步与服务的注册全部通过 HTTP 协议实现
4. 定时任务(发送心跳、定时清理过期服务、节点同步等)通过 JDK 自带的 Timer 实现
5. 内存缓存使用Google的guava包实现

### - Eureka Server

该操作只适合IDEA（因为它牛逼）

1.创建项目，选择Spring Initalizr -> Next

![创建项目，选择Spring Initalizr](一起来学SpringCloud之-注册中心(EurekaConsul).assets/eureka-1.png)

创建项目，选择Spring Initalizr

2.填写项目基本信息 -> Next

![填写项目基本信息](一起来学SpringCloud之-注册中心(EurekaConsul).assets/eureka-2.png)

填写项目基本信息

3.Cloud Discovery 选择当前最新版本（SpringBoot1.5.4），以及创建Eureka Server -> Next

![Cloud Discovery 选择当前最新版本（SpringBoot1.5.4），以及创建Eureka Server](一起来学SpringCloud之-注册中心(EurekaConsul).assets/eureka-3.png)

Cloud Discovery 选择当前最新版本（SpringBoot1.5.4），以及创建Eureka Server

4.Maven工程最后一步填写的东西 -> Finish

![Maven工程最后一步填写的东西](一起来学SpringCloud之-注册中心(EurekaConsul).assets/eureka-4.png)

Maven工程最后一步填写的东西

至此项目创建完毕，从项目中可以看到IDEA为我们生成的pom.xml

#### - pom.xml

```
<dependencies>
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-eureka-server</artifactId>
	</dependency>

	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-test</artifactId>
		<scope>test</scope>
	</dependency>
</dependencies>
```

#### - BattcnCloudDiscoveryApplication.java

```
@SpringBootApplication
@EnableEurekaServer
public class BattcnCloudDiscoveryApplication {

	public static void main(String[] args) {
		SpringApplication.run(BattcnCloudDiscoveryApplication.class, args);
	}
}
```

#### - bootstrap.yml

```
server:
  port: 8761  	#官方写的就是 8761
spring:
  application:
    name: battcn-cloud-discovery
  cloud:
    inetutils:
      ignored-interfaces:             #忽略docker0网卡以及 veth开头的网卡
        - docker0
        - veth.*
      preferred-networks:             #使用正则表达式,使用指定网络地址
        - 192.168
        - 10.0

eureka:
  instance:
    hostname: localhost 	#配置主机名
  client:
    register-with-eureka: false 	#配置服务注册中心是否以自己为客户端进行注册(配置false)
    fetch-registry: false 	#是否取得注册信息(配置false)
    service-url:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
      #配置eureka客户端的缺省域(该配置可能没有提示,请复制或者手动输入,切勿使用有提示的service-url会引起内置tomcat报错)
```

如果开启Eureka健康检查

```
eureka:
  client:
    healthcheck:
      enabled: true
```

官方：（WARNING）eureka.client.healthcheck.enabled=true应该只能设置application.yml。设置值bootstrap.yml将导致不期望的副作用，例如在具有UNKNOWN状态的eureka中注册。

官方链接：http://cloud.spring.io/spring-cloud-static/Dalston.SR1/#netflix-eureka-client-starter

#### - 测试

访问：http://localhost:8761/，看到下图表示Eureka Server已经运行成功

![测试结果](一起来学SpringCloud之-注册中心(EurekaConsul).assets/eureka-5.png)

测试结果

### - Eureka Client

项目创建方式同上，只是选择上从第二项的 `Eureka Server` 变成 `Eureka Discovery`

1.Eureka Client 可以看成是我们一个一个的服务，比如 `battcn-cloud-hello` ， `battcn-cloud-order` 它都是需要注册到Eureka中去

#### - pom.xml

```
<dependencies>
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-eureka</artifactId>
	</dependency>

	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-test</artifactId>
		<scope>test</scope>
	</dependency>
</dependencies>
```

#### - BattcnCloudHelloApplication.java

```
@SpringBootApplication
@EnableEurekaClient
@RestController
public class BattcnCloudHelloApplication {

    @Value("${spring.application.name}")
    String applicationName;

    @RequestMapping("/hello")
    public String home(@RequestParam String email) {
        return "My Name's :" + applicationName + " Email:" + email;
    }

    public static void main(String[] args) {
        SpringApplication.run(BattcnCloudHelloApplication.class, args);
    }
}
```

#### - bootstrap.yml

```
server:
  port: 8762
spring:
  application:
    name: battcn-cloud-hello
  cloud:
    inetutils:
      ignored-interfaces:             #忽略docker0网卡以及 veth开头的网卡
        - docker0
        - veth.*
      preferred-networks:             #使用正则表达式,使用指定网络地址
        - 192.168
        - 10.0
  profiles:
    active: eureka

---

# eureka 环境(我这里为了偷懒，就丢一个文件和一个项目了，用不同注册中心，请看博客对应的java代码)
spring:
  profiles: eureka
eureka:
  instance:
    hostname: localhost 	#配置主机名
  client:
    service-url:
      default-zone: http://localhost:8761/eureka/ 	#配置Eureka Server
---

#  consul 环境
spring:
  profiles: consul
  cloud:
    consul:
      host: localhost
      port: 8500
      enabled: true
      discovery:
        enabled: true
        prefer-ip-address: true
```

#### - 测试

访问：http://localhost:8762/，看到下图表示注册成功

![测试结果](一起来学SpringCloud之-注册中心(EurekaConsul).assets/eureka-6.png)

测试结果

代表我们的服务已经注册成功了

访问：http://localhost:8762/hello?email=1837307557@qq.com

可以看到：`My Name's :battcn-cloud-hello Email:1837307557@qq.com`



## - Consul

consul是分布式的、高可用、横向扩展的。consul提供的一些关键特性：

service discovery：consul通过DNS或者HTTP接口使服务注册和服务发现变的很容易，一些外部服务，例如saas提供的也可以一样注册。
health checking：健康检测使consul可以快速的告警在集群中的操作。和服务发现的集成，可以防止服务转发到故障的服务上面。
key/value storage：一个用来存储动态配置的系统。提供简单的HTTP接口，可以在任何地方操作。
multi-datacenter：无需复杂的配置，即可支持任意数量的区域。

总结：只要知道它是解决我上一部分提出的问题就行，其它的东西慢慢理解

### - 安裝方式

官网：https://www.consul.io/downloads.html

#### - Windows

![下载方式](一起来学SpringCloud之-注册中心(EurekaConsul).assets/consul-down.png)

下载方式

解压，在当前目录进入到命令界面输入 `consul agent -dev`

![启动Consul](一起来学SpringCloud之-注册中心(EurekaConsul).assets/consul-1.png)

启动Consul

访问：[http://localhost:8500](http://localhost:8500/)

![管理页](一起来学SpringCloud之-注册中心(EurekaConsul).assets/consul-2.png)

管理页

1.基于consul做注册中心，服务注册方式（还是用的battcn-cloud-hello） 只是做了改造

#### - pom.xml

```
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-consul-discovery</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

#### - BattcnCloudHelloApplication.java

```
@SpringBootApplication
@EnableDiscoveryClient
@RestController
public class BattcnCloudHelloApplication {

    @Value("${spring.application.name}")
    String applicationName;

    @RequestMapping("/hello")
    public String home(@RequestParam String email) {
        return "My Name's :" + applicationName + " Email:" + email;
    }


    public static void main(String[] args) {
        SpringApplication.run(BattcnCloudHelloApplication.class, args);
    }
}
```

#### - bootstrap.yml

```
server:
  port: 8762
spring:
  application:
    name: battcn-cloud-hello
  cloud:
    inetutils:
      ignored-interfaces:             #忽略docker0网卡以及 veth开头的网卡
        - docker0
        - veth.*
      preferred-networks:             #使用正则表达式,使用指定网络地址
        - 192.168
        - 10.0
  profiles:
    active: consul

---

# eureka 环境(我这里为了偷懒，就丢一个文件和一个项目了，用不同注册中心，请看博客对应的java代码)
spring:
  profiles: eureka
eureka:
  instance:
    hostname: localhost #配置主机名
  client:
    service-url:
      default-zone: http://localhost:8761/eureka/ #配置Eureka Server
---

#  consul 环境
spring:
  profiles: consul
  cloud:
    consul:
      host: localhost
      port: 8500
      enabled: true
      discovery:
        enabled: true
        prefer-ip-address: true
```

#### - 测试

访问：[http://localhost:8500](http://localhost:8500/)

![测试结果](一起来学SpringCloud之-注册中心(EurekaConsul).assets/consul-3.png)

测试结果

代表我们的服务已经注册成功了

访问：http://localhost:8762/hello?email=1837307557@qq.com

可以看到：`My Name's :battcn-cloud-hello Email:1837307557@qq.com`



全文代码：https://github.com/yukmingyu/battcn-cloud/tree/master/battcn-cloud-discovery

