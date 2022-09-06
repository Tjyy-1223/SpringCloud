# SpringCloud-MicroSerivices

SpringCloud+RabbitMQ+Docker+Redis+搜索+分布式，微服务技术栈学习

### Day1 SpringCloud01

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



### Day2 SpringCloud02

#### 1 Nacos配置管理

+ 统一配置管理

当微服务部署的实例越来越多，达到数十、数百时，逐个修改微服务配置就会让人抓狂，而且很容易出错。我们需要一种统一配置管理方案，可以集中管理所有实例的配置。

+ 从微服务中拉取配置 + 本地配置文件

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
