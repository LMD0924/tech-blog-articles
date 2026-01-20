# MySQL 常用字符串函数

## 📌 字符串基本操作

### 1. **连接函数**
```sql
-- CONCAT: 连接多个字符串
SELECT CONCAT('Hello', ' ', 'World'); -- 'Hello World'

-- CONCAT_WS: 用分隔符连接
SELECT CONCAT_WS(', ', 'John', 'Doe', 'USA'); -- 'John, Doe, USA'
```

### 2. **长度相关**
```sql
-- LENGTH: 字节数（受字符集影响）
SELECT LENGTH('你好'); -- 6 (UTF-8)

-- CHAR_LENGTH / CHARACTER_LENGTH: 字符数
SELECT CHAR_LENGTH('你好'); -- 2

-- BIT_LENGTH: 位数
SELECT BIT_LENGTH('abc'); -- 24 (3*8)
```

## 🔍 字符串查找与提取

### 3. **查找函数**
```sql
-- INSTR: 返回子串位置（从1开始）
SELECT INSTR('Hello World', 'World'); -- 7

-- LOCATE / POSITION: 同 INSTR
SELECT LOCATE('lo', 'Hello World'); -- 4

-- FIND_IN_SET: 在逗号分隔的列表中查找
SELECT FIND_IN_SET('b', 'a,b,c,d'); -- 2
```

### 4. **提取函数**
```sql
-- SUBSTRING / SUBSTR: 提取子串
SELECT SUBSTRING('Hello World', 7); -- 'World'
SELECT SUBSTRING('Hello World', 1, 5); -- 'Hello'

-- LEFT / RIGHT: 从左/右提取
SELECT LEFT('Hello World', 5); -- 'Hello'
SELECT RIGHT('Hello World', 5); -- 'World'

-- MID: 同 SUBSTRING
SELECT MID('Hello World', 7, 5); -- 'World'
```

## 🔧 字符串修改

### 5. **大小写转换**
```sql
-- UPPER / UCASE: 转大写
SELECT UPPER('hello'); -- 'HELLO'

-- LOWER / LCASE: 转小写
SELECT LOWER('WORLD'); -- 'world'
```

### 6. **修剪函数**
```sql
-- TRIM: 去除两端空格或指定字符
SELECT TRIM('  Hello  '); -- 'Hello'
SELECT TRIM(LEADING '0' FROM '000123'); -- '123'
SELECT TRIM(TRAILING 'x' FROM 'abcxxx'); -- 'abc'

-- LTRIM / RTRIM: 去除左/右空格
SELECT LTRIM('  Hello'); -- 'Hello'
SELECT RTRIM('Hello  '); -- 'Hello'
```

### 7. **替换函数**
```sql
-- REPLACE: 替换所有匹配
SELECT REPLACE('Hello World', 'World', 'MySQL'); -- 'Hello MySQL'

-- INSERT: 插入/替换字符串
SELECT INSERT('Hello World', 7, 5, 'MySQL'); -- 'Hello MySQL'
-- 从第7个位置开始，替换5个字符
```

## 📐 填充与格式化

### 8. **填充函数**
```sql
-- LPAD / RPAD: 左/右填充
SELECT LPAD('123', 5, '0'); -- '00123'
SELECT RPAD('123', 5, '*'); -- '123**'

-- SPACE: 生成空格字符串
SELECT CONCAT('Hello', SPACE(3), 'World'); -- 'Hello   World'
```

## 🎯 字符串比较

### 9. **比较函数**
```sql
-- STRCMP: 字符串比较
SELECT STRCMP('abc', 'abc'); -- 0 (相等)
SELECT STRCMP('abc', 'abd'); -- -1 (小于)
SELECT STRCMP('abd', 'abc'); -- 1 (大于)
```

## 🔄 编码与转换

### 10. **编码转换**
```sql
-- ASCII: 返回第一个字符的ASCII值
SELECT ASCII('A'); -- 65

-- CHAR: ASCII转字符
SELECT CHAR(65); -- 'A'

-- BIN / OCT / HEX: 转二进制/八进制/十六进制
SELECT HEX('abc'); -- '616263'
SELECT UNHEX('616263'); -- 'abc'

-- ORD: 返回第一个字符的编码值
SELECT ORD('€'); -- 8364
```

