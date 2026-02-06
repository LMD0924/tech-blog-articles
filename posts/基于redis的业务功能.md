# 基于 Redis 和 JWT 实现短信验证登录系统

## 一、**整体架构设计**

```
用户 → 发送手机号 → 生成验证码 → Redis存储 → 短信发送
        ↓
用户输入验证码 → 验证Redis → 生成JWT令牌 → 返回给客户端
```

---

## 二、**依赖配置**

### **1. pom.xml 依赖**

```xml
<!-- Spring Boot Starter -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<!-- Redis -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>

<!-- JWT -->
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.11.5</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.11.5</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>0.11.5</version>
    <scope>runtime</scope>
</dependency>

<!-- Lombok -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>

<!-- 验证码生成 -->
<dependency>
    <groupId>com.github.penggle</groupId>
    <artifactId>kaptcha</artifactId>
    <version>2.3.2</version>
</dependency>
```

### **2. application.yml 配置**

```yaml
spring:
  redis:
    host: localhost
    port: 6379
    password: 
    database: 0
    timeout: 3000ms
    lettuce:
      pool:
        max-active: 8
        max-idle: 8
        min-idle: 0

# JWT配置
jwt:
  secret: "your-secret-key-here-min-256-bit"  # 至少32位
  expiration: 86400000  # 24小时（毫秒）
  header: Authorization
  
# 短信验证码配置
sms:
  code:
    length: 6           # 验证码长度
    expire-minutes: 5   # 过期时间（分钟）
    prefix: "sms:code:" # Redis key前缀
    resend-interval: 60 # 重发间隔（秒）
```

---

## 三、**核心组件实现**

### **1. Redis 配置类**

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.StringRedisSerializer;

@Configuration
public class RedisConfig {
    
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(factory);
        
        // 使用String序列化器
        StringRedisSerializer stringSerializer = new StringRedisSerializer();
        template.setKeySerializer(stringSerializer);
        template.setValueSerializer(stringSerializer);
        template.setHashKeySerializer(stringSerializer);
        template.setHashValueSerializer(stringSerializer);
        
        template.afterPropertiesSet();
        return template;
    }
}
```

### **2. JWT 工具类**

```java
import io.jsonwebtoken.*;
import io.jsonwebtoken.security.Keys;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;
import javax.crypto.SecretKey;
import java.util.Date;
import java.util.Map;
import java.util.HashMap;

@Component
public class JwtTokenUtil {
    
    @Value("${jwt.secret}")
    private String secret;
    
    @Value("${jwt.expiration}")
    private Long expiration;
    
    @Value("${jwt.header}")
    private String header;
    
    // 生成密钥
    private SecretKey getSigningKey() {
        return Keys.hmacShaKeyFor(secret.getBytes());
    }
    
    /**
     * 生成JWT令牌
     */
    public String generateToken(String phone, Map<String, Object> claims) {
        if (claims == null) {
            claims = new HashMap<>();
        }
        
        return Jwts.builder()
            .setClaims(claims)
            .setSubject(phone)  // 手机号作为主题
            .setIssuedAt(new Date())  // 签发时间
            .setExpiration(new Date(System.currentTimeMillis() + expiration))  // 过期时间
            .signWith(getSigningKey(), SignatureAlgorithm.HS256)  // 签名算法
            .compact();
    }
    
    /**
     * 生成JWT令牌（简化版）
     */
    public String generateToken(String phone) {
        return generateToken(phone, null);
    }
    
    /**
     * 从令牌中获取手机号
     */
    public String getPhoneFromToken(String token) {
        Claims claims = getClaimsFromToken(token);
        return claims.getSubject();
    }
    
    /**
     * 获取令牌中的声明
     */
    public Claims getClaimsFromToken(String token) {
        return Jwts.parserBuilder()
            .setSigningKey(getSigningKey())
            .build()
            .parseClaimsJws(token)
            .getBody();
    }
    
    /**
     * 验证令牌是否有效
     */
    public boolean validateToken(String token, String phone) {
        String tokenPhone = getPhoneFromToken(token);
        return (tokenPhone.equals(phone) && !isTokenExpired(token));
    }
    
    /**
     * 判断令牌是否过期
     */
    public boolean isTokenExpired(String token) {
        Date expiration = getExpirationDateFromToken(token);
        return expiration.before(new Date());
    }
    
    /**
     * 获取令牌过期时间
     */
    public Date getExpirationDateFromToken(String token) {
        Claims claims = getClaimsFromToken(token);
        return claims.getExpiration();
    }
    
    /**
     * 获取请求头名称
     */
    public String getHeader() {
        return header;
    }
}
```

### **3. 短信验证码服务**

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.stereotype.Service;
import java.util.Random;
import java.util.concurrent.TimeUnit;

@Service
public class SmsCodeService {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    @Value("${sms.code.length}")
    private int codeLength;
    
    @Value("${sms.code.expire-minutes}")
    private int expireMinutes;
    
    @Value("${sms.code.prefix}")
    private String redisPrefix;
    
    @Value("${sms.code.resend-interval}")
    private int resendInterval;
    
    /**
     * 生成短信验证码
     */
    public String generateCode() {
        Random random = new Random();
        StringBuilder code = new StringBuilder();
        for (int i = 0; i < codeLength; i++) {
            code.append(random.nextInt(10));  // 0-9随机数字
        }
        return code.toString();
    }
    
    /**
     * 发送短信验证码
     */
    public boolean sendSmsCode(String phone) {
        // 1. 检查是否可以重发
        String resendKey = redisPrefix + "resend:" + phone;
        if (Boolean.TRUE.equals(redisTemplate.hasKey(resendKey))) {
            return false;  // 还在重发间隔期内
        }
        
        // 2. 生成验证码
        String code = generateCode();
        
        // 3. 存储到Redis（验证码）
        String codeKey = redisPrefix + phone;
        redisTemplate.opsForValue().set(codeKey, code, expireMinutes, TimeUnit.MINUTES);
        
        // 4. 设置重发限制
        redisTemplate.opsForValue().set(resendKey, "1", resendInterval, TimeUnit.SECONDS);
        
        // 5. 实际发送短信（这里模拟发送）
        System.out.println("发送短信给 " + phone + "，验证码：" + code);
        
        // TODO: 集成实际短信服务商（阿里云、腾讯云等）
        // smsClient.send(phone, code);
        
        return true;
    }
    
    /**
     * 验证短信验证码
     */
    public boolean verifyCode(String phone, String code) {
        String key = redisPrefix + phone;
        String storedCode = (String) redisTemplate.opsForValue().get(key);
        
        if (storedCode == null) {
            return false;  // 验证码不存在或已过期
        }
        
        boolean isValid = storedCode.equals(code);
        
        // 验证成功后删除验证码（防止重复使用）
        if (isValid) {
            redisTemplate.delete(key);
        }
        
        return isValid;
    }
    
    /**
     * 获取剩余过期时间
     */
    public Long getExpireTime(String phone) {
        String key = redisPrefix + phone;
        return redisTemplate.getExpire(key, TimeUnit.SECONDS);
    }
    
    /**
     * 检查是否可以重发
     */
    public boolean canResend(String phone) {
        String resendKey = redisPrefix + "resend:" + phone;
        return !Boolean.TRUE.equals(redisTemplate.hasKey(resendKey));
    }
    
    /**
     * 获取重发等待时间
     */
    public Long getResendWaitTime(String phone) {
        String resendKey = redisPrefix + "resend:" + phone;
        return redisTemplate.getExpire(resendKey, TimeUnit.SECONDS);
    }
}
```

