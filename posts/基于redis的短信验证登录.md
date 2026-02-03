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