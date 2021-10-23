## [Maven 依赖关系机制](http://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html)

>背景： 有一次在maven中突然出现了一个自己未引用的包，并且无法正确计算出来包的引用路径。所以需要了解一下maven依赖有哪些是我不知道的，
>
>最终原因发现因为在测试过程中，父项目路径中创建过同样坐标的项目，导致加载出来期望之外的JAR.

### Transitive Dependencies

>  TO

### Dependency Scope

- **compile**
  This is the default scope, used if none is specified. Compile dependencies are available in all classpaths of a project. Furthermore, those dependencies are propagated to dependent projects.

  默认编译需要

- **provided**
  This is much like `compile`, but indicates you expect the JDK or a container to provide the dependency at runtime. For example, when building a web application for the Java Enterprise Edition, you would set the dependency on the Servlet API and related Java EE APIs to scope `provided` because the web container provides those classes. A dependency with this scope is added to the classpath used for compilation and test, but not the runtime classpath. It is not transitive.

  外部容器提供运行是要求的JAR

- **runtime**
  This scope indicates that the dependency is not required for compilation, but is for execution. Maven includes a dependency with this scope in the runtime and test classpaths, but not the compile classpath.

  编译不需要，但是运行时需要

- **test**
  This scope indicates that the dependency is not required for normal use of the application, and is only available for the test compilation and execution phases. This scope is not transitive. Typically this scope is used for test libraries such as JUnit and Mockito. It is also used for non-test libraries such as Apache Commons IO if those libraries are used in unit tests (src/test/java) but not in the model code (src/main/java).

  标记只是在Maven test 阶段需要使用这个包

- **system**
  This scope is similar to `provided` except that you have to provide the JAR which contains it explicitly. The artifact is always available and is not looked up in a repository.

  运行时需要，但是文件是使用本地文件方式提供

- **import**
  This scope is only supported on a dependency of type `pom` in the `<dependencyManagement>` section. It indicates the dependency is to be replaced with the effective list of dependencies in the specified POM's `<dependencyManagement>` section. Since they are replaced, dependencies with a scope of `import` do not actually participate in limiting the transitivity of a dependency.

  导入`POM`文件， pom中可以进一步定义依赖关系，相当于依赖关系的一个封装

### Dependency Management

用于Maven project继承的情况处理， 主要是提供共同的依赖管理版本信息，子module不需要再设定版本



