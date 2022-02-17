# 什么是SPI机制
```text
    仿照Java中的SPI扩展机制实现的解耦。SPI全名为Service Provider Interface.这个机制是针对厂商或者插件的。
```
# Spring Boot的SPI机制
```text
    Spring中也有一种类似于SPI的加载机制。它被放在META-INF/spring。factories文件中。配置接口的实现类名称，然后在程序中读取这些配置文件并实例
化，并且注入到容器中。
```
```text
    org.springframework.boot.autoconfigure.EnableAutoConfiguration= 加载配置类
    org.springframework.boot.env.EnvironmentPostProcessor= key类型需要实例化的类
```
# Spring Factories实现原理
```text
    spring-core包里包含了SpringFactoriesLoader类，这个类实现检索META-INF/spring.factories文件，并获得指定接口的配置的功能。
```
![Image](../aimage/spring%20factories加载方法.png)