# 一、Mybatis简介

1.Mybaits将重要的步骤抽取出来可以人工定制，其他步骤自动化
2.重要的步骤都是写在配置文件中
3.完全解决数据库的优化问题
4.Mybatis底层就是对JDBC的一个简单封装
5.即将java编码和sql抽取了出来，还不会失去自动化功能，半自动的持久化层框架
6.mybatis是一个轻量级框架

# 二、用mybatis操作数据库

## 2.1以前

* 导包
   mybatis,mysql-connector-java,log4j
* 环境搭建
  创建一个java工程->创建测试库->封装数据的javabean->以及操作数据库的dao

****

## 2.2使用Mybatis

* 导包

* 写配置（两个：全局配置文件（知道mybatis运行的），dao接口的实现文件（描述dao中每个方法怎样工作的））
  * 第一个配置文件，称为mybatis的全局配置文件，知道mybatis如何正确运行，比如连接哪个数据库

  * 第二个配置文件，编写每一个方法如何向数据库发送sql语句，如何执行。。。相当于接口的实现类
     1)   将mapper的namespace属性改为接口的全类名
     2)   配置细节

     ```xml
     <?xml version="1.0" encoding="UTF-8" ?>
     <!DOCTYPE mapper
                 PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
                 "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
     <!--namespace:名称空间，写接口的全类名，相当于告诉mybatis这个配置文件是实现哪个接口的-->
     <mapper namespace="com.sinvie.dao.EmployeeDao">
         <!--select: 用来定义查询操作
             id:方法名，相当于这个配置是对于某个方法的实现
             resultType:指定方法的返回类型，查询操作必须指定的
             #{属性名}代表取出传递过来的某个参数的值
             -->
         <select id="getEmployeeId" resultType="getEmployeeId">
             select * from Blog where id = #{id}
         </select>
     </mapper>
     
     
     ```

     3）我们写的dao接口的实现文件mybatis默认不知道，我们需要在全局配置文件中注册

     ```xml
     <!--引入我们自己编写的每一个接口的实现文件-->
         <mappers>
             <!--resource表示从类路径下找资源-->
             <mapper resource="EmployeeDao.xml"/>
         </mappers>
     ```

     

* 测试，根据全局配置文件先创建

  ```java
  String resource = "mybatis-config.xml";
  InputStream inputStream = Resources.getResourceAsStream(resource);
  SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
  //获取sqlSession对象操作数据库即可
  	Employee employee = null;
          SqlSession openSession = null;
          try {
              openSession = sqlSessionFactory.openSession();
              EmployeeDao employeeDao = openSession.getMapper(EmployeeDao.class);
              employee = employeeDao.getEmployeeId(1);
          }
          finally {
              openSession.close();
          }
          System.out.println(employee);
  ```

  

* 如何xml有提示，只要idea找到了约束文件
  找到文件->绑定位置

****

总结：**两个文件**
1）、全局配置文件：mybatis-config.xml指导mybatis正确运行的一些全局设置
2）、SQL映射文件：EmployeeDao.xml 相当于对Dao接口的一个实现描述
  细节：1.获取的是接口的代理对象：mybatis自动创建的
        2.SqlSessionFactory创建SqlSession，Factory只new一次
        3.SqlSession相当于和数据库进行交互的，和数据库的一次会话，就应该创建一个新的SqlSssion

## 2.3参数传递

1.单个参数：基本类型-取值#{参数名随便写} ，
2.多个参数
#{参数名}无效，可用0,1参数的索引，或者param1,param2第几个参数
原因只要传入了多个参数，mybatis会自动的将这些参数封装在一个map中，封装时使用的key就是参数的索引和参数的第几个表示
#{key}就是从这个map的值
可用告诉mybatis，@Param为参数指定key,命名参数
可以传map
无论传入什么参数都要正确的取出值
1）#{key}取值的时候可以规定一些规则

#{}${}取值的区别
#{属性名}：是参数预编译的方式，参数的位置都是用？代替，参数后来都是预编译设置进去的：安全，不会有mysql注入问题
${属性名}不是参数预编译，而是直接和sql语句进行拼串：不安全
虽然不安全但是也有应用场景：sql语句只有参数位置是支持预编译的；
select * from t_employee where id = #{id}
表格动态查询（在不支持参数预编译的位置要进行取值就使用${}）：
select * from ${t_employee} where id = #{id}

# 三、SSM

## 3.1整合过程

SSM;spring+springmvc+mybatis
1.导包
spring: ioc核心，jdbc核心 测试
springmvc: web核心,上传下载，数据校验，Ajax,验证码
mybatis:mybatis核心、encache
数据源驱动

2.写配置
web配置：启动Spring的ioc容器（指定spring配置文件的）
springmvc配置：拦截器->编码->文件上传下载等
spring配置：扫描包->配数据源->

mybatis配置: 全局配置->Dao映射文件配置
整合的关键：注册得到SqlSessionFactoryBean;将dao接口添加到ioc容器中

```xml
<!--根据配置文件得到SqlSessionFactory-->
    <bean class="org.mybatis.spring.SqlSessionFactoryBean" id="sqlSessionFactoryBean">
        <property name="configLocation" value="classpath:mybatis/mybatis-config.xml"></property>
        <property name="dataSource" ref="dataSource"/>
        <!--指定xml映射文件的位置-->
        <property name="mapperLocations" value="mybatis/mapper/*.xml"/>
    </bean>
    <!--要把每个dao接口的实现加入到ioc容器-->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.sinvie.dao"></property>
    </bean>
```

**整合步骤**
1、导入整合包，（能将dao的实现加入到容器中）（mybatis的核心是sqlSessionFactory根据全局配置文件闯将sqlSession对象，sqlSession对象根据映射文件配置进行增删改查）
mybatis-spring-.jar

## 3.2测试

## 3.3Mybatis MBG

1）导包

```xml
<dependency>
    <groupId>org.mybatis.generator</groupId>
    <artifactId>mybatis-generator-core</artifactId>
    <version>1.4.0</version>
</dependency>
```

