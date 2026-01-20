# MySQL 常用日期时间函数

## 📅 日期时间获取

### 1. **当前时间获取**
```sql
-- 当前日期和时间
SELECT NOW();        -- 2024-01-19 10:30:45
SELECT SYSDATE();    -- 系统日期时间（执行时的时间）
SELECT CURRENT_TIMESTAMP(); -- 当前时间戳

-- 当前日期
SELECT CURDATE();    -- 2024-01-19
SELECT CURRENT_DATE();

-- 当前时间
SELECT CURTIME();    -- 10:30:45
SELECT CURRENT_TIME();

-- UTC时间
SELECT UTC_DATE();   -- 2024-01-19
SELECT UTC_TIME();   -- 02:30:45
SELECT UTC_TIMESTAMP();
```

### 2. **日期时间各部分提取**
```sql
-- 年、月、日
SELECT YEAR('2024-01-19');     -- 2024
SELECT MONTH('2024-01-19');    -- 1
SELECT MONTHNAME('2024-01-19');-- January
SELECT DAY('2024-01-19');      -- 19
SELECT DAYOFMONTH('2024-01-19'); -- 19

-- 星期相关
SELECT DAYNAME('2024-01-19');   -- Friday
SELECT DAYOFWEEK('2024-01-19'); -- 6 (1=Sunday)
SELECT WEEKDAY('2024-01-19');   -- 4 (0=Monday)
SELECT WEEK('2024-01-19');      -- 3 (第几周)

-- 季度
SELECT QUARTER('2024-01-19');   -- 1

-- 时、分、秒
SELECT HOUR('10:30:45');        -- 10
SELECT MINUTE('10:30:45');      -- 30
SELECT SECOND('10:30:45');      -- 45

-- 微秒
SELECT MICROSECOND('10:30:45.123456'); -- 123456
```

## 🔄 日期时间计算

### 3. **日期加减运算**
```sql
-- DATE_ADD / ADDDATE: 日期加
SELECT DATE_ADD('2024-01-19', INTERVAL 1 DAY);    -- 2024-01-20
SELECT DATE_ADD('2024-01-19', INTERVAL 2 MONTH);  -- 2024-03-19
SELECT DATE_ADD('2024-01-19', INTERVAL 3 YEAR);   -- 2027-01-19
SELECT DATE_ADD('10:30:45', INTERVAL 1 HOUR);     -- 11:30:45

-- DATE_SUB / SUBDATE: 日期减
SELECT DATE_SUB('2024-01-19', INTERVAL 1 WEEK);   -- 2024-01-12
```

### 4. **快速加减函数**
```sql
-- 加减天数
SELECT '2024-01-19' + INTERVAL 1 DAY;   -- 2024-01-20
SELECT '2024-01-19' - INTERVAL 1 DAY;   -- 2024-01-18

-- 组合使用
SELECT '2024-01-19 10:30:00' + 
       INTERVAL '1 2:30:00' DAY_SECOND; -- 2024-01-20 13:00:00
```

## 📊 日期时间差计算

### 5. **差值计算**
```sql
-- DATEDIFF: 日期差（天数）
SELECT DATEDIFF('2024-01-20', '2024-01-19'); -- 1

-- TIMEDIFF: 时间差
SELECT TIMEDIFF('11:30:00', '10:30:00');    -- 01:00:00

-- TIMESTAMPDIFF: 时间戳差（可指定单位）
SELECT TIMESTAMPDIFF(DAY, '2024-01-01', '2024-01-19');     -- 18
SELECT TIMESTAMPDIFF(HOUR, '2024-01-19 10:00', '2024-01-19 12:30'); -- 2
SELECT TIMESTAMPDIFF(MONTH, '2024-01-01', '2024-03-01');   -- 2
SELECT TIMESTAMPDIFF(YEAR, '2020-01-01', '2024-01-01');    -- 4
```

### 6. **时间单位换算**
```sql
-- 支持的INTERVAL单位：
-- MICROSECOND, SECOND, MINUTE, HOUR, DAY, WEEK, 
-- MONTH, QUARTER, YEAR, SECOND_MICROSECOND, 
-- MINUTE_MICROSECOND, MINUTE_SECOND, 
-- HOUR_MICROSECOND, HOUR_SECOND, HOUR_MINUTE,
-- DAY_MICROSECOND, DAY_SECOND, DAY_MINUTE, DAY_HOUR
```

## 🎯 日期时间格式化

### 7. **格式化输出**
```sql
-- DATE_FORMAT: 日期格式化
SELECT DATE_FORMAT(NOW(), '%Y-%m-%d');              -- 2024-01-19
SELECT DATE_FORMAT(NOW(), '%Y年%m月%d日');          -- 2024年01月19日
SELECT DATE_FORMAT(NOW(), '%W, %M %d, %Y');         -- Friday, January 19, 2024
SELECT DATE_FORMAT(NOW(), '%H:%i:%s');              -- 10:30:45
SELECT DATE_FORMAT(NOW(), '%Y%m%d%H%i%s');          -- 20240119103045
```

