# SSM框架搭建

本章节主要是给大家介绍如何整合SpringMVC、Spring、Mybatis的框架的步骤，使大家能够顺利搭建一个完整的Java EE企业级开发框架来开发软件项目。

本次知识单元包含如下知识点和能力要求：

![image-20251226133826297](E:\GitHub\tech-blog-articles\posts\SSM框架搭建.assets\image-20251226133826297.png)

SSM框架整合，顾名思义这SSM是三个框架的缩写（S：SpringMVC S:Spring M：Mybatis），之前我们都在前面的课程中学习了Web层的SpringMVC框架、Service层的Spring框架以及Dao层的Mybatis框架，但是我们都是分别去进行使用的，没有把他们整合到一起来使用。由于MyBatis框架和我们Spring系列框架不是一个公司的出品，所以我们需要在整合前下载一些整合需要用的Jar包。

### 1）框架简介

#### 1.1、Spring

Spring 是一个开源框架， Spring 是于 2003 年兴起的一个轻量级的 Java 开发框架，由 Rod Johnson 在其著作 Expert One-On-One J2EE Development and Design 中阐述的部分理念和原型衍生而来。它是为了解决企业应用开发的复杂性而创建的。 Spring 使用基本的 JavaBean 来完成以前只可能由 EJB 完成的事情。然而， Spring 的用途不仅限于服务器端的开发。从简单性、可测试性和松耦合的角度而言，任何 Java 应用都可以从 Spring 中受益。 简单来说， Spring 是一个轻量级的控制反转（ IoC ）和面向切面（ AOP ）的容器框架。

 

#### 1.2、SpringMVC

Spring MVC 属于 SpringFrameWork 的后续产品，已经融合在 Spring Web Flow 里面。 Spring MVC 分离了 控制器 、模型 对象 、分派器以及处理程序对象的角色，这种分离让它们更容易进行定制。

 

#### 1.3、MyBatis

MyBatis本是 apache 的一个开源项目iBatis, 2010年这个项目由apache software foundation 迁移到了google code，并且改名为 MyBatis 。 MyBatis 是一个基于 Java 的 持久层 框架。 iBATIS 提供的 持久层 框架包括 SQL Maps 和 Data Access Objects （ DAO ） MyBatis 消除了几乎所有的 JDBC 代码和参数的手工设置以及结果集的检索。 MyBatis 使用简单的 XML 或注解用于配置和原始映射，将接口和 Java 的 POJOs （ Plain Old Java Objects ，普通的 Java对象）映射成数据库中的记录。

###  2） 所需Jar包

![image-20251226134055685](E:\GitHub\tech-blog-articles\posts\SSM框架搭建.assets\image-20251226134055685.png)

### 3）简单给大家介绍一下上述Jar包的作用

  ① AOP所需要用到的Jar包（Spring框架中的非必须）：

 aopalliance

aspectjweaver

cglib-nodep

Spring-aop

Spring AOPjar包

（包含动态代理等）



**注：Spring5之后，Spring-aop jar包成为Spring框架必须jar包组件之一，必须导入**

 **② Spring 框架（包含Spring MVC）所需Jar包：**

`Spring-beans`

`Spring-core`

`Spring-context`

`Spring-expression`



**Spring 框架必须jar包**

`Spring-webmvc`

`Spring-web`



**Spring MVC框架所需jar包**

`Spring-jdbc`

`Spring-tx`



**SpringJDBC与事务jar包**

`Log4j`

`Commons-log`



**Spring日志相关jar包**

 **③ Mybatis框架所需Jar包：**

`Mysql-connection-java`

`Druid`

`Mybatis`

`Mybatis-spring`



1.数据库连接jar包

2.数据库连接池jar包

3.mybatisjar包

4.与spring整合jar包

​    **④ 其他工具Jar包：**

`Jstl`

`Standard`

**JSTL所需jar包**

`Fastjson`

`Lombok`

 Json与快速开发所需jar包

大家通过上面的jar包也发现，在Mybatis框架中多了一个我们没有用过的整合jar包（Mybatis-spring），这个Jar包就是我们整合Mybatis框架与Spring框架中间的连接jar包，它就相当于是桥接我们两个框架之间的纽带，必须要有它我们才可以整合SSM框架。

### 导入Log4j及数据库配置信息文件

Log4j.properties

```properties

# rootLogger是所有日志的根日志,修改该日志属性将对所有日志起作用

# 下面的属性配置中,所有日志的输出级别是info,输出源是console

log4j.rootLogger=info,console

# 定义输出源的输入位置是控制台

log4j.appender.console=org.apache.log4j.ConsoleAppender

# 定义输出日志的布局采用的类

log4j.appender.console.layout=org.apache.log4j.PatternLayout

# 定义日志输出布局

log4j.appender.console.layout.ConversionPattern=%d %p [%c] - %m%n
```

