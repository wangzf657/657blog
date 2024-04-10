---
title: SpringCloud学习笔记之GateWay
date: 2021-04-05 13:56:09
categories:
  - 后端
  - SpringCloud
tags:
  - 框架搭建
  - SpringCloud
---

## 网关配置

[官方文档](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#gateway-starter)

- 在网关模块 light-gateway 的 pom 文件引入依赖

```xml
<!--GateWay 网关-->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```

- nacos 后台,light-gateway-dev.yaml 中,编辑

```yaml
spring:
  cloud:
    gateway:
      # 路由数组[路由 就是指定当请求满足什么条件的时候转到哪个微服务]
      routes:
        # id 当前路由的标识, 要求唯一
        - id: light-mall
          # lb 从 nacos 中按照名称获取微服务,并遵循负载均衡策略
          uri: lb://light-mall
          # 断言(就是路由转发要满足的条件)
          predicates:
            # 当请求路径满足Path指定的规则时,才进行路由转发
            - Path=/light-mall/**
          ## 过滤器
          filters:
            # 转发之前去掉第一层的路径,过滤调用附加路径
            - StripPrefix=1
        - id: light-order
          uri: lb://light-order
          predicates:
            - Path=/light-order/**
          filters:
            - StripPrefix=1
        - id: light-user
          uri: lb://light-user
          predicates:
            - Path=/light-user/**
          filters:
            - StripPrefix=1
```

- 测试

```java
// mall模块编写测试接口
@GetMapping("/test-gateway")
public String testGateway(){
    return "我被调用啦~~~~~~~";
}

```

## 路由（route）

- 路由是网关的基本单元，由 ID、URI、一组 Predicate、一组 Filter 组成，根据 Predicate 进行匹配转发。

## 断言（Predicate）

- 路由转发的判断条件，目前 SpringCloud Gateway 支持多种方式，常见如：Path、Query、Method、Header 等，写法必须遵循 key=vlue 的形式,_**Route 和 Predicate 必须同时申明**_
- Spring Cloud GateWay 内置 Predicate 的使用,简单写几个例子,更多规则看[官方文档](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#gateway-request-predicates-factories)

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: light-mall
          uri: lb://light-mall
          predicates:
            # 当请求的路径为gate、rule开头的时，转发
            - Path=/gate/,/rule/
            # 在某个时间之前的请求才会被转发
            - Before=2022-01-20T17:42:47.789-07:00[America/Denver]
            # 名为chocolate的表单或者满足正则ch.p的表单才会被匹配到进行请求转发
            - Cookie=chocolate, ch.p
            # GET方法才会匹配转发请求
            - Method=GET
            # 当主机名为www.hd123.com的时候直接转发
            - Host=www.hd123.com
          filters:
            - StripPrefix=1
```

## 过滤器（Filter）

- 过滤器是路由转发请求时所经过的过滤逻辑，可用于修改请求、响应内容
- Spring Cloud GateWay 内置 Filter 的使用,简单写几个例子,更多规则看[官方文档](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#gatewayfilter-factories)

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: light-mall
          uri: lb://light-mall
          predicates:
            - Path=/light-mall/**
          filters:
            # 在请求路径前加上app
            - PrefixPath=/app
            # 访问localhost:9022/test,请求会转发到localhost:8001/app/test
            - RewritePath=/test, /app/test
            # 通过模板设置路径，转发的规则时会在路径前增加app，{path}表示原请求路径
            - SetPath=/app/{path}
            # 重定向，配置包含重定向的返回码和地址
            - RedirectTo=302, https://acme.org
            # 去掉某个请求头信息
            - RemoveRequestHeader=X-Request-Foo
```

## 自定义过滤器

Spring Cloud Gateway 虽然自带了很多内置的 Filter，但是针对一些业务场景，还是需要写一些过滤的业务逻辑，比如登录鉴权等；

### 自定义 GatewayFilter

Gateway 网关过滤器，又称局部过滤器，需要实现 GatewayFilter，并且跟单个路由绑定，其功能是针对访问的 URL 起到一定的过滤效果。

_**命名 xxxxGatewayFiterFactory，- xxxx - 为 yml 自定义的名称+GatewayFiterFactory；xxxx 也为配置文件 filter 下的 key 名称**_

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: light-mall
          uri: lb://light-mall
          predicates:
            - Path=/light-mall/**
          filters:
            - Authorize=afafa
```

```java
@Slf4j
@Component
public class AuthorizeGatewayFilterFactory extends  AbstractGatewayFilterFactory<AuthorizeGatewayFilterFactory.Config> {

        public AuthorizeGatewayFilterFactory() {
            super(Config.class);
        }


        @Override
        public List<String> shortcutFieldOrder() {
            return Arrays.asList("param");
        }

        @Override
        public GatewayFilter apply(Config config) {
            return (exchange, chain) -> {
                //获取请求参数中param对应的参数名 的参数值
                ServerHttpRequest request = exchange.getRequest();
                if (request.getQueryParams().containsKey(config.param)) {
                    request.getQueryParams().get(config.param).forEach((v) -> {
                        System.out.print("--局部过滤器--获得参数 " + config.param + "=" + v);
                    });
                }
                return chain.filter(exchange);//执行请求
            };
        }

        public static class Config {
            //对应配置在application.yml配置文件中的过滤器参数名
            private String param;

            public String getParam() {
                return param;
            }

            public void setParam(String param) {
                this.param = param;
            }
        }

    }
```

### 自定义 GlobalFilter

全局过滤器，需要实现 GlobalFilter 接口，这其中可以根据进行权限的验证，HTTP 请求的头部添加等等。

```java
    @Component
    @Slf4j
    public class AuthorizeGlobalFilter implements GlobalFilter, Ordered {
        @Override
        public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
            System.out.println("=========全局过滤器");
            String token = exchange.getResponse().getHeaders().getFirst("token");//获取第一个名为token的请求头
            //无权限
            if (StringUtils.isBlank(token)) {
                exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);//如果不是返回状态码401
                return exchange.getResponse().setComplete();
            }
            //有权限
            return chain.filter(exchange);
        }

        @Override
        public int getOrder() {
            // 值越小,约优先执行
            return 1;
        }
    }
```
