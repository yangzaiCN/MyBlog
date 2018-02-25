---
title: 单元测试之junit5(一)
categories:
- 测试
tags:
- junit5

---
现在的开发大都是敏捷开发，在快速版本迭代的过程中，开发人员越来越重视单元测试，坚持边写边测是一种良好的习惯。
在java的世界里，大都集成了junit，包括现在流行的testng、spock都借鉴了junit的思想。所以junit是用好单元测试
的基础，而JUnit5在增加了很多的新特性的同时，又保持了对JUnit4的向后兼容性。本文对 JUnit5进行了详细的介绍。
<!-- more -->

### 什么是junit5 ###

junit5与之前的junit4有所不同，它是由三个模块组成：
> JUnit 5 = JUnit Platform + JUnit Jupiter + JUnit Vintage

** JUnit Platforms: **是基于JVM启动测试框架基础，支持gradle与maven命令行执行
** JUnit Jupiter: ** 包含了 JUnit 5 最新的编程模型和扩展机制。
** JUnit Vintage: ** 对JUnit3 和 JUnit4 的测试用例支持

- JUnit Platform:
  >junit-platform-commons:内部使用库
  >junit-platform-console：控制台执行测试引擎
  >junit-platform-console-standalone：控制台独立执行所需要的全部依赖。
  >junit-platform-engine：扩展引擎提供接口
  >junit-platform-gradle-plugin：对gradle支持
  >junit-platform-launcher：给IDE提供配置、启动测试用例接口
  >junit-platform-runner：兼容junit4
  >junit-platform-suite-api：提供注解支持
  >junit-platform-surefire-provider：对maven支持

- JUnit Jupiter
  >junit-jupiter-api:提供扩展接口,注解支持
  >junit-jupiter-engine：运行时引擎
  >junit-jupiter-params：参数化支持
  >junit-jupiter-migrationsupport：迁移支持

- JUnit Vintage
  >junit-vintage-engine:对老式junit的支持测试引擎

以下是junit5中各个模块的组件图

![](https://i.imgur.com/dFPWKpy.png)

### 运行环境 ###
>JUnit 5 对 Java 运行环境的最低要求是 Java 8。本文的示例基于 IntelliJ IDEA 上开发，并使用 Gradle 作为构建工具。

### 写单元测试 ###

想用好junit5，首先要了解其提供了哪些注解，以下是提供的注解：

|注解|含义|
| -- | -- |
|@Test|表明该方法是测试方法，并且该注解是没有属性的|