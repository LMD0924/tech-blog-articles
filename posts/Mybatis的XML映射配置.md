# 📝 MyBatis XML映射配置与日志系统全解析

## 🎯 引言

在现代Java企业级开发中，数据持久化和日志记录是两个至关重要的环节。MyBatis作为优秀的ORM框架，提供了灵活的SQL映射方式；而Logback作为成熟的日志框架，则是应用监控和问题排查的利器。本文将深入探讨这两种技术的配置与最佳实践。

---

## 📊 第一章：MyBatis XML映射配置详解

### 1.1 XML配置 vs 注解配置：如何选择？

MyBatis提供了两种SQL映射方式：

| 方式         | 优点                                                     | 缺点                                              | 适用场景                                   |
| ------------ | -------------------------------------------------------- | ------------------------------------------------- | ------------------------------------------ |
| **XML配置**  | SQL与Java代码分离，便于维护<br>支持动态SQL<br>可读性更好 | 文件较多<br>需要额外维护                          | 复杂SQL语句<br>需要动态SQL<br>团队规范要求 |
| **注解配置** | 简洁直观<br>减少文件数量<br>快速开发                     | SQL混在代码中<br>复杂SQL可读性差<br>不支持动态SQL | 简单CRUD操作<br>原型开发<br>微服务简单查询 |

### 1.2 XML映射配置三大黄金法则

#### 🎯 法则一：名称与位置对应
```
src/main/java/
└── com/example/mapper/
    ├── UserMapper.java          # Mapper接口
    └── UserMapper.xml          # XML映射文件（同名）
```

#### 🎯 法则二：namespace精确匹配
```xml
<!-- XML中namespace必须与Mapper接口全限定名一致 -->
<mapper namespace="com.example.mapper.UserMapper">
    <!-- SQL语句 -->
</mapper>
```

#### 🎯 法则三：方法名与SQL ID一致
```java
// Mapper接口
@Mapper
public interface UserMapper {
    List<User> findAll();      // 方法名：findAll
    User findById(Long id);    // 方法名：findById
}
```

```xml
<!-- XML映射文件 -->
<mapper namespace="com.example.mapper.UserMapper">
    <!-- id与接口方法名一致 -->
    <select id="findAll" resultType="com.example.entity.User">
        SELECT * FROM user
    </select>
    
    <select id="findById" resultType="com.example.entity.User">
        SELECT * FROM user WHERE id = #{id}
    </select>
</mapper>
```