### **4. 用户服务（模拟）**

```java
import org.springframework.stereotype.Service;
import java.util.concurrent.ConcurrentHashMap;

@Service
public class UserService {
    
    // 模拟用户存储（实际应该用数据库）
    private ConcurrentHashMap<String, User> userStore = new ConcurrentHashMap<>();
    
    /**
     * 注册或登录用户
     */
    public User registerOrLogin(String phone) {
        User user = userStore.get(phone);
        if (user == null) {
            // 新用户，自动注册
            user = new User();
            user.setId(System.currentTimeMillis());
            user.setPhone(phone);
            user.setUsername("用户_" + phone.substring(7));  // 取后4位
            user.setCreateTime(System.currentTimeMillis());
            
            userStore.put(phone, user);
        }
        return user;
    }
    
    /**
     * 获取用户信息
     */
    public User getUserByPhone(String phone) {
        return userStore.get(phone);
    }
    
    /**
     * 更新用户信息
     */
    public void updateUser(User user) {
        userStore.put(user.getPhone(), user);
    }
}

@Data
class User {
    private Long id;
    private String phone;
    private String username;
    private Long createTime;
    private Long lastLoginTime;
}
```

---

## 四、**控制器实现**

### **1. 短信验证码控制器**

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;
import java.util.HashMap;
import java.util.Map;

@RestController
@RequestMapping("/api/sms")
public class SmsController {
    
    @Autowired
    private SmsCodeService smsCodeService;
    
    /**
     * 发送短信验证码
     */
    @PostMapping("/send")
    public ApiResponse sendCode(@RequestParam String phone) {
        // 1. 验证手机号格式
        if (!isValidPhone(phone)) {
            return ApiResponse.error("手机号格式不正确");
        }
        
        // 2. 检查是否可以重发
        if (!smsCodeService.canResend(phone)) {
            Long waitTime = smsCodeService.getResendWaitTime(phone);
            return ApiResponse.error(waitTime + "秒后可重发");
        }
        
        // 3. 发送验证码
        boolean success = smsCodeService.sendSmsCode(phone);
        
        if (success) {
            Map<String, Object> data = new HashMap<>();
            data.put("phone", phone);
            data.put("expire", smsCodeService.getExpireTime(phone));
            return ApiResponse.success("验证码发送成功", data);
        } else {
            return ApiResponse.error("验证码发送失败");
        }
    }
    
    /**
     * 验证短信验证码
     */
    @PostMapping("/verify")
    public ApiResponse verifyCode(@RequestParam String phone, 
                                  @RequestParam String code) {
        boolean isValid = smsCodeService.verifyCode(phone, code);
        
        if (isValid) {
            return ApiResponse.success("验证码正确");
        } else {
            return ApiResponse.error("验证码错误或已过期");
        }
    }
    
    /**
     * 获取验证码状态
     */
    @GetMapping("/status/{phone}")
    public ApiResponse getCodeStatus(@PathVariable String phone) {
        Long expireTime = smsCodeService.getExpireTime(phone);
        boolean canResend = smsCodeService.canResend(phone);
        
        Map<String, Object> data = new HashMap<>();
        data.put("hasCode", expireTime != null && expireTime > 0);
        data.put("expireTime", expireTime);
        data.put("canResend", canResend);
        
        if (!canResend) {
            data.put("waitTime", smsCodeService.getResendWaitTime(phone));
        }
        
        return ApiResponse.success(data);
    }
    
