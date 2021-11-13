## Maven Plugin

### 查看已有的Maven Plugin的主入口

查看对于的Plugin 核心包对应的MAINFEST.MF 中的Main-Class信息，比如在mybatis-generator中有如下的信息

```
Main-Class: org.mybatis.generator.api.ShellRunner
```

由此我们知道mybatis-generator的主入口是 ShellRunner

### [开发自己的Maven Plugin](http://maven.apache.org/plugin-developers/index.html)

见官方文档



[Maven Plugin]: http://maven.apache.org/plugins/index.html	"Maven Plugin List"