### 1.3 完整的XML映射文件示例

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" 
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.example.mapper.UserMapper">
    
    <!-- 结果映射配置 -->
    <resultMap id="UserResultMap" type="com.example.entity.User">
        <id property="id" column="id" />
        <result property="username" column="username" />
        <result property="password" column="password" />
        <result property="name" column="name" />
        <result property="age" column="age" />
        <result property="createTime" column="create_time" />
        <result property="updateTime" column="update_time" />
    </resultMap>
    
    <!-- 基础查询：查询所有用户 -->
    <select id="findAll" resultMap="UserResultMap">
        SELECT id, username, password, name, age, 
               create_time, update_time
        FROM user
        WHERE status = 1
        ORDER BY create_time DESC
    </select>
    
    <!-- 条件查询：根据ID查询 -->
    <select id="findById" resultMap="UserResultMap">
        SELECT id, username, password, name, age, 
               create_time, update_time
        FROM user
        WHERE id = #{id} AND status = 1
    </select>
    
    <!-- 动态SQL：多条件查询 -->
    <select id="findByCondition" resultMap="UserResultMap">
        SELECT * FROM user
        WHERE status = 1
        <if test="username != null and username != ''">
            AND username LIKE CONCAT('%', #{username}, '%')
        </if>
        <if test="minAge != null">
            AND age >= #{minAge}
        </if>
        <if test="maxAge != null">
            AND age &lt;= #{maxAge}
        </if>
        <choose>
            <when test="orderBy == 'age'">
                ORDER BY age ${orderType}
            </when>
            <otherwise>
                ORDER BY create_time DESC
            </otherwise>
        </choose>
    </select>
    
    <!-- 插入数据 -->
    <insert id="insert" useGeneratedKeys="true" keyProperty="id">
        INSERT INTO user (
            username, password, name, age, 
            create_time, update_time
        ) VALUES (
            #{username}, #{password}, #{name}, #{age},
            NOW(), NOW()
        )
    </insert>
    
    <!-- 更新数据 -->
    <update id="update">
        UPDATE user
        <set>
            <if test="username != null">username = #{username},</if>
            <if test="password != null">password = #{password},</if>
            <if test="name != null">name = #{name},</if>
            <if test="age != null">age = #{age},</if>
            update_time = NOW()
        </set>
        WHERE id = #{id} AND status = 1
    </update>
    
    <!-- 逻辑删除 -->
    <update id="deleteById">
        UPDATE user 
        SET status = 0, update_time = NOW()
        WHERE id = #{id}
    </update>
    
</mapper>
```

### 1.4 XML映射文件位置配置

在Spring Boot的`application.yml`中配置：

```yaml
# MyBatis配置
mybatis:
  # XML映射文件位置（支持通配符）
  mapper-locations: 
    - classpath:mapper/**/*.xml
    - classpath*:com/**/mapper/*.xml
  
  # 类型别名包扫描
  type-aliases-package: com.example.entity
  
  # 配置XML文件中可以使用简短的类名
  configuration:
    map-underscore-to-camel-case: true  # 自动转换下划线到驼峰
    default-fetch-size: 100             # 默认获取数量
    default-statement-timeout: 30       # 超时时间(秒)
    
  # 全局配置
  global-config:
    db-config:
      id-type: auto                     # ID自增
      logic-delete-value: 0             # 逻辑删除值
      logic-not-delete-value: 1         # 逻辑未删除值
```

**多环境配置示例：**
```yaml
# application-dev.yml (开发环境)
mybatis:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl  # 打印SQL日志

# application-prod.yml (生产环境)
mybatis:
  configuration:
    log-impl: org.apache.ibatis.logging.slf4j.Slf4jImpl    # 使用Slf4j日志
```

---

## 📁 第二章：配置文件格式详解

### 2.1 三种配置文件格式对比

| 格式           | 扩展名         | 优点                               | 缺点                               | 适用场景                                  |
| -------------- | -------------- | ---------------------------------- | ---------------------------------- | ----------------------------------------- |
| **Properties** | `.properties`  | 简单易读<br>广泛支持<br>键值对清晰 | 不支持层级结构<br>配置复杂时冗长   | 简单配置<br>系统属性<br>兼容性要求高      |
| **YAML**       | `.yml`/`.yaml` | 结构清晰<br>支持层级<br>可读性好   | 缩进敏感易出错<br>部分工具支持不足 | Spring Boot项目<br>复杂配置<br>微服务配置 |
| **XML**        | `.xml`         | 结构严谨<br>验证方便<br>工具支持好 | 冗长繁琐<br>可读性较差             | 传统项目<br>需要DTD/XSD验证               |

### 2.2 YAML配置文件最佳实践

```yaml
# application.yml - 完整示例
server:
  port: 8080
  servlet:
    context-path: /api
    encoding:
      charset: UTF-8
      enabled: true
      force: true

spring:
  application:
    name: user-service
    
  # 数据源配置
  datasource:
    url: jdbc:mysql://localhost:3306/user_db?useSSL=false&serverTimezone=Asia/Shanghai
    username: root
    password: 123456
    driver-class-name: com.mysql.cj.jdbc.Driver
    hikari:
      connection-timeout: 30000
      maximum-pool-size: 20
      minimum-idle: 5
      idle-timeout: 600000
      
  # JPA配置（可选）
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    properties:
      hibernate:
        dialect: org.hibernate.dialect.MySQL8Dialect
        format_sql: true
        
# MyBatis配置
mybatis:
  mapper-locations: classpath*:mapper/**/*.xml
  configuration:
    map-underscore-to-camel-case: true
    cache-enabled: true
    
# 日志配置
logging:
  level:
    com.example.mapper: debug    # MyBatis Mapper层日志
    org.springframework.web: info
    root: warn
  file:
    name: logs/application.log
    max-size: 10MB
    max-history: 30
    
# 自定义配置
app:
  config:
    upload:
      max-size: 10MB
      allowed-types: jpg,png,pdf
    security:
      token-expire: 7200         # token过期时间(秒)
      secret-key: my-secret-key-2024
```

### 2.3 多环境配置策略

```yaml
# 激活指定配置文件
# 命令行参数：--spring.profiles.active=dev
# 环境变量：SPRING_PROFILES_ACTIVE=dev

# 默认配置（所有环境共享）
spring:
  profiles:
    active: @activatedProperties@  # Maven属性替换

---
# 开发环境配置
spring:
  config:
    activate:
      on-profile: dev

server:
  port: 8081

logging:
  level:
    root: debug

---
# 测试环境配置
spring:
  config:
    activate:
      on-profile: test

server:
  port: 8082

datasource:
  url: jdbc:mysql://test-db:3306/user_db

---
# 生产环境配置
spring:
  config:
    activate:
      on-profile: prod

server:
  port: 80
  ssl:
    enabled: true

logging:
  level:
    root: warn
  file:
    name: /var/log/user-service/application.log
```

---

## 📋 第三章：Logback日志框架深度解析

### 3.1 日志框架架构

```
Logback架构层次：
┌─────────────────────────────────────┐
│   Application Code                  │
│   Logger.debug("message")           │
├─────────────────────────────────────┤
│   Logback Core                      │
│   ┌─────────┐ ┌─────────┐           │
│   │Logger   │ │Appender │           │
│   └─────────┘ └─────────┘           │
├─────────────────────────────────────┤
│   Logback Classic                   │
│   ┌─────────┐ ┌─────────┐ ┌───────┐ │
│   │Encoder  │ │Layout   │ │Filter │ │
│   └─────────┘ └─────────┘ └───────┘ │
└─────────────────────────────────────┘
```

### 3.2 依赖引入（Spring Boot项目）

Spring Boot Starter已包含Logback依赖，无需显式添加：
```xml
<!-- Spring Boot Web Starter已传递依赖Logback -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<!-- 如需自定义配置，可排除默认日志配置 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

### 3.3 日志记录基础用法

```java
package com.example.service;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Service;

@Service
public class UserService {
    
    // 最佳实践：使用当前类作为Logger名称
    private static final Logger log = LoggerFactory.getLogger(UserService.class);
    
    // 对于Lombok用户，可以使用注解简化
    // @Slf4j
    // @Component
    // public class UserService { ... }
    
    public User getUserById(Long id) {
        // TRACE级别：最详细的日志，通常用于调试
        log.trace("进入getUserById方法，参数id={}", id);
        
        // DEBUG级别：调试信息，开发时使用
        log.debug("开始查询用户，ID: {}", id);
        
        try {
            User user = userMapper.findById(id);
            
            if (user == null) {
                // WARN级别：警告信息，需要注意但不是错误
                log.warn("未找到用户，ID: {}", id);
                return null;
            }
            
            // INFO级别：重要业务流程信息
            log.info("成功查询到用户: {}, 用户名: {}", id, user.getUsername());
            
            // 敏感信息不要记录在日志中！
            // ❌ log.info("用户密码: {}", user.getPassword());
            
            return user;
            
        } catch (Exception e) {
            // ERROR级别：错误信息，需要立即处理
            log.error("查询用户失败，ID: {}", id, e);
            throw new ServiceException("查询用户失败", e);
        } finally {
            log.trace("退出getUserById方法");
        }
    }
    
    /**
     * 日志记录最佳实践示例
     */
    public void processBatch(List<Long> ids) {
        long startTime = System.currentTimeMillis();
        
        log.info("开始批量处理，数量: {}", ids.size());
        
        int successCount = 0;
        int failCount = 0;
        
        for (Long id : ids) {
            try {
                processSingle(id);
                successCount++;
                
                // 每处理100条记录一次进度
                if (successCount % 100 == 0) {
                    log.debug("处理进度: {}/{}", successCount, ids.size());
                }
                
            } catch (Exception e) {
                failCount++;
                log.error("处理记录失败，ID: {}, 错误: {}", id, e.getMessage());
            }
        }
        
        long endTime = System.currentTimeMillis();
        long duration = endTime - startTime;
        
        log.info("批量处理完成，成功: {}，失败: {}，耗时: {}ms", 
                 successCount, failCount, duration);
        
        if (failCount > 0) {
            log.warn("本次处理有{}条记录失败", failCount);
        }
    }
}
```

### 3.4 Logback配置文件详解

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- logback.xml - 企业级配置示例 -->
<configuration 
    xmlns="http://ch.qos.logback/xml/ns/logback"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://ch.qos.logback/xml/ns/logback 
    https://raw.githubusercontent.com/enricopulatzo/logback-XSD/master/src/main/xsd/logback.xsd">
    
    <!-- 定义属性变量 -->
    <property name="LOG_HOME" value="logs" />
    <property name="APP_NAME" value="user-service" />
    <property name="LOG_PATTERN" value="%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n" />
    <property name="MAX_HISTORY" value="30" />
    <property name="MAX_FILE_SIZE" value="10MB" />
    
    <!-- 控制台输出 -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <!-- 彩色日志输出 -->
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} %highlight(%-5level) %cyan(%logger{36}) - %msg%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
        <!-- 过滤器：只输出WARN及以上级别到控制台 -->
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>WARN</level>
        </filter>
    </appender>
    
    <!-- 开发环境控制台输出（更详细） -->
    <appender name="DEV_CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} %highlight(%-5level) %cyan(%logger{20}) - %msg%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>
    
    <!-- 文件输出：所有日志 -->
    <appender name="FILE_ALL" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_HOME}/${APP_NAME}.log</file>
        <!-- 滚动策略：按时间+大小 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 每天一个文件，保存30天 -->
            <fileNamePattern>${LOG_HOME}/${APP_NAME}.%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy 
                class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>${MAX_FILE_SIZE}</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <maxHistory>${MAX_HISTORY}</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>${LOG_PATTERN}</pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>
    
    <!-- 文件输出：错误日志单独文件 -->
    <appender name="FILE_ERROR" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_HOME}/${APP_NAME}-error.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${LOG_HOME}/${APP_NAME}-error.%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>60</maxHistory> <!-- 错误日志保留更久 -->
        </rollingPolicy>
        <encoder>
            <pattern>${LOG_PATTERN}</pattern>
            <charset>UTF-8</charset>
        </encoder>
        <!-- 过滤器：只记录ERROR级别 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>ERROR</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>
    
    <!-- 异步日志输出（提升性能） -->
    <appender name="ASYNC_FILE" class="ch.qos.logback.classic.AsyncAppender">
        <!-- 不丢失日志。默认如果队列剩余80%，则丢弃TRACT、DEBUG、INFO级别日志 -->
        <discardingThreshold>0</discardingThreshold>
        <!-- 更改默认的队列深度，该值会影响性能。默认256 -->
        <queueSize>1024</queueSize>
        <!-- 添加附加的appender，最多只能添加一个 -->
        <appender-ref ref="FILE_ALL"/>
    </appender>
    
    <!-- 日志级别详解 -->
    <!-- 
        TRACE < DEBUG < INFO < WARN < ERROR
        日志级别继承：子包继承父包级别，除非单独设置
    -->
    
    <!-- 根日志配置 -->
    <root level="INFO">
        <appender-ref ref="CONSOLE" />
        <appender-ref ref="ASYNC_FILE" />
        <appender-ref ref="FILE_ERROR" />
    </root>
    
    <!-- 开发环境特殊配置 -->
    <springProfile name="dev">
        <root level="DEBUG">
            <appender-ref ref="DEV_CONSOLE" />
        </root>
        
        <!-- MyBatis SQL日志 -->
        <logger name="com.example.mapper" level="DEBUG" />
        
        <!-- Spring框架日志 -->
        <logger name="org.springframework" level="INFO" />
        <logger name="org.springframework.web" level="DEBUG" />
        <logger name="org.springframework.transaction" level="DEBUG" />
    </springProfile>
    
    <!-- 生产环境特殊配置 -->
    <springProfile name="prod">
        <root level="WARN">
            <appender-ref ref="CONSOLE" />
            <appender-ref ref="ASYNC_FILE" />
            <appender-ref ref="FILE_ERROR" />
        </root>
        
        <!-- 业务日志 -->
        <logger name="com.example.service" level="INFO" />
        <logger name="com.example.controller" level="WARN" />
        
        <!-- 第三方库日志 -->
        <logger name="org.apache" level="WARN" />
        <logger name="com.zaxxer" level="WARN" />
    </springProfile>
    
    <!-- 特定包日志级别 -->
    <logger name="com.example.controller" level="INFO" />
    <logger name="com.example.service" level="DEBUG" />
    <logger name="com.example.mapper" level="DEBUG" />
    
    <!-- 第三方框架日志控制 -->
    <logger name="org.mybatis" level="INFO" />
    <logger name="org.hibernate" level="WARN" />
    <logger name="org.apache.http" level="WARN" />
    
    <!-- 关闭不需要的日志 -->
    <logger name="org.apache.ibatis.io" level="OFF" />
    <logger name="org.springframework.boot.autoconfigure" level="OFF" />
    
