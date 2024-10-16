## 微服务

微服务是一种软件架构风格，由许多负责单一职责的小项目组合成大项目

### 导入项目

启动项目时,可以右键编辑选择启用的配置文件

![image-20241014201543918](new.6.%E5%BE%AE%E6%9C%8D%E5%8A%A1.assets/image-20241014201543918.png)

### 认识微服务

- 单体架构: 所有的业务功能集中在一个项目中开发,打成一个包部署
  - 优点: 架构简单,部署成本低
  - 缺点: 协作成本高,发布效率低,系统可用性差
- 微服务架构: 服务化思想指导下的一套最佳的架构方案。服务化，就是把单体架构中的功能模块拆分为多个独立项目。
  - 拆分粒度要小
  - 团队自治：一个小项目由一个团队负责
  - 服务自治：部署的时候可以单独部署

SpringCloud技术栈：SpringCloud集成了各种微服务功能组件,基于SpringBoot实现了这些组件的自动装配，从而提供了良好的开箱即用的体验。

- 服务注册发现：Eureka，Nacos，Consul
- 服务远程调用：OpenFeign，Dubbo
- 服务链路监控：Zipkin，Sleuth
- 统一配置管理：SpringCloudCOnfig，Nacos
- 统一网关路由：SpringCloudGateway，Zuul
- 流控、降级、保护：Hystix，Sentinel

依赖管理: 在父模块的pom.xml文件中配置如下,即可实现自动指定springcloud的组件的版本

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.7.12</version>
    <relativePath/>
</parent>

<properties>
    <spring-cloud.version>2021.0.3</spring-cloud.version>
    <spring-cloud-alibaba.version>2021.0.4.0</spring-cloud-alibaba.version>
</properties>

<dependencyManagement>
    <dependencies>
        <!--spring cloud-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>${spring-cloud.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-dependencies</artifactId>
            <version>${spring-cloud-alibaba.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

### 服务拆分原则

什么时候拆分?

- 创业型项目:先采用单体架构,快速开发,快速试错,随着规模扩大,逐渐拆分。
- 确定的大型项目：资金充足，目标明确，可以直接选择微服务，避免后续拆分的麻烦。

拆分的目标:

- 高内聚: 每个微服务的职责单一,包含的业务关联度高,完整度高。
- 低耦合:每个微服务的功能要相对独立，尽量减少对其他服务的依赖。

拆分方向：

- 纵向拆分：按照业务模块来划分。
- 横向拆分：抽取公共服务，提高复用性。

工程结构：

- 独立Project，每个服务都是一个Project
- Maven聚合，每个服务都是一个Module

### 拆分流程

1. 在父项目下创建新的Module
2. 拷贝并修改pom.xml依赖
3. 创建启动类和mapper,service等包
4. 复制配置文件application.yml等并修改数据库端口等配置
5. 拷贝代码

注意:pom.xml文件中有如下片段可能导致运行失败

```xml
<packaging>pom</packaging>
```

这个配置的含义是项目在打包时不会生成jar包,只做项目的整合。

### 远程调用

Spring提供的 RestTemplate 可以发送Http请求

存在问题: 服务调用者无法感知到服务提供者的变化

## 服务治理

注册中心原理:

1. 服务提供者向注册中心注册自己的信息
2. 服务调用者订阅服务提供者的信息
3. 进行负载均衡,选择其中一个提供者
4. 远程调用
5. 提供者和注册中心保持心跳连接,注册中心实时向调用者推送变更

![image-20241015195149341](new.6.%E5%BE%AE%E6%9C%8D%E5%8A%A1.assets/image-20241015195149341.png)

### Nacos

#### 服务注册

1. 引入maven依赖

   ```xml
   <dependency>
       <groupId>com.alibaba.cloud</groupId>
       <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
   </dependency>
   ```

2. 配置注册中心地址

   ```yml
   spring:
     application:
       name: item-service
     cloud:
       nacos:
         server-addr: 192.168.128.128:8848
   ```

#### 服务发现

1. 引入maven依赖

   ```xml
   <dependency>
       <groupId>com.alibaba.cloud</groupId>
       <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
   </dependency>
   ```

2. 配置注册中心地址

   ```yml
   spring:
     application:
       name: item-service
     cloud:
       nacos:
         server-addr: 192.168.128.128:8848
   ```

3. 调用API获取实例信息

   ```java
   @Resource
   DiscoveryClient discoveryClient;
   
   private void handleCartItems(List<CartVO> vos) {
       // 获取服务列表
       List<ServiceInstance> instances = discoveryClient.getInstances("item-service");
       // 负载均衡获取其中一个实例
       ServiceInstance instance = instances.get(RandomUtil.randomInt(instances.size()));
       // 获取实例的uri
       URI uri = instance.getUri();
   }
   ```

### OpenFeign

OpenFeign是一个声明式的http客户端

#### 快速入门

1. 导入依赖

   ```xml
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-openfeign</artifactId>
   </dependency>
   
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-loadbalancer</artifactId>
   </dependency>
   ```

2. 在启动类开启OpenFeign

   ```java
   @EnableFeignClients
   public class CartApplication {
   	...
   }
   ```

3. 编写FeignClient

   ```java
   @FeignClient("item-service")
   public interface ItemClient {
       @GetMapping("/items")
       List<ItemDTO> queryItemByIds(@RequestParam("ids") Collection<Long> ids);
   }
   ```

4. 使用FeignClient

   ```java
   @Resource
   // 依赖注入
   ItemClient itemClient;
   
   private void handleCartItems(List<CartVO> vos) {
   
       // 1.获取商品id
       Set<Long> itemIds = vos.stream().map(CartVO::getItemId).collect(Collectors.toSet());
       // 2.查询商品
       List<ItemDTO> items = itemClient.queryItemByIds(itemIds);
   }
   ```

#### 连接池

OpenFeign对Http请求做了优雅的封装，不过它底层发起Http请求依赖于其他的框架。共有以下三种：

- HttpURLConnection：默认实现，不支持连接池
- Apache HttpClient：支持连接池
- OKHttp：支持连接池

具体源码可以参考FeignBlockingLoadBalancerClient类中的delegate变量

1. 引入依赖

   ```xml
   <dependency>
       <groupId>io.github.openfeign</groupId>
       <artifactId>feign-okhttp</artifactId>
   </dependency>
   ```

2. 开启连接池功能

   ```yml
   feign:
     okhttp:
       enabled: true
   ```

#### 最佳实践

方案一：服务提供者拆分模块，将DTO和Client接口放在子模块中

![image-20241016211208988](new.6.%E5%BE%AE%E6%9C%8D%E5%8A%A1.assets/image-20241016211208988.png)

方案二：开一个通用的api模块，将所有的DTO和要暴露的接口放在这个模块内

![image-20241016211327555](new.6.%E5%BE%AE%E6%9C%8D%E5%8A%A1.assets/image-20241016211327555.png)

采用这两种方案的时候需要配置Feign扫描包

```java
@EnableFeignClients(basePackages = "com.hmall.api.client")
```