### 8. **常用格式符号**
```sql
-- 年：%Y(2024), %y(24)
-- 月：%m(01-12), %c(1-12), %M(January), %b(Jan)
-- 日：%d(01-31), %e(1-31), %D(1st, 2nd)
-- 时：%H(00-23), %k(0-23), %h(01-12), %I(01-12), %l(1-12)
-- 分：%i(00-59)
-- 秒：%s(00-59)
-- 上下午：%p(AM/PM)
-- 星期：%W(Sunday), %a(Sun), %w(0-6, 0=Sunday)
-- 一年中的第几天：%j(001-366)
-- 一年中的第几周：%U(00-53, Sunday开始), %u(00-53, Monday开始)
```

### 9. **字符串转日期**
```sql
-- STR_TO_DATE: 字符串转日期
SELECT STR_TO_DATE('2024-01-19', '%Y-%m-%d');       -- 2024-01-19
SELECT STR_TO_DATE('01/19/2024', '%m/%d/%Y');       -- 2024-01-19
SELECT STR_TO_DATE('19 January 2024', '%d %M %Y');  -- 2024-01-19

-- UNIX时间戳转日期
SELECT FROM_UNIXTIME(1705645845);                   -- 2024-01-19 10:30:45
SELECT FROM_UNIXTIME(1705645845, '%Y-%m-%d %H:%i:%s');
```

### 10. **日期转字符串/数字**
```sql
-- 日期转字符串
SELECT DATE_FORMAT(NOW(), '%Y%m%d');                -- 20240119

-- 日期转数字
SELECT YEAR(NOW()) * 10000 + MONTH(NOW()) * 100 + DAY(NOW()); -- 20240119

-- UNIX时间戳
SELECT UNIX_TIMESTAMP();                           -- 1705645845
SELECT UNIX_TIMESTAMP('2024-01-19 10:30:45');
```

## 🧮 特殊日期计算

### 11. **月份相关计算**
```sql
-- 月份第一天/最后一天
SELECT DATE_FORMAT(NOW(), '%Y-%m-01');              -- 当月第一天
SELECT LAST_DAY(NOW());                             -- 当月最后一天
SELECT LAST_DAY('2024-02-15');                      -- 2024-02-29

-- 月份加减（保持月末）
SELECT LAST_DAY(DATE_ADD('2024-01-31', INTERVAL 1 MONTH)); -- 2024-02-29
```

### 12. **日期截断**
```sql
-- DATE: 提取日期部分
SELECT DATE('2024-01-19 10:30:45');                 -- 2024-01-19

-- TIME: 提取时间部分
SELECT TIME('2024-01-19 10:30:45');                 -- 10:30:45

-- 提取日期时间各部分
SELECT EXTRACT(YEAR FROM NOW());                    -- 2024
SELECT EXTRACT(YEAR_MONTH FROM NOW());              -- 202401
SELECT EXTRACT(DAY_HOUR FROM NOW());                -- 1910
SELECT EXTRACT(HOUR_MINUTE FROM NOW());             -- 1030
```

## 📈 周和季度计算

### 13. **周相关函数**
```sql
-- 周开始和结束
SELECT DATE_SUB(NOW(), INTERVAL WEEKDAY(NOW()) DAY); -- 本周一
SELECT DATE_ADD(NOW(), INTERVAL 6 - WEEKDAY(NOW()) DAY); -- 本周日

-- 一年中的第几周
SELECT WEEK(NOW());                                 -- 3 (默认周日开始)
SELECT WEEK(NOW(), 1);                              -- 3 (周一开始)
SELECT WEEKOFYEAR(NOW());                           -- 3
```

### 14. **季度计算**
```sql
-- 季度开始和结束
SELECT CONCAT(YEAR(NOW()), '-', (QUARTER(NOW())*3-2), '-01'); -- 季度第一天
SELECT LAST_DAY(
    MAKEDATE(YEAR(NOW()), 1) + 
    INTERVAL QUARTER(NOW())*3-1 MONTH
); -- 季度最后一天
```

## 🔢 日期时间生成

### 15. **构造日期时间**
```sql
-- MAKEDATE: 根据年份和天数构造
SELECT MAKEDATE(2024, 19);                          -- 2024-01-19

-- MAKETIME: 构造时间
SELECT MAKETIME(10, 30, 45);                        -- 10:30:45

-- 组合日期时间
SELECT TIMESTAMP('2024-01-19', '10:30:45');         -- 2024-01-19 10:30:45
SELECT DATE('2024-01-19') + INTERVAL '10:30:45' HOUR_SECOND;
```

## 📊 日期时间验证

