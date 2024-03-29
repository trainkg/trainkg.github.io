[#官方文档URL](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features)

### 1. spring boot默认使用 commons logging.

默认的配置支持 ， java util logging, log4j2, logback(slf4j)

### 2. debug 方式

```bash
JVM command $ java -jar myapp.jar --debug
```

`debug=true` in your application.properties

### 3. 如果控制断支持 ANSI。 可以针对不同的日志设定颜色

spring.output.ansi.enabled 开启特性  
  示例

```
%clr(%5p)
%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){yellow}
```

* blue
* cyan
* faint
* green
* magenta
* red
* yellow

### 4. 文件输出位置

使用配置 

```
logging.file.name
logging.file.path   
```

### 5. LOG GROUP 特性 

示例

```
logging.group.tomcat=org.apache.catalina, org.apache.coyote, org.apache.tomcat
logging.level.tomcat=TRACE 集中定义tomcat级别
```

spring默认方式

```
web > logging.group.web
sql > logging.group.sql
```

### 自定义配置文件

logging.config

```
org.springframework.boot.logging.LoggingSystem 强制使用指定的日志系统
```

* Logback   
  `logback-spring.xml, logback-spring.groovy, logback.xml, or logback.groovy`

* Log4j2
  `log4j2-spring.xml or log4j2.xml`

* JDK (Java Util Logging)

  `logging.properties`







   


?    

  

  


?    
?    