</configuration>
```

### 3.5 日志级别详解与使用场景

| 级别      | 数值 | 使用场景                       | 示例                                   |
| --------- | ---- | ------------------------------ | -------------------------------------- |
| **TRACE** | 0    | 最详细的调试信息，生产环境关闭 | `log.trace("方法入参: {}", param)`     |
| **DEBUG** | 10   | 开发调试信息，帮助排查问题     | `log.debug("SQL: {}", sql)`            |
| **INFO**  | 20   | 重要业务过程信息，生产环境保留 | `log.info("用户{}登录成功", username)` |
| **WARN**  | 30   | 警告信息，需要注意但不是错误   | `log.warn("缓存未命中，查询数据库")`   |
| **ERROR** | 40   | 错误信息，需要立即处理         | `log.error("数据库连接失败", e)`       |

**日志级别配置建议：**
- 开发环境：DEBUG
- 测试环境：INFO  
- 生产环境：WARN或INFO

### 3.6 日志记录最佳实践

✅ **应该做的：**
```java
// 1. 使用参数化日志，避免字符串拼接
log.info("用户 {} 登录成功，IP: {}", username, ip);  // ✅ 正确
log.info("用户 " + username + " 登录成功，IP: " + ip); // ❌ 避免

// 2. 异常日志要包含堆栈信息
try {
    // ...
} catch (Exception e) {
    log.error("处理失败，用户: {}", userId, e);  // ✅ 包含异常
    log.error("处理失败，用户: " + userId);      // ❌ 缺少异常信息
}