## 📊 字符串分割与组合

### 11. **分割与组合**
```sql
-- SUBSTRING_INDEX: 按分隔符分割
SELECT SUBSTRING_INDEX('www.mysql.com', '.', 1); -- 'www'
SELECT SUBSTRING_INDEX('www.mysql.com', '.', -2); -- 'mysql.com'

-- REPEAT: 重复字符串
SELECT REPEAT('MySQL ', 3); -- 'MySQL MySQL MySQL '
```

## 🔍 正则表达式

### 12. **正则表达式函数**
```sql
-- REGEXP / RLIKE: 正则匹配
SELECT 'hello' REGEXP '^h'; -- 1 (true)
SELECT '123' REGEXP '^[0-9]+$'; -- 1

-- REGEXP_REPLACE (MySQL 8.0+)
SELECT REGEXP_REPLACE('abc123def456', '[0-9]+', 'X'); -- 'abcXdefX'

-- REGEXP_SUBSTR (MySQL 8.0+)
SELECT REGEXP_SUBSTR('Email: test@example.com', '[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,}'); -- 'test@example.com'
```

## 📝 格式化与显示

### 13. **格式化函数**
```sql
-- FORMAT: 数字格式化
SELECT FORMAT(1234567.89, 2); -- '1,234,567.89'

-- QUOTE: 添加引号
SELECT QUOTE('Hello'); -- "'Hello'"

-- ELT: 返回索引位置的字符串
SELECT ELT(2, 'Apple', 'Banana', 'Cherry'); -- 'Banana'

-- FIELD: 返回字符串在列表中的位置
SELECT FIELD('Banana', 'Apple', 'Banana', 'Cherry'); -- 2
```

## 🛡️ 安全相关

### 14. **安全函数**
```sql
-- WEIGHT_STRING: 显示字符串权重（用于校对）
SELECT WEIGHT_STRING('abc');

-- SOUNDEX: 语音编码（用于相似发音）
SELECT SOUNDEX('hello'), SOUNDEX('hallo'); -- 相似值
```

## 💡 实用组合示例

### 示例1：格式化姓名
```sql
SELECT CONCAT(
    UPPER(LEFT(first_name, 1)),
    LOWER(SUBSTRING(first_name, 2)),
    ' ',
    UPPER(last_name)
) AS full_name
FROM users;
```

### 示例2：提取域名
```sql
SELECT 
    SUBSTRING_INDEX(
        SUBSTRING_INDEX(email, '@', -1),
        '.',
        1
    ) AS domain
FROM users;
```

### 示例3：安全显示手机号
```sql
SELECT CONCAT(
    LEFT(phone, 3),
    '****',
    RIGHT(phone, 4)
) AS masked_phone
FROM contacts;
```

### 示例4：分割逗号分隔值
```sql
-- 假设有 'a,b,c,d'
SELECT 
    SUBSTRING_INDEX(SUBSTRING_INDEX('a,b,c,d', ',', 1), ',', -1) AS item1,
    SUBSTRING_INDEX(SUBSTRING_INDEX('a,b,c,d', ',', 2), ',', -1) AS item2,
    SUBSTRING_INDEX(SUBSTRING_INDEX('a,b,c,d', ',', 3), ',', -1) AS item3,
    SUBSTRING_INDEX(SUBSTRING_INDEX('a,b,c,d', ',', 4), ',', -1) AS item4;
```

## 🎯 版本注意事项

| 函数           | MySQL 5.7 | MySQL 8.0 | 说明     |
| -------------- | --------- | --------- | -------- |
| REGEXP_REPLACE | ❌ 不支持  | ✅ 支持    | 正则替换 |
| REGEXP_SUBSTR  | ❌ 不支持  | ✅ 支持    | 正则提取 |
| JSON 函数      | 有限支持  | 完整支持  | JSON处理 |

这些函数覆盖了 MySQL 中 95% 的日常字符串操作需求，掌握它们能极大提高数据库查询和处理效率！