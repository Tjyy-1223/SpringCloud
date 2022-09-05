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