// 3. 敏感信息脱敏
log.info("用户登录，手机号: {}", maskPhoneNumber(phone)); // ✅ 脱敏
```

❌ **应该避免的：**
```java
// 1. 不要在生产环境使用System.out
System.out.println("用户登录");  // ❌ 避免

// 2. 不要记录敏感信息
log.info("密码: {}", password);  // ❌ 危险！

// 3. 避免在循环中记录大量日志
for (User user : users) {
    log.debug("处理用户: {}", user.getId());  // ❌ 可能产生大量日志
}

// 4. 不要使用字符串拼接判断日志级别
if (log.isDebugEnabled()) {  // ✅ 正确
    log.debug("大量数据: {}", expensiveOperation());
}
```

---

## 🏢 第四章：企业级整合方案

### 4.1 MyBatis + Logback整合配置

```yaml
# application.yml - 完整整合配置
spring:
  profiles:
    active: dev
  
  datasource:
    url: jdbc:mysql://localhost:3306/demo
    username: root
    password: 123456
  
mybatis:
  mapper-locations: classpath:mapper/**/*.xml
  type-aliases-package: com.example.entity
  configuration:
    map-underscore-to-camel-case: true
    log-impl: org.apache.ibatis.logging.slf4j.Slf4jImpl  # 关键配置
    
