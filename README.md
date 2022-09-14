# SpringCloud-MicroSerivices

SpringCloud+RabbitMQ+Docker+Redis+搜索+分布式，微服务技术栈学习

## Day1 SpringCloud01

#### 1 认识微服务

+ 单体架构
+ 分布式架构
+ 服务管理：如何拆分、维护、调用
+ 微服务架构特征
+ SpringCloud

#### 2 服务拆分和远程调用

+ 服务拆分原则
+ 服务拆分demo实例
+ 实现微服务之间的远程调用 - RestTemplate
+ 提供者和消费者

#### 3 Eureka注册中心

+  Eureka的结构和作用

问题1:order-service如何得知user-service实例地址?

问题2:order-service如何从多个user-service实例中选择具体的实例?

问题3:order-service如何得知某个user-service实例是否依然健康，是不是已经宕机?

+ 搭建**eureka-server** - 是一个独立的微服务
+ eureka服务注册
+ eureka服务发现

#### 4 Ribbon负载均衡

+ 负载均衡原理
+ Ribbon服贼均衡流程总结
+ 自定义负载均衡策略
+ 饥饿加载

Ribbon默认是采用懒加载，即第一次访问时才会去创建LoadBalanceClient，请求时间会很⻓；而饥饿加载则会在项目启动时创建。

#### 5 **Nacos**注册中心

+ 认识和安装nacos 

```
sh startup.sh -m standalone
http://127.0.0.1:8848/nacos/#/login
```

+ 服务注册到nacos

```
引入nacos.discovery依赖
配置nacos地址spring.cloud.nacos.server-addr
```

+ Nacos服务分级存储模型

一级是服务，二级是集群，三级是实例。设置集群属性：

```
cloud:
  nacos:
    server-addr: localhost:8848
    discovery:
      cluster-name: SH
```

修改负载均衡规则：优先同一集群的负载调度

```
userservice:
  ribbon:
		NFLoadBalancerRuleClassName: com.alibaba.cloud.nacos.ribbon.NacosRule # 负载均衡规则
```

+ 权重配置 - 平滑升级

在nacos注册中心中修改权重，控制访问频率，权重越大则访问频率越高。

+ nacos环境隔离

Nacos提供了namespace来实现环境隔离功能，nacos中可以有多个namespace namespace下可以有group、service等 。不同namespace之间相互隔离，例如不同namespace的服务互相不可⻅。

**修改微服务的命名空间：**

```
spring:
  cloud:
    nacos:
      server-addr: localhost:8848
      discovery:
        cluster-name: HZ
        namespace: 492a7d5d-237b-46a1-a99a-fa8e98e4b0f9 # 命名空间，填ID
```

+ **Nacos**与**Eureka**的区别

```
Nacos与eureka的共同点:
	都支持服务注册和服务拉取
	都支持服务提供者心跳方式做健康检测
	
Nacos与Eureka的区别:
	Nacos支持服务端主动检测提供者状态:临时实例采用心跳模式，非临时实例采用主动检测模式 
	临时实例心跳不正常会被剔除，非临时实例则不会被剔除 
	Nacos支持服务列表变更的消息推送模式，服务列表更新更及时 
	Nacos集群默认采用AP方式，当集群中存在非临时实例时，采用CP模式;Eureka采用AP方式
```



## Day2 SpringCloud02

#### 1 Nacos配置管理

+ 统一配置管理

当微服务部署的实例越来越多，达到数十、数百时，逐个修改微服务配置就会让人抓狂，而且很容易出错。我们需要一种统一配置管理方案，可以集中管理所有实例的配置。

+ 从微服务中拉取配置 + 本地配置文件

```
SpringCloud 2.4版本之后不再优先读取bootstrap文件，导致bootstrap不起作用；需要在pom.xml文件中引入如下依赖后，就可以正常读取bootstrap.yml配置文件了

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bootstrap</artifactId>
    <version>3.0.2</version>
</dependency>

config:
  file-extension: yaml # 文件后缀名
  namespace: 9f97404c-51e3-4d6d-b4f7-3ed102069817
```

**注意：**配置文件的namespace要和启动文件的namespace相统一！！！不然会找不到配置文件中的配置

+ 配置热更新

```
在@Value注入的变量所在类上添加注解@RefreshScope

使用@ConfigurationProperties注解代替@Value注解
@Data
@Component
@ConfigurationProperties(prefix = "pattern")
public class PatternProperties {
    private String dateformat;
}
```

+ 多环境配置共享

在配置中添加 多环境配置共享属性值 ： pattern.envSharedValue

