## Maven

### 1. Maven �ֿ�ѡ��

[�������ṩ�б�](https://developer.aliyun.com/mvn/guide)

### 2. ���������Ŀ�н��а汾����

ʹ��dependencyManagement�ڶ���Model�й���������JAR�汾����module�����趨�������İ汾
����

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

Spring Boots ����������趨��ʽ�趨�����汾

### 3. Maven ʹ���ֲ�

```
http://maven.apache.org/guides/getting-started/index.html
```

