## [Maven ������ϵ����](http://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html)

>������ ��һ����maven��ͻȻ������һ���Լ�δ���õİ��������޷���ȷ���������������·����������Ҫ�˽�һ��maven��������Щ���Ҳ�֪���ģ�
>
>����ԭ������Ϊ�ڲ��Թ����У�����Ŀ·���д�����ͬ���������Ŀ�����¼��س�������֮���JAR.

### Transitive Dependencies

>  TO

### Dependency Scope

- **compile**
  This is the default scope, used if none is specified. Compile dependencies are available in all classpaths of a project. Furthermore, those dependencies are propagated to dependent projects.

  Ĭ�ϱ�����Ҫ

- **provided**
  This is much like `compile`, but indicates you expect the JDK or a container to provide the dependency at runtime. For example, when building a web application for the Java Enterprise Edition, you would set the dependency on the Servlet API and related Java EE APIs to scope `provided` because the web container provides those classes. A dependency with this scope is added to the classpath used for compilation and test, but not the runtime classpath. It is not transitive.

  �ⲿ�����ṩ������Ҫ���JAR

- **runtime**
  This scope indicates that the dependency is not required for compilation, but is for execution. Maven includes a dependency with this scope in the runtime and test classpaths, but not the compile classpath.

  ���벻��Ҫ����������ʱ��Ҫ

- **test**
  This scope indicates that the dependency is not required for normal use of the application, and is only available for the test compilation and execution phases. This scope is not transitive. Typically this scope is used for test libraries such as JUnit and Mockito. It is also used for non-test libraries such as Apache Commons IO if those libraries are used in unit tests (src/test/java) but not in the model code (src/main/java).

  ���ֻ����Maven test �׶���Ҫʹ�������

- **system**
  This scope is similar to `provided` except that you have to provide the JAR which contains it explicitly. The artifact is always available and is not looked up in a repository.

  ����ʱ��Ҫ�������ļ���ʹ�ñ����ļ���ʽ�ṩ

- **import**
  This scope is only supported on a dependency of type `pom` in the `<dependencyManagement>` section. It indicates the dependency is to be replaced with the effective list of dependencies in the specified POM's `<dependencyManagement>` section. Since they are replaced, dependencies with a scope of `import` do not actually participate in limiting the transitivity of a dependency.

  ����`POM`�ļ��� pom�п��Խ�һ������������ϵ���൱��������ϵ��һ����װ

### Dependency Management

����Maven project�̳е�������� ��Ҫ���ṩ��ͬ����������汾��Ϣ����module����Ҫ���趨�汾



