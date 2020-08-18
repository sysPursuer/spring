# 一、SpringBoot入门

## 1. springBoo简介

    简化Spring应用开发的一个框架；
    整个Spring技术栈的一个大整合；
    J2EE开发的一站式解决方案。

## 2. 微服务

2014，matin fowler
微服务：架构风格
一个应用应该是一组小型服务：可以通过HTTP的方式进行互通

每一个功能元素最终都是一个可独立替换和独立升级的软件单元。

学习SpringBoot必须掌握以下内容：

* Spring框架使用的经验[谷粒学院](http://www.gulixueyuan.com/)
* 环境约束：
* jdk1.8:spring boot 1.7及以上 java =version
* maven3.x  mvn -v
* IntellijIDEA2017、STS
* SpringBoot 1.5.9RELEASE

## 3.统一环境

### MAVEN设置

给maven的settings.xml配置文件的profiles标签添加

```xml
<profile>
    <id>jdk-1.8</id>
    <activation>
        <activeByDefault>true</activeByDefault>
        <jdk>1.8</jdk>
    </activation>
    <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
    </properties>
</profile>
```

## 4.SpringBoot HelloWorld

一个功能: 浏览器发送hello请求，服务器接收请求并处理，响应helloworld字符串。

### 4.1 创建一个maven工程（jar)

### 4.2 导入springBoot的依赖

```xml
<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.2.RELEASE</version>
    </parent>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
```



### 4.3 编写一个主程序

```java
/**
@SpringBoorApplication:标注一个主程序类，说明这是一个Spring Boot应用
 */
@SpringBootApplication
public class HelloWorldMainApplication {
    public static void main(String [] args){
        //Spring应用启动起来
        SpringApplication.run(HelloWorldMainApplication.class,args);
    }
}
```

### 4.4 编写相关的Controller、Service

```java
@Controller
public class HelloController {
    
    @ResponseBody
    @RequestMapping("/hello")
    public String hello(){
        return "Hello World";
        
    }
}
```

### 4.5 运行主程序

### 4.6 简化部署

```xml
<!--这个插件可以应用打包成一个可执行的jar包-->
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
```

将这个应用打成jar包，直接使用java -jar的命令进行执行

## 5.HelloWorld探究

### 5.1 POM文件

父项目

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.2.RELEASE</version>
</parent>
他的父项目是
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>1.5.2.RELEASE</version>
    <relativePath>../../spring-boot-dependencies</relativePath>
</parent>
他来真正管理SpringBoot应用里面的所有依赖版本；
```

Spring Boot的版本仲裁中心：

以后我们导入依赖默认是不需要写版本；（没有在dependencies里面管理的依赖自然需要声明版本号）

### 5.2 导入的依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>	
```

spring-boot-starter-web:

spring-boot-starter:spring-boot场景启动器；帮我们导入了web模块正常运行所依赖的组件；

SpringBoot将所有的功能场景都抽取出来做成一个个的starters(启动器)，只需要在项目里引入这些starter相关场景的所有依赖都会导入进来。要用什么功能就导入什么场景的启动器

### 5.3主程序类，主入口类



@SpringBootApplication: SpringBoot应用标注在某个类上说明这个类是SpringBoot的主配置类，SpringBoot就应该运行这个类的main方法来启动SpringBoot应用

```java
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {	
```

@**SpringBootConfiguration** SpringBoot的配置类；标注在某个类上，表示这是一个SpringBoot的配置类；

​		@**Configuration** 配置类上来标注这是个注解；配置类......配置文件；

​		配置类也是容器的一个组件；@Component

@**EnableAutoConfiguration** 开启自动配置功能

​		以前我们需要配置的东西，SpringBoot帮我们自动配置；@**EnableAutoConfiguration** 告诉SpringBoot开始自动配置功能；这样自动配置才能生效。

```java
@AutoConfigurationPackage
@Import({EnableAutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
```

@**AutoConfigurationPackage** 自动配置包

​	@Import({Registrar.class})： Spring的底层注解，给容器中导入一个组件；导入的组件有Registrar.class

​	**将主配置类（@SpringBootApplication标注的类）的所在包及下面所有子包里面的所有组件扫描到Spring容器**，若不在主程序类所在包下的包或者类将不能够被找到。

​	@Import({EnableAutoConfigurationImportSelector.class})将所有需要导入的组件以全类名的方式返回，这些组件就会被添加到容器中；会给容器中导入非常多的自动配置类；就是给容器这个场景需要的所有组件，并配置好

​	spring注解版（谷粒学院）

![自动配置](images/img2.jpg)

有了自动配置类，就免去了我们手动编写配置和注入功能组件等的工作；

**SpringFactoriesLoader.loadFactoryNames（EnableAutoConfiguration.class，classLoader）从类路径下的"META-INF/spring.factories"中获取EnableAutoConfiguration指定的值，将这些值作为自动配置类导入到容器中，自动配置类就生效，帮我们进行自动配置工作**；以前我们需要自己配置的东西，自动配置类帮我们完成了。

J2EE整体整合解决方案和自动配置都在spring-boot-autoconfigure-1.5.9.RELEASE.jar;

## 6.使用Spring Initializer快速创建SpringBoot项目

IDE都支持使用spring的项目，快速创建一个springboot项目：

选择我们需要的模块；向导会联网SpringBoot创建项目；

默认生成的SpringBoot项目；

* 主程序已经生成好了，只需要编写业务逻辑
* resources文件夹中的目录结构
  * static:保存所有的静态资源；js css images;
  * templates:保存所有的模板页面；（springBoot默认jar包使用嵌入式的tomcat,默认不支持jsp页面）；可以使用模板引擎（freemarker,thymeleaf）;
  * application.properties:SpringBoot应用的配置文件，可以修改一些默认设置

# 二、配置文件

## 1.配置文件

SpringBoot使用一个全局的配置文件，配置文件名是固定的；

* application.properties
* application.yml

配置文件的作用：修改SpringBoot自动配置的默认值；springBoot在底层都给我们自动配置好

YAML是一个标记语言，不是一个标记语言

标记语言：

​		以前的配置文件，大多都使用的是**xxx.xml**文件；	

​		YAML 以数据为中心，比json,xml等更适合做配置文件

## 2.YAML语法

### 2.1 基本语法

```yaml
k: v 标识一个键值对（空格必须有）
以空格的缩进来控制层级关系；只有左对齐的一列数据都是同一个层级的
server: 
	port: 8081
	path: /hello
属性和值都是大小写敏感的	
```



### 2.2 值的写法

#### 字面量；普通的值（数字、字符串、布尔）

k: v字面量直接写，自粗创不用加上单引号或者双引号；""双引号不会转义字符串里面的特殊字符；特殊字符会作为本身想表示的意思，‘’单引号回转义特殊字符，特殊字符最终只是一个普通的字符串数据

```yaml
name: "zhangsan\nlisi" 输出：zhangsan换行李四
name: 'zhangsan\nlisi' 输出：zhangsan\nlisi
```

#### 对象、Map（属性和值/键值对）

K: V对象还是k: v的方式

```yaml
friends:
	lastname: zhangsan
	age: 20
```

行内写法：

```yaml
friends: {lastname: zhangsan,age: 18}
```

#### 数组

用-值表示数组中的一个元素

```yaml
pets:
	- car
	- dog
	- pig
```

行内写法：

```yaml
pets: [cat,dog,pig]
```

### 2.3 配置文件值注入

#### ConfigurationProperties全局配置文件进行属性自动注入

```yaml
person:
  lastName: hello
  age: 19
  boss: false
  birth: 2017/12/12
  maps: {k1: v1,k2: 12}
  lists:
    - lisi
    - zhaoliu
  dog:
    name: 小狗
    age: 2
```

javabean

```java
/**
 * 将配置文件中配置的每一个属性的值，映射到这个组件
 * @ConfigurationProperties:告诉SpringBoot将本类中的所有属性和配置文件中相关的配置进行绑定
 *  prefix = "person"：配置文件中哪个下面的所有属性进行一一映射
 *
 *  只有这个组件是容器中组件，才能使用容易提供的功能
 */

@Component
@ConfigurationProperties(prefix = "person")
public class Person {
    private String lastName;
    private Integer age;
    private Boolean boss;
    private Date birth;

    private Map<String,Object> maps;
    private List<Object> lists;
    private Dog dog;

```

我们可以导入配置文件处理器，以后编写配置就有提示了

```xml
<!--导入配置文件处理器-->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-configuration-processor</artifactId>
			<optional>true</optional>
		</dependency>
```

properties配置文件默认为utf8编码，若出现乱码，可以在设置中file encoding将其转为ascii

```java
<property name="lastName" value="字面量/${key}从环境变量或配置文件中取值/#{SpEL}springExpressionLanguage表达式">
@Value注解也可以对属性的值进行注入
```

两种对javaBean注入值的区别

|                | @ConfigurationProperties | @Value     |
| -------------- | ------------------------ | ---------- |
| 功能           | 批量注入配置文件中的属性 | 一个个指定 |
| 松散语法绑定   | 支持                     | 不支持     |
| SpEL           | 不支持                   | 支持       |
| JSR303数据校验 | 支持                     | 不支持     |

配置文件yml还是properties都能获取值

如果只是在某个业务逻辑中获取一下配置文件中的某项值，就使用@Value

#### @PropertySource、@ImportResource

* @PropertySource只能用于properties文件，读取指定文件而不是全局的配置文件

  ```java
  //写一个对应bean.properties,将配置上的属性值自动绑定到对应bean的属性
  
  @PropertySource(value = {"classpath:bean.properties"})
  public class Bean{
      //...
  }
  ```

* @ImportResource读取外部配置文件，让配置文件里面的内容生效.

  springboot里面灭有Spring的配置文件，我们自己写的配置文件也不能自动识别，

  想让spring的配置文件生效加载进来，把@ImportResource标注在一个类上

  ```java
  //写一个beans.xml，使用<bean>标签进行属性值绑定
  @ImportResource(locations = "classpath:beans.xml")
  public class Bean{
      //...
  }
  ```

* SpringBoot推荐给容器中添加组件的方式：推荐使用全注解的方式

  ```java
  /**
   * @Configuration 指明当前类是一个配置类，就是来代替之前的spring配置文件
   *
   * 在配置文件中用<bean></bean>标签添加组件
   */
  @Configuration
  public class MyConfig {
  
      //将方法的返回值添加到容器中；容器中的组件默认的id就是方法名
      @Bean
      public SemanticAnalysis semanticAnalysis(){
          return new SemanticAnalysis();
      }
  }
  ```

## 3.profile多环境支持

### 3.1 多profile文件

​	我们在主配置文件编写的时候，文件名可以是application-{profile}.properties/yml

​	默认使用的是application.properties的配置

### 3.2 yml支持多文档格式

```yml
server:
	port: 8081
spring:
	profiles:
		active: dev	#指定激活哪个环境
---
server:
	port: 8082
spring:
	profiles: dev
---
server:
	port: 8083
spring:
	profiles: prod
```

### 3.3 激活指定profile

1、 在默认配置文件中指定spring.profiles.active=dev

2、命令行：`--spring.profiles.active=dev`

```java
java -jar spring-boot-02.jar --spring.profiles.active=dev
```

3、虚拟机参数: `-Dspring.profiles.active=dev`

## 4.配置文件的加载位置

application.properties/yml可以放在：

* 当前项目文件路径下/config
* 当前项目文件路径下
* 类路径下/config
* 类路径下

**优先级由高到低，高优先级的配置会覆盖低优先级的配置；springboot会从这四个位置全部加载主配置文件；互补配置**；

****

​	我们还可以通过spring.config.location来改变默认的配置文件位置

​	项目打包好后，我们可以使用命令行参数的形式，启动项目的时候来指定配置文件的新位置；指定配置文件和默认加载的这些配置文件共同起作用形成互补配置；

​	打包后进行外部配置

****

所有的加载来源可以参考[官方文档](https://docs.spring.io/spring-boot/docs/2.2.0.RELEASE/reference/htmlsingle/#boot-features-external-config)

## 5. 自动配置原理

配置文件到底能写什么？怎么写？自动配置原理；

[配置文件能配置的属性参照](https://docs.spring.io/spring-boot/docs/2.2.0.RELEASE/reference/htmlsingle/#common-application-properties)

### 5.1自动配置原理

* Springboot启动的时候加载主配置类、开启了自动配置功能`@EnableAutoConfiguration`

* `@EnableAutoConfiguration`作用：

  * 利用AutoConfigurationImportSelector给容器中导入一些组件
  * 可以查看selectorImports()方法的内容便知道导入哪些组件
  * `List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);`获取候选的配置
  * `SpringFactoriesLoader.loadFactoryNames`扫描所有类路径下`META-INF/spring.factories`的jar包
  * 把扫描到的这些文件的内容包装成properties对象
  * 从properties中获取到EnableAutoConfiguration.class类(类名)对应的值，把他们添加的容器中

* 将类路径下`META-INF/spring.factories`里面配置的所有EnableAutoConfiguration.class的值加入到容器中：

  ```properties
  # Auto Configure
  org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
  org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
  org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
  org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,\
  org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration,\
  org.springframework.boot.autoconfigure.cassandra.CassandraAutoConfiguration,\
  org.springframework.boot.autoconfigure.context.ConfigurationPropertiesAutoConfiguration,\
  org.springframework.boot.autoconfigure.context.LifecycleAutoConfiguration,\
  org.springframework.boot.autoconfigure.context.MessageSourceAutoConfiguration,\
  org.springframework.boot.autoconfigure.context.PropertyPlaceholderAutoConfiguration,\
  org.springframework.boot.autoconfigure.couchbase.CouchbaseAutoConfiguration,\
  org.springframework.boot.autoconfigure.dao.PersistenceExceptionTranslationAutoConfiguration,\
  org.springframework.boot.autoconfigure.data.cassandra.CassandraDataAutoConfiguration,\
  org.springframework.boot.autoconfigure.data.cassandra.CassandraReactiveDataAutoConfiguration,\
  org.springframework.boot.autoconfigure.data.cassandra.CassandraReactiveRepositoriesAutoConfiguration,\
  org.springframework.boot.autoconfigure.data.cassandra.CassandraRepositoriesAutoConfiguration,\
  org.springframework.boot.autoconfigure.data.couchbase.CouchbaseDataAutoConfiguration,\
  ...
  ```

  每一个这样的AutoConfiguration类都是容器中的一个组件，用他们来做自动配置

  * 每一个自动配置类进行自动配置功能

  * 以**HttpEncodingAutoConfiguration**为例解释自动配置原理

    ```java
    @Configuration(proxyBeanMethods = false)	//表示这是一个配置类，和以前编写的配置文件一样，可以给容器中添加组件
    @EnableConfigurationProperties(ServerProperties.class)	//启动指定类的ConfigurationProperties功能，即将配置文件中对应的值和ServerProperties绑定起来，并把ServerProperties加入到ioc容器中
    
    @ConditionalOnWebApplication(type = ConditionalOnWebApplication.Type.SERVLET)//spring底层注解，根据不同的条件，如果满足指定的条件，整个配置类里面的配置就会生效；判断当前应用是否为web应用，如果是，当前配置类生效
    
    @ConditionalOnClass(CharacterEncodingFilter.class)//判断当前项目有没有这个类CharacterEncodingFilter，即springmvc进行乱码解决的过滤器
    @ConditionalOnProperty(prefix = "server.servlet.encoding", value = "enabled", matchIfMissing = true)//判断配置文件中是否存在某个配置server.servlet.encoding，如果不存在，判断也是成立的。即使配置文件中不配置server.servlet.encoding，也是默认生效
    public class HttpEncodingAutoConfiguration {
    
        //他已经和springBoot的配置文件映射了
    	private final Encoding properties;
    	//只有一个有参构造器的情况下，参数的值就会从容器中拿
    	public HttpEncodingAutoConfiguration(ServerProperties properties) {
    		this.properties = properties.getServlet().getEncoding();
    	}
    
    	@Bean	//给容器中添加一个组件，这个组件的某些值需要从properties中获取
    	@ConditionalOnMissingBean
    	public CharacterEncodingFilter characterEncodingFilter() {
    		CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
    		filter.setEncoding(this.properties.getCharset().name());
    		filter.setForceRequestEncoding(this.properties.shouldForce(Encoding.Type.REQUEST));
    		filter.setForceResponseEncoding(this.properties.shouldForce(Encoding.Type.RESPONSE));
    		return filter;
    	}
    ```

    根据当前不同的条件判断，决定这个配置是否生效。

    一旦这个配置类生效；这个配置类就会给容器添加各种组件；这些组件的属性是从对应的properties类中获取的，这些类里面的每一个属性又是和配置文件绑定的

  * 所有的在配置文件中能配置的属性都是在xxxProperties类中封装着，配置文件能配置什么就可以参照某个功能对应的这个属性类

* **精髓：**
  * **SpringBoot启动会加载大量的自动配置类**
  * **我们看我们需要的功能有没有SpringBoot默认写好的自动配置类**
  * **我们再来看这个自动配置类中到底配置了哪些组件，只要我们要用的组件有，我们就不需要再来配置了**
  * **给容器中自动配置类添加组件的时候，会从properties类中获取某些属性。我们就可以在配置文件中指定这些属性的值**
* **xxxAutoConfiguration:自动配置类；给容器中添加组件：xxxProperties封装配置文件中相关属性**

### 5.2细节

#### 1.@Conditional派生注释(spring注解版原生的@Conditional作用)

* 作用：必须是@Conditional指定的条件成立，才给容器中添加组件，配置类里的内容才生效。

|                           |                          |
| ------------------------- | ------------------------ |
| @ConditionalOnMissingBean | 容器中是否存在指定的bean |
| @ConditionalOnClass       | 系统中有指定的类         |
| ...                       |                          |

* 自动配置类必须在一定的条件下才能生效；

* 我们可以通过启用`debug=true`属性，来让控制台打印自动配置报告，这样我们就可以很方便的知道哪些自动配置类生效

  ```java
  Positive matches:
  -----------------
  
     AopAutoConfiguration matched:
        - @ConditionalOnProperty (spring.aop.auto=true) matched (OnPropertyCondition)
  
     AopAutoConfiguration.ClassProxyingConfiguration matched:
        - @ConditionalOnMissingClass did not find unwanted class 'org.aspectj.weaver.Advice' (OnClassCondition)
        - @ConditionalOnProperty (spring.aop.proxy-target-class=true) matched (OnPropertyCondition)
  
     DispatcherServletAutoConfiguration matched:
        - @ConditionalOnClass found required class 'org.springframework.web.servlet.DispatcherServlet' (OnClassCondition)
        - found 'session' scope (OnWebApplicationCondition)
  
     DispatcherServletAutoConfiguration.DispatcherServletConfiguration matched:
        - @ConditionalOnClass found required class 'javax.servlet.ServletRegistration' (OnClassCondition)
        - Default DispatcherServlet did not find dispatcher servlet beans (DispatcherServletAutoConfiguration.DefaultDispatcherServletCondition)
  ...
  Negative matches:
  -----------------
  
     ActiveMQAutoConfiguration:
        Did not match:
           - @ConditionalOnClass did not find required class 'javax.jms.ConnectionFactory' (OnClassCondition)
  
     AopAutoConfiguration.AspectJAutoProxyingConfiguration:
        Did not match:
           - @ConditionalOnClass did not find required class 'org.aspectj.weaver.Advice' (OnClassCondition)
  
     ArtemisAutoConfiguration:
        Did not match:
           - @ConditionalOnClass did not find required class 'javax.jms.ConnectionFactory' (OnClassCondition)
  ...
  ```

# 三、日志

## 3.1日志框架

日志门面(抽象层)：SLF4J

日志实现：Logback

SpringBoot:底层是spring框架，spring框架默认使用的是JCL;SpringBoot选用SLF4j和Logback

## 3.2 SLF4j使用

### 3.2.1如何在系统中使用SLF4j

以后开发的时候，日志记录方法的调用，不应该来直接调用日志的实现类，而是调用日志抽象层里面的方法;给系统里面导入SLF4j的jar包和logback的实现jar

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class HelloWorld {
  public static void main(String[] args) {
    Logger logger = LoggerFactory.getLogger(HelloWorld.class);
    logger.info("Hello World");
  }
}
```

图示：

<img src="./images/concrete-bindings.png" alt="image2" style="zoom:60%;" />

每一个日志的实现框架都有自己的配置文件。使用slf4j以后，**配置文件还是做成日志实现框架的配置文件**

### 3.2.2 遗留问题

a(slf4j+logback):spring(commons-logging),hiberbate(jboss-logging),Mybatis...

统一日志记录，即使是别的框架，也和我一起统一使用slf4j进行输出

<img src="./images/legacy.png" alt="image3" style="zoom:75%;" />

**如何让系统中的所有日志都统一到slf4j**

* 将系统中其他日志框架先排除出去
* 用中间包来替换原有的日志框架
* 我们导入slf4j其他的实现

## 3.3 SpringBoot日志关系



在pom中右击选择diagrams->show dependecies显示项目的依赖关系

![image5](./images/img4.png)

****

总结：

* SpringBoot底层也是使用slf4j+logback的方式进行日志记录

* SpringBoot也把其他的日志都替换成slf4j

* 中间的替换包

  ![img5](./images/img5.png)

* 如果我们要引入其他框架？一定要把这个框架的默认日志依赖移除掉

  **引用SpringBoot能自动适配所有的日志，而且底层使用slf4j+logback的方式记录日志，引入其他框架的时候只需要把这个框架依赖的日志框架排除掉。**

## 3.4日志使用

```java
Logger logger = LoggerFactory.getLogger(getClass());
    @Test
    void contextLoads() {
        //日志的级别由低到高 trace<debug<info<warn<error
        //可以调整输出的日志级别，日志就会在这个级别及以上的高级别生效
        logger.trace("这是trace日志...");
        logger.debug("这是debug日志...");
        //没有指定，springBoot默认使用info级别,
        logger.info("这是info日志...");
        logger.warn("这是warn日志...");
        logger.error("这是error日志...");
    }
```

```html
<!-- 日志输出格式
    %d表示日期时间
    %thread表示线程名
    %-5level表示从左显示5个字符宽度
    %logger{50}表示logger名字最长50个字符，否则按照句点分割
    %msg日志消息
    %n换行符 -->
```

springBoot修改日志的默认配置

```properties
logging.level.com.sinvie.springboot=debug
#不指定路径，则在当前项目下生成springboot.log
#logging.file.name=springboot.log
#在当前磁盘根路径下创建spring文件夹和里面的log文件夹，，使用spring.log作为默认文件名
logging.file.path=/spring/log

#在控制台输出的日志的格式
logging.pattern.console=%d{yyyy-MM-dd} [%thread] %-5level %logger{50} - %msg%n
#指定文件中日志输出的格式
logging.pattern.file=%d{yyyy-MM-dd} [%thread] %-5level %logger{50} - %msg%n

```