+ Nacos集群搭建



#### 2 Http客户端Feign

Feign是一个声明式的http客户端，官方地址:https://github.com/OpenFeign/feign ，其作用就是帮助我们优雅的实现http请求的发送，解决上面提到的问题。

+ **Feign**替代**RestTemplate**

1）引入依赖

```
 <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

2）添加注解 - @EnableFeignClients

3）编写Feign客户端

```java
@FeignClient("userservice")
public interface UserClient {
    @GetMapping("/user/{id}")
    User findById(@PathVariable Long id);
}
```

4）测试

+ 自定义Feign配置

Feign运行自定义配置来覆盖默认配置，可以修改一些配置

```
 feign:
  client:
		config:
			userservice: # 针对某个微服务的配置
				loggerLevel: FULL # 日志级别
```

+ Feign的性能优化

提高Feign的性能主要手段就是使用连接池代替默认的URLConnection。

1）引入依赖

```
<!--httpClient的依赖 --> 
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-httpclient</artifactId>
</dependency>
```

2)配置依赖池

```
feign:
  httpclient:
		enabled: true # 开启feign对HttpClient的支持 
		max-connections: 200 # 最大的连接数 
		max-connections-per-route: 50 # 每个路径的最大连接数
```

+ Feign的最佳实践

**分为继承方式和抽取方式**，其中比较常用的方式为抽取方式：

将Feign的Client抽取为独立模块，并且把接口有关的POJO、默认的Feign配置都放到这个模块中，提供给所有消费者使用。例如，将UserClient、User、Feign的默认配置都抽取到一个feign-api包中，所有微服务引用该依赖包，即可直接使用。



#### 3 统一网管GateWay

+ 为什么需要网关

Gateway网关是我们服务的守⻔神，所有微服务的统一入口。 网关的核心功能特性:

**请求路由和负载均衡：**一切请求都必须先经过gateway，但网关不处理业务，而是根据某种规则，把请求转发到某个微服务，这个过程叫做路由。当然路由的目标服务有多个时，还需要做负载均衡。

**权限控制：**网关作为微服务入口，需要校验用户是是否有请求资格，如果没有则进行拦截。

**限流：**当请求流量过高时，在网关中按照下流的微服务能够接受的速度来放行请求，避免服务压力过大。

+ GateWay快速入门

**基本步骤如下:**

1. 创建SpringBoot工程gateway，引入网关依赖 
2. 编写启动类
3. 编写基础配置和路由规则
4. 启动网关服务进行测试

```
server:
  port: 10010
