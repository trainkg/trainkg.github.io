# [Integrating Groovy into applications](http://groovy-lang.org/integrating.html)

### 集成机制

### Eval

```groovy
import groovy.util.Eval

assert Eval.me('33*3') == 99
assert Eval.me('"foo".toUpperCase()') == 'FOO'
```

Eval 只能执行简单脚本，这里的单行脚本指的不是代码写在一行内，而是通俗的说不能出现`;`

### GroovySheel

类 `groovy.lang.GroovyShell` 可以执行多行脚本并且可以缓存脚本对象， 但是局限于单个groovy script.

```groovy
def shell = new GroovyShell()                           
def result = shell.evaluate '3*5'                       
def result2 = shell.evaluate(new StringReader('3*5'))   
assert result == result2
def script = shell.parse '3*5'                          
assert script instanceof groovy.lang.Script
assert script.run() == 15                               
```

#### Sharing data between a script and the application

脚本执行是需要参数的，所以需要能够接入application的上下文，所以这个共享数据的能力是必不可少的。

```groovy
def sharedData = new Binding()                          
def shell = new GroovyShell(sharedData)                 
def now = new Date()
sharedData.setProperty('text', 'I am shared data!')     
sharedData.setProperty('date', now)                     

String result = shell.evaluate('"At $date, $text"')     

assert result == "At $now, I am shared data!"
```

通过`groovy.lang.Binding` 实现，实现script从外部获取参数

> @思考.  是否支持复合对象，复合对象内script如何解析或者范围

```groovy
def sharedData = new Binding()
def shell = new GroovyShell(sharedData)

shell.evaluate('int foo=123') // 返回值未定义

try {
    assert sharedData.getProperty('foo')
} catch (MissingPropertyException e) {
    println "foo is defined as a local variable"
}          
```

同时，script 也可以通过binding向外分享数据,

>! You must be very careful when using shared data in a **multithreaded environment.** The `Binding` instance that you pass to `GroovyShell` is **not** **thread safe**, and shared by **all scripts**.

```groovy
def shell = new GroovyShell()

def b1 = new Binding(x:3)                       
def b2 = new Binding(x:4)          
// 注意这个地方两个提供数据的binding使用的是同一个script对象， 是不能保证线程安全的，如果在多线程缓解中
def script = shell.parse('x = 2*x')
script.binding = b1
script.run()
script.binding = b2
script.run()
assert b1.getProperty('x') == 6
assert b2.getProperty('x') == 8
assert b1 != b2
```

补充：如果我们在全局缓存了所有的groovy脚本，但是在多线程环境下面使用它，那么会导致一些并发问题

### GroovyClassLoader

补充： 常规我们正常项目是不可能使用简单脚本的，如果是这样的话，我觉得也没有必要使用groovy, 使用其他的`javascript engine ` ，所以`GroovyClassLoader` 和 `GroovyScriptEngine` 才是我们真正需要的东西

负责运行时编译和加载类的核心类。

```groovy
import groovy.lang.GroovyClassLoader
// 	create a new GroovyClassLoader
def gcl = new GroovyClassLoader()                                           
// parseClass will return an instance of Class
def clazz = gcl.parseClass('class Foo { void doIt() { println "ok" } }')    
// 断言检查
assert clazz.name == 'Foo'                                                  
// 创建实例
def o = clazz.newInstance()                                                 
// 执行方法
o.doIt()    
```

补充： 和JAVA看起来很类似，唯独可以直接加载字符串作为class来源，不像Java载入的是`class`文件， 这点也是groovy的动态能力的体现

> 注意： groovy 加载器会持有所有的类定义，并且在执行两次加载的时候，会有两个独立的类定义，所以重复执行会造成内存泄漏.

```groovy
import groovy.lang.GroovyClassLoader

def gcl = new GroovyClassLoader()
def clazz1 = gcl.parseClass('class Foo { }')                                
def clazz2 = gcl.parseClass('class Foo { }')                                
assert clazz1.name == 'Foo'                                                 
assert clazz2.name == 'Foo'
// 虽然同名，但是这两个类并不是同一个类
assert clazz1 != clazz2                                                     
```

造成上面的原因是因为groovy不提供这样的明文字符串监控，如果你想两次得到相同的class定义，**必须使用文件的方式提供groovy脚本**

