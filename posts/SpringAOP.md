# Spring AOP 面向切面编程完全指南 🚀

## 一、什么是 AOP？ 🤔

**面向切面编程**（AOP）是 Spring 框架的核心功能之一，它允许开发者将横切关注点（如日志记录、事务管理、安全控制等）从业务逻辑中分离出来，实现代码的模块化和复用。就像电影特效团队🎬，他们专注于特效制作，而不需要关心剧本内容！

## 二、快速开始 ⚡

### 1. 导入依赖 📦

```xml
<!-- Spring Boot AOP 依赖 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

### 2. 第一个 AOP 程序：方法执行时间监控 ⏱️

```java
@Slf4j
@Aspect         // 声明为切面类 🎯
@Component      // 交给 IOC 容器管理 🏭
public class RecordTimeAspect {
    
    /**
     * 切入点表达式：监控 com.example.service 包下所有类的所有方法
     */
    @Around("execution(* com.example.service.*.*(..))")
    public Object recordTime(ProceedingJoinPoint pjp) throws Throwable {
        // 记录开始时间
        long beginTime = System.currentTimeMillis();
        
        // 执行原始方法 🏃‍♂️
        Object result = pjp.proceed();
        
        // 记录结束时间并计算耗时
        long endTime = System.currentTimeMillis();
        long costTime = endTime - beginTime;
        
        // 获取方法签名信息
        String methodName = pjp.getSignature().getName();
        String className = pjp.getTarget().getClass().getSimpleName();
        
        log.info("🎯 [{}] - [{}] 执行耗时: {}ms", className, methodName, costTime);
        
        return result;
    }
}
```

## 三、AOP 核心概念详解 🎓

### 1. 核心术语 📚

| 概念                   | 说明                           | 类比               | 表情 |
| ---------------------- | ------------------------------ | ------------------ | ---- |
| **连接点 (JoinPoint)** | 程序执行过程中可以插入切面的点 | 电影中的每个场景   | 🎬    |
| **通知 (Advice)**      | 切面在特定连接点执行的动作     | 特效团队的工作     | 🎨    |
| **切入点 (Pointcut)**  | 匹配连接点的谓词               | 需要加特效的场景   | 📍    |
| **切面 (Aspect)**      | 通知和切入点的组合             | 完整的特效设计方案 | 📋    |
| **目标对象 (Target)**  | 被一个或多个切面所通知的对象   | 原始电影片段       | 🎥    |

### ![image-20251221183926108](SSM框架搭建.assets/image-20251221183926108.png)

### 2. AOP 底层原理：动态代理 🧙‍♂️

Spring AOP 默认使用两种代理方式：
- **JDK 动态代理**：基于接口实现 ✨
- **CGLIB 代理**：基于继承实现 🔄

## 四、通知类型详解 🎪

### 1. 五种通知类型 🖐️

```java
@Aspect
@Component
public class AllAdviceExample {
    
    /**
     * 1. 环绕通知 - 最强大的通知类型 💪
     */
    @Around("execution(* com.example.service.*.*(..))")
    public Object aroundAdvice(ProceedingJoinPoint pjp) throws Throwable {
        log.info("🔄 @Around - 方法执行前");
        try {
            Object result = pjp.proceed();  // 必须显式调用原始方法
            log.info("✅ @Around - 方法执行后");
            return result;
        } catch (Exception e) {
            log.error("❌ @Around - 方法执行异常");
            throw e;
        }
    }
    
    /**
     * 2. 前置通知 - 在目标方法执行前执行 ⬆️
     */
    @Before("execution(* com.example.service.*.*(..))")
    public void beforeAdvice(JoinPoint joinPoint) {
        log.info("⬆️ @Before - 方法执行前");
    }
    
    /**
     * 3. 后置通知 - 在目标方法执行后执行（无论是否异常） ⬇️
     */
    @After("execution(* com.example.service.*.*(..))")
    public void afterAdvice(JoinPoint joinPoint) {
        log.info("⬇️ @After - 方法执行后（总执行）");
    }
    
    /**
     * 4. 返回后通知 - 在目标方法正常返回后执行 ✅
     */
    @AfterReturning(value = "execution(* com.example.service.*.*(..))", 
                   returning = "result")
    public void afterReturningAdvice(JoinPoint joinPoint, Object result) {
        log.info("✅ @AfterReturning - 方法正常返回，结果: {}", result);
    }
    
    /**
     * 5. 异常通知 - 在目标方法抛出异常后执行 ❌
     */
    @AfterThrowing(value = "execution(* com.example.service.*.*(..))", 
                   throwing = "ex")
    public void afterThrowingAdvice(JoinPoint joinPoint, Exception ex) {
        log.error("❌ @AfterThrowing - 方法执行异常: {}", ex.getMessage());
    }
}
```

### 2. 通知执行顺序 📊

默认执行顺序（同一切面内）：  
**🔄 @Around**（前半部分） → **⬆️ @Before** → **🎯 目标方法** → **🔄 @Around**（后半部分） → **✅ @AfterReturning/❌ @AfterThrowing** → **⬇️ @After**

## 五、切入点表达式 ✨

### 1. execution 表达式 🎯

```java
@Aspect
@Component
public class PointcutExamples {
    