logging:
  config: classpath:logback-spring.xml  # 指定Logback配置文件
  level:
    com.example.mapper: debug  # MyBatis Mapper日志级别
```

### 4.2 多模块项目日志配置

```
项目结构：
my-project/
├── pom.xml
├── common/                    # 公共模块
├── user-service/             # 用户服务
│   ├── src/main/resources/
│   │   ├── logback-spring.xml
│   │   └── mapper/
│   └── application.yml
├── order-service/            # 订单服务
└── gateway/                  # 网关服务
```

**公共日志配置：**
```xml
<!-- common模块中的logback-base.xml -->
<included>
    <!-- 定义公共的Pattern -->
    <property name="COMMON_PATTERN" value="%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level [%X{traceId:-}] %logger{40} - %msg%n"/>
    
    <!-- 公共的Appender配置 -->
    <appender name="COMMON_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 基础配置 -->
    </appender>
</included>
```

### 4.3 日志分析与监控

```xml
<!-- JSON格式日志输出，便于ELK等系统收集 -->
<appender name="JSON_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <file>${LOG_HOME}/${APP_NAME}.json.log</file>
    <encoder class="net.logstash.logback.encoder.LogstashEncoder">
        <customFields>{"app":"${APP_NAME}","env":"${ENV}"}</customFields>
        <includeContext>false</includeContext>
        <timestampPattern>yyyy-MM-dd'T'HH:mm:ss.SSS</timestampPattern>
        <includeMdcKeyName>traceId</includeMdcKeyName>
        <includeMdcKeyName>userId</includeMdcKeyName>
    </encoder>
