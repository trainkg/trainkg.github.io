## Maven �ṹ��Ϣ

### 1. ����JAR ����·��

<code class="hljs groovy">mvn <span class="hljs-attr">dependency:</span>tree -Dincludes=io.<span class="hljs-attr">netty:</span>netty-all</code>

**��ʽ��ϢΪ��**

<code class="hljs less"><span class="hljs-selector-attr">[groupId]</span>:<span class="hljs-selector-attr">[artifactId]</span>:<span class="hljs-selector-attr">[type]</span>:<span class="hljs-selector-attr">[version]</span></code>

����

```shell
mvn dependency:tree -Dincludes=org.springframework.boot:spring-boot-starter-web
```