    // 1. 抽取公共切入点表达式 📝
    @Pointcut("execution(* com.example.service.*.*(..))")
    public void serviceLayer() {}
    
    // 2. 精确匹配方法 🎯
    @Pointcut("execution(public String com.example.service.UserService.getUserById(Integer))")
    public void specificMethod() {}
    
    // 3. 匹配包下所有方法 📁
    @Pointcut("execution(* com.example.service..*.*(..))")  // ..表示子包
    public void allServiceMethods() {}
    
    // 4. 匹配特定注解的方法 🏷️
    @Pointcut("@annotation(com.example.annotation.Log)")
    public void logAnnotation() {}
    
    // 5. 组合切入点表达式 🔗
    @Pointcut("serviceLayer() && !specificMethod()")
    public void serviceButNotSpecific() {}
    
    @Around("serviceLayer()")  // 引用切入点表达式
    public Object adviceMethod(ProceedingJoinPoint pjp) throws Throwable {
        // 业务逻辑
        return pjp.proceed();
    }
}
```

### 2. @annotation 表达式（基于注解） 🏷️

```java
/**
 * 自定义日志注解
 */
@Target(ElementType.METHOD)    // 只能用在方法上
@Retention(RetentionPolicy.RUNTIME)  // 运行时生效
public @interface Log {
    String value() default "";
    boolean recordParams() default true;
    boolean recordResult() default true;
}
```

## 六、实战：操作日志记录到数据库 💾

### 1. 创建操作日志实体 📋

```java
@Data
@TableName("t_operate_log")  // MyBatis-Plus 注解
public class OperateLog {
    
    @TableId(type = IdType.AUTO)
    private Long id;
    
    private Long operateUserId;        // 操作人ID 👤
    private String operateUserName;    // 操作人姓名
    
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    private LocalDateTime operateTime; // 操作时间 ⏰
    
    private String className;          // 类名
    private String methodName;         // 方法名
    
    @TableField(typeHandler = JacksonTypeHandler.class)
    private Object methodParams;       // 方法参数（JSON格式） 📄
    
    @TableField(typeHandler = JacksonTypeHandler.class)
    private Object returnValue;        // 返回值（JSON格式） 📋
    
    private Boolean success;           // 操作是否成功 ✅
    private String errorMessage;       // 错误信息 ❌
    private Long costTime;             // 耗时（毫秒） ⏱️
    private String clientIp;           // 客户端IP 🌐
    private String requestUri;         // 请求URI 🔗
}
```

### 2. Mapper 接口 🔧

```java
@Mapper
public interface OperateLogMapper extends BaseMapper<OperateLog> {
    // 这里可以添加自定义查询方法
}
```

### 3. 用户上下文工具类（ThreadLocal） 🧵

```java
@Component
public class UserContext {
    
    private static final ThreadLocal<CurrentUser> USER_CONTEXT = new ThreadLocal<>();
    
    /**
     * 设置当前用户信息 👤
     */
    public static void setCurrentUser(CurrentUser user) {
        USER_CONTEXT.set(user);
    }
    
    /**
     * 获取当前用户ID 🔑
     */
    public static Long getCurrentUserId() {
        CurrentUser user = USER_CONTEXT.get();
        return user != null ? user.getId() : null;
    }
    
    /**
     * 获取当前用户信息 📋
     */
    public static CurrentUser getCurrentUser() {
        return USER_CONTEXT.get();
    }
    
    /**
     * 清除用户信息（防止内存泄漏） 🧹
     */
    public static void clear() {
        USER_CONTEXT.remove();
    }
    
    @Data
    public static class CurrentUser {
        private Long id;
        private String username;
        private String name;
        private String role;
    }
}
```

### 4. 切面类实现 🛡️

```java
@Slf4j
@Aspect
@Component
public class OperateLogAspect {
    
    @Autowired
    private OperateLogMapper operateLogMapper;
    
    @Autowired
    private HttpServletRequest request;
    
