# spring-cloud-sidecar集成非java服务端（php,node.js,paython）到spring cloud
-

## 一、大致介绍

``` 
1、在一些稍微复杂点系统中，往往都不是单一代码写的服务，而恰恰相反集成了各种语言写的系统，并且我们还要很好的解耦合集成到自己的系统中；
2、出于上述现状，SpringCloud 生态圈中给我们提供了很好的插件式服务，利用 sidecar 我们也可以轻松方便的集成异构系统到我们自己的系统来；
3、而本章节目的就是为了解决轻松简便的集成异构系统到自己的微服务系统中来的；
```


## 二、实现步骤

### 2.1 添加 maven 引用包
``` 
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.springcloud</groupId>
	<artifactId>spring-sidecar</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<name>spring-sidecar</name>
	<description>Demo project for Spring Boot</description>

	<parent>
		<groupId>com.springcloud</groupId>
		<artifactId>springcloud</artifactId>
		<version>1.0-SNAPSHOT</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<dependencies>
		<!-- 异构系统模块 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-netflix-sidecar</artifactId>
		</dependency>

		<!-- 客户端发现模块，由于文档说 Zuul 的依赖里面不包括 eureka 客户端发现模块，所以这里还得单独添加进来 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-eureka</artifactId>
		</dependency>
	</dependencies>

</project>

```


### 2.2 添加spring-sidecar应用配置文件application.yml
``` 
spring:
  application:
    name: spring-sidecar
server:
  port: 8210
eureka:
  datacenter: SpringCloud   # 修改 http://localhost:8761 地址 Eureka 首页上面 System Status 的 Data center 显示信息
  environment: Test         # 修改 http://localhost:8761 地址 Eureka 首页上面 System Status 的 Environment 显示信息
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka
    healthcheck:  # 健康检查
      enabled: true

#####################################################################################################
# 异构微服务的配置， port 代表异构微服务的端口；health-uri 代表异构微服务的操作链接地址
sidecar:
  port: 8205
  health-uri: http://localhost:8205/health.json   #此处为非java服务端的一个接口地址，返回必须为 {"status": "UP"}，用着eureka监听
#####################################################################################################

```




### 2.3 添加spring-sidecar微服务启动类pringSidecarApplication.java
``` 
package com.springcloud.springsidecar;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.cloud.netflix.sidecar.EnableSidecar;

/**
 *  * 集成异构微服务系统到 SpringCloud 生态圈中(比如集成 nodejs 微服务)。
 *
 * 注意 EnableSidecar 注解能注册到 eureka 服务上，是因为该注解包含了 eureka 客户端的注解，该 EnableZuulProxy 是一个复合注解。
 *
 * @EnableSidecar --> { @EnableCircuitBreaker、@EnableDiscoveryClient、@EnableZuulProxy } 包含了 eureka 客户端注解，同时也包含了 Hystrix 断路器模块注解，还包含了 zuul API网关模块。
 */
@EnableSidecar
//@EnableEurekaClient
@SpringBootApplication
public class SpringSidecarApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringSidecarApplication.class, args);
	}
}

```



## 三、测试

``` 
/****************************************************************************************
 一、集成非java服务端（php,node.js,paython）到spring cloud 生态圈中(比如集成 php 后端服务)（正常情况测试）：

 1、启动 spring-eureka-server 模块服务，启动1个端口8761；
 2、启动 spring-zuul 模块服务，启动1个端口8200；
 3、启动 spring-sidecar 模块服务，启动1个端口8210；
 4、启动 php后端 服务，启动1个端口8205；

 6、新起网页页签，输入 http://localhost:8205/ 正常情况下是:
 "{"index": "欢迎来php融入spring cloud欢迎界面"}";
 7、新起网页页签，输入 http://localhost:8205/findName 正常情况下是:
 " {"name": "Li Ming"}";
 8、新起网页页签，然后输入 http://localhost:8205/health.json，正常情况下是: 
 "{"status":"UP"}"；

 总结一：php后端服务，自己访问自己都是正常的；

 9、新起网页页签，输入 http://localhost:8200/spring-sidecar/ 正常情况下是:
 "{"index": "欢迎来php融入spring cloud欢迎界面"}";
 10、新起网页页签，输入 http://localhost:8200/spring-sidecar/findName 正常情况下是:
 " {"name": "Li Ming"}";
 11、新起网页页签，然后输入 http://localhost:8200/spring-sidecar/health.json，正常情况下是: 
 "{"status":"UP"}"；

 总结二：通过在yml配置文件中添加 sidecar 属性，就可以将php后端服务添加到SpringCloud生态圈中，完美无缝衔接；
 ****************************************************************************************/

```


## 四、下载地址

[https://github.com/chouxin/springcloud.git](https://gitee.com/ylimhhmily/SpringCloudTutorial.git)






