spring:
  application:
    name: geteway
  cloud:
    nacos:
      server-addr: localhost:8848
      discovery:
        namespace: 0f5f7e8c-7704-4f33-a470-ca171eb75b9d
    gateway:
      routes:
        - id: user-service
          uri: lb://userservice
          predicates:
            - Path=/user/**
        - id: order-service
          uri: lb://orderservice
          predicates:
            - Path=/order/**
```

+ 路由断言工厂 Route Predicate Factory

我们在配置文件中写的断言规则只是字符串，这些字符串会被Predicate Factory读取并处理，转变为路由判断的条件

+ 过滤器工厂

GatewayFilter是网关中提供的一种过滤器，可以对进入网关的请求和微服务返回的响应做处理:

过滤器的作用是什么?

1）对路由的请求或响应做加工处理，比如添加请求头 

2）配置在路由下的过滤器只对当前路由的请求生效 

defaultFilters的作用是什么?	

1）对所有路由都生效的过滤器

+ 全局过滤器

全局过滤器的作用也是处理一切进入网关的请求和微服务响应，与GatewayFilter的作用一样。区别在于GatewayFilter通过配置定义，处理逻辑是固定的;而GlobalFilter的逻辑需要自己写代码实现。**定义方式是实现GlobalFilter接口：**

```java
@Order(-1)
@Component
public class AuthorizeFilter implements GlobalFilter {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        // 1 获取请求参数
        ServerHttpRequest request = exchange.getRequest();
        MultiValueMap<String, String> params = request.getQueryParams();
        // 2  获取auth参数
        String auth = params.getFirst("authorization");
        // 3 判断参数是否等于admin
        if("admin".equals(auth)){
            // 4 是 执行
            return chain.filter(exchange);
        }
        // 5 否 拦截
        exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
        return exchange.getResponse().setComplete();
    }
}
```

+ 过滤器执行顺序

请求路由后，会将当前路由过滤器和DefaultFilter、GlobalFilter，合并到一个过滤器链(集合)中，排序后依次执行每个过滤器:当过滤器的order值一样时，会按照 defaultFilter > 路由过滤器 > GlobalFilter的顺序执行。

+ 跨域配置

**跨域问题: **    **浏览器**禁止请求的发起者与服务端发生跨域ajax请求，请求被浏览器拦截的问题。

```
spring:
  cloud:
		gateway: 
			# 。。。
			globalcors: # 全局的跨域处理
				add-to-simple-url-handler-mapping: true # 解决options请求被拦截问题 
				corsConfigurations:
					'[/**]':
					allowedOrigins: # 允许哪些网站的跨域请求
						- "http://localhost:8090" allowedMethods: # 允许的跨域ajax的请求方式
              - "GET"
              - "POST"
              - "DELETE"
              - "PUT"
              - "OPTIONS"
          allowedHeaders: "*" # 允许在请求中携带的头信息
          allowCredentials: true # 是否允许携带cookie 
          maxAge: 360000 # 这次跨域检测的有效期
```



## Day3 Docker

#### 1 初识Docker

**什么是Docker**

+ Docker将用户程序与所需要调用的系统(比如Ubuntu)函数库一起打包
+ Docker运行到不同操作系统时，直接基于打包的函数库，借助于操作系统的Linux内核来运行

问题：

Docker如何解决大型项目依赖关系复杂，不同组件依赖的兼容性问题?

Docker如何解决开发、测试、生产环境有差异的问题?

Docker是一个快速交付应用、运行应用的技术，具备优势？



**Docker与虚拟机的区别**

**虚拟机：**模拟硬件设备，然后再运行另一个操作系统，应用调用内置操作系统，与Hypervisor与外部环境交互，性能较差

**Docker：**封装函数库，并没有模拟完整的操作系统

+ docker是一个系统进程;虚拟机是在操作系统中的操作系统 

+ docker体积小、启动速度快、性能好;虚拟机体积大、启动速度慢、性能一般



**Docker架构**

**镜像(Image):**Docker将应用程序及其所需的依赖、函数库、环境、配置等文件打包在一起，称为镜像。

**容器(Container):**镜像中的应用程序运行后形成的进程就是容器，只是Docker会给容器进程做隔离，对外不可⻅。

Docker是一个CS架构的程序，由两部分组成:

**服务端(server):**Docker守护进程，负责处理Docker指令，管理镜像、容器等 

**客户端(client):**通过命令或RestAPI向Docker服务端发送指令。可以在本地或远程向服务端发送指令。





#### 2 Docker的基本操作

**镜像操作**

```
docker build 构建镜像
docker pull 拉取镜像
docker rmi 删除镜像
docker push 推送镜像
docker save 保存镜像为一个压缩包
docker load 加载压缩包成为镜像
```

**docker 运行nginx的案例**

```
拉取nginx镜像：
docker pull nginx

查看目前拥有的镜像：
docker images
```



**容器相关操作**

容器保护三个状态:

+ 运行:进程正常运行 
+ 暂停:进程暂停，CPU不再运行，并不释放内存 
+ 停止:进程终止，回收进程占用的内存、CPU等资源

```
docker run:创建并运行一个容器，处于运行状态 
docker pause:让一个运行的容器暂停
docker unpause:让一个容器从暂停状态恢复运行 
docker stop:停止一个运行的容器
docker start:让一个停止的容器再次运行 
docker rm:删除一个容器
```

```
nginx容器的运行命令：
docker run --name containerName -p 80:80 -d nginx

如何在容器中运行代码：案例 - 进入Nginx容器，修改HTML文件内容，添加“tjyy hello”
进入容器：docker exec -it mn bash
进入目录: cd /usr/share/nginx/html
修改： sed -i -e 's#Welcome to nginx#传智教育欢迎您#g' -e 's#<head>#<head><meta charset="utf- 8">#g' index.html
停止：docker stop mn

redis:docker run --name mr -p 6379:6379 -d redis redis-server --appendonly yes
```



**数据卷：**

在之前的nginx案例中，修改nginx的html⻚面时，需要进入nginx内部。并且因为没有编辑器，修改文件也很麻烦。这就是因为容器与数据(容器内文件)耦合带来的后果。

**数据卷(volume)**是一个虚拟目录，指向宿主机文件系统中的某个目录;一旦完成数据卷挂载，对容器的一切操作都会作用在数据卷对应的宿主机目录了。

**数据卷操作命令：**

```
docker volume [COMMAND]
	create 创建一个volume
	inspect 显示一个或多个volume的信息 
	ls 列出所有的volume
	prune 删除未使用的volume
	rm 删除一个或多个指定的volume
```