    // 简单的手机号验证（实际应该更严谨）
    private boolean isValidPhone(String phone) {
        return phone != null && phone.matches("^1[3-9]\\d{9}$");
    }
}
```

### **2. 认证控制器**

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;
import java.util.HashMap;
import java.util.Map;

@RestController
@RequestMapping("/api/auth")
public class AuthController {
    
    @Autowired
    private SmsCodeService smsCodeService;
    
    @Autowired
    private JwtTokenUtil jwtTokenUtil;
    
    @Autowired
    private UserService userService;
    
    /**
     * 短信登录/注册
     */
    @PostMapping("/login-by-sms")
    public ApiResponse loginBySms(@RequestParam String phone, 
                                  @RequestParam String code) {
        // 1. 验证手机号
        if (!isValidPhone(phone)) {
            return ApiResponse.error("手机号格式不正确");
        }
        
        // 2. 验证验证码
        boolean codeValid = smsCodeService.verifyCode(phone, code);
        if (!codeValid) {
            return ApiResponse.error("验证码错误或已过期");
        }
        
        // 3. 获取或创建用户
        User user = userService.registerOrLogin(phone);
        user.setLastLoginTime(System.currentTimeMillis());
        userService.updateUser(user);
        
        // 4. 生成JWT令牌
        Map<String, Object> claims = new HashMap<>();
        claims.put("userId", user.getId());
        claims.put("username", user.getUsername());
        
        String token = jwtTokenUtil.generateToken(phone, claims);
        
        // 5. 返回结果
        Map<String, Object> data = new HashMap<>();
        data.put("token", token);
        data.put("tokenType", "Bearer");
        data.put("expiresIn", jwtTokenUtil.getExpiration());
        data.put("user", user);
        
        return ApiResponse.success("登录成功", data);
    }
    
    /**
     * 获取当前用户信息
     */
    @GetMapping("/me")
    public ApiResponse getCurrentUser(@RequestHeader("Authorization") String authHeader) {
        try {
            // 从Header中提取token
            String token = extractTokenFromHeader(authHeader);
            
            // 解析token获取手机号
            String phone = jwtTokenUtil.getPhoneFromToken(token);
            
            // 获取用户信息
            User user = userService.getUserByPhone(phone);
            if (user == null) {
                return ApiResponse.error("用户不存在");
            }
            
            return ApiResponse.success(user);
        } catch (Exception e) {
            return ApiResponse.error("令牌无效或已过期");
        }
    }
    
    /**
     * 刷新令牌
     */
    @PostMapping("/refresh-token")
    public ApiResponse refreshToken(@RequestHeader("Authorization") String authHeader) {
        try {
            String oldToken = extractTokenFromHeader(authHeader);
            String phone = jwtTokenUtil.getPhoneFromToken(oldToken);
            
            // 检查用户是否存在
            User user = userService.getUserByPhone(phone);
            if (user == null) {
                return ApiResponse.error("用户不存在");
            }
            
            // 生成新令牌
            Map<String, Object> claims = new HashMap<>();
            claims.put("userId", user.getId());
            claims.put("username", user.getUsername());
            
            String newToken = jwtTokenUtil.generateToken(phone, claims);
            
            Map<String, Object> data = new HashMap<>();
            data.put("token", newToken);
            data.put("tokenType", "Bearer");
            data.put("expiresIn", jwtTokenUtil.getExpiration());
            
            return ApiResponse.success("令牌刷新成功", data);
        } catch (Exception e) {
            return ApiResponse.error("令牌刷新失败");
        }
    }
    
    /**
     * 登出
     */
    @PostMapping("/logout")
    public ApiResponse logout(@RequestHeader("Authorization") String authHeader) {
        // TODO: 可以将令牌加入黑名单（存储在Redis中）
        // 这里简单实现，客户端删除本地token即可
        return ApiResponse.success("登出成功");
    }
    
    private String extractTokenFromHeader(String authHeader) {
        if (authHeader != null && authHeader.startsWith("Bearer ")) {
            return authHeader.substring(7);
        }
        throw new RuntimeException("无效的Authorization头");
    }
    
    private boolean isValidPhone(String phone) {
        return phone != null && phone.matches("^1[3-9]\\d{9}$");
    }
}
```

### **3. 全局响应封装**

```java
import lombok.Data;

@Data
public class ApiResponse<T> {
    private int code;
    private String message;
    private T data;
    private long timestamp;
    
    private ApiResponse(int code, String message, T data) {
        this.code = code;
        this.message = message;
        this.data = data;
        this.timestamp = System.currentTimeMillis();
    }
    
    public static <T> ApiResponse<T> success(T data) {
        return new ApiResponse<>(200, "success", data);
    }
    
    public static <T> ApiResponse<T> success(String message, T data) {
        return new ApiResponse<>(200, message, data);
    }
    
    public static <T> ApiResponse<T> success(String message) {
        return new ApiResponse<>(200, message, null);
    }
    
    public static <T> ApiResponse<T> error(String message) {
        return new ApiResponse<>(400, message, null);
    }
    
    public static <T> ApiResponse<T> error(int code, String message) {
        return new ApiResponse<>(code, message, null);
    }
    
    public static <T> ApiResponse<T> error(int code, String message, T data) {
        return new ApiResponse<>(code, message, data);
    }
}
```

---

## 五、**安全拦截器**

### **1. JWT认证拦截器**

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import org.springframework.web.servlet.HandlerInterceptor;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@Component
public class JwtAuthenticationInterceptor implements HandlerInterceptor {
    
    @Autowired
    private JwtTokenUtil jwtTokenUtil;
    
    // 不需要拦截的路径
    private static final String[] WHITE_LIST = {
        "/api/sms/send",
        "/api/sms/verify",
        "/api/auth/login-by-sms",
        "/swagger-ui.html",
        "/swagger-resources",
        "/v2/api-docs",
        "/webjars"
    };
    
    @Override
    public boolean preHandle(HttpServletRequest request, 
                           HttpServletResponse response, 
                           Object handler) throws Exception {
        
        // 检查是否在白名单中
        String requestURI = request.getRequestURI();
        for (String path : WHITE_LIST) {
            if (requestURI.startsWith(path)) {
                return true;
            }
        }
        
        // 从请求头获取token
        String authHeader = request.getHeader(jwtTokenUtil.getHeader());
        
        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
            response.getWriter().write("{\"code\":401,\"message\":\"未提供有效的令牌\"}");
            return false;
        }
        
        String token = authHeader.substring(7);
        
        try {
            // 验证令牌
            if (jwtTokenUtil.isTokenExpired(token)) {
                response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
                response.getWriter().write("{\"code\":401,\"message\":\"令牌已过期\"}");
                return false;
            }
            
            // 将手机号设置到请求属性中，供后续使用
            String phone = jwtTokenUtil.getPhoneFromToken(token);
            request.setAttribute("currentPhone", phone);
            
        } catch (Exception e) {
            response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
            response.getWriter().write("{\"code\":401,\"message\":\"令牌无效\"}");
            return false;
        }
        
        return true;
    }
}
```

### **2. 拦截器配置**

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class WebConfig implements WebMvcConfigurer {
    
    @Autowired
    private JwtAuthenticationInterceptor jwtAuthenticationInterceptor;
    
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(jwtAuthenticationInterceptor)
            .addPathPatterns("/api/**")
            .excludePathPatterns("/api/sms/**")
            .excludePathPatterns("/api/auth/login-by-sms");
    }
}
```

---

## 六、**短信服务集成（示例）**

### **阿里云短信服务集成**