</appender>
```

---

## 🚀 第五章：高级特性与性能优化

### 5.1 MyBatis性能优化配置

```xml
<!-- mybatis-config.xml -->
<configuration>
    <!-- 二级缓存配置 -->
    <settings>
        <setting name="cacheEnabled" value="true"/>
        <setting name="lazyLoadingEnabled" value="true"/>
        <setting name="aggressiveLazyLoading" value="false"/>
        <setting name="localCacheScope" value="SESSION"/>
    </settings>
    
    <!-- 插件配置 -->
    <plugins>
        <!-- 分页插件 -->
        <plugin interceptor="com.github.pagehelper.PageInterceptor">
            <property name="reasonable" value="true"/>
            <property name="supportMethodsArguments" value="true"/>
        </plugin>
        
        <!-- SQL执行性能分析插件（开发环境） -->
        <plugin interceptor="com.github.sql-helper.sql-helper-starter"/>
    </plugins>
</configuration>
```

### 5.2 异步日志性能优化

```xml
<appender name="ASYNC_PERFORMANCE" class="ch.qos.logback.classic.AsyncAppender">
    <!-- 队列深度，默认256，根据应用调整 -->
    <queueSize>2048</queueSize>
    
    <!-- 当队列剩余容量低于此阈值时，丢弃级别较低的日志 -->
    <discardingThreshold>20</discardingThreshold>
    
    <!-- 设置是否从不丢弃日志 -->
    <neverBlock>true</neverBlock>
    
    <!-- 引用实际的appender -->
    <appender-ref ref="FILE_ALL"/>
</appender>
```

### 5.3 动态日志级别调整

```java
@RestController
@RequestMapping("/admin/log")
public class LogLevelController {
    
    @PostMapping("/level")
    public ResponseEntity<String> changeLogLevel(
            @RequestParam String loggerName,
            @RequestParam String level) {
        
        LoggerContext loggerContext = (LoggerContext) LoggerFactory.getILoggerFactory();
        Logger logger = loggerContext.getLogger(loggerName);
        
        if ("ROOT".equalsIgnoreCase(loggerName)) {
            logger = loggerContext.getLogger(Logger.ROOT_LOGGER_NAME);
        }
        
        logger.setLevel(Level.valueOf(level.toUpperCase()));
        
        return ResponseEntity.ok(
            String.format("日志级别已修改: %s -> %s", loggerName, level)
        );
    }
    