### 16. **有效性检查**
```sql
-- 检查日期有效性
SELECT DATE('2024-02-30');                          -- NULL（无效日期）
SELECT IS_DATE('2024-02-30');                       -- 0 (false)

-- 检查时间有效性
SELECT TIME('25:00:00');                            -- NULL（无效时间）

-- 完整验证
SELECT STR_TO_DATE('2024-02-30', '%Y-%m-%d');       -- NULL
```

## 💡 实用示例

### 示例1：年龄计算
```sql
-- 计算年龄（精确到年）
SELECT TIMESTAMPDIFF(YEAR, birth_date, CURDATE()) AS age
FROM users;

-- 计算精确年龄（年月日）
SELECT 
    TIMESTAMPDIFF(YEAR, birth_date, CURDATE()) AS years,
    TIMESTAMPDIFF(MONTH, birth_date, CURDATE()) % 12 AS months,
    DATEDIFF(CURDATE(), 
        DATE_ADD(birth_date, 
            INTERVAL TIMESTAMPDIFF(MONTH, birth_date, CURDATE()) MONTH
        )
    ) AS days
FROM users;
```

### 示例2：本月统计
```sql
-- 本月第一天和最后一天
SELECT 
    DATE_FORMAT(NOW(), '%Y-%m-01') AS month_start,
    LAST_DAY(NOW()) AS month_end;

-- 本月订单统计
SELECT COUNT(*)
FROM orders
WHERE order_date >= DATE_FORMAT(NOW(), '%Y-%m-01')
  AND order_date <= LAST_DAY(NOW());
```

### 示例3：工作日计算
```sql
-- 计算下一个工作日（跳过周末）
SELECT 
    CASE 
        WHEN WEEKDAY(@date) = 4 THEN DATE_ADD(@date, INTERVAL 3 DAY)
        WHEN WEEKDAY(@date) = 5 THEN DATE_ADD(@date, INTERVAL 2 DAY)
        ELSE DATE_ADD(@date, INTERVAL 1 DAY)
    END AS next_workday;
```

### 示例4：时间段查询
```sql
-- 最近7天
SELECT * FROM logs 
WHERE log_time >= DATE_SUB(NOW(), INTERVAL 7 DAY);

-- 本月
SELECT * FROM orders 
WHERE YEAR(order_date) = YEAR(NOW()) 
  AND MONTH(order_date) = MONTH(NOW());

-- 今天
SELECT * FROM visits 
WHERE DATE(visit_time) = CURDATE();

-- 特定时间段
SELECT * FROM events 
WHERE event_time BETWEEN '09:00:00' AND '18:00:00';
```

### 示例5：生日提醒
```sql
-- 本月过生日的人
SELECT name, birth_date
FROM users
WHERE MONTH(birth_date) = MONTH(NOW());

-- 7天内过生日的人
SELECT name, birth_date
FROM users
WHERE 
    DATE_FORMAT(birth_date, '%m-%d') BETWEEN 
        DATE_FORMAT(NOW(), '%m-%d') AND 
        DATE_FORMAT(DATE_ADD(NOW(), INTERVAL 7 DAY), '%m-%d')
    OR (
        -- 处理跨年情况
        DATE_FORMAT(birth_date, '%m-%d') >= DATE_FORMAT(NOW(), '%m-%d')
        AND DATE_FORMAT(birth_date, '%m-%d') <= '12-31'
    ) AND (
        DATE_FORMAT(birth_date, '%m-%d') <= DATE_FORMAT(DATE_ADD(NOW(), INTERVAL 7 DAY), '%m-%d')
        OR DATE_FORMAT(DATE_ADD(NOW(), INTERVAL 7 DAY), '%m-%d') < DATE_FORMAT(NOW(), '%m-%d')
    );
```

## 🚀 性能优化建议

1. **避免在WHERE条件中使用函数**
   ```sql
   -- 不好：无法使用索引
   SELECT * FROM orders WHERE DATE(order_date) = CURDATE();
   
   -- 好：可以使用索引
   SELECT * FROM orders 
   WHERE order_date >= CURDATE() 
     AND order_date < CURDATE() + INTERVAL 1 DAY;
   ```

2. **使用正确的数据类型**
   - `DATE`: 仅日期
   - `TIME`: 仅时间
   - `DATETIME`: 日期时间
   - `TIMESTAMP`: 时间戳（带时区）
   - `YEAR`: 年份

3. **建立索引**
   ```sql
   -- 对经常查询的日期字段建立索引
   CREATE INDEX idx_order_date ON orders(order_date);
   CREATE INDEX idx_created_at ON users(created_at);
   ```

## 📚 MySQL 8.0 新增功能

```sql
-- 时区支持
SELECT NOW() AT TIME ZONE 'Asia/Shanghai';

-- 更多日期函数
SELECT DATE_BIN('15 minutes', NOW(), '2001-01-01'); -- 时间分箱

-- 更好的JSON支持
SELECT JSON_EXTRACT('{"date": "2024-01-19"}', '$.date');
```

这些函数覆盖了 MySQL 中 95% 的日期时间处理需求，合理使用能极大简化开发工作！