DBInfo.properties

```properties

jdbc.driver=com.mysql.jdbc.Driver

#自己数据库的地址

jdbc.url=jdbc:mysql://localhost:3306/test 

#自己数据库的用户名

jdbc.username=root

#自己数据库的密码

jdbc.password=xxx
```

### 配置Spring的配置文件applicationContext.xml文件

```xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"

         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"

         xmlns:context="http://www.springframework.org/schema/context"

         xmlns:aop="http://www.springframework.org/schema/aop"

       xmlns:tx="http://www.springframework.org/schema/tx"     

       xsi:schemaLocation="

          http://www.springframework.org/schema/beans

          http://www.springframework.org/schema/beans/spring-beans-4.0.xsd

          http://www.springframework.org/schema/tx

        http://www.springframework.org/schema/tx/spring-tx-4.0.xsd

          http://www.springframework.org/schema/context

          http://www.springframework.org/schema/context/spring-context-4.0.xsd

          http://www.springframework.org/schema/aop

        http://www.springframework.org/schema/aop/spring-aop-4.0.xsd">

  <context:property-placeholder   location="classpath:DBInfo.properties"/>

  <!-- 1.创建数据库连接池对象 -->

  <bean   id="dataSource"   class="com.alibaba.druid.pool.DruidDataSource">

    <!-- 1.2配置数据库信息 -->

    <!-- 基本配置 -->

    <property   name="url" value="${jdbc.url}"></property>

        <property   name="driverClassName" value="${jdbc.driver}" />

        <property   name="username"   value="${jdbc.username}"></property>

        <property   name="password"   value="${jdbc.password}"></property>

        <!-- 数据库连接池其他配置 -->

         <!-- 配置初始化大小 最大和最小 -->

        <property   name="initialSize" value="1"></property>

        <property   name="minIdle" value="1"></property>

        <property   name="maxActive" value="200"></property>

        <!-- 配置获取连接等待超时的时间 -->

        <property   name="maxWait" value="60000"></property>

        <!-- 配置间隔多久进行一次检测，检测需要关闭的空闲连接，单位是毫秒 -->

        <property   name="timeBetweenEvictionRunsMillis"   value="60000"></property>

        <!-- 配置一个连接在池中最小生存的时间 单位是毫秒 -->

        <property   name="minEvictableIdleTimeMillis"   value="300000"></property>

        <!-- 打开PSCache，并且指定每个连接上PSCache的大小 -->

        <property   name="poolPreparedStatements" value="true"/>

        <property   name="maxPoolPreparedStatementPerConnectionSize"   value="20"/>

        <!-- 配置监控统计拦截的filters，去掉后监控界面sql无法统计 -->

        <property   name="filters" value="stat"/>

  </bean>

  <!-- 2.创建SqlSessionFactory对象 -->

  <bean   id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">

    <!-- 数据源导入 -->

    <property   name="dataSource" ref="dataSource"></property>

    <!-- 主配置文件 -->

    <property   name="configLocation"   value="classpath:mybatis-config.xml"></property>

  </bean>

  <!--2.2.配置mybatis注解 (假设不使用注解的配置方式，这步可以省略)-->

    <bean   class="org.mybatis.spring.mapper.MapperScannerConfigurer">

        <property   name="basePackage" value="com.iflytek.dao" />

    </bean>

      <!-- 3.配置事务管理器 -->

    <!-- class 为在spring jdbc.jar包当中 -->

    <bean id="txManger"   class="org.springframework.jdbc.datasource.DataSourceTransactionManager">

     <!-- 3.1 对于哪个数据源进行事务管理 -->

     <property name="dataSource"   ref="dataSource"></property>

    </bean>

    <!-- 4.声明式事务 -->

    <tx:annotation-driven   transaction-manager="txManger"/>

    <!-- 5.注解扫描器 -->

    <context:component-scan   base-package="com.iflytek.*">

     <context:exclude-filter type="annotation"   expression="org.springframework.stereotype.Controller"/>

      </context:component-scan>

</beans>
```

### Spring MVC配置文件

springmvc-config.xml