    @GetMapping("/levels")
    public Map<String, String> getLogLevels() {
        LoggerContext loggerContext = (LoggerContext) LoggerFactory.getILoggerFactory();
        Map<String, String> levels = new LinkedHashMap<>();
        
        for (ch.qos.logback.classic.Logger logger : loggerContext.getLoggerList()) {
            if (logger.getLevel() != null) {
                levels.put(logger.getName(), logger.getLevel().toString());
            }
        }
        
        return levels;
    }
}
```

---

## 📊 第六章：监控与告警

### 6.1 关键指标监控

```yaml
# 日志监控配置
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,loggers
  metrics:
    export:
      prometheus:
        enabled: true
  endpoint:
    loggers:
      enabled: true
```

### 6.2 日志告警规则示例

```java
@Component
public class LogMonitor {
    
    private final AtomicInteger errorCount = new AtomicInteger(0);
    private final AtomicLong lastAlertTime = new AtomicLong(0);
    
    @EventListener
    public void handleLogEvent(LoggingEvent event) {
        // 监听ERROR级别日志
        if (event.getLevel().levelInt >= Level.ERROR_INT) {
            int count = errorCount.incrementAndGet();
            long currentTime = System.currentTimeMillis();
            
            // 10分钟内错误超过100次，触发告警
            if (count > 100 && 
                currentTime - lastAlertTime.get() > 10 * 60 * 1000) {
                
                sendAlert("系统错误频发", 
                    String.format("10分钟内错误次数: %d", count));
                
                lastAlertTime.set(currentTime);
                errorCount.set(0);
            }
        }
    }
    
    // 定时重置计数器
    @Scheduled(fixedRate = 10 * 60 * 1000)
    public void resetCounters() {
        errorCount.set(0);
    }
}
```

---

## 🎯 总结与最佳实践

### 7.1 配置检查清单

✅ **MyBatis配置检查：**
- [ ] XML文件与Mapper接口同名同包
- [ ] namespace配置正确
- [ ] SQL id与方法名一致
- [ ] 开启了驼峰映射（如果需要）
- [ ] 配置了正确的日志实现

✅ **日志配置检查：**
- [ ] 不同环境有不同日志级别
- [ ] 生产环境关闭DEBUG日志
- [ ] 配置了日志轮转策略
- [ ] 错误日志单独文件存储
- [ ] 敏感信息已脱敏

### 7.2 性能优化建议

1. **MyBatis优化：**
   - 使用二级缓存减少数据库访问
   - 合理使用批量操作
   - 避免N+1查询问题

2. **日志优化：**
   - 生产环境使用异步日志
   - 合理设置日志级别
   - 定期清理历史日志

3. **监控建议：**
   - 关键业务日志添加traceId
   - 配置日志聚合系统（ELK）
   - 设置关键错误告警

### 7.3 故障排查指南

| 问题现象      | 可能原因         | 解决方案                        |
| ------------- | ---------------- | ------------------------------- |
| SQL执行无日志 | 日志级别配置过高 | 设置`com.example.mapper: DEBUG` |
| XML映射不生效 | 文件位置不正确   | 检查`mapper-locations`配置      |
| 日志文件过大  | 未配置轮转策略   | 添加`TimeBasedRollingPolicy`    |
| 性能下降      | 同步日志阻塞     | 切换为异步日志Appender          |

---

## 📚 延伸学习资源

1. **官方文档：**
   - [MyBatis官方文档](https://mybatis.org/mybatis-3/zh/index.html)
   - [Logback官方手册](http://logback.qos.ch/manual/)

2. **推荐工具：**
   - [MyBatis Generator](https://mybatis.org/generator/) - 代码生成器
   - [MyBatis PageHelper](https://pagehelper.github.io/) - 分页插件
   - [Logstash Logback Encoder](https://github.com/logstash/logstash-logback-encoder) - JSON日志编码

3. **监控方案：**
   - ELK Stack（Elasticsearch + Logstash + Kibana）
   - Grafana + Prometheus
   - SkyWalking分布式追踪

---

**🌟 核心要点总结：**
- MyBatis XML配置提供更好的SQL维护性
- 日志是系统可观测性的基础
- 合理的配置可以显著提升系统性能
- 监控告警是生产环境的必需品

希望这篇详细的指南能帮助你在实际项目中更好地配置和使用MyBatis与Logback。记住，好的配置是成功的一半！ 🚀
