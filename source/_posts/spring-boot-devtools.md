---
path: https://wx-10045722.cos.ap-shanghai.myqcloud.com/blog-srping-boot-1/spring-dev-tools-97a5ba18daef214b4cef0ab62f847b94-a3725.jpg
date: 2018/11/18 23:46:25
title: SpringBoot | 第五章：使用Spring Boot DevTools加快开发速度
categories:
    - SpringBoot
tags:
    - java
    - spring
    - spring boot 
    - devtools
---
> 英语 [原文](https://www.vojtechruzicka.com/spring-boot-devtools)
[![Spring Boot DevTools](https://wx-10045722.cos.ap-shanghai.myqcloud.com/blog-srping-boot-1/spring-dev-tools-97a5ba18daef214b4cef0ab62f847b94-a3725.jpg)](https://wx-10045722.cos.ap-shanghai.myqcloud.com/blog-srping-boot-1/spring-dev-tools-97a5ba18daef214b4cef0ab62f847b94-a3725.jpg)

如何使用DevTools进一步加快Spring Boot开发速度并提升效率？

# 建立

与Spring Boot一样，设置非常简单。你需要做的就是添加`Devtools`的依赖，`Spring Boot`会自动配置`DevTools`。

Maven：

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <optional>true</optional>
</dependency>
```

Gradle：

```
configurations {
    developmentOnly
    runtimeClasspath {
        extendsFrom developmentOnly
    }
}
dependencies {
    developmentOnly("org.springframework.boot:spring-boot-devtools")
}
```

注意，请将`optional`设置为`true`或者在`gradle` 中加入`developmentOnly`

## 自动重启

每当类路径中的文件发生更改时，`DevTools`会自动重新启动正在运行的应用程序，并应用新的更改。在本地开发时，这个功能非常实用，因为你不需要手动重新部署应用程序。

### 在`IDE`中触发重新启动

只要`classpaht`中的类发生了改变，就会触发重新启动。所以在IDE中只改`.java`文件不够的 ，需要更新`classpaht`中的`.class`文件。

使用IntelliJ IDEA时，你需要手动构建项目（*Ctrl + F9*或*Build→Build Project*），或者将IDEA设置为自动构建，你也可以打开Spring Boot运行配置并定义触发应用程序更新条件（*Ctrl + F10*）：

[![Intellij IDEA Spring Boot运行配置](https://wx-10045722.cos.ap-shanghai.myqcloud.com/blog-srping-boot-1/intellij-idea-update-2a187cd547b9066b5441734203997a6f-55803.png)](https://wx-10045722.cos.ap-shanghai.myqcloud.com/blog-srping-boot-1/intellij-idea-update-2a187cd547b9066b5441734203997a6f-55803.png)

在`On Update action`中，你可以选择`Update trigger file`在调用`Update`操作时触发`DevTools`重新启动。或者，你甚至可以选择`Host swap classes and update trigger file if failed`。

在`On frame deactivation`中，选择`Update classes and resources`

在Eclipse中，仅保存文件就足够了。

## 仅用于开发环境中

Spring Boot DevTools仅用于开发环境，而不能用于生产环境。如果`Spring boot `检测到你正在生产中运行，则会自动禁用DevTools。

为此，只要你以jar包的形式运行程序时，都会被视为在生产环境中运行，如：

```
java -jar devtools-example-1.0.0.jar
```

当你的应用程序通过特殊的类加载器启动时也是如此，例如在应用程序服务器上。相反，当你IDE中运行时，将被视为开发模式。使用spring-boot-plugin运行应用时也被视为开发模式：

Maven的：

```
mvn spring-boot:run
```

Gradle：

```
gradle bootRun
```

## Live Reload 插件

[LiveReload](http://livereload.com/)是一个有用的工具，它可以在你对文件中进行更改时立即在浏览器中更新页面，如HTML，CSS，图像等。也可以根据需要预处理文件 - 可以自动编译SASS或LESS文件。

![Live Reload在行动中](https://wx-10045722.cos.ap-shanghai.myqcloud.com/blog-srping-boot-1/live-reload-bd91e4e886709035cba31d16978657c3.gif)

Spring DevTools自动在本地启动LiveReload服务的实例，该服务会器监视你的文件。你只需要安装一个[浏览器扩展](http://livereload.com/extensions/)，就可以了。它不仅可用于开发应用程序的前端，还可用于监视和重新加载REST API的输出。

## Devtools 配置

在本地开发应用程序时，通常需要在不同的环境中使用不同的配置文件。比如缓存。在生产中，依赖各种缓存是非常重要的（例如模板引擎的缓存，静态资源的缓存头等）。但是在开发过程中，你需要实时的拿到最新的数据。

自己管理多组配置是非常复杂的。好消息是Spring Boot DevTools为你的本地开发配置了许多开箱即用的属性。

```
spring.thymeleaf.cache=false
spring.freemarker.cache=false
spring.groovy.template.cache=false
spring.mustache.cache=false
server.servlet.session.persistent=true
spring.h2.console.enabled=true
spring.resources.cache.period=0
spring.resources.chain.cache=false
spring.template.provider.cache=false
spring.mvc.log-resolved-exception=true
server.servlet.jsp.init-parameters.development=true
spring.reactor.stacktrace-mode.enabled=true
```

具体的你可以查看[DevToolsPropertyDefaultsPostProcessor](https://github.com/spring-projects/spring-boot/blob/v2.0.6.RELEASE/spring-boot-project/spring-boot-devtools/src/main/java/org/springframework/boot/devtools/env/DevToolsPropertyDefaultsPostProcessor.java)中所有属性的[列表](https://github.com/spring-projects/spring-boot/blob/v2.0.6.RELEASE/spring-boot-project/spring-boot-devtools/src/main/java/org/springframework/boot/devtools/env/DevToolsPropertyDefaultsPostProcessor.java)。

## 远程连接

除本地开发外，你还可以连接到运行`DevTools`的远程应用程序。这不适用于生产环境，因为它可能是一个严重的安全风险。但是，它在预生产环境中非常有用。

### 启用远程连接

默认情况下不启用远程连接。你需要通过修改pom文件显式启用它：

```
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <excludeDevtools>false</excludeDevtools>
            </configuration>
        </plugin>
    </plugins>
</build>
```

或者使用gradle，[你需要设置](https://docs.spring.io/spring-boot/docs/current/gradle-plugin/reference/html/#packaging-executable-configuring-excluding-devtools) `excludeDevtools = false`：

```
bootWar {
    excludeDevtools = false
}
```

然后，你需要设置一个密码，以便在连接到远程应用程序时用于身份验证

```
spring.devtools.remote.secret=somesecret
```

### 连接到远程应用程序

远程应用程序运行后，你可以启动远程连接会话。现在，你需要做的就是通过`org.springframework.boot.devtools.RemoteSpringApplication`使用远程应用程序的URL作为参数启动。请注意，如果可能，请使用`https`。

在IDE中轻松运行远程连接。在IDEA中，你只需创建一个新的运行配置。转到`Run → Edit Configurations...`并点击左上角的图标`+`创建一个的新配置。选择`Application`类型。

使用`RemoteSpringApplication` 作为Main class，从DevTools模块中选择URL并作为程序启动参数.

[![IDEA中的远程连接配置](https://wx-10045722.cos.ap-shanghai.myqcloud.com/blog-srping-boot-1/intellij-idea-update-2a187cd547b9066b5441734203997a6f-55803.png)](https://wx-10045722.cos.ap-shanghai.myqcloud.com/blog-srping-boot-1/intellij-idea-update-2a187cd547b9066b5441734203997a6f-55803.png)

运行此配置后，如果与远程应用程序的连接成功，你应该会看到类似的输出。

```
  .   ____          _                                              __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _          ___               _      \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` |        | _ \___ _ __  ___| |_ ___ \ \ \ \
 \\/  ___)| |_)| | | | | || (_| []::::::[]   / -_) '  \/ _ \  _/ -_) ) ) ) )
  '  |____| .__|_| |_|_| |_\__, |        |_|_\___|_|_|_\___/\__\___|/ / / /
 =========|_|==============|___/===================================/_/_/_/
 :: Spring Boot Remote ::  (v2.0.6.RELEASE)

2018-11-02 17:24:42.126  INFO 16640 --- [           main] o.s.b.devtools.RemoteSpringApplication   : Starting RemoteSpringApplication v2.0.6.RELEASE on DESKTOP-6NJV4ON with PID 16640 (C:\Users\vojte\.m2\repository\org\springframework\boot\spring-boot-devtools\2.0.6.RELEASE\spring-boot-devtools-2.0.6.RELEASE.jar started by vojte in C:\projects\rest-docs-starter)
2018-11-02 17:24:42.130  INFO 16640 --- [           main] o.s.b.devtools.RemoteSpringApplication   : No active profile set, falling back to default profiles: default
2018-11-02 17:24:42.172  INFO 16640 --- [           main] s.c.a.AnnotationConfigApplicationContext : Refreshing org.springframework.context.annotation.AnnotationConfigApplicationContext@3daa422a: startup date [Fri Nov 02 17:24:42 CET 2018]; root of context hierarchy
2018-11-02 17:24:42.679  WARN 16640 --- [           main] o.s.b.d.r.c.RemoteClientConfiguration    : The connection to http://localhost:8080 is insecure. You should use a URL starting with 'https://'.
2018-11-02 17:24:42.800  WARN 16640 --- [           main] o.s.b.d.a.OptionalLiveReloadServer       : Unable to start LiveReload server
2018-11-02 17:24:42.829  INFO 16640 --- [           main] o.s.b.devtools.RemoteSpringApplication   : Started RemoteSpringApplication in 1.212 seconds (JVM running for 1.877)
```

连接到远程应用程序后，`DevTools`监视`classpath`下的变动，与本地开发相同。但是，它不是本地重新启动，而是将更改推送到远程服务器并在那里触发重新启动。这比构建应用程序和部署到远程计算机要快得多。



## 限制

### Live Reload

使用DevTools的Spring应用程序会自动启动LiveReload服务。不幸的是，此服务器中只有一个实例可以同时运行。更确切地说，只有第一个可运行。这不仅适用于使用DevTools的Spring应用程序的多个实例，也适用于任何其他应用程序。

如果要将Spring应用程序配置为不启动LiveReload服务，可以在`application.properties`添加配置：

```
spring.devtools.livereload.enabled=false
```

### 关机钩

DevTools依赖[shutdown hook](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/SpringApplication.html#setRegisterShutdownHook-boolean-)的`SpringApplication`。如果你使用以下方法手动禁用`Shutdown hook`，Devtools将无法正常工作：

```
springApplication.setRegisterShutdownHook(false);
```

默认情况下，Hook是启用的。

### 与第三方库的冲突

虽然DevTools通常都可以正常运行，但它可能与第三方库有冲突。特别是，使用标准进行反序列化存在[已知问题](https://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-devtools.html#using-boot-devtools-known-restart-limitations)`ObjectInputStream`。

如果发生冲突，可以通过设置禁用自动重启：

```
spring.devtools.restart.enabled=false
```

虽然不再触发重启。但是，仍使用重启类加载器。如果你需要完全禁用类加载器，则需要在启动应用程序之前执行此操作：

```
public static void main(String[] args) {
    System.setProperty("spring.devtools.restart.enabled", "false");
    SpringApplication.run(MyApp.class, args);
}
```



## 结论

DevTools通过提供自动重启和LiveReload功能，使你更快，更轻松地开发Spring Boot应用程序。除此之外，它还将各种属性设置为更适合本地开发的值。此外，它允许你远程连接到你的应用程序，并仍然使用其大部分功能。在生产中运行时，不要使用DevTools。有关详细信息，请参阅[官方文档](https://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-devtools.html)。