```java
import com.aliyun.dysmsapi20170525.Client;
import com.aliyun.dysmsapi20170525.models.SendSmsRequest;
import com.aliyun.dysmsapi20170525.models.SendSmsResponse;
import com.aliyun.teaopenapi.models.Config;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;

@Service
public class AliyunSmsService {
    
    @Value("${aliyun.sms.accessKeyId}")
    private String accessKeyId;
    
    @Value("${aliyun.sms.accessKeySecret}")
    private String accessKeySecret;
    
    @Value("${aliyun.sms.signName}")
    private String signName;
    
    @Value("${aliyun.sms.templateCode}")
    private String templateCode;
    
    /**
     * 发送短信
     */
    public boolean sendSms(String phone, String code) {
        try {
            Client client = createClient();
            
            SendSmsRequest request = new SendSmsRequest()
                .setPhoneNumbers(phone)
                .setSignName(signName)
                .setTemplateCode(templateCode)
                .setTemplateParam("{\"code\":\"" + code + "\"}");
            
            SendSmsResponse response = client.sendSms(request);
            return "OK".equals(response.getBody().getCode());
            
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }
    
    private Client createClient() throws Exception {
        Config config = new Config()
            .setAccessKeyId(accessKeyId)
            .setAccessKeySecret(accessKeySecret);
        config.endpoint = "dysmsapi.aliyuncs.com";
        
        return new Client(config);
    }
}
```

---

## 七、**完整业务流程图**

### **1. 发送验证码流程**

```
1. 用户输入手机号 → 前端调用 /api/sms/send
2. 服务端验证手机号格式
3. 检查Redis重发限制
4. 生成6位随机验证码
5. 存储到Redis（5分钟过期）
6. 设置重发限制（60秒）
7. 调用短信服务发送
8. 返回发送结果
```

### **2. 登录流程**

```
1. 用户输入手机号+验证码 → 前端调用 /api/auth/login-by-sms
2. 验证手机号格式
3. 验证Redis中的验证码
4. 验证成功后删除Redis验证码
5. 查询/创建用户
6. 生成JWT令牌
7. 返回令牌和用户信息
```

---

## 八、**测试接口**

### **使用 curl 测试**

```bash
# 1. 发送验证码
curl -X POST "http://localhost:8080/api/sms/send?phone=13800138000"

# 2. 验证验证码
curl -X POST "http://localhost:8080/api/sms/verify" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "phone=13800138000&code=123456"

# 3. 登录（获取JWT令牌）
curl -X POST "http://localhost:8080/api/auth/login-by-sms" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "phone=13800138000&code=123456"

# 4. 访问需要认证的接口
curl -X GET "http://localhost:8080/api/auth/me" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN_HERE"

# 5. 刷新令牌
curl -X POST "http://localhost:8080/api/auth/refresh-token" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN_HERE"
```

### **使用 Postman 测试**

1. **发送验证码**：POST `/api/sms/send?phone=13800138000`
2. **登录**：POST `/api/auth/login-by-sms` (Body: `phone=13800138000&code=123456`)
3. **访问受保护接口**：添加Header `Authorization: Bearer {token}`

---

## 九、**安全增强建议**

### **1. 防止暴力破解**

```java
@Service
public class SecurityService {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    /**
     * 检查验证码错误次数
     */
    public boolean checkCodeErrorCount(String phone) {
        String key = "sms:error:" + phone;
        String countStr = (String) redisTemplate.opsForValue().get(key);
        int count = countStr == null ? 0 : Integer.parseInt(countStr);
        
        if (count >= 5) {  // 最多错误5次
            return false;
        }
        
        return true;
    }
    
    /**
     * 增加错误计数
     */
    public void incrementErrorCount(String phone) {
        String key = "sms:error:" + phone;
        Long count = redisTemplate.opsForValue().increment(key);
        
        // 24小时后过期
        if (count != null && count == 1) {
            redisTemplate.expire(key, 24, TimeUnit.HOURS);
        }
    }
}
```

### **2. IP限制**

```java
@Service  
public class IpLimitService {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    /**
     * 检查IP发送频率
     */
    public boolean checkIpLimit(String ip) {
        String key = "sms:ip:" + ip;
        String countStr = (String) redisTemplate.opsForValue().get(key);
        int count = countStr == null ? 0 : Integer.parseInt(countStr);
        
        // 1小时内最多发10条
        if (count >= 10) {
            return false;
        }
        
        redisTemplate.opsForValue().increment(key);
        if (count == 0) {
            redisTemplate.expire(key, 1, TimeUnit.HOURS);
        }
        
        return true;
    }
}
```

### **3. 设备指纹**

```java
public class DeviceFingerprintUtil {
    
    /**
     * 生成设备指纹
     */
    public static String generateFingerprint(HttpServletRequest request) {
        String userAgent = request.getHeader("User-Agent");
        String ip = request.getRemoteAddr();
        String acceptLanguage = request.getHeader("Accept-Language");
        
        // 简单实现，生产环境应该更复杂
        return DigestUtils.md5DigestAsHex(
            (userAgent + ip + acceptLanguage).getBytes()
        );
    }
}
```

---

## 十、**监控和日志**

### **1. 发送日志记录**

```java
@Service
public class SmsLogService {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    /**
     * 记录短信发送日志
     */
    public void logSmsSend(String phone, String code, boolean success, String ip) {
        String key = "sms:log:" + System.currentTimeMillis();
        
        Map<String, Object> log = new HashMap<>();
        log.put("phone", phone);
        log.put("code", code);
        log.put("success", success);
        log.put("ip", ip);
        log.put("timestamp", System.currentTimeMillis());
        
        redisTemplate.opsForHash().putAll(key, log);
        redisTemplate.expire(key, 30, TimeUnit.DAYS);  // 保留30天
    }
}
```

---

## **总结**

### **核心优势：**

1. **安全性**：验证码存储于Redis，过期自动删除
2. **防刷**：重发间隔、IP限制、错误次数限制
3. **无状态认证**：JWT令牌，服务端无需存储会话
4. **高性能**：Redis内存操作，响应快速
5. **可扩展**：易于集成多种短信服务商

### **生产环境建议：**

1. **增加验证码图形验证**：防止机器恶意请求
2. **增加语音验证码**：作为短信的备用方案
3. **使用HTTPS**：防止中间人攻击
4. **定期更换JWT密钥**：增加安全性
5. **监控异常请求**：及时发现攻击行为
6. **多级缓存**：热点数据增加本地缓存

### **可优化的点：**

1. **验证码复杂度**：可配置数字+字母
2. **发送策略**：根据时间段调整发送频率
3. **黑白名单**：特定手机号/IP的特殊处理
4. **异步发送**：使用消息队列提高吞吐量
5. **多通道发送**：短信+邮件+APP推送

