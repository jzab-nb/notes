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

#### 日志

OpenFeign只会在FeignClient所在的包的日志级别是DEBUG时才会输出日志。而且日志级别有四级：

- **NONE**：不记录任何日志，默认值
- **BASIC**：仅记录请求的方法，URL以及响应状态码和执行时间
- **HEADERS**：在BASIC的基础上，额外记录了请求和响应的头信息
- **FULL**：记录所有请求和响应的明细，包括头信息、请求体、元数据。

Feign默认的日志级别就是NONE，所以默认我们看不到请求日志。

1. 配置日志级别

```java
import feign.Logger;
import org.springframework.context.annotation.Bean;
/**
 * @author JZAB
 * @from http://vip.jzab.xyz
 */
public class LogConfig {
    @Bean
    public Logger.Level feignLogLevel(){
        return Logger.Level.FULL;
    }
}
```

2. 配置日志生效

```java
// 局部生效
@FeignClient(value = "item-service",configuration = LogConfig.class)
// 全局生效
@EnableFeignClients(basePackages = "com.hmall.api.client",defaultConfiguration = LogConfig.class)
```

### 网关

负责请求的路由、转发、身份校验

SpringCloudGateway

- spring官方出品
- 基于WebFlux响应式编程
- 无需调优即可获得优异性能

Netfilx Zuul

- Netflix出品
- 基于Servlet的阻塞式编程
- 需要调优才能获得和Gateway类似的性能

#### 路由配置

1. 导入相关依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>

