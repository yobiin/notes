## 1. 介绍

### 1.1. 本指南所涵盖的方面

本指南涵盖了Spring Web Flow所有方面的内容。它涵盖了最终用户应用程序的实现流程以和使用功能集。它同样涵盖了扩展框架以及总体架构模型。

### 1.2. Web Flow 的运行环境

Java 1.8 或者更高版本。

Spring 5.0 或者更高版本。

### 1.3. 资源

你可以在StackOverflow的指定的标签中提问或者参与互动，具体请看[Spring at StackOverflow](https://spring.io/questions)。

你可以通过[Spring Issue Tracker](https://jira.spring.io/)报告bugs和提交问题。

提交推送请求和使用源码，请使用[Web Flow on Github](https://github.com/spring-projects/spring-webflow)。

### 1.4. 如何通过Maven获取Web Flow组件

Web Flow 发行版的每一个jar包都可以在[Maven Central Repository](https://search.maven.org/)中找到。如果你已经在使用Maven来管理并构建你的Web项目，那么你很容易就可以将Web Flow集成进来。

如下所示，在你项目的pom文件中定义一个依赖：

~~~xml
<dependency>
    <groupId>org.springframework.webflow</groupId>
    <artifactId>spring-webflow</artifactId>
    <version>x.y.z.RELEASE</version>
</dependency>
~~~

注意，如果你使用JavaServer Faces进行开发，则需要在项目的pom文件中添加以下的依赖：

~~~xml
<dependency>
    <groupId>org.springframework.webflow</groupId>
    <artifactId>spring-faces</artifactId>
    <version>x.y.z.RELEASE</version>
</dependency>
~~~

### 1.5. 如何获取Web Flow每晚的构建和里程碑版本

使用Maven可以获取Web Flow开发分支的每晚快照。这些快照构建对于在下一个版本之前测试你依赖的修复是非常有用的，并为你提供了一个方便的方式来提交相关的修复是否满足你的需求的反馈。

#### 1.5.1. 通过Maven获取Web Flow快照和里程碑版本

你需要通过SpringSource仓库来获取Web Flow的快照版本和里程碑版本。如下，把SpringSource仓库添加到pom.xml文件：

~~~xml
<repository>
    <id>spring</id>
    <name>Spring Repository</name>
    <url>http://repo.spring.io/snapshot</url>
</repository>
~~~

然后定义如下的依赖：

~~~xml
<dependency>
    <groupId>org.springframework.webflow</groupId>
    <artifactId>spring-webflow</artifactId>
    <version>x.y.z.BUILD-SNAPSHOT</version>
</dependency>
~~~

如果使用JSF，则添加以下的依赖：

~~~xml
<dependency>
    <groupId>org.springframework.webflow</groupId>
    <artifactId>spring-faces</artifactId>
    <version>x.y.z.BUILD-SNAPSHOT</version>
</dependency>
~~~