# 优惠券秒杀

## 1. 核心表结构设计

### 1.1 优惠券基础表 `coupon_base`

sql

```sql
CREATE TABLE `coupon_base` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '优惠券ID',
  `code` varchar(50) NOT NULL COMMENT '优惠券编码（唯一）',
  `name` varchar(100) NOT NULL COMMENT '优惠券名称',
  `type` tinyint(4) NOT NULL DEFAULT 1 COMMENT '优惠券类型：1-普通券，2-秒杀券',
  `description` text COMMENT '优惠券描述',
  `discount_type` tinyint(4) NOT NULL DEFAULT 1 COMMENT '折扣类型：1-满减，2-折扣，3-固定金额',
  `discount_value` decimal(10,2) NOT NULL COMMENT '折扣值（根据类型不同含义不同）',
  `min_amount` decimal(10,2) DEFAULT 0.00 COMMENT '最低消费金额',
  `max_discount_amount` decimal(10,2) DEFAULT NULL COMMENT '最大折扣金额',
  `total_quantity` int(11) NOT NULL DEFAULT 0 COMMENT '发行总量',
  `remaining_quantity` int(11) NOT NULL DEFAULT 0 COMMENT '剩余数量',
  `per_user_limit` int(11) DEFAULT 1 COMMENT '每人限领张数',
  `validity_type` tinyint(4) NOT NULL DEFAULT 1 COMMENT '有效期类型：1-固定时间段，2-领取后N天有效',
  `start_time` datetime DEFAULT NULL COMMENT '有效期开始时间',
  `end_time` datetime DEFAULT NULL COMMENT '有效期结束时间',
  `valid_days` int(11) DEFAULT NULL COMMENT '领取后有效天数',
  `status` tinyint(4) NOT NULL DEFAULT 1 COMMENT '状态：1-待发布，2-已发布，3-已下架，4-已过期',
  `apply_scope` tinyint(4) DEFAULT 1 COMMENT '适用范围：1-全场通用，2-指定分类，3-指定商品',
  `seckill_info` json DEFAULT NULL COMMENT '秒杀专用信息（仅type=2时使用）',
  `creator_id` bigint(20) DEFAULT NULL COMMENT '创建人ID',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_code` (`code`),
  KEY `idx_type_status` (`type`,`status`),
  KEY `idx_start_end_time` (`start_time`,`end_time`),
  KEY `idx_creator` (`creator_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='优惠券基础信息表';
```



### 1.2 秒杀专用信息表 `coupon_seckill`（可选，也可以使用JSON字段）

sql

```sql
CREATE TABLE `coupon_seckill` (
  `coupon_id` bigint(20) NOT NULL COMMENT '优惠券ID',
  `seckill_start_time` datetime NOT NULL COMMENT '秒杀开始时间',
  `seckill_end_time` datetime NOT NULL COMMENT '秒杀结束时间',
  `seckill_price` decimal(10,2) DEFAULT NULL COMMENT '秒杀价（如果优惠券本身是商品）',
  `preheat_time` datetime DEFAULT NULL COMMENT '预热开始时间',
  `purchase_limit` int(11) DEFAULT 1 COMMENT '限购数量',
  `flash_sale_strategy` tinyint(4) DEFAULT 1 COMMENT '秒杀策略：1-定时上架，2-阶梯秒杀，3-随机秒杀',
  `stock_sync_method` tinyint(4) DEFAULT 1 COMMENT '库存同步方式：1-实时，2-预热',
  `virtual_stock` int(11) DEFAULT 0 COMMENT '虚拟库存（用于超卖控制）',
  `actual_sold` int(11) DEFAULT 0 COMMENT '实际已售数量',
  `seckill_status` tinyint(4) DEFAULT 1 COMMENT '秒杀状态：1-未开始，2-进行中，3-已结束，4-已取消',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`coupon_id`),
  KEY `idx_seckill_time` (`seckill_start_time`,`seckill_end_time`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='秒杀优惠券专用信息表';
```



### 1.3 用户领取记录表 `coupon_user`

sql

```sql
CREATE TABLE `coupon_user` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '记录ID',
  `user_id` bigint(20) NOT NULL COMMENT '用户ID',
  `coupon_id` bigint(20) NOT NULL COMMENT '优惠券ID',
  `coupon_code` varchar(50) NOT NULL COMMENT '优惠券编码',
  `coupon_type` tinyint(4) NOT NULL COMMENT '优惠券类型（冗余字段）',
  `status` tinyint(4) NOT NULL DEFAULT 1 COMMENT '状态：1-未使用，2-已使用，3-已过期，4-已锁定（下单中）',
  `source` tinyint(4) DEFAULT 1 COMMENT '领取来源：1-主动领取，2-系统发放，3-活动赠送',
  `receive_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '领取时间',
  `valid_start_time` datetime NOT NULL COMMENT '有效开始时间',
  `valid_end_time` datetime NOT NULL COMMENT '有效结束时间',
  `use_time` datetime DEFAULT NULL COMMENT '使用时间',
  `order_id` varchar(50) DEFAULT NULL COMMENT '使用的订单ID',
  `use_platform` tinyint(4) DEFAULT NULL COMMENT '使用平台：1-PC，2-APP，3-小程序',
  `is_seckill` tinyint(1) DEFAULT 0 COMMENT '是否秒杀领取',
  `seckill_order_no` varchar(50) DEFAULT NULL COMMENT '秒杀订单号',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_user_coupon` (`user_id`,`coupon_id`) COMMENT '用户优惠券唯一',
  KEY `idx_user_status` (`user_id`,`status`),
  KEY `idx_coupon_status` (`coupon_id`,`status`),
  KEY `idx_valid_time` (`valid_end_time`,`status`),
  KEY `idx_seckill_order` (`seckill_order_no`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='用户优惠券领取记录表';
```



### 1.4 优惠券使用范围表 `coupon_scope`

sql

```sql
CREATE TABLE `coupon_scope` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `coupon_id` bigint(20) NOT NULL COMMENT '优惠券ID',
  `scope_type` tinyint(4) NOT NULL COMMENT '范围类型：1-商品分类，2-商品，3-品牌，4-店铺',
  `scope_id` bigint(20) NOT NULL COMMENT '范围ID（商品ID、分类ID等）',
  `scope_name` varchar(100) DEFAULT NULL COMMENT '范围名称（冗余）',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  KEY `idx_coupon_scope` (`coupon_id`,`scope_type`),
  KEY `idx_scope` (`scope_type`,`scope_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='优惠券适用范围表';
```



### 1.5 秒杀活动参与记录表 `seckill_participant`

sql

```sql
CREATE TABLE `seckill_participant` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `seckill_id` bigint(20) NOT NULL COMMENT '秒杀活动ID（对应coupon_id）',
  `user_id` bigint(20) NOT NULL COMMENT '用户ID',
  `session_id` varchar(100) DEFAULT NULL COMMENT '会话ID（用于防刷）',
  `ip_address` varchar(50) DEFAULT NULL COMMENT 'IP地址',
  `user_agent` varchar(500) DEFAULT NULL COMMENT '用户代理',
  `participate_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '参与时间',
  `result` tinyint(4) DEFAULT NULL COMMENT '参与结果：1-成功，2-失败，3-排队中',
  `queue_number` int(11) DEFAULT NULL COMMENT '排队号',
  `queue_start_time` datetime DEFAULT NULL COMMENT '排队开始时间',
  `queue_end_time` datetime DEFAULT NULL COMMENT '排队结束时间',
  `error_reason` varchar(200) DEFAULT NULL COMMENT '失败原因',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_user_seckill` (`user_id`,`seckill_id`) COMMENT '用户秒杀活动唯一',
  KEY `idx_seckill_result` (`seckill_id`,`result`),
  KEY `idx_participate_time` (`participate_time`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='秒杀活动参与记录表';
```



### 1.6 优惠券库存流水表 `coupon_stock_flow`

sql

```sql
CREATE TABLE `coupon_stock_flow` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `coupon_id` bigint(20) NOT NULL COMMENT '优惠券ID',
  `flow_type` tinyint(4) NOT NULL COMMENT '流水类型：1-发放，2-领取，3-核销，4-回退，5-作废',
  `change_quantity` int(11) NOT NULL COMMENT '变更数量（正数为增加，负数为减少）',
  `before_quantity` int(11) NOT NULL COMMENT '变更前数量',
  `after_quantity` int(11) NOT NULL COMMENT '变更后数量',
  `user_id` bigint(20) DEFAULT NULL COMMENT '操作用户ID',
  `order_id` varchar(50) DEFAULT NULL COMMENT '关联订单ID',
  `remark` varchar(200) DEFAULT NULL COMMENT '备注',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  KEY `idx_coupon_flow` (`coupon_id`,`flow_type`),
  KEY `idx_create_time` (`create_time`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='优惠券库存流水表';
```

我来帮您完善这个秒杀下单的完整实现，解决超卖和一人一单问题。

## 🛠️ 完整完善版本

### 1. 数据库表结构

```sql
-- 秒杀优惠券表
CREATE TABLE `seckill_voucher` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `voucher_id` bigint(20) NOT NULL COMMENT '关联的优惠券id',
  `title` varchar(255) NOT NULL COMMENT '标题',
  `stock` int(11) NOT NULL COMMENT '库存',
  `begin_time` datetime NOT NULL COMMENT '开始时间',
  `end_time` datetime NOT NULL COMMENT '结束时间',
  `create_time` datetime DEFAULT CURRENT_TIMESTAMP,
  `update_time` datetime DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_voucher_id` (`voucher_id`),
  KEY `idx_time` (`begin_time`,`end_time`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='秒杀优惠券表';

-- 优惠券订单表
CREATE TABLE `voucher_order` (
  `id` bigint(20) NOT NULL COMMENT '订单id',
  `user_id` bigint(20) NOT NULL COMMENT '下单的用户id',
  `voucher_id` bigint(20) NOT NULL COMMENT '购买的代金券id',
  `pay_type` tinyint(1) DEFAULT '1' COMMENT '支付方式 1：余额支付；2：支付宝；3：微信',
  `status` tinyint(1) DEFAULT '1' COMMENT '订单状态，1：未支付；2：已支付；3：已核销；4：已取消',
  `create_time` datetime DEFAULT CURRENT_TIMESTAMP COMMENT '下单时间',
  `pay_time` datetime DEFAULT NULL COMMENT '支付时间',
  `use_time` datetime DEFAULT NULL COMMENT '核销时间',
  `refund_time` datetime DEFAULT NULL COMMENT '退款时间',
  `update_time` datetime DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `idx_user_voucher` (`user_id`,`voucher_id`) COMMENT '防止用户重复购买',
  KEY `idx_user_id` (`user_id`),
  KEY `idx_voucher_id` (`voucher_id`),
  KEY `idx_create_time` (`create_time`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='优惠券订单表';
```

### 2. 实体类

```java
// SeckillVoucher.java
@Data
@TableName("seckill_voucher")
public class SeckillVoucher {
    private Long id;
    private Long voucherId;
    private String title;
    private Integer stock;
    private LocalDateTime beginTime;
    private LocalDateTime endTime;
    private LocalDateTime createTime;
    private LocalDateTime updateTime;
}

// VoucherOrder.java
@Data
@TableName("voucher_order")
public class VoucherOrder {
    @TableId(type = IdType.INPUT)  // 手动输入ID
    private Long id;
    private Long userId;
    private Long voucherId;
    private Integer payType;
    private Integer status;  // 1-未支付 2-已支付 3-已核销 4-已取消
    private LocalDateTime createTime;
    private LocalDateTime payTime;
    private LocalDateTime useTime;
    private LocalDateTime refundTime;
    private LocalDateTime updateTime;
}
```

### 3. 完善后的Service层

```java
@Service
@Slf4j
public class VoucherOrderServiceImpl extends ServiceImpl<VoucherOrderMapper, VoucherOrder> 
    implements IVoucherOrderService {

    @Resource
    private ISeckillVoucherService seckillVoucherService;
    
    @Resource
    private RedisIdWorker redisIdWorker;
    
    @Resource
    private StringRedisTemplate stringRedisTemplate;  // 用于分布式锁

    /**
     * 秒杀下单 - 完整版本（解决超卖和一人一单）
     */
    @Override
    @Transactional(rollbackFor = Exception.class)
    public Result seckillVoucher(Long voucherId) {
        // 1. 查询优惠券信息
        SeckillVoucher voucher = seckillVoucherService.getById(voucherId);
        if (voucher == null) {
            return Result.fail("优惠券不存在");
        }
        
        // 2. 判断秒杀是否开始
        LocalDateTime now = LocalDateTime.now();
        if (now.isBefore(voucher.getBeginTime())) {
            return Result.fail("秒杀尚未开始！开始时间：" + voucher.getBeginTime());
        }
        
        // 3. 判断秒杀是否已经结束
        if (now.isAfter(voucher.getEndTime())) {
            return Result.fail("秒杀已经结束！结束时间：" + voucher.getEndTime());
        }
        
        // 4. 判断库存是否充足
        if (voucher.getStock() < 1) {
            return Result.fail("库存不足");
        }
        
        // 5. 一人一单校验（获取当前用户）
        Long userId = UserHolder.getUser().getId();
        
        // 方法一：使用synchronized锁（单机版）
        // synchronized (userId.toString().intern()) {
        //     return createVoucherOrder(userId, voucherId, voucher);
        // }
        
        // 方法二：使用分布式锁（集群版）
        String lockKey = "order:" + userId + ":" + voucherId;
        RLock lock = null;
        try {
            // 尝试获取锁，最多等待5秒，锁过期时间10秒
            lock = redissonClient.getLock(lockKey);
            boolean isLock = lock.tryLock(5, 10, TimeUnit.SECONDS);
            
            if (!isLock) {
                return Result.fail("请勿重复操作");
            }
            
            // 执行下单逻辑
            return createVoucherOrder(userId, voucherId, voucher);
            
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            return Result.fail("系统繁忙，请稍后再试");
        } finally {
            if (lock != null && lock.isHeldByCurrentThread()) {
                lock.unlock();
            }
        }
    }
    
    /**
     * 创建订单的核心方法（加锁后执行）
     */
    @Transactional(rollbackFor = Exception.class)
    public Result createVoucherOrder(Long userId, Long voucherId, SeckillVoucher voucher) {
        // 1. 再次校验一人一单（防止并发问题）
        int count = query().eq("user_id", userId)
                          .eq("voucher_id", voucherId)
                          .count();
        if (count > 0) {
            return Result.fail("您已经购买过此优惠券！");
        }
        
        // 2. 扣减库存（使用乐观锁解决超卖问题）
        boolean success = seckillVoucherService.update()
                .setSql("stock = stock - 1")
                .eq("voucher_id", voucherId)
                .gt("stock", 0)  // 乐观锁：库存必须大于0
                // 或者使用版本号乐观锁：
                // .eq("version", voucher.getVersion())
                // .set("version", voucher.getVersion() + 1)
                .update();
        
        if (!success) {
            // 扣减失败，可能是库存不足或已被其他用户抢购
            return Result.fail("库存不足，抢购失败！");
        }
        
        // 3. 创建订单
        VoucherOrder voucherOrder = new VoucherOrder();
        
        // 生成分布式ID
        long orderId = redisIdWorker.nextId("order");
        voucherOrder.setId(orderId);
        voucherOrder.setUserId(userId);
        voucherOrder.setVoucherId(voucherId);
        voucherOrder.setStatus(1);  // 未支付
        voucherOrder.setCreateTime(LocalDateTime.now());
        
        // 4. 保存订单
        boolean saved = save(voucherOrder);
        if (!saved) {
            throw new RuntimeException("创建订单失败");
        }
        
        // 5. 记录日志（可选）
        log.info("用户 {} 成功抢购优惠券 {}，订单号：{}", userId, voucherId, orderId);
        
        // 6. 返回订单ID
        return Result.ok(orderId);
    }
    
    /**
     * 方法三：使用数据库唯一索引防止一人一单（最简单可靠）
     * 在voucher_order表创建唯一索引：UNIQUE KEY `uk_user_voucher` (`user_id`,`voucher_id`)
     * 然后直接创建订单，让数据库抛出异常
     */
    @Transactional(rollbackFor = Exception.class)
    public Result seckillVoucherSimple(Long voucherId) {
        // ... 前面校验逻辑相同 ...
        
        Long userId = UserHolder.getUser().getId();
        
        try {
            // 1. 扣减库存
            boolean success = seckillVoucherService.update()
                    .setSql("stock = stock - 1")
                    .eq("voucher_id", voucherId)
                    .gt("stock", 0)
                    .update();
            
            if (!success) {
                return Result.fail("库存不足");
            }
            
            // 2. 创建订单（依赖数据库唯一索引防止重复）
            VoucherOrder voucherOrder = new VoucherOrder();
            voucherOrder.setId(redisIdWorker.nextId("order"));
            voucherOrder.setUserId(userId);
            voucherOrder.setVoucherId(voucherId);
            voucherOrder.setStatus(1);
            voucherOrder.setCreateTime(LocalDateTime.now());
            
            save(voucherOrder);
            
            return Result.ok(voucherOrder.getId());
            
        } catch (DuplicateKeyException e) {
            // 捕获唯一索引冲突异常
            log.warn("用户重复购买，userId: {}, voucherId: {}", userId, voucherId);
            return Result.fail("您已经购买过此优惠券");
        } catch (Exception e) {
            log.error("下单异常", e);
            throw e;
        }
    }
}
```

### 4. 工具类：UserHolder（获取当前用户）

```java
@Component
public class UserHolder {
    private static final ThreadLocal<UserDTO> tl = new ThreadLocal<>();
    
    public static void saveUser(UserDTO user) {
        tl.set(user);
    }
    
    public static UserDTO getUser() {
        return tl.get();
    }
    
    public static Long getUserId() {
        UserDTO user = getUser();
        return user != null ? user.getId() : null;
    }
    
    public static void removeUser() {
        tl.remove();
    }
    
    @Data
    public static class UserDTO {
        private Long id;
        private String username;
        private String phone;
        // 其他用户信息...
    }
}
```

### 5. 拦截器（从Token获取用户）

```java
@Slf4j
@Component
public class LoginInterceptor implements HandlerInterceptor {
    
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        // 1. 从请求头获取token
        String token = request.getHeader("Authorization");
        if (StringUtils.isBlank(token)) {
            throw new BusinessException(401, "未登录");
        }
        
        // 2. 解析token获取用户信息
        UserDTO userDTO = parseToken(token);
        if (userDTO == null) {
            throw new BusinessException(401, "登录已过期");
        }
        
        // 3. 保存用户信息到ThreadLocal
        UserHolder.saveUser(userDTO);
        return true;
    }
    
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        // 移除用户信息，防止内存泄漏
        UserHolder.removeUser();
    }
    
    private UserDTO parseToken(String token) {
        // 解析JWT token的实现
        // 这里简化处理，实际应该验证token签名和过期时间
        try {
            // 示例：假设token是userId
            Long userId = Long.parseLong(token);
            UserDTO userDTO = new UserDTO();
            userDTO.setId(userId);
            // 可以从数据库查询用户信息
            return userDTO;
        } catch (Exception e) {
            log.error("解析token失败", e);
            return null;
        }
    }
}
```

### 6. 全局异常处理

```java
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {
    
    /**
     * 业务异常处理
     */
    @ExceptionHandler(BusinessException.class)
    public Result handleBusinessException(BusinessException e) {
        log.warn("业务异常: {}", e.getMessage());
        return Result.fail(e.getMessage());
    }
    
    /**
     * 唯一键冲突异常（一人一单）
     */
    @ExceptionHandler(DuplicateKeyException.class)
    public Result handleDuplicateKeyException(DuplicateKeyException e) {
        log.warn("数据重复: {}", e.getMessage());
        return Result.fail("请勿重复操作");
    }
    
    /**
     * 乐观锁异常
     */
    @ExceptionHandler(OptimisticLockingFailureException.class)
    public Result handleOptimisticLockException(OptimisticLockingFailureException e) {
        log.warn("并发冲突: {}", e.getMessage());
        return Result.fail("抢购过于火爆，请稍后重试");
    }
    
    /**
     * 全局异常处理
     */
    @ExceptionHandler(Exception.class)
    public Result handleException(Exception e) {
        log.error("系统异常", e);
        return Result.fail("系统繁忙，请稍后再试");
    }
}

// 自定义业务异常
@Data
@EqualsAndHashCode(callSuper = true)
public class BusinessException extends RuntimeException {
    private Integer code;
    
    public BusinessException(Integer code, String message) {
        super(message);
        this.code = code;
    }
}
```

### 7. Result统一返回格式

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Result {
    private Boolean success;
    private String message;
    private Object data;
    
    public static Result ok() {
        return new Result(true, null, null);
    }
    
    public static Result ok(Object data) {
        return new Result(true, null, data);
    }
    
    public static Result fail(String message) {
        return new Result(false, message, null);
    }
    
    public static Result fail(String message, Object data) {
        return new Result(false, message, data);
    }
}
```

### 8. 优化建议：使用Redis预减库存

```java
@Service
@Slf4j
public class SeckillServiceV2 {
    
    @Resource
    private StringRedisTemplate stringRedisTemplate;
    
    // 秒杀活动开始前，将库存加载到Redis
    public void loadStockToRedis(Long voucherId) {
        SeckillVoucher voucher = seckillVoucherService.getById(voucherId);
        String key = "seckill:stock:" + voucherId;
        stringRedisTemplate.opsForValue().set(key, voucher.getStock().toString());
    }
    
    /**
     * 秒杀流程优化：
     * 1. Redis预减库存（快速失败）
     * 2. 消息队列异步下单
     */
    public Result seckillVoucherWithRedis(Long voucherId) {
        Long userId = UserHolder.getUserId();
        
        // 1. 资格校验（一人一单）
        String userOrderKey = "seckill:user:" + voucherId + ":" + userId;
        Boolean isFirst = stringRedisTemplate.opsForValue().setIfAbsent(userOrderKey, "1", 24, TimeUnit.HOURS);
        if (Boolean.FALSE.equals(isFirst)) {
            return Result.fail("您已经参与过此活动");
        }
        
        // 2. Redis预减库存
        String stockKey = "seckill:stock:" + voucherId;
        Long stock = stringRedisTemplate.opsForValue().decrement(stockKey);
        if (stock == null || stock < 0) {
            // 库存不足，恢复用户的购买资格
            stringRedisTemplate.delete(userOrderKey);
            return Result.fail("库存不足");
        }
        
        // 3. 发送消息到队列异步下单
        VoucherOrderMessage message = new VoucherOrderMessage();
        message.setUserId(userId);
        message.setVoucherId(voucherId);
        message.setOrderId(redisIdWorker.nextId("order"));
        
        // 发送到RabbitMQ或Kafka
        // rabbitTemplate.convertAndSend("seckill.exchange", "seckill.order", message);
        
        // 4. 立即返回排队中
        return Result.ok("抢购成功，正在生成订单...");
    }
    
    /**
     * 消息队列消费者：异步处理订单
     */
    @RabbitListener(queues = "seckill.order.queue")
    public void handleSeckillOrder(VoucherOrderMessage message) {
        try {
            // 异步下单，这里可以慢慢处理数据库操作
            createVoucherOrder(message.getUserId(), message.getVoucherId(), message.getOrderId());
        } catch (Exception e) {
            log.error("处理秒杀订单失败", e);
            // 失败后需要恢复Redis中的库存
            stringRedisTemplate.opsForValue().increment("seckill:stock:" + message.getVoucherId());
            // 移除用户限制
            stringRedisTemplate.delete("seckill:user:" + message.getVoucherId() + ":" + message.getUserId());
        }
    }
}
```

## 🎯 总结解决方案

### 超卖问题解决方案（优先级）：
1. **乐观锁**（推荐）：`UPDATE ... SET stock = stock - 1 WHERE stock > 0`
2. **Redis原子操作**：预减库存，快速失败
3. **悲观锁**：在极端高并发下使用，但性能较差

### 一人一单解决方案（优先级）：
1. **数据库唯一索引**（最简单可靠）：创建`(user_id, voucher_id)`唯一索引
2. **Redis setnx**：快速校验，配合数据库唯一索引
3. **分布式锁**：复杂场景使用，如Redisson

### 完整流程建议：
1. **请求入口**：校验用户登录状态
2. **资格校验**：时间、库存、用户是否已购买
3. **库存扣减**：Redis预减库存（快速）
4. **订单创建**：消息队列异步处理
5. **返回结果**：立即返回，异步通知

这样设计既保证了系统的**高并发性能**，又确保了**数据的一致性**。