```groovy
def gcl = new GroovyClassLoader()
def clazz1 = gcl.parseClass(file)                                           
// 来源于文件，这样groovy会监控文件是否发生过变化
def clazz2 = gcl.parseClass(new File(file.absolutePath))                    
assert clazz1.name == 'Foo'                                                 
assert clazz2.name == 'Foo'
assert clazz1 == clazz2                                                     
```

### GroovyScriptEngine

`groovy.util.GroovyScriptEngine` 提供了非常灵活的基础功能用来解决脚本依赖关系问题。 多种的加载方式

The `groovy.util.GroovyScriptEngine` class provides a flexible foundation for applications which rely on script reloading and script dependencies. While `GroovyShell` focuses on standalone `Script`s and `GroovyClassLoader` handles dynamic compilation and loading of any Groovy class, the `GroovyScriptEngine` will add a layer on top of `GroovyClassLoader` to handle both script dependencies and reloading.

文件 **ReloadingTest.groovy** 内容

```groovy
class Greeter {
    String sayHello() {
        def greet = "Hello, world!"
        greet
    }
}

new Greeter()
```

```groovy
def binding = new Binding()
// 载入groovy 的脚本根目录
def engine = new GroovyScriptEngine([tmpDir.toURI().toURL()] as URL[])          

// 循环遍历
while (true) {
    // 载入指定的groovy脚本
    def greeter = engine.run('ReloadingTest.groovy', binding)                   
    println greeter.sayHello()                                                  
    Thread.sleep(1000)
}
```

输出内容：

```
Hello, world!
Hello, world!
...
```

不停止程序更新groovy脚本

```groovy
class Greeter {
    String sayHello() {
        def greet = "Hello, Groovy!"
        greet
    }
}

new Greeter()
```

输出内容将发生变化

```
Hello, world!
...
Hello, Groovy!
Hello, Groovy!
...
```

升级， 当我们修改的脚本存在依赖关系，即存在import 其他的文件

下面有两个脚本 `Depencency.groovy`,

```groovy
class Dependency {
    String message = 'Hello, dependency 1'
}
```

`ReloadingTest` 升级内容，**存在依赖关系**

```groovy
import Dependency

class Greeter {
    String sayHello() {
        // 升级存在依赖关系
        def greet = new Dependency().message
        greet
    }
}

new Greeter()
```

输出信息将会变更为

```
Hello, Groovy!
...
Hello, dependency 1!
Hello, dependency 1!
...
```

这个时候，我们在此更新脚本 `Dependency.groovy` , 但是不修改 `ReloadingTest`

```groovy
class Dependency {
    String message = 'Hello, dependency 2'
}
```

输出信息将会变更为

```
Hello, dependency 1!
...
Hello, dependency 2!
Hello, dependency 2!
```

还是发生了变化，因为groovy engine 管理着所有的依赖关系

### CompilationUnit

Ultimately, it is possible to perform more operations during compilation by relying directly on the `org.codehaus.groovy.control.CompilationUnit` class. This class is responsible for determining the various steps of compilation and would let you introduce new steps or even stop compilation at various phases. This is for example how stub generation is done, for the joint compiler.

However, overriding `CompilationUnit` is not recommended and should only be done if no other standard solution works.

补充： 可以监控到文件变化过程? 我理解应该是这样，暂时不使用到这样的信息监控

### Bean Scripting Framework

补充： 忘记了，一直在找groovy的官方文档和java的集成方式，但是忘记了万能的集成神器spring. 应该去看一下spring官方文档关于groovy的集成方式

推荐使用[JSR-223](http://groovy-lang.org/integrating.html#jsr223)规范接入groovy.

#### 2.1. Getting started

#### 2.2. Passing in variables

#### 2.3. Other calling options

#### 2.4. Access to the scripting engine

### 3. JSR 223 javax.script API

JSR-223 is a standard API for calling scripting frameworks in Java. It is available since Java 6 and aims at providing a common framework for calling multiple languages from Java. Groovy provides its own richer integration mechanisms, and if you don’t plan to use multiple languages in the same application, it is recommended that you use the Groovy integration mechanisms **instead of** the limited JSR-223 API.

补充：有意思的文档， 前面说BSF长时间不更新，建议使用JSR223, 到下面又建议直接使用groovy自身的集成方案, 先不看了，先考虑推荐groovy方案





