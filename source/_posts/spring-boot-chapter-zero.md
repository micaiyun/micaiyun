---
title: SpringBoot | 第一章：第一个SpringBoot应用
categories:
    - SpringBoot
tags:
    - java
    - spring
    - spring boot 
---

### SpringBoot | 第一章：第一个SpringBoot应用(非原创)

> [原文](https://blog.lqdev.cn/2018/07/11/springboot/chapter-one/)


### springboot简单介绍

#### 概述

> 随着动态语言的流行（Ruby、Groovy、Scala、Node.js），Java的开发显得格外的笨重：繁多的配置、低下的开发效率、复杂的部署流程以及第三方技术集成难度大。
> 在上述环境下，`Springboot`应运而生。它使用”习惯优于配置”（项目中存在大量的配置，此外还内置一个习惯性的配置，让你无须手动进行配置）的理念让你的项目快速运行起来。使用springboot很容易创建一个独立运行（运行jar，内嵌servlet容器）、准生产级别的基于Spring框架的项目，使用`springboot`你可以不用或者只需要很少的Spring配置。

------

#### Spring Boot 的核心功能

- 独立运行的Spring项目

  > Spring Boot可以以jar包的形式独立运行，运行一个Spring Boot项目只需要通过java -jar xx.jar。

- 内置Servlet容器

  > Spring Boot可选择内嵌Tomcat、Jetty或者Undertow，这样无须以war包形式部署。

- 提供starter简化maven配置

  > Spring提供了一系列的starter pom来简化maven依赖加载，例如：当你使用了spring-boot-starter-web时，会自动加入相关依赖，无需你手动一个一个的添加坐标依赖。

- 自动配置Spring

  > Spring Boot会根据在类路径中的jar包、类，为jar包里的类自动配置Bean，这样会极大地减少我们要使用的配置。当然，Spring Boot只是考虑了大多数的开发场景，并不是所有场景，若在实际开发中，我们需要自动配置bean，而Spring Boot没有提供支持，则可以自定义自动配置。

- 无代码生成和xml配置

  > Spring Boot的神奇的不是借助于代码生成来实现的，而是通过条件注解来实现的，这是Spring 4.x提供的新特性，Spring 4.x提倡使用java配置和注解配置相结合，而Spring Boot不需要任何xml配置即可实现Sping Boot的所有配置。

#### 优缺点

##### 优点

1. 快速构建项目：省略了繁琐且重复的xml配置，分分钟构建一个web工程；
2. 对主流开发框架的无配置集成：提供了很多Starter 依赖包，开箱即用，无需多余配置；
3. 项目可独立运行：无需外部依赖Servlet容器；
4. 极大地提供了开发、部署效率；
5. 监控简单：提供了actuator包，可以使用它来对你的应用进行监控。

##### 缺点

1. 依赖太多：一个简单的SpringBoot应用都有好几十M只有；
2. 缺少监控集成方案、安全管理方案：只提供基础监控，要实现生产级别的监控，监控方案需要自己动手解决；(后期讲解`soringCloud`时，会结合[pinpoint](https://github.com/naver/pinpoint)和[skywalking](https://github.com/apache/incubator-skywalking)分布式链路工具进行应用监控)

### 工程搭建

> 使用的工具为：`Spring Tool Suite(3.9.3.RELEASE)`
> SpringBoot：1.5.14.RELEASE

**Spring Tool Suite 下载地址：https://spring.io/tools/sts/all**

#### 创建项目

> 利用**Spring Initializr**进行快速创建项目

- **选择Dashboard–>CREATE SPRING STARTER PROJECT进行创建项目，或者可以选择file–>new–>Spring Starter Project，打开创建面板**

第一种方式：
[![img](https://wx-10045722.cos.ap-shanghai.myqcloud.com/blog-srping-boot-1/73137443.jpg)](https://wx-10045722.cos.ap-shanghai.myqcloud.com/blog-srping-boot-1/73137443.jpg)

第二种方式：
[![img](https://wx-10045722.cos.ap-shanghai.myqcloud.com/blog-srping-boot-1/72277454.jpg)](https://wx-10045722.cos.ap-shanghai.myqcloud.com/blog-srping-boot-1/72277454.jpg)

- 出现创建面板，填写项目信息

  > 这里url建议直接填写:`https://start.spring.io`(默认是http方式)

[![img](https://wx-10045722.cos.ap-shanghai.myqcloud.com/blog-srping-boot-1/71296322.jpg)](https://wx-10045722.cos.ap-shanghai.myqcloud.com/blog-srping-boot-1/71296322.jpg)

**maven相关命名说明**

1. **Group**：一般为逆向域名格式，如本博客域名为`lqdev.cn`，则group一般以`cn.lqdev`开头
2. **Artifact**：唯一标识，一般为项目名称。
   *具体maven相关信息，可自行搜索,这里只简单阐述*

- **选择依赖包和版本**

[![img](https://wx-10045722.cos.ap-shanghai.myqcloud.com/blog-srping-boot-1/891167.jpg)](https://wx-10045722.cos.ap-shanghai.myqcloud.com/blog-srping-boot-1/891167.jpg)

**除此下载包时，可能会比较慢，建议替换成阿里云的maven镜像**

#### 项目结构

```
- src
    -main
        -java
            -cn.lqdev.learning.springboot.chapter1
                #主函数，启动类，运行它如果运行了 Tomcat、Jetty、Undertow 等容器
                -Chapter1Application	
        -resouces
            #存放静态资源 js/css/images 等
            - statics
            #存放 html 模板文件
            - templates
            #主要的配置文件，SpringBoot启动时候会自动加载application.properties/bootstrap.properties		
            - application.properties
    #测试文件存放目录		
    -test
 # pom.xml 文件是Maven构建的基础，里面包含了我们所依赖JAR和Plugin的信息
- pom
```

[![img](https://wx-10045722.cos.ap-shanghai.myqcloud.com/blog-srping-boot-1/76756887.jpg)](https://wx-10045722.cos.ap-shanghai.myqcloud.com/blog-srping-boot-1/76756887.jpg)

#### pom依赖

> 由于使用了`Spring Initializr`直接创建项目，相关依赖自动添加好了。

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
 
	<groupId>cn.lqdev.learning</groupId>
	<artifactId>springboot-chapter1</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<name>chapter-1</name>
	<description>Spring Boot | 第一章：第一个Springboot应用</description>

    <!-- Springboot的版本，大家选择时，应该选择 RELEASE 版本 -->
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.14.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<java.version>1.8</java.version>
	</properties>

	<dependencies>
	    <!-- 内嵌了tomcat服务器，开发简单的web应用，此依赖即可  -->
 		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
        <!-- 测试包 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>
```

#### 主入口

```
/**
 *  启动类
 * @author oKong 
 *
 */
@SpringBootApplication
public class Chapter1Application {

	public static void main(String[] args) {
		SpringApplication.run(Chapter1Application.class, args);
	}
}
```

#### 编写controller

```
/**
 * 第一个springboot应用 
 * @author oKong
 *
 */
//@RestController = @Controller + @ResponseBody 默认直接返回json
@RestController
public class DemoController {

	@RequestMapping(value = "/demo", method = RequestMethod.GET)
	public String demo() {
		return "hello,SpringBoot!";
	}
}
```

#### 启动应用

> 直接`Chapter1Application`,右键 `run as` –> `Spring Boot App` 即可。

看见以下提示，说明启动成功：

```
2018-07-11 22:47:38.170  INFO 15396 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat started on port(s): 8080 (http)
```



**简单说明**

1. springboot 默认的端口号为：8080，此时浏览器访问：`127.0.0.1:8080/demo`即可查看。

2. 需要修改默认端口号时及上下文路径时，只需要在

   ```
   application.properties
   ```

   设置以下属性：

   ```
   # 端口号
   server.port=8888
   # 应用上下文路径
   server.context-path=/chapter1
   ```

访问：<http://127.0.0.1:8888/chapter1/demo>

[![img](https://wx-10045722.cos.ap-shanghai.myqcloud.com/blog-srping-boot-1/87292757.jpg)](https://wx-10045722.cos.ap-shanghai.myqcloud.com/blog-srping-boot-1/87292757.jpg)

**自此，一个简单的SpringBoot就开发完成了。比起原来的springmvc时的开发效率，简直是一个质的飞跃，无需再烦扰烦人的xml配置文件了。终于可以快乐的撸代码了~**

### 总结

> 目前互联网上很多大佬都有`springboot`系列教程，如有雷同，请多多包涵了。本文是作者在电脑前一字一句敲的，每一步都是亲身实际过的。若文中有所错误之处，还望提出，谢谢。