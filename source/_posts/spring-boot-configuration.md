---
title: SpringBoot | 第三章：springboot配置详解
categories:
    - SpringBoot
tags:
    - java
    - spring
    - spring boot 
---
> 非原创 [原文](https://blog.lqdev.cn/2018/07/14/springboot/chapter-third/)

基于springboot的`约定优于配置`的原则，在多数情况下，启动一个应用时，基本上无需做太多的配置，应用就能正常启动。但在大部分开发环境下，添加额外配置是无所避免的，比如自定义应用端口号(比较在机器比较少的情况下，一台机器还是需要部署多个应用的，当然利用`docker`的话，是可避免的，这是后话了)、mq的服务地址、缓存服务的服务地址、数据库的配置等，都或多或少的需要一些外部的配置项。

## 配置文件格式简要说明

`springboot`默认的全局配置文件名为application.properties或者application.yml(spring官方推荐使用的格式是`.yml`格式，目前官网都是实例都是使用yml格式进行配置讲解的),应用启动时会自动加载此文件，无需手动引入。除此之外还有一个`bootstrap`的全局文件，它的加载顺序在`application`配置文件之前，主要是用于在**应用程序上下文的引导阶段**，在后期讲解`springCloudConfig`时，主要是利用此特性，进行配置文件的动态修改，在此不表，在通常情况下，此两个配置文件是没有差别的，所以一般上都只需要配置`application`即可。

## 自定义属性值

`application.properties`配置文件支持自定义属性的支持，比如

```
blog.address=https://blog.lqdev.cn
blog.author=oKong
```



然后可通过`@Value("${blog.author}")`的形式获取属性值。

```
@RestController
public class DemoController {

	@Value("${blog.address}")
	String address;
	
	@Value("${blog.author}")
	String author;
	
	@Value("${blog.desc}")	
	String desc;
	
	@RequestMapping("/")
	public String demo() {
		return desc;
	}
}
```



> 这里提醒下，在填写一些默认的比如，数据库属性时，可使用`alt+/`的方式，IDE会自动显示提示，避免了手动嵌入属性值或者忘记属性的尴尬。

[![img](https://wx-10045722.cos.ap-shanghai.myqcloud.com/blog-srping-boot-1/95060000.jpg)](https://wx-10045722.cos.ap-shanghai.myqcloud.com/blog-srping-boot-1/95060000.jpg)

**关于自定义属性时，特别是一些公用包，会使用到属性值时，建议在创建additional-spring-configuration-metadata.json属性元文件，这样在使用上述快捷方式时，会进行提示，包括属性名和属性说明，这样也方便调用者询问属性名是啥。**

[![img](https://wx-10045722.cos.ap-shanghai.myqcloud.com/blog-srping-boot-1/32633042.jpg)](https://wx-10045722.cos.ap-shanghai.myqcloud.com/blog-srping-boot-1/32633042.jpg)

[![img](https://wx-10045722.cos.ap-shanghai.myqcloud.com/blog-srping-boot-1/64663506.jpg)](https://wx-10045722.cos.ap-shanghai.myqcloud.com/blog-srping-boot-1/64663506.jpg)

相关`configuration-metadata`说明可查看：<https://docs.spring.io/spring-boot/docs/current/reference/html/configuration-metadata.html>

## 属性引用

在配置文件中，各个属性参数可进行引用的，比如：

```
blog.address=https://blog.lqdev.cn
blog.author=oKong
blog.desc=${blog.author},${blog.address}
```



最后`blog.desc`的值即可：`oKong,https://blog.lqdev.cn`。利用此特性，并可实现一些特殊的功能。比如后期讲解`spring cloud`时，注册`eurka`注册中心的实例名时，并会使用类似如下配置，使得实例名一眼就知道哪台服务地址：

```
eureka.instance.instance-id=${spring.cloud.client.ipAddress}:${server.port}
```



**这里需要注意，由于springboot在读取properties文件时，使用的是PropertiesPropertySourceLoader类进行读取，默认读取的编码是ISO 8859-1，故在默认的配置文件中使用中文时，会出现乱码，此时可以将中文转成Unicode编码或者使用yml配置格式(默认就支持utf-8),再不济可以将作为配置写入到一个自定义配置文件，利用@PropertySource注解的encoding属性指定编码**

## 随机数

Spring Boot的属性配置文件中可以通过`${random}`来产生int值、long值或者string字符串，来支持属性的随机值。

```
# 随机字符串
.blog.value=${random.value}
# 随机int
.blog.number=${random.int}
# 随机long
.blog.bignumber=${random.long}
# 10以内的随机数
.blog.test1=${random.int(10)}
# 1-20的随机数
.blog.test2=${random.int[1,20]}
```



## 自定义配置文件

在多数情况下，配置信息基本上都是放入`application.properties`文件中，但在一些场景下，比如某个配置项比较多时，为了分开存放，也可自定义配置文件，如`my.properties`。由于自定义的文件，系统不会自动加载，这个时候就需要手动引入了。
利用`@PropertySource`注解既可以引入配置文件，需要引入多个时，可使用`@PropertySources`设置数组，引入多个文件。

```
@SpringBootApplication
@PropertySource(value="classpath:my.properties",encoding="utf-8")
public class Chapter3Application {

	public static void main(String[] args) {
		SpringApplication.run(Chapter3Application.class, args);
	}
}
```



## 配置绑定对象

虽然使用`@Value()`方式，能方便的引入自定义的属性值，但在多某个配置项属于某一配置时，希望对应到一个实体配置类中，springboot也提供了支持。利用`@ConfigurationProperties`属性，即可完成
my.properties配置文件：

```
config.code=code
config.name=趔趄的猿
config.hobby[0]=看电影
config.hobby[1]=旅游
```



实体类：

```
@Component
//@EnableConfigurationProperties(value= {Config.class})
@ConfigurationProperties(prefix="config")
@Data
public class Config {
	
	String code;
	
	String name;
	
	List<String> hobby;
}
```



**这里可直接加入@Component使其在启动时被自动扫描到，或者使用@EnableConfigurationProperties注解注册此实体bean.其次，在引入@ConfigurationProperties时，IDE会提示你引入spring-boot-configuration-processor依赖，前面提到，在自定义属性时，创建additional-spring-configuration-metadata.json可进行属性提示，而此依赖功能类似，会编译时自动生成spring-configuration-metadata.json文件，此文件主要给IDE使用，用于提示使用。添加后在配置文件点击属性时，会自动跳转到对应绑定的实体类中**

[![img](https://wx-10045722.cos.ap-shanghai.myqcloud.com/blog-srping-boot-1/30490525.jpg)](https://wx-10045722.cos.ap-shanghai.myqcloud.com/blog-srping-boot-1/30490525.jpg)

## 数组形式

配置了以下配置，然后利用`List<String>`就能获取hobby的值了。

```
config.code=code
config.name=趔趄的猿
config.hobby[0]=看电影
config.hobby[1]=旅游
```



[![img](https://wx-10045722.cos.ap-shanghai.myqcloud.com/blog-srping-boot-1/15003045.jpg)](https://wx-10045722.cos.ap-shanghai.myqcloud.com/blog-srping-boot-1/15003045.jpg)

## 总结

> 目前互联网上很多大佬都有`springboot`系列教程，如有雷同，请多多包涵了。本文是作者在电脑前一字一句敲的，每一步都是亲身实际过的。若文中有所错误之处，还望提出，谢谢。