    /**
     * 基于注解的切入点 🎯
     */
    @Around("@annotation(logAnnotation)")
    public Object logOperate(ProceedingJoinPoint pjp, Log logAnnotation) throws Throwable {
        // 记录开始时间 ⏰
        long startTime = System.currentTimeMillis();
        
        // 创建日志对象 📝
        OperateLog operateLog = new OperateLog();
        operateLog.setOperateTime(LocalDateTime.now());
        operateLog.setSuccess(true);
        
        // 获取方法信息 🔍
        MethodSignature signature = (MethodSignature) pjp.getSignature();
        Method method = signature.getMethod();
        
        // 设置基本信息
        operateLog.setClassName(pjp.getTarget().getClass().getName());
        operateLog.setMethodName(method.getName());
        
        // 获取当前用户 👤
        Long currentUserId = UserContext.getCurrentUserId();
        operateLog.setOperateUserId(currentUserId);
        
        // 获取请求信息 🌐
        if (request != null) {
            operateLog.setClientIp(getClientIp(request));
            operateLog.setRequestUri(request.getRequestURI());
        }
        
        // 记录方法参数（根据配置） 📄
        if (logAnnotation.recordParams()) {
            Object[] args = pjp.getArgs();
            String[] paramNames = signature.getParameterNames();
            Map<String, Object> params = new HashMap<>();
            
            for (int i = 0; i < args.length; i++) {
                // 过滤掉敏感参数 🚫
                if (!isSensitiveParam(paramNames[i])) {
                    params.put(paramNames[i], args[i]);
                }
            }
            operateLog.setMethodParams(params);
        }
        
        Object result = null;
        try {
            // 执行目标方法 🏃‍♂️
            result = pjp.proceed();
            
            // 记录返回值（根据配置） ✅
            if (logAnnotation.recordResult()) {
                operateLog.setReturnValue(result);
            }
            
            // 计算耗时 ⏱️
            long endTime = System.currentTimeMillis();
            operateLog.setCostTime(endTime - startTime);
            
            log.info("✅ 操作日志记录成功: {}.{}", 
                    operateLog.getClassName(), 
                    operateLog.getMethodName());
            
        } catch (Throwable throwable) {
            // 记录异常信息 ❌
            operateLog.setSuccess(false);
            operateLog.setErrorMessage(throwable.getMessage());
            operateLog.setCostTime(System.currentTimeMillis() - startTime);
            
            log.error("❌ 操作执行失败: {}", throwable.getMessage());
            throw throwable;
            
        } finally {
            // 异步保存日志到数据库 💾
            saveLogAsync(operateLog);
        }
        
        return result;
    }
    
    /**
     * 异步保存日志 🚀
     */
    private void saveLogAsync(OperateLog operateLog) {
        CompletableFuture.runAsync(() -> {
            try {
                operateLogMapper.insert(operateLog);
                log.debug("💾 操作日志已保存到数据库");
            } catch (Exception e) {
                log.error("❌ 保存操作日志失败: {}", e.getMessage());
            }
        });
    }
    
    /**
     * 获取客户端IP地址 🌐
     */
    private String getClientIp(HttpServletRequest request) {
        String ip = request.getHeader("X-Forwarded-For");
        if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
            ip = request.getHeader("Proxy-Client-IP");
        }
        if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
            ip = request.getHeader("WL-Proxy-Client-IP");
        }
        if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
            ip = request.getRemoteAddr();
        }
        return ip;
    }
    
    /**
     * 判断是否为敏感参数 🚫
     */
    private boolean isSensitiveParam(String paramName) {
        List<String> sensitiveParams = Arrays.asList("password", "token", "secret");
        return sensitiveParams.stream()
                .anyMatch(paramName::contains);
    }
}
```

## 七、最佳实践与注意事项 ⚠️

### 1. 性能优化建议 🚀
- **合理使用切入点表达式**：避免过于宽泛的匹配
- **异步处理耗时操作**：如数据库日志保存
- **缓存频繁访问的数据**：如方法签名信息

### 2. 常见陷阱 🕳️
- **自调用问题**：同一个类中方法互相调用，AOP不会生效
- **异常处理**：确保异常被正确处理和传播
- **内存泄漏**：ThreadLocal使用后必须清理

### 3. 调试技巧 🔧
```java
// 在通知中打印详细信息
@Before("execution(* com.example..*.*(..))")
public void debugAdvice(JoinPoint joinPoint) {
    log.debug("🎯 目标类: {}", joinPoint.getTarget().getClass().getName());
    log.debug("🔧 方法名: {}", joinPoint.getSignature().getName());
    log.debug("📋 参数: {}", Arrays.toString(joinPoint.getArgs()));
}
```

## 八、总结 🎉

Spring AOP 是一个强大的面向切面编程框架，它通过动态代理机制实现了横切关注点的模块化。掌握 AOP 可以让你的代码更加：

- ✅ **干净整洁** - 分离关注点
- ✅ **易于维护** - 集中化管理
- ✅ **高度复用** - 一处定义，多处使用
- ✅ **灵活扩展** - 非侵入式增强

希望这篇指南能帮助你更好地理解和使用 Spring AOP！Happy coding! 😊👨‍💻👩‍💻

---

**小提示**：在实际项目中，建议将AOP配置放在独立的配置类中，便于管理和维护。记得根据具体业务场景选择合适的通知类型和切入点表达式哦！✨
