## Maven 结构信息

### 1. 查找JAR 依赖路径

<code class="hljs groovy">mvn <span class="hljs-attr">dependency:</span>tree -Dincludes=io.<span class="hljs-attr">netty:</span>netty-all</code>

**格式信息为：**

<code class="hljs less"><span class="hljs-selector-attr">[groupId]</span>:<span class="hljs-selector-attr">[artifactId]</span>:<span class="hljs-selector-attr">[type]</span>:<span class="hljs-selector-attr">[version]</span></code>

范例

```shell
mvn dependency:tree -Dincludes=org.springframework.boot:spring-boot-starter-web
```

