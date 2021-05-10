# Gradle的使用教程

## 相关介绍

​    Gradle是一个好用的构建工具 ，使用它的原因是：

- 配置相关依赖代码量少，不会像maven一样xml过多 
- 打包编译测试发布都有，而且使用起来方便 
- 利用自定义的任务可以完成自己想要的功能

## 安装

 下载地址[http://services.gradle.org/distributions/ ](http://services.gradle.org/distributions/) ，下载你所需要对应的版本，我这里下载的是gradle-4.7-bin.zip。下载后解压到你想要的目录即可，然后设置环境变量：

![image-20210510215955478](https://gitee.com/kongyin/picture_bed/raw/master/wx_picture/image-20210510215955478.png)

接下来需要在PATH配置相关bin

![image-20210510220209652](https://gitee.com/kongyin/picture_bed/raw/master/wx_picture/image-20210510220209652.png)

> 类似maven 环境变量配置

在cmd模式下查看，出现以下信息证明安装成功

![image-20210510220308892](https://gitee.com/kongyin/picture_bed/raw/master/wx_picture/image-20210510220308892.png)

然后我们可以在在环境变量里配置gradle默认的仓库地址（和maven不太一样）：

配置名为`GRADLE_USER_HOME`的环境变量



以下为gradle项目配置文件（类似Maven Pom）

```groovy
// buildscript 代码块中脚本优先执行
buildscript {
 
	// ext 用于定义动态属性
	ext {
		springBootVersion = '1.5.2.RELEASE'
	}
			
	// 自定义  Thymeleaf 和 Thymeleaf Layout Dialect 的版本
	ext['thymeleaf.version'] = '3.0.3.RELEASE'
	ext['thymeleaf-layout-dialect.version'] = '2.2.0'
	
	// 自定义  Hibernate 的版本
	ext['hibernate.version'] = '5.2.8.Final'
 
	// 使用了 Maven 的中央仓库（你也可以指定其他仓库）
	repositories {
		//mavenCentral()
		maven {
			url 'http://maven.aliyun.com/nexus/content/groups/public/'
		}
	}
	
	// 依赖关系
	dependencies {
		// classpath 声明说明了在执行其余的脚本时，ClassLoader 可以使用这些依赖项
		classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
	}
}
 
// 使用插件
apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'org.springframework.boot'
 
// 打包的类型为 jar，并指定了生成的打包的文件名称和版本
jar {
	baseName = 'springboot-test'
	version = '1.0.0'
}
 
// 指定编译 .java 文件的 JDK 版本
sourceCompatibility = 1.8
 
// 默认使用了 Maven 的中央仓库。这里改用自定义的镜像库
repositories {
	//mavenCentral()
	maven {
		url 'http://maven.aliyun.com/nexus/content/groups/public/'
	}
}
 
// 依赖关系
dependencies {
 
	// 该依赖对于编译发行是必须的
	compile('org.springframework.boot:spring-boot-starter-web')
 
	// 添加 Thymeleaf 的依赖
	compile('org.springframework.boot:spring-boot-starter-thymeleaf')
 
	// 添加  Spring Security 依赖
	compile('org.springframework.boot:spring-boot-starter-security')
	
	// 添加 Spring Boot 开发工具依赖
 	//compile("org.springframework.boot:spring-boot-devtools")
 
	// 添加 Spring Data JPA 的依赖
	compile('org.springframework.boot:spring-boot-starter-data-jpa')
	
	// 添加 MySQL连接驱动 的依赖
	compile('mysql:mysql-connector-java:6.0.5')
	
	// 添加   Thymeleaf Spring Security 依赖，与 Thymeleaf 版本一致都是 3.x
	compile('org.thymeleaf.extras:thymeleaf-extras-springsecurity4:3.0.2.RELEASE')
	
	// 添加  Apache Commons Lang 依赖
	compile('org.apache.commons:commons-lang3:3.5')
	
	// 该依赖对于编译测试是必须的，默认包含编译产品依赖和编译时依
	testCompile('org.springframework.boot:spring-boot-starter-test')
	
 
}
```

## Groovy基础

### Groovy的变量和方法声明

在Groovy中，通过 def 关键字来声明变量和方法，比如：

```groovy
def a = 1;
def b = "hello world";
def int c = 1;
 
def hello() {
   println ("hello world");
   return 1;
}
```

在Groovy中，很多东西都是可以省略的，比如

- 语句后面的分号是可以省略的
- 变量的类型和方法的返回值也是可以省略的
- 方法调用时，括号也是可以省略的
- 甚至语句中的return都是可以省略的

所以上面的代码也可以写成如下形式：

```groovy
def a = 1
def b = "hello world"
def int c = 1
 
def hello() {
   println "hello world" // 方法调用省略括号
   1;                    // 方法返回值省略return
}
 
def hello(String msg) {
   println (msg)
}
 
// 方法省略参数类型
int hello(msg) {
   println (msg)
   return 1
}
 
// 方法省略参数类型
int hello(msg) {
   println msg
   return 1 // 这个return不能省略
   println "done"
}
```

> **总结**
>
> - 在Groovy中，类型是弱化的，所有的类型都可以动态推断，但是Groovy仍然是强类型的语言，类型不匹配仍然会报错；
> - 在Groovy中很多东西都可以省略，所以寻找一种自己喜欢的写法；
> - Groovy中的注释和Java中相同。

### Groovy的数据类型

在Groovy中，数据类型有：

- Java中的基本数据类型
- Java中的对象
- Closure（闭包）
- 加强的List、Map等集合类型
- 加强的File、Stream等IO类型

类型可以显示声明，也可以用 def 来声明，用 def 声明的类型Groovy将会进行类型推断。

基本数据类型和对象这里不再多说，和Java中的一致，只不过在Gradle中，对象默认的修饰符为public。下面主要说下String、闭包、集合和IO等。

**1. String**

String的特色在于字符串的拼接，比如

```groovy
def a = 1
def b = "hello"
def c = "a=${a}, b=${b}"
println c
 
outputs:
a=1, b=hello
```

**2. 闭包**

Groovy中有一种特殊的类型，叫做Closure，翻译过来就是闭包，这是一种类似于C语言中函数指针的东西。闭包用起来非常方便，在Groovy中，闭包作为一种特殊的数据类型而存在，闭包可以作为方法的参数和返回值，也可以作为一个变量而存在。

如何声明闭包？

```groovy
{ parameters ->
   code
}
```

闭包可以有返回值和参数，当然也可以没有。下面是几个具体的例子：

```groovy
def closure = { int a, String b ->
   println "a=${a}, b=${b}, I am a closure!"
}
 
// 这里省略了闭包的参数类型
def test = { a, b ->
   println "a=${a}, b=${b}, I am a closure!"
}
 
def ryg = { a, b ->
   a + b
}
 
closure(100, "renyugang")
test.call(100, 200)
def c = ryg(100,200)
println c
```

闭包可以当做函数一样使用，在上面的例子中，将会得到如下输出：

```groovy
a=100, b=renyugang, I am a closure!
a=100, b=200, I am a closure!
300
```

另外，如果闭包不指定参数，那么它会有一个隐含的参数 it

```groovy
// 这里省略了闭包的参数类型
def test = {
   println "find ${it}, I am a closure!"
}
test(100)
 
outputs:
find 100, I am a closure! 
```

> 闭包的一个难题是如何确定闭包的参数，尤其当我们调用Groovy的API时，这个时候没有其他办法，只有查询Groovy的文档：
>
> http://www.groovy-lang.org/api.html
>
> http://docs.groovy-lang.org/latest/html/groovy-jdk/index-all.html
>
> 下面会结合具体的例子来说明如何查文档。

**3. List和Map**

Groovy加强了Java中的集合类，比如List、Map、Set等。

List的使用如下：

```groovy
def emptyList = []
 
def test = [100, "hello", true]
test[1] = "world"
println test[0]
println test[1]
test << 200
println test.size
 
outputs:
100
world
4
```

>  List还有一种看起来很奇怪的操作符<<，其实这并没有什么大不了，左移位表示向List中添加新元素的意思

其实Map也有左移操作，这如果不查文档，将会非常费解。

Map的使用如下：

```groovy
def emptyMap = [:]
def test = ["id":1, "name":"renyugang", "isMale":true]
test["id"] = 2
test.id = 900
println test.id
println test.isMale
 
outputs:
900
true
```

**4. 加强的IO**

根据File的eachLine方法，我们可以写出如下遍历代码，可以看到，eachLine方法也是支持1个或2个参数的，这两个参数分别是什么意思，就需要我们学会读文档了，一味地从网上搜例子，多累啊，而且很难彻底掌握：

```groovy
def file = new File("a.txt")
println "read file using two parameters"
file.eachLine { line, lineNo ->
   println "${lineNo} ${line}"
}
 
println "read file using one parameters"
file.eachLine { line ->
   println "${line}"
}
 
outputs:
read file using two parameters
1 欢迎
2 关注
3 玉刚说
 
read file using one parameters
欢迎
关注
玉刚说
```

除了eachLine，File还提供了很多Java所没有的方法，大家需要浏览下大概有哪些方法，然后需要用的时候再去查就行了，这就是学习Groovy的正道。

下面我们再来看看访问xml文件，也是比Java中简单多了。
Groovy访问xml有两个类：XmlParser和XmlSlurper，二者几乎一样，在性能上有细微的差别，如果大家感兴趣可以从文档上去了解细节，不过这对于本文不重要。

在下面的链接中找到XmlParser的API文档，参照例子即可编程，

http://docs.groovy-lang.org/docs/latest/html/api/。

假设我们有一个xml，attrs.xml，如下所示：

```xml
<resources>
<declare-styleable name="CircleView">
 
   <attr name="circle_color" format="color">#98ff02</attr>
   <attr name="circle_size" format="integer">100</attr>
   <attr name="circle_title" format="string">renyugang</attr>
</declare-styleable>
 
</resources>
```

那么如何遍历它呢？

```groovy
def xml = new XmlParser().parse(new File("attrs.xml"))
// 访问declare-styleable节点的name属性
println xml['declare-styleable'].@name[0]
 
// 访问declare-styleable的第三个子节点的内容
println xml['declare-styleable'].attr[2].text()
 
 
outputs：
CircleView
renyugang
```

- **Class是一等公民**

在Groovy中，所有的Class类型，都可以省略.class，比如：

```groovy
func(File.class)
func(File)
 
def func(Class clazz) {
}
```

- **Getter和Setter**

在Groovy中，Getter/Setter和属性是默认关联的，比如：

```groovy
class Book {
   private String name
   String getName() { return name }
   void setName(String name) { this.name = name }
}
 
class Book {
   String name
}
```

> 上述两个类完全一致，只有有属性就有Getter/Setter；同理，只要有Getter/Setter，那么它就有隐含属性。

- **with操作符**

在Groovy中，当对同一个对象进行操作时，可以使用with，比如：

```groovy
Book bk = new Book()
bk.id = 1
bk.name = "android art"
bk.press = "china press"
 
可以简写为：
Book bk = new Book() 
bk.with {
   id = 1
   name = "android art"
   press = "china press"
}
```

- **判断是否为真**

在Groovy中，判断是否为真可以更简洁：

```groovy
if (name != null && name.length > 0) {}
 
可以替换为：
if (name) {}
```

- **简洁的三元表达式**

在Groovy中，三元表达式可以更加简洁，比如：

```groovy
def result = name != null ? name : "Unknown"
 
// 省略了name
def result = name ?: "Unknown"
```

- **简洁的非空判断**

在Groovy中，非空判断可以用?表达式，比如

```groovy
if (order != null) {
   if (order.getCustomer() != null) {
       if (order.getCustomer().getAddress() != null) {
       System.out.println(order.getCustomer().getAddress());
       }
   }
}
 
可以简写为：
println order?.customer?.address
```

- **使用断言**

在Groovy中，可以使用assert来设置断言，当断言的条件为false时，程序将会抛出异常：

```groovy
def check(String name) {
   // name non-null and non-empty according to Gro    ovy Truth
   assert name
   // safe navigation + Groovy Truth to check
   assert name?.size() > 3
}
```

- **switch方法**

在Groovy中，switch方法变得更加灵活，可以同时支持更多的参数类型：

```groovy
def x = 1.23
def result = ""
switch (x) {
   case "foo": result = "found foo"
   // lets fall through
   case "bar": result += "bar"
   case [4, 5, 6, 'inList']: result = "list"
   break
   case 12..30: result = "range"
   break
   case Integer: result = "integer"
   break
   case Number: result = "number"
   break
   case { it > 3 }: result = "number > 3"
   break
   default: result = "default"
}
assert result == "number"
```

- **==和equals**

在Groovy中，==相当于Java的equals，，如果需要比较两个对象是否是同一个，需要使用.is()。

```groovy
Object a = new Object()
Object b = a.clone()
 
assert a == b
assert !a.is(b)
```

## 编译、运行Groovy

在当面目录下创建build.gradle文件，在里面创建一个task，然后在task中编写Groovy代码即可，如下所示：

```groovy
task(yugangshuo).doLast {
   println "start execute yuangshuo"
   haveFun()
}
 
def haveFun() {
   println "have fun!"
   System.out.println("have fun!");
   1
   def file1 = new File("a.txt")
   def file2 = new File("a.txt")
   assert file1 == file2
   assert !file1.is(file2)
}
 
class Book {
   private String name
   String getName() { return name }
   void setName(String name) { this.name = name }
}
```

只需要在haveFun方法中编写Groovy代码即可，如下命令即可运行：

```groovy
gradle yugangshuo
```