<!--服务注册发现-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
<!--负载均衡-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-loadbalancer</artifactId>
</dependency>
```

2. yml文件配置路由

```yml
spring:
 cloud:
   gateway:
     routes:
     - id: route1 # 微服务的ID
       uri: lb://微服务名字 # 转发的地址,lb代表负载均衡,也可以传递http地址
       predicates: # 匹配规则,有xxx开头的路径会匹配上,可以用逗号分隔配置多个,也可以换行配置多个
       - Path=/xxx/**,/search/**
       filters: # 过滤器,
       - AddRequestHeader=ab, 114514
     default-filters: # 默认过滤器,所有路由都生效
       - AddRequestHeader=ab, 114514
```

##### **路由断言**

![image-20241021194054773](new.6.%E5%BE%AE%E6%9C%8D%E5%8A%A1.assets/image-20241021194054773.png)

##### **路由过滤器**

![image-20241021194217930](new.6.%E5%BE%AE%E6%9C%8D%E5%8A%A1.assets/image-20241021194217930.png)

##### **网关处理流程**

![image-20241021195152535](new.6.%E5%BE%AE%E6%9C%8D%E5%8A%A1.assets/image-20241021195152535.png)

##### 自定义过滤器  

网关的过滤器有两种:

- GateWayFilter：路由过滤器，作用于任意指定的路由，默认不生效，要配置到路由后生效
- GlobalFilter：全局过滤器，作用于所有的路由，声明后自动生效。

**自定义全局过滤器**

```java
// 注册成Spring组件
@Component
public class MyGlobalFilter implements GlobalFilter, Ordered {
    @Override
    // 主要实现过滤的方法  
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        // 获取请求头
        exchange.getRequest().getHeaders().get("token");
        // 过滤器放行
        return chain.filter(exchange);
    }

    @Override
    // 排序,值越小越优先执行
    public int getOrder() {
        return 0;
    }
}

```

**自定义路由过滤器**

不能直接定义,而是要定义工厂类

且工厂类的后缀必须为GatewayFilterFactory

前缀则会变成配置在yml文件中的部分

```java
@Component
@Slf4j
// 定义工厂类
public class AbGatewayFilterFactory extends AbstractGatewayFilterFactory<AbGatewayFilterFactory.Config> {

    @Override
    // 返回实际的过滤器
    public GatewayFilter apply(Config config) {
        // 利用装饰模式实现排序
        return new OrderedGatewayFilter((exchange, chain) -> {
            log.error("{}-{}",config,exchange.getRequest().getURI());
            return chain.filter(exchange);
        },1);
    }

    @Data
    // 定义存储配置内容的类
    public static class Config{
        private String name;
        private Integer age;
    }

    @Override
    // 定义参数位置和变量的映射关系
    public List<String> shortcutFieldOrder() {
        return List.of("name","age");
    }

    // 构造函数,调用父类的构造函数,并把配置类传入
    public AbGatewayFilterFactory(){
        super(Config.class);
    }
}
```

yml文件配置

```yml
default-filters:
	- Ab=JZAB,100
```



### 登录校验

1. 在网关部分配置过滤器，处理token并将信息写入请求头

```java
// 注册成Spring组件
@Component
@Slf4j
public class MyGlobalFilter implements GlobalFilter, Ordered {
    @Resource
    AuthProperties authProperties;

    @Resource
    JwtTool jwtTool;

    private final AntPathMatcher antPathMatcher = new AntPathMatcher(  );

    public boolean checkPath(String path){
        List<String> include = authProperties.getIncludePaths();
        List<String> exclude = authProperties.getExcludePaths();
        // 是否允许放行
        for (String p : exclude) {
            if(antPathMatcher.match(p,path)) return true;
        }
        return false;
    }

    @Override
    // 主要实现过滤的方法
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        log.error("{}",exchange.getRequest( ).getURI());
        // 1.获取请求头
        ServerHttpRequest request = exchange.getRequest( );
        // 2.判断是否允许放行
        String path = request.getURI().getPath();
        if(checkPath(path)) return chain.filter(exchange);

        // 3.获取token
        String token = request.getHeaders( ).getFirst("authorization");
        try{
            // 4.解析token
            Long id = jwtTool.parseToken(token);
            // 5.传递用户信息
            log.error("{}",id);
            // 构建请求头并获取新的exchange
            exchange = exchange.mutate( ).request(
                    req -> req.header("user-id", id.toString( ))
            ).build( );

        }catch (Exception e){
            // 产生异常了,返回401
            ServerHttpResponse response = exchange.getResponse( );
            response.setStatusCode(HttpStatus.UNAUTHORIZED);
            return response.setComplete();
        }
        // 6.放行
        return chain.filter(exchange);
    }

    @Override
    // 排序,值越小越优先执行
    public int getOrder() {
        return 0;
    }
}
```

2. 在公共模块书写拦截器相关配置

拦截器

```java
public class UserIdInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        // 获取用户ID
        String id = request.getHeader("user-id");
        System.out.println("id="+id );
        // 不为空则保存用户信息
        if(StrUtil.isNotBlank(id)){
            UserContext.setUser(Long.valueOf(id));
        }
        return true;
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        UserContext.removeUser();
    }
}
```

MVC配置类

```java
@Configuration
// 条件,当有SpringMVC时才引入
@ConditionalOnClass(DispatcherServlet.class)
public class MvcConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new UserIdInterceptor());
    }
}
```

自动装配配置文件 resources/META-INF/spring.factories

```factoris
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.hmall.common.config.MyBatisConfig,\
  com.hmall.common.config.JsonConfig,\
  com.hmall.common.config.MvcConfig
```

feign的拦截器配置，使用Feign发送请求时会带上请求头

```java
@Bean
public RequestInterceptor requestInterceptor(){
    return requestTemplate -> {
        Long user = UserContext.getUser( );
        if(user!=null) requestTemplate.header("user-id", user.toString());
    };
}
```

### 配置管理

不同微服务之间重复的配置过多

有些配置是业务配置经常变动

网关路由配置可能存在变动

#### 配置共享

在nacos的界面上进行配置，将application.yml中的配置按模块划分

${}代表变量,里面可以用冒号指定默认值

```yml
spring:
  datasource:
    url: jdbc:mysql://${hm.db.host:192.168.128.128}:${hm.db.port:3306}/${hm.db.database}?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: ${hm.db.un:root}
    password: ${hm.db.pw:102099}
mybatis-plus:
  configuration:
    default-enum-type-handler: com.baomidou.mybatisplus.core.handlers.MybatisEnumTypeHandler
  global-config:
    db-config:
      update-strategy: not_null
      id-type: auto
```

![image-20241022212542327](new.6.%E5%BE%AE%E6%9C%8D%E5%8A%A1.assets/image-20241022212542327.png)

要想让共享配置生效，还需要进行如下操作

1. 引入依赖

   ```xml
   <!--配置管理-->
   <dependency>
       <groupId>com.alibaba.cloud</groupId>
       <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
   </dependency>
   <!--读取bootstrap文件-->
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-bootstrap</artifactId>
   </dependency>
   ```

2. 新建bootstrap.yml文件

   ```yml
   spring:
     application:
       name: item-service
     profiles:
       active: dev
     cloud:
       nacos:
         server-addr: 192.168.128.128:8848
         config:
           file-extension: yaml # 文件后缀名
           shared-configs: # 共享配置
             - dataId: shared-jdbc.yaml
             - dataId: shared-log.yaml
             - dataId: shared-swagger.yaml
   ```

3. 精简application.yml文件

   ```yml
   server:
     port: 8081
   hm:
     db:
       database: hm-item
     swagger:
       title: "黑马商城商品服务接口文档"
       desc: "黑马商城订单服务接口文档"
       package: com.hmall.item.controller
   ```

#### 配置热更新

修改完配置后不需要重启变动就能生效。

前提条件:

1. nacos上要有一个和微服务相关的配置文件 命名规则: 服务名[-活动配置文件].后缀 例如: item-service-dev.yaml

2. 要用特定的方式读取要热更新的属性

   ```java
   @Data
   @ConfigurationProperties(prefix = "hm.jwt")
   public class JwtProperties {
   
   }
   
   ```

   

#### 动态路由

