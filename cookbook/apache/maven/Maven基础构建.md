## Maven

### 1. Maven 仓库选择

[阿里云提供列表](https://developer.aliyun.com/mvn/guide)

### 2. 如何在子项目中进行版本管理

使用dependencyManagement在顶层Model中管理依赖的JAR版本，子module不再设定依赖包的版本
例如

```xml
<dependency>
	<!-- Import dependency management from Spring Boot -->
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-dependencies</artifactId>
	<version>2.3.4.RELEASE</version>
	<type>pom</type>
	<scope>import</scope>
</dependency>
```

Spring Boots 采用上面的设定方式设定依赖版本

### 3. Maven 使用手册

```
http://maven.apache.org/guides/getting-started/index.html
```