```xml
springmvc-config.xml:

<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"

         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"

       xmlns:context="http://www.springframework.org/schema/context"

         xmlns:mvc="http://www.springframework.org/schema/mvc"

         xsi:schemaLocation="http://www.springframework.org/schema/beans

           http://www.springframework.org/schema/beans/spring-beans.xsd

        http://www.springframework.org/schema/context

          http://www.springframework.org/schema/context/spring-context-4.3.xsd

          http://www.springframework.org/schema/mvc

          http://www.springframework.org/schema/mvc/spring-mvc-4.3.xsd">

  <!-- 1.注解扫描器 -->

  <context:component-scan   base-package="com.iflytek.*">

    <context:include-filter   type="annotation"   expression="org.springframework.stereotype.Controller"/>

  </context:component-scan>

  <!-- 2.视图解析器 -->

  <bean   id="internalResourceViewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">

        <!-- 前缀 -->

<!--         <property   name="prefix" value=""/> -->

        <!-- 后缀 -->

        <property   name="suffix" value=".jsp"/>

    </bean>

    <!-- 3.注解驱动 -->

    <mvc:annotation-driven   conversion-service="dateCoverter">

     <!-- 2.1消息转换器 -->

        <!-- 解决@ResponseBody中文乱码 -->

          <mvc:message-converters register-defaults="true">

            <bean   class="org.springframework.http.converter.StringHttpMessageConverter">

                <property   name="supportedMediaTypes"   value="text/html;charset=UTF-8"/>

            </bean>

          </mvc:message-converters>

    </mvc:annotation-driven>

    <!-- 4.静态资源过滤 -->

    <mvc:resources   location="/" mapping="/*.html"></mvc:resources>

      <mvc:default-servlet-handler/>

  <!-- 5.日期转换器 -->

  <bean   id="dateCoverter"   class="org.springframework.format.support.FormattingConversionServiceFactoryBean">

    <property   name="converters">

      <set>

        <bean   class="com.iflytek.util.DateConvert"></bean>

      </set>

    </property>

  </bean>

<!-- 6.文件上传 -->

  <bean   id="multipartResolver"   class="org.springframework.web.multipart.commons.CommonsMultipartResolver">

     <property   name="maxUploadSize" value="104857600" />

     <property   name="maxInMemorySize" value="4096" />

     <property   name="defaultEncoding" value="UTF-8"></property>

  </bean>

</beans>
```

### Mybatis框架配置

mybatis-config.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!DOCTYPE configuration 

  PUBLIC   "-//mybatis.org//DTD Config 3.0//EN" 

    "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>

    <!-- 配置显示SQL语句   -->

  <settings>

    <setting   name="logImpl" value="STDOUT_LOGGING" />

   </settings>

  <mappers>

    <mapper   class="com.iflytek.dao.xxxDao"/>

  </mappers>

</configuration>
```

配置好上述我们SSM框架配置文件就已经都配置完成了，接下来我们就要配置Web.xml文件了

### 配置我们Web.xml文件，加载所有配置

Web.xml

```xml

<?xml version="1.0" encoding="UTF-8"?>

<web-app   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"   xmlns="http://xmlns.jcp.org/xml/ns/javaee"   xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee   http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd" id="WebApp_ID"   version="3.1">

    <display-name>SSM-18</display-name>

  <welcome-file-list>

      <welcome-file>index.html</welcome-file>

      <welcome-file>index.htm</welcome-file>

      <welcome-file>index.jsp</welcome-file>

      <welcome-file>default.html</welcome-file>

    <welcome-file>default.htm</welcome-file>

      <welcome-file>default.jsp</welcome-file>

  </welcome-file-list>

  <!-- 1.SpringMVC 核心分发器   -->

  <servlet>

   <servlet-name>SpringMVC</servlet-name>

   <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>

   <!-- 初始化参数 -->

   <init-param>

    <param-name>contextConfigLocation</param-name>

    <param-value>classpath:springmvc-config.xml</param-value>

   </init-param>

      <!-- 自启 -->

   <load-on-startup>1</load-on-startup>

  </servlet>

  <servlet-mapping>

   <servlet-name>SpringMVC</servlet-name>

   <url-pattern>/</url-pattern>

  </servlet-mapping>

  <!-- 2.配置Spring主配置文件   -->

  <context-param>

   <param-name>contextConfigLocation</param-name>

   <param-value>classpath:applicationContext.xml</param-value>

  </context-param>

  <!-- 3.配置Spring监听器 -->

  <listener>

   <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>

  </listener>

   <!-- 第四步：防止参数乱码问题（可选） -->

  <!-- 使用过滤器解决中文参数乱码问题 -->

 <filter>

      <filter-name>encodingFilter</filter-name>

      <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>

    <init-param>

        <param-name>encoding</param-name>

        <param-value>UTF-8</param-value>

    </init-param>

    <init-param>

        <param-name>forceEncoding</param-name>

        <param-value>true</param-value>

    </init-param>

  </filter>

  <filter-mapping>

      <filter-name>encodingFilter</filter-name>

    <url-pattern>/*</url-pattern>

  </filter-mapping>

</web-app>
```

整合好的项目已经正常启动了，没有报错。这样我们的SSM框架就已经搭建好了。