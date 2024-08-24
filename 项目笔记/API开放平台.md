# API开放平台

场景:

​	前端需要现成的数据接口

​	需要使用别人现成的接口

需要考虑的点:

1. 防止攻击

2. 禁止随意调用

3. 计费, 流量保护, 统计调用次数

4. api的接入

## 项目介绍

一个提供API接口调用的平台，用户可以注册登录，开通接口调用权限。用户可以使用接口，并且每次调用会进行统计。管理员可以发布接口、下线接口、接入接口，以及可视化接口的调用情况、数据

![image-20231007160858872](API%E5%BC%80%E6%94%BE%E5%B9%B3%E5%8F%B0.assets/image-20231007160858872.png)

## 技术选型

前端: Ant Design Pro，React，Ant Design Pro components，Umi，Umi Request(Axios的封装)

后端：Spring Boot，Spring Boot Starter(SDK开发)，？？？(网关实现)，

## 快捷键

ctrl+shift+R全局搜索替换

## 数据库表设计

### 接口信息表

interface_info: 

​	id, isDelete, createTime, updateTime

​	name(接口名称),

​	description,

​	url,

​	type,

​	requestHeader

​	responseHeader

​	status 状态 0- 关闭 1-开启

## API签名认证

本质:

1. 签发签名
2. 使用签名(校验签名)

为什么需要?

1. 保证安全性,不能随便一个人调用

怎么实现:

​	accesskey: 调用的标识,知道是谁

​	secretKey: 密钥

​	类似用户名和密码，但他是无状态的

用户参数和secretKey拼接, 计算哈希值, 将哈希值发送给服务端

服务端根据参数和secretKey用相同的方法计算哈希值, 进行比对

防止重放: 加随机数, 加时间戳

## 开发starter

### 0.删除pom.xml中的打包插件

### 1.书写配置类

```java
@Configuration
// 自动装配的配置前缀
@ConfigurationProperties("abapi.client")
@Data
@ComponentScan
public class AbApiClientConfig {
    // 要被自动装配的属性
    private String accessKey;
    private String secretKey;
    // 放到bean中的实体类
    @Bean
    public AbapiClient abapiClient(){
        return new AbapiClient(accessKey, secretKey);
    }
}
```

### 2.书写spring.factories文件

在resources目录下，创建META-INF/spring.factories文件并写入如下内容

```factories
# Spring Boot Starter
org.springframework.boot.autoconfigure.EnableAutoConfiguration=xyz.jzab.abapiclientsdk.AbApiClientConfig
```

### 3.安装到maven本地仓库

点击maven工具的install即可

# API网关

统一的处理请求

## 应用场景:

### 1.路由

网关记录路由地址和实际地址的对应关系，收到请求后进行转发

/a => 接口A

/b => 接口B

### 2.负载均衡

在路由的基础上，，一个地址配置多台机器，随机转发到某一台机器

### 3.统一鉴权

判断用户是否有权限进行操作，所有接口都统一判断权限

### 4. 统一处理跨域

网关统一处理跨域，不用在每个项目里单独处理

### 5.统一业务处理

每个项目中相同的业务逻辑进行提取，如统计调用次数

### 6.访问控制

黑白名单，控制业务逻辑之外的用户访问权限，如有人DDOS那就禁止他访问

### 7.发布控制

灰度发布，比如接口发布新版本，先分配少部分流量过去，再逐渐增大流量，直到把就接口全部替代

### 8.流量染色

给请求添加一些标识，一般是设置在请求头中，比如标识请求来自网关的转发而不是直接请求的后面的地址

### 9.统一接口保护

#### 限制请求

比如限制请求的大小

#### 信息脱敏

删除响应头中的敏感信息

#### 降级(熔断)

接口调用失败，让用户访问其他的

#### 限流

限制每分钟最高访问次数

#### 超时时间

长时间不返回直接中断

### 10.统一日志

记录每次请求，响应的信息

### 11.统一文档

将下游项目的文档进行聚合

## 分类

1. 全局网关(接入层网关): 作用是负载均衡、请求日志等、不和业务逻辑绑定
2. 业务网关(微服务网关): 会有一些业务逻辑，作用是将请求转发到不同的业务/项目/接口/服务

## 实现

1. Nginx（全局网关）、Kong网关（API网关）
2. Spring Cloud Gateway （取代了Zuul）性能高、可以用Java代码来写逻辑

## Spring Cloud Gateway使用

路由: 根据什么规则转发到什么地址

断言: 一组规则，条件，规定何时转发

过滤器: 与web开发中的过滤器相同，统一处理请求

### 工作流程

1. 客户端发起请求
2. Handler Mapping：根据断言转发请求
3. Web Handler：处理请求(一层层过滤器)
4. 实际调用服务

<img src="API%E5%BC%80%E6%94%BE%E5%B9%B3%E5%8F%B0.assets/image-20231022205246803.png" alt="image-20231022205246803" style="zoom:50%;" />

### 两种配置方式

1. 在yaml配置文件中进行配置（方便,规范）

   ```yaml
   spring:
     cloud:
       gateway:
         routes:
         - id: after_route
           uri: https://example.org
           predicates:
   		- 各种断言
   ```

   

2. 使用代码流程进行配置(灵活,但相对麻烦)

### 开启日志

```yaml
logging:
  level:
    org:
      springframework:
        cloud:
          gateway: trace
```

### 断言

```yaml
1. After 在某个时间之后
After=2017-01-20T17:42:47.789-07:00[America/Denver]
2. Before 在某个时间之前
Before=2017-01-20T17:42:47.789-07:00[America/Denver]
3. Between 在两个时间之间
Between=2017-01-20T17:42:47.789-07:00[America/Denver], 2017-01-21T17:42:47.789-07:00[America/Denver]
4. Cookie 请求头中包含对应键值的Cookie
Cookie=键, 值
5. Header 请求头中包含对应的键值对,且值符合某个规则(如正则)
Header=X-Request-Id, \d+
6. Host 根据主机名(域名)来进行跳转
Host=**.somehost.org,**.anotherhost.org
7. Method 根据请求的方法
Method=GET,POST
8. Path 根据请求的路径进行匹配(通配符为**而不是*)
Path=/red/{segment},/blue/{segment}
9. Query 请求参数(问号传参)包含某个或某个参数有某个值的时候进行转发
Query=green
Query=val, 1
10. RemoteAddr 根据请求方的地址来判断是否转发
RemoteAddr=192.168.1.1/24
11. Weight，分组按权重分配，
Weight=group1, 8
Weight=group1, 2
12. XForwardedRemoteAddr 根据XForwarded远程地址来判断是否转发
XForwardedRemoteAddr=192.168.1.1/24
```

### 过滤器

```yaml
语法:
spring:
  cloud:
    gateway:
      routes:
      - id: route1
        uri: 转发到的地址
        predicates:
        - 断言选项
        filters:
        - 过滤器选项
1.添加请求头
AddRequestHeader=key, value
2.添加请求参数
AddRequestParameter=key, value

```

### 默认过滤器

给所有请求使用的过滤器

```yaml
spring:
  cloud:
    gateway:
      default-filters:
      - AddResponseHeader=X-Response-Default-Red, Default-Blue
      - PrefixPath=/httpbin
```

