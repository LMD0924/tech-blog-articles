# Redis

## Redis快速入门

**键值数据库**

### 认识NoSQL

|          | SQL                                              | NoSQL                                                |
| -------- | ------------------------------------------------ | ---------------------------------------------------- |
| 数据结构 | 结构化                                           | 非结构化                                             |
| 数据关联 | 关联的                                           | 无关联的                                             |
| 查询方式 | SQL查询                                          | 非SQL                                                |
| 事务特性 | ACID                                             | BASE                                                 |
| 存储方式 | 磁盘                                             | 内存                                                 |
| 扩展性   | 垂直                                             | 水平                                                 |
| 使用场景 | 数据结构固定。相关业务对数据安全性，一致性要求高 | 数据结构不固定。对一致性，安全性要求不高，对性能要求 |

### 认识Redis

**特征：**

- 键值型，value支持多种不同数据结构，功能丰富
- 单线程，每个命令具备原子性
- 低延迟，速度块（基于内存，IO多路复用，良好的编码）；
- 支持数据持久化
- 支持主从集群，分片集群
- 支持多语言客户端

### Redis命令行客户端

```text
redis-cli [options] [commonds]
```

其中常见的options有：

- -h 127.0.0.1：指定要连接的redis节点的IP地址，默认是127.0.0.1
- -p 6379：指定要连接的节点端口，默认6379
- -a：指定密码

其中commonds就是Redis的操作命令：

ping：与redis服务端做心跳测试，服务端正常会返回pong

### Redis通用命令

KEYS：查看符合模板的所有key，不建议使用

DEL：删除指定的key

EXISTS：判断key是否存在

EXPIRE：给一个key设置有效期，有效期到期时该key会被自动删除

TTL：查看key的剩余有效期

### String类型

#### 一、String 类型特点

Redis 的 String 类型是最基础的数据类型，是**二进制安全的**（可以存储任何数据，包括图片、序列化对象等）。一个 String 类型的值最大可以是 **512MB**。

#### 二、常用命令

#### 1. **基本操作**

| 命令       | 语法                                                   | 说明           | 示例              |
| :--------- | :----------------------------------------------------- | :------------- | :---------------- |
| **SET**    | `SET key value [EX seconds] [PX milliseconds] [NX|XX]` | 设置键值对     | `SET name "John"` |
| **GET**    | `GET key`                                              | 获取键的值     | `GET name`        |
| **DEL**    | `DEL key [key ...]`                                    | 删除键         | `DEL name`        |
| **EXISTS** | `EXISTS key [key ...]`                                 | 检查键是否存在 | `EXISTS name`     |

**SET 选项说明：**

- `EX seconds`：设置过期时间（秒）
- `PX milliseconds`：设置过期时间（毫秒）
- `NX`：仅当键不存在时设置
- `XX`：仅当键存在时设置

#### 2. **批量操作**

| 命令     | 语法                             | 说明               |
| :------- | :------------------------------- | :----------------- |
| **MSET** | `MSET key value [key value ...]` | 批量设置多个键值对 |
| **MGET** | `MGET key [key ...]`             | 批量获取多个键的值 |

#### 3. **数值操作**（针对整数字符串）

| 命令            | 语法                        | 说明                 |
| :-------------- | :-------------------------- | :------------------- |
| **INCR**        | `INCR key`                  | 将键的值增加1        |
| **DECR**        | `DECR key`                  | 将键的值减少1        |
| **INCRBY**      | `INCRBY key increment`      | 将键的值增加指定数值 |
| **DECRBY**      | `DECRBY key decrement`      | 将键的值减少指定数值 |
| **INCRBYFLOAT** | `INCRBYFLOAT key increment` | 增加浮点数值         |

#### 4. **字符串操作**

| 命令         | 语法                        | 说明                          |
| :----------- | :-------------------------- | :---------------------------- |
| **APPEND**   | `APPEND key value`          | 追加值到原值末尾              |
| **STRLEN**   | `STRLEN key`                | 获取值的长度                  |
| **GETRANGE** | `GETRANGE key start end`    | 获取子字符串（类似substring） |
| **SETRANGE** | `SETRANGE key offset value` | 从指定偏移量开始覆盖字符串    |

### Hash类型

Redis Hash 是一个 **field-value 映射表**，适合存储对象。

#### 二、**基本命令**

#### **1. 设置和获取字段值**

| 命令        | 语法                                     | 说明                 | 示例                                |
| :---------- | :--------------------------------------- | :------------------- | :---------------------------------- |
| **HSET**    | `HSET key field value [field value ...]` | 设置一个或多个字段值 | `HSET user:1001 name "John" age 30` |
| **HGET**    | `HGET key field`                         | 获取指定字段的值     | `HGET user:1001 name`               |
| **HDEL**    | `HDEL key field [field ...]`             | 删除一个或多个字段   | `HDEL user:1001 age`                |
| **HEXISTS** | `HEXISTS key field`                      | 检查字段是否存在     | `HEXISTS user:1001 name`            |

```sql
# 创建用户对象
127.0.0.1:6379> HSET user:1001 name "张三" age 25 email "zhangsan@example.com"
(integer) 3  # 返回设置的字段数量

# 获取单个字段
127.0.0.1:6379> HGET user:1001 name
"张三"

# 检查字段是否存在
127.0.0.1:6379> HEXISTS user:1001 age
(integer) 1  # 存在
127.0.0.1:6379> HEXISTS user:1001 address
(integer) 0  # 不存在
```

### **2. 批量操作**

| 命令        | 语法                                      | 说明                                 |
| :---------- | :---------------------------------------- | :----------------------------------- |
| **HMSET**   | `HMSET key field value [field value ...]` | 批量设置字段值（已弃用，用HSET替代） |
| **HMGET**   | `HMGET key field [field ...]`             | 批量获取字段值                       |
| **HGETALL** | `HGETALL key`                             | 获取所有字段和值                     |

```sql
# 批量设置（Redis 4.0+ HSET已支持多字段）
127.0.0.1:6379> HSET product:1001 title "iPhone 15" price 6999 stock 100 category "phone"
(integer) 4

# 批量获取
127.0.0.1:6379> HMGET product:1001 title price
1) "iPhone 15"
2) "6999"

# 获取所有字段和值
127.0.0.1:6379> HGETALL product:1001
1) "title"
2) "iPhone 15"
3) "price"
4) "6999"
5) "stock"
6) "100"
7) "category"
8) "phone"
```

### **3. 数值操作**

| 命令             | 语法                               | 说明             |
| :--------------- | :--------------------------------- | :--------------- |
| **HINCRBY**      | `HINCRBY key field increment`      | 字段值增加整数   |
| **HINCRBYFLOAT** | `HINCRBYFLOAT key field increment` | 字段值增加浮点数 |

**示例：**

bash

```sql
# 计数器
127.0.0.1:6379> HSET article:1001 views 0 likes 0
(integer) 2

# 增加阅读量
127.0.0.1:6379> HINCRBY article:1001 views 1
(integer) 1
127.0.0.1:6379> HINCRBY article:1001 views 5
(integer) 6

# 增加点赞（浮点数示例）
127.0.0.1:6379> HINCRBYFLOAT article:1001 rating 0.5
"0.5"
```



### **4. 获取信息**

| 命令        | 语法                | 说明                   |
| :---------- | :------------------ | :--------------------- |
| **HLEN**    | `HLEN key`          | 获取字段数量           |
| **HKEYS**   | `HKEYS key`         | 获取所有字段名         |
| **HVALS**   | `HVALS key`         | 获取所有字段值         |
| **HSTRLEN** | `HSTRLEN key field` | 获取字段值的字符串长度 |

**示例：**

bash

```sql
127.0.0.1:6379> HGETALL user:1001
1) "name"
2) "张三"
3) "age"
4) "25"
5) "email"
6) "zhangsan@example.com"

127.0.0.1:6379> HLEN user:1001
(integer) 3

127.0.0.1:6379> HKEYS user:1001
1) "name"
2) "age"
3) "email"

127.0.0.1:6379> HVALS user:1001
1) "张三"
2) "25"
3) "zhangsan@example.com"

127.0.0.1:6379> HSTRLEN user:1001 name
(integer) 2  # "张三"的字节长度
```

### List类型

特征与LinkedList类似：

- 有序
- 元素可重复
- 插入和删除快
- 查询速度一般

#### 一、**基础操作**

#### **1. 添加元素**
| 命令       | 语法                              | 说明                 | 时间复杂度 |
| ---------- | --------------------------------- | -------------------- | ---------- |
| **LPUSH**  | `LPUSH key element [element ...]` | 左侧插入（头部）     | O(1)       |
| **RPUSH**  | `RPUSH key element [element ...]` | 右侧插入（尾部）     | O(1)       |
| **LPUSHX** | `LPUSHX key element`              | 列表存在时才左侧插入 | O(1)       |
| **RPUSHX** | `RPUSHX key element`              | 列表存在时才右侧插入 | O(1)       |

```bash
# 示例
LPUSH tasks "task1" "task2"      # 列表: ["task2", "task1"]
RPUSH tasks "task3"              # 列表: ["task2", "task1", "task3"]
LPUSHX notexist "task"           # 列表不存在，操作无效
```

---

#### **2. 弹出元素**
| 命令      | 语法                          | 说明             | 时间复杂度 |
| --------- | ----------------------------- | ---------------- | ---------- |
| **LPOP**  | `LPOP key [count]`            | 左侧弹出（头部） | O(1)       |
| **RPOP**  | `RPOP key [count]`            | 右侧弹出（尾部） | O(1)       |
| **BLPOP** | `BLPOP key [key ...] timeout` | 阻塞式左侧弹出   | O(1)       |
| **BRPOP** | `BRPOP key [key ...] timeout` | 阻塞式右侧弹出   | O(1)       |

```bash
# 示例
LPOP tasks                      # 弹出 "task2"
RPOP tasks                      # 弹出 "task3"
BLPOP tasks 5                   # 阻塞5秒等待元素
```

---

#### **3. 读取元素**
| 命令       | 语法                    | 说明             | 时间复杂度 |
| ---------- | ----------------------- | ---------------- | ---------- |
| **LRANGE** | `LRANGE key start stop` | 获取指定范围元素 | O(S+N)     |
| **LINDEX** | `LINDEX key index`      | 获取指定位置元素 | O(N)       |
| **LLEN**   | `LLEN key`              | 获取列表长度     | O(1)       |

```bash
# 示例
LRANGE tasks 0 -1               # 获取所有元素
LINDEX tasks 1                  # 获取索引1的元素
LLEN tasks                      # 获取列表长度
```

---

#### 二、**插入/删除操作**

#### **4. 插入元素**
| 命令        | 语法                                      | 说明               | 时间复杂度 |
| ----------- | ----------------------------------------- | ------------------ | ---------- |
| **LINSERT** | `LINSERT key BEFORE\|AFTER pivot element` | 在指定元素前后插入 | O(N)       |

```bash
# 示例
RPUSH list "A" "C"
LINSERT list BEFORE "C" "B"    # 列表: ["A", "B", "C"]
```

---

#### **5. 删除元素**
| 命令      | 语法                     | 说明                   | 时间复杂度 |
| --------- | ------------------------ | ---------------------- | ---------- |
| **LREM**  | `LREM key count element` | 删除指定元素           | O(N)       |
| **LTRIM** | `LTRIM key start stop`   | 修剪列表，保留指定范围 | O(N)       |

```bash
# 示例
RPUSH list "A" "B" "A" "C" "A"
LREM list 2 "A"                # 从左删除2个"A"，列表: ["B", "C", "A"]
LTRIM list 0 1                 # 只保留前2个元素: ["B", "C"]
```

---

#### 三、**修改/阻塞操作**

#### **6. 修改元素**
| 命令     | 语法                     | 说明             | 时间复杂度 |
| -------- | ------------------------ | ---------------- | ---------- |
| **LSET** | `LSET key index element` | 设置指定位置元素 | O(N)       |

```bash
# 示例
LSET list 1 "new_value"        # 修改索引1的元素
```

---

#### **7. 弹出并推送**
| 命令           | 语法                                    | 说明                         | 时间复杂度 |
| -------------- | --------------------------------------- | ---------------------------- | ---------- |
| **RPOPLPUSH**  | `RPOPLPUSH source destination`          | 从源列表弹出，推送到目标列表 | O(1)       |
| **BRPOPLPUSH** | `BRPOPLPUSH source destination timeout` | 阻塞式版本                   | O(1)       |

```bash
# 示例
RPUSH list1 "a" "b" "c"
RPUSH list2 "x" "y"
RPOPLPUSH list1 list2          # list1: ["a", "b"], list2: ["c", "x", "y"]
```

---

#### 四、**快速查询参考**

### **按功能分类：**

#### **添加元素**
```bash
LPUSH key value         # 左边加
RPUSH key value         # 右边加
```

#### **获取元素**
```bash
LRANGE key 0 -1         # 获取全部
LINDEX key 0            # 获取第一个
LLEN key                # 获取长度
```

#### **删除元素**
```bash
LPOP key                # 左边删
RPOP key                # 右边删
LREM key count value    # 删除指定值
```

#### **阻塞操作**
```bash
BLPOP key timeout       # 阻塞弹出（队列）
BRPOP key timeout
```

---

#### 五、**实际应用场景命令**

#### **1. 消息队列**
```bash
# 生产者
LPUSH queue "message1"
LPUSH queue "message2"

# 消费者
BRPOP queue 0           # 0=无限等待
```

### **2. 最新列表**
```bash
# 添加最新文章
LPUSH news:latest "article:1001"
LTRIM news:latest 0 9   # 只保留最新的10条
```

### **3. 任务调度**
```bash
# 添加任务
LPUSH pending_tasks "task1"
LPUSH pending_tasks "task2"

# 处理任务
BRPOPLPUSH pending_tasks processing_tasks 0
```

---

#### 六、**内存优化提示**

```bash
# 长列表使用 LRANGE 分页
LRANGE key 0 9          # 前10条
LRANGE key 10 19        # 第2页

# 及时清理过期数据
LTRIM key 0 99          # 只保留100条
```

---

#### **最常用命令（80%场景）**

```bash
# 1. 压入元素
LPUSH/RPUSH

# 2. 弹出元素  
LPOP/RPOP/BRPOP

# 3. 读取元素
LRANGE/LINDEX/LLEN

# 4. 维护列表
LTRIM/LREM
```

记住这10个命令，就能应对大多数List使用场景。

### Set类型

与HashSet类似：

- 无序
- 元素不可重复
- 查找快
- 支持交集，并集，差集等功能

# Redis Set 类型常用命令

## 一、**基础操作**

### **1. 添加/删除元素**
| 命令     | 语法                           | 说明         | 时间复杂度 |
| -------- | ------------------------------ | ------------ | ---------- |
| **SADD** | `SADD key member [member ...]` | 添加元素     | O(1)       |
| **SREM** | `SREM key member [member ...]` | 删除元素     | O(1)       |
| **SPOP** | `SPOP key [count]`             | 随机弹出元素 | O(1)       |

```bash
# 示例
SADD tags "redis" "database" "cache"     # 添加标签
SREM tags "cache"                        # 删除标签
SPOP tags 2                              # 随机弹出2个元素
```

---

### **2. 查询元素**
| 命令            | 语法                      | 说明         | 时间复杂度 |
| --------------- | ------------------------- | ------------ | ---------- |
| **SMEMBERS**    | `SMEMBERS key`            | 获取所有元素 | O(N)       |
| **SISMEMBER**   | `SISMEMBER key member`    | 判断是否成员 | O(1)       |
| **SRANDMEMBER** | `SRANDMEMBER key [count]` | 随机获取元素 | O(1)       |
| **SCARD**       | `SCARD key`               | 获取元素数量 | O(1)       |

```bash
# 示例
SMEMBERS tags                           # 获取所有标签
SISMEMBER tags "redis"                  # 1=存在，0=不存在
SRANDMEMBER tags 3                      # 随机获取3个元素
SCARD tags                              # 获取标签数量
```

---

## 二、**集合运算**

### **3. 交集**
| 命令            | 语法                                    | 说明             | 时间复杂度 |
| --------------- | --------------------------------------- | ---------------- | ---------- |
| **SINTER**      | `SINTER key [key ...]`                  | 多个集合的交集   | O(N*M)     |
| **SINTERSTORE** | `SINTERSTORE destination key [key ...]` | 交集存储到新集合 | O(N*M)     |

```bash
# 示例
SADD set1 "a" "b" "c"
SADD set2 "b" "c" "d"
SINTER set1 set2                        # 返回 ["b", "c"]
SINTERSTORE set3 set1 set2              # 交集存到set3
```

---

### **4. 并集**
| 命令            | 语法                                    | 说明             | 时间复杂度 |
| --------------- | --------------------------------------- | ---------------- | ---------- |
| **SUNION**      | `SUNION key [key ...]`                  | 多个集合的并集   | O(N)       |
| **SUNIONSTORE** | `SUNIONSTORE destination key [key ...]` | 并集存储到新集合 | O(N)       |

```bash
# 示例
SUNION set1 set2                        # 返回 ["a","b","c","d"]
SUNIONSTORE set4 set1 set2              # 并集存到set4
```

---

### **5. 差集**
| 命令           | 语法                                   | 说明                       | 时间复杂度 |
| -------------- | -------------------------------------- | -------------------------- | ---------- |
| **SDIFF**      | `SDIFF key [key ...]`                  | 第一个集合与其他集合的差集 | O(N)       |
| **SDIFFSTORE** | `SDIFFSTORE destination key [key ...]` | 差集存储到新集合           | O(N)       |

```bash
# 示例
SDIFF set1 set2                         # set1有但set2没有: ["a"]
SDIFF set2 set1                         # set2有但set1没有: ["d"]
```

---

## 三、**移动/扫描**

### **6. 移动元素**
| 命令      | 语法                              | 说明                 | 时间复杂度 |
| --------- | --------------------------------- | -------------------- | ---------- |
| **SMOVE** | `SMOVE source destination member` | 移动元素到另一个集合 | O(1)       |

```bash
# 示例
SMOVE set1 set2 "a"                     # 把"a"从set1移到set2
```

---

### **7. 迭代扫描**
| 命令      | 语法                                             | 说明         | 时间复杂度   |
| --------- | ------------------------------------------------ | ------------ | ------------ |
| **SSCAN** | `SSCAN key cursor [MATCH pattern] [COUNT count]` | 增量迭代集合 | O(1)每次调用 |

```bash
# 示例
SSCAN tags 0 MATCH "r*" COUNT 10       # 迭代匹配"r*"的元素
```

---

## 四、**应用场景命令**

### **1. 标签系统**
```bash
# 给文章打标签
SADD article:1001:tags "tech" "programming" "redis"

# 获取文章标签
SMEMBERS article:1001:tags

# 查找有相同标签的文章
SINTER article:1001:tags article:1002:tags
```

### **2. 好友关系**
```bash
# 添加好友
SADD user:1001:friends "1002" "1003" "1004"

# 共同好友
SINTER user:1001:friends user:1002:friends

# 可能认识的人（好友的好友）
SDIFF user:1002:friends user:1001:friends
SREM user:1002:friends "1001"          # 排除自己
```

### **3. 抽奖/随机推荐**
```bash
# 参与抽奖用户
SADD lottery:users "user1" "user2" "user3" "user4" "user5"

# 随机抽取3名中奖者
SRANDMEMBER lottery:users 3

# 或弹出（不重复抽奖）
SPOP lottery:users 3
```

### **4. 唯一计数器**
```bash
# 记录唯一访客（IP）
SADD daily:visitors:20240101 "192.168.1.1" "192.168.1.2"

# 当日UV
SCARD daily:visitors:20240101

# 连续访问用户（交集）
SINTER daily:visitors:20240101 daily:visitors:20240102
```

### **5. 黑白名单**
```bash
# 黑名单
SADD blacklist:ip "10.0.0.1" "10.0.0.2"
SADD blacklist:user "spammer1" "spammer2"

# 检查是否在黑名单
SISMEMBER blacklist:ip "10.0.0.1"
```

---

## 五、**最常用命令速查**

### **每日必用（8个核心命令）**
```bash
# 增删查改
SADD key member1 member2     # 添加
SREM key member              # 删除
SMEMBERS key                 # 查看所有
SISMEMBER key member         # 是否存在
SCARD key                    # 数量

# 随机操作
SRANDMEMBER key [count]      # 随机取（不删除）
SPOP key [count]             # 随机取（删除）

# 集合运算
SINTER key1 key2             # 交集
SUNION key1 key2             # 并集
SDIFF key1 key2              # 差集
```

---

## 六、**性能注意事项**

```bash
# 1. 大集合慎用 SMEMBERS
# 使用 SSCAN 分批获取
SSCAN key 0 COUNT 100

# 2. 集合运算复杂度高
# 大集合运算可考虑结果缓存
SINTERSTORE cache:result set1 set2
EXPIRE cache:result 60

# 3. 合理使用 SPOP 和 SRANDMEMBER
SPOP key count      # 弹出并删除（抽奖场景）
SRANDMEMBER key count  # 只读不删（推荐场景）
```

---

## 七、**Set 与 List 对比**

| 特性     | Set                 | List             |
| -------- | ------------------- | ---------------- |
| 元素     | 唯一，无序          | 可重复，有序     |
| 查找元素 | O(1)（SISMEMBER）   | O(N)（需遍历）   |
| 随机访问 | O(1)（SRANDMEMBER） | O(N)（LINDEX）   |
| 去重     | 自动                | 需手动           |
| 应用场景 | 标签、好友、抽奖    | 消息队列、时间线 |

---

## **一句话场景指南**

```bash
# 需要去重 → Set
# 需要顺序 → List
# 需要快速查找是否存在 → Set
# 需要排序/分页 → List 或 Sorted Set
# 需要集合运算（交集/并集） → Set
```

记住：**Set = 数学上的集合 = 自动去重 + 无序存储 + 集合运算**

# Redis Sorted Set (ZSet) 类型常用命令

## 一、**基础操作**

### **1. 添加/更新元素**
| 命令        | 语法                                                         | 说明                    | 时间复杂度 |
| ----------- | ------------------------------------------------------------ | ----------------------- | ---------- |
| **ZADD**    | `ZADD key [NX\|XX] [CH] [INCR] score member [score member ...]` | 添加/更新元素（带分数） | O(log N)   |
| **ZINCRBY** | `ZINCRBY key increment member`                               | 增加成员分数            | O(log N)   |

```bash
# 示例
ZADD leaderboard 100 "Alice" 200 "Bob" 150 "Charlie"
ZADD leaderboard NX 120 "David"       # 仅当不存在时添加
ZADD leaderboard XX CH 180 "Alice"    # 仅当存在时更新，返回变更数
ZINCRBY leaderboard 50 "Bob"          # Bob分数增加50
```

**选项说明：**
- `NX`：仅添加新成员
- `XX`：仅更新已存在成员
- `CH`：返回变更数量（新增+更新）
- `INCR`：类似 ZINCRBY

---

### **2. 删除元素**
| 命令                 | 语法                             | 说明             | 时间复杂度 |
| -------------------- | -------------------------------- | ---------------- | ---------- |
| **ZREM**             | `ZREM key member [member ...]`   | 删除指定成员     | O(M*log N) |
| **ZREMRANGEBYRANK**  | `ZREMRANGEBYRANK key start stop` | 按排名范围删除   | O(log N+M) |
| **ZREMRANGEBYSCORE** | `ZREMRANGEBYSCORE key min max`   | 按分数范围删除   | O(log N+M) |
| **ZPOPMAX**          | `ZPOPMAX key [count]`            | 弹出分数最高成员 | O(log N)   |
| **ZPOPMIN**          | `ZPOPMIN key [count]`            | 弹出分数最低成员 | O(log N)   |

```bash
# 示例
ZREM leaderboard "Charlie"                    # 删除Charlie
ZREMRANGEBYRANK leaderboard 0 2               # 删除排名0-2的成员
ZREMRANGEBYSCORE leaderboard 0 100            # 删除分数0-100的成员
ZPOPMAX leaderboard 2                         # 弹出前2名
ZPOPMIN leaderboard                           # 弹出最后1名
```

---

### **3. 查询元素**
| 命令         | 语法                  | 说明                   | 时间复杂度 |
| ------------ | --------------------- | ---------------------- | ---------- |
| **ZSCORE**   | `ZSCORE key member`   | 获取成员分数           | O(1)       |
| **ZCARD**    | `ZCARD key`           | 获取成员数量           | O(1)       |
| **ZCOUNT**   | `ZCOUNT key min max`  | 统计分数区间成员数     | O(log N)   |
| **ZRANK**    | `ZRANK key member`    | 获取成员排名（低到高） | O(log N)   |
| **ZREVRANK** | `ZREVRANK key member` | 获取成员排名（高到低） | O(log N)   |

```bash
# 示例
ZSCORE leaderboard "Alice"                    # 获取Alice分数
ZCARD leaderboard                             # 总成员数
ZCOUNT leaderboard 100 200                    # 分数100-200的成员数
ZRANK leaderboard "Bob"                       # Bob的正向排名（从0开始）
ZREVRANK leaderboard "Bob"                    # Bob的逆向排名（从高到低）
```

---

## 二、**范围查询**

### **4. 按排名查询**
| 命令            | 语法                                           | 说明           | 时间复杂度 |
| --------------- | ---------------------------------------------- | -------------- | ---------- |
| **ZRANGE**      | `ZRANGE key start stop [WITHSCORES]`           | 按排名升序查询 | O(log N+M) |
| **ZREVRANGE**   | `ZREVRANGE key start stop [WITHSCORES]`        | 按排名降序查询 | O(log N+M) |
| **ZRANGEBYLEX** | `ZRANGEBYLEX key min max [LIMIT offset count]` | 按字典序查询   | O(log N+M) |

```bash
# 示例
ZRANGE leaderboard 0 2                        # 前3名（分数低到高）
ZRANGE leaderboard 0 -1 WITHSCORES            # 所有成员带分数
ZREVRANGE leaderboard 0 2 WITHSCORES          # 前3名（分数高到低）
ZRANGEBYLEX names [A [Z                       # 名字A-Z的成员
```

---

### **5. 按分数查询**
| 命令                 | 语法                                                         | 说明           | 时间复杂度 |
| -------------------- | ------------------------------------------------------------ | -------------- | ---------- |
| **ZRANGEBYSCORE**    | `ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]` | 按分数升序查询 | O(log N+M) |
| **ZREVRANGEBYSCORE** | `ZREVRANGEBYSCORE key max min [WITHSCORES] [LIMIT offset count]` | 按分数降序查询 | O(log N+M) |

```bash
# 示例
ZRANGEBYSCORE leaderboard 100 200             # 分数100-200的成员
ZRANGEBYSCORE leaderboard 100 (200            # 100≤分数<200
ZRANGEBYSCORE leaderboard -inf +inf           # 所有分数
ZRANGEBYSCORE leaderboard 100 200 LIMIT 0 5   # 前5个
ZREVRANGEBYSCORE leaderboard 200 100          # 分数200-100的成员（降序）
```

**区间表示法：**
- `(100`：大于100（不包含100）
- `100`：大于等于100
- `-inf`：负无穷
- `+inf`：正无穷

---

## 三、**集合运算**

### **6. 聚合运算**
| 命令            | 语法                                                         | 说明     | 时间复杂度        |
| --------------- | ------------------------------------------------------------ | -------- | ----------------- |
| **ZUNIONSTORE** | `ZUNIONSTORE destination numkeys key [key ...] [WEIGHTS weight] [AGGREGATE SUM\|MIN\|MAX]` | 并集存储 | O(N)+O(M log M)   |
| **ZINTERSTORE** | `ZINTERSTORE destination numkeys key [key ...] [WEIGHTS weight] [AGGREGATE SUM\|MIN\|MAX]` | 交集存储 | O(N*K)+O(M log M) |

```bash
# 示例
ZADD set1 1 "A" 2 "B"
ZADD set2 3 "A" 4 "C"
ZUNIONSTORE union 2 set1 set2 WEIGHTS 1 2 AGGREGATE SUM  # 并集，分数相加
ZINTERSTORE inter 2 set1 set2                            # 交集，默认SUM
```

**AGGREGATE选项：**
- `SUM`：分数相加（默认）
- `MIN`：取最小分数
- `MAX`：取最大分数

---

## 四、**迭代扫描**

### **7. 增量迭代**
| 命令      | 语法                                             | 说明     | 时间复杂度   |
| --------- | ------------------------------------------------ | -------- | ------------ |
| **ZSCAN** | `ZSCAN key cursor [MATCH pattern] [COUNT count]` | 增量迭代 | O(1)每次调用 |

```bash
# 示例
ZSCAN leaderboard 0 MATCH "A*" COUNT 10       # 迭代名字以A开头的成员
```

---

## 五、**应用场景命令**

### **1. 排行榜系统**
```bash
# 玩家得分
ZADD leaderboard 2500 "player1" 1800 "player2" 3200 "player3"

# 获取前10名
ZREVRANGE leaderboard 0 9 WITHSCORES

# 获取玩家排名
ZREVRANK leaderboard "player1"                # 第几名（从0开始）
ZSCORE leaderboard "player1"                  # 分数

# 增加分数
ZINCRBY leaderboard 100 "player2"

# 获取区间排名（第11-20名）
ZREVRANGE leaderboard 10 19 WITHSCORES
```

### **2. 延迟队列**
```bash
# 添加延迟任务（时间戳为分数）
ZADD delay_queue 1640995200 "task1"  # 2022-1-1执行
ZADD delay_queue 1641081600 "task2"  # 2022-1-2执行

# 获取已到期的任务
ZRANGEBYSCORE delay_queue 0 1640995200

# 处理并删除任务
ZREMRANGEBYSCORE delay_queue 0 1640995200
```

### **3. 时间线/新闻feed**
```bash
# 用户发布动态（时间戳为分数）
ZADD user:1001:timeline 1640995200 "post:1001"
ZADD user:1001:timeline 1641081600 "post:1002"

# 获取最新动态
ZREVRANGE user:1001:timeline 0 9

# 分页获取
ZREVRANGE user:1001:timeline 10 19
```

### **4. 带权重的标签系统**
```bash
# 文章标签热度（点击数为分数）
ZADD article:1001:tags 150 "redis" 80 "database" 200 "cache"

# 获取热门标签
ZREVRANGE article:1001:tags 0 4

# 合并多篇文章的标签热度
ZUNIONSTORE hot_tags 2 article:1001:tags article:1002:tags
ZREVRANGE hot_tags 0 9 WITHSCORES
```

### **5. 实时竞价系统**
```bash
# 广告竞价（出价为分数）
ZADD ad:slot1 5.2 "advertiser1" 4.8 "advertiser2" 6.0 "advertiser3"

# 获取最高出价
ZREVRANGE ad:slot1 0 0 WITHSCORES

# 获取前3名出价
ZREVRANGE ad:slot1 0 2 WITHSCORES

# 更新出价
ZADD ad:slot1 XX CH 5.5 "advertiser2"
```

---

## 六、**最常用命令速查（10个核心命令）**

```bash
# 1. 增删改查
ZADD key score member              # 添加/更新
ZREM key member                    # 删除
ZSCORE key member                  # 查分数
ZCARD key                          # 查总数

# 2. 排名相关
ZRANK key member                   # 正序排名
ZREVRANK key member                # 倒序排名
ZRANGE key 0 -1 WITHSCORES         # 正序范围
ZREVRANGE key 0 -1 WITHSCORES      # 倒序范围

# 3. 分数范围
ZRANGEBYSCORE key min max          # 按分数查询
ZCOUNT key min max                 # 分数区间统计
```

---

## 七、**性能注意事项**

```bash
# 1. 大范围查询优化
# 使用 LIMIT 分页
ZREVRANGE leaderboard 0 99              # 前100名（不是所有）

# 2. 避免 ZRANGE 0 -1 在大集合上
# 使用 ZSCAN 分批
ZSCAN key 0 COUNT 100

# 3. ZUNIONSTORE/ZINTERSTORE 可能很重
# 考虑结果缓存
ZINTERSTORE cache:result 2 set1 set2
EXPIRE cache:result 60

# 4. 排名更新的代价
# ZADD 更新分数会导致排名重排 O(log N)
```

---

## 八、**Sorted Set 特性总结**

### **核心特性：**
1. **成员唯一**，分数可重复
2. **按分数排序**，分数相同时按字典序
3. **支持范围查询**（排名、分数、字典序）
4. **支持集合运算**（并集、交集）

### **适用场景：**
- ✅ 需要排序的集合
- ✅ 需要范围查询
- ✅ 需要按权重/分数排序
- ✅ 需要排名统计

### **不适用场景：**
- ❌ 需要重复成员的集合（用 List）
- ❌ 只需要简单去重，不需要排序（用 Set）
- ❌ 需要存储复杂对象（用 Hash）

---

## **一句话选择指南**

```bash
# 问：用什么数据结构？
# 答：
# 要排序 + 去重 → Sorted Set
# 只要去重，不要排序 → Set
# 要顺序，可重复 → List
# 要快速查单个值 → Hash
# 只要简单键值对 → String
```

**记住**：Sorted Set = **Set的去重** + **List的顺序** + **自定义分数排序**

# Jedis

### Jedis连接池

```java
public class JedisConnectionFactory{
    private static final JedisPool jedisPool;
    static {
        JedisPoolConfig jedisisPoolConfig = new JedisPoolConfig();
        //最大连接
        jedisPoolConfig.setMaxTotal(8);
        //最大空闲连接
        jedisPoolConfig.setMaxIdle(8);
        //最小空闲连接
        jedisPoolConfig.setMinIdle(0);
        ///设置最长等待时间
        jedisPoolConfig.setMaxWaitMillis(200);
        jedisPool=new JedisPool(jedisPoolConfig,"ip地址",端口号,超时时间，密码);
        
    }
    //获取Jedis对象
    public static Jedis getJedis(){
        return jedisPool.getResource();
    }
}
```

# SpringDataRedis

1.引入依赖

```
引入redis依赖
引入连接池依赖common-pool
```

2.配置文件

```yaml
spring:
	redis:
		host: ip地址
		port: 端口号
		password: 密码
		lettuce:
			pool:
				max-active: 8
				max-idle: 8
				min-idle: 0
				max-wait: 100ms
```

3.实现

```java
@Autowired
private RedisTemplate redisTemplate;
//写入一条String数据
redisTempate.opsForValue().set("name","zhangsan");
//获取String数据
Object name =redisTempate.opsForValue().get("name");
System.out.prinln("name="+ name);
```

# RedisTemplate 自定义序列化配置

## 一、**默认序列化问题**

Spring Boot 默认的 RedisTemplate 使用 **JdkSerializationRedisSerializer**，会导致：
1. Redis 中存储的是二进制数据，不可读
2. 键有乱码前缀：`\xac\xed\x00\x05t\x00\x04name`
3. 值不可读，不利于调试

---

## 二、**配置自定义序列化**

### **方案1：配置类方式（推荐）**

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.*;

@Configuration
public class RedisConfig {
    
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory connectionFactory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(connectionFactory);
        
        // 使用 StringRedisSerializer 序列化和反序列化 key
        StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();
        template.setKeySerializer(stringRedisSerializer);
        template.setHashKeySerializer(stringRedisSerializer);
        
        // 使用 GenericJackson2JsonRedisSerializer 序列化和反序列化 value
        GenericJackson2JsonRedisSerializer jsonRedisSerializer = new GenericJackson2JsonRedisSerializer();
        template.setValueSerializer(jsonRedisSerializer);
        template.setHashValueSerializer(jsonRedisSerializer);
        
        template.afterPropertiesSet();
        return template;
    }
}
```

---

### **方案2：使用 StringRedisTemplate（简单场景）**

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.data.redis.core.ValueOperations;

@Service
public class RedisService {
    
    @Autowired
    private StringRedisTemplate stringRedisTemplate;  // 专门处理字符串
    
    public void test() {
        // 写入字符串
        stringRedisTemplate.opsForValue().set("name", "zhangsan");
        
        // 读取字符串
        String name = stringRedisTemplate.opsForValue().get("name");
        System.out.println("name=" + name);
        
        // 存储对象（需要自己序列化）
        User user = new User("张三", 25);
        String userJson = objectMapper.writeValueAsString(user);
        stringRedisTemplate.opsForValue().set("user:1001", userJson);
    }
}
```

---

### **方案3：混合序列化（Key用String，Value用JSON）**

```java
@Configuration
public class RedisConfig {
    
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory connectionFactory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(connectionFactory);
        
        // Key 的序列化方式
        template.setKeySerializer(new StringRedisSerializer());
        template.setHashKeySerializer(new StringRedisSerializer());
        
        // Value 的序列化方式
        Jackson2JsonRedisSerializer<Object> jsonSerializer = new Jackson2JsonRedisSerializer<>(Object.class);
        
        // 解决 Jackson 反序列化问题
        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        objectMapper.activateDefaultTyping(
            objectMapper.getPolymorphicTypeValidator(),
            ObjectMapper.DefaultTyping.NON_FINAL
        );
        jsonSerializer.setObjectMapper(objectMapper);
        
        template.setValueSerializer(jsonSerializer);
        template.setHashValueSerializer(jsonSerializer);
        
        template.afterPropertiesSet();
        return template;
    }
}
```

---

## 三、**完整示例代码**

### **1. 配置类**
```java
import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.jsontype.impl.LaissezFaireSubTypeValidator;
import org.springframework.cache.annotation.CachingConfigurerSupport;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

@Configuration
public class RedisConfig extends CachingConfigurerSupport {
    
    @Bean
    @SuppressWarnings("all")
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(factory);
        
        // JSON 序列化配置
        Jackson2JsonRedisSerializer<Object> jackson2JsonRedisSerializer = 
            new Jackson2JsonRedisSerializer<>(Object.class);
        
        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        
        // 启用默认类型信息，解决反序列化问题
        objectMapper.activateDefaultTyping(
            LaissezFaireSubTypeValidator.instance,
            ObjectMapper.DefaultTyping.NON_FINAL
        );
        
        jackson2JsonRedisSerializer.setObjectMapper(objectMapper);
        
        // String 序列化配置
        StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();
        
        // Key 采用 String 序列化
        template.setKeySerializer(stringRedisSerializer);
        template.setHashKeySerializer(stringRedisSerializer);
        
        // Value 采用 Jackson 序列化
        template.setValueSerializer(jackson2JsonRedisSerializer);
        template.setHashValueSerializer(jackson2JsonRedisSerializer);
        
        template.afterPropertiesSet();
        return template;
    }
}
```

### **2. Service 使用示例**
```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.stereotype.Service;
import java.util.concurrent.TimeUnit;

@Service
public class UserService {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    // 存储简单字符串
    public void saveString() {
        redisTemplate.opsForValue().set("name", "zhangsan");
        
        // 带过期时间
        redisTemplate.opsForValue().set("token", "abc123", 30, TimeUnit.MINUTES);
    }
    
    // 读取字符串
    public void getString() {
        Object name = redisTemplate.opsForValue().get("name");
        System.out.println("name=" + name);
    }
    
    // 存储对象
    public void saveUser() {
        User user = new User("张三", 25, "zhangsan@example.com");
        redisTemplate.opsForValue().set("user:1001", user);
    }
    
    // 读取对象
    public void getUser() {
        User user = (User) redisTemplate.opsForValue().get("user:1001");
        System.out.println("user=" + user);
    }
    
    // 存储Hash
    public void saveUserHash() {
        Map<String, Object> userMap = new HashMap<>();
        userMap.put("name", "李四");
        userMap.put("age", 30);
        userMap.put("email", "lisi@example.com");
        
        redisTemplate.opsForHash().putAll("user:hash:1002", userMap);
    }
    
    // 获取Hash字段
    public void getUserHash() {
        String name = (String) redisTemplate.opsForHash().get("user:hash:1002", "name");
        Integer age = (Integer) redisTemplate.opsForHash().get("user:hash:1002", "age");
        System.out.println("name=" + name + ", age=" + age);
    }
    
    // 存储List
    public void saveList() {
        List<String> list = Arrays.asList("item1", "item2", "item3");
        redisTemplate.opsForList().rightPushAll("mylist", list);
    }
    
    // 存储Set
    public void saveSet() {
        Set<String> set = new HashSet<>(Arrays.asList("tag1", "tag2", "tag3"));
        redisTemplate.opsForSet().add("tags", set.toArray());
    }
    
    // 存储Sorted Set
    public void saveSortedSet() {
        redisTemplate.opsForZSet().add("leaderboard", "player1", 100);
        redisTemplate.opsForZSet().add("leaderboard", "player2", 200);
        redisTemplate.opsForZSet().add("leaderboard", "player3", 150);
    }
}

// 实体类
@Data
@AllArgsConstructor
@NoArgsConstructor
class User implements Serializable {
    private String name;
    private Integer age;
    private String email;
}
```

---

## 四、**不同序列化器对比**

| 序列化器                               | 优点                   | 缺点               | 适用场景            |
| -------------------------------------- | ---------------------- | ------------------ | ------------------- |
| **StringRedisSerializer**              | 可读性好，性能高       | 只能序列化字符串   | Key序列化，简单值   |
| **Jackson2JsonRedisSerializer**        | 可读性好，支持复杂对象 | 需要处理类型信息   | Value序列化（对象） |
| **GenericJackson2JsonRedisSerializer** | 自动处理类型信息       | 占用空间稍大       | Value序列化（推荐） |
| **JdkSerializationRedisSerializer**    | Java原生，无需配置     | 不可读，二进制格式 | 兼容旧系统          |
| **GenericToStringSerializer**          | 通用字符串转换         | 需要类型转换器     | 特定类型转换        |

---

## 五、**常见问题解决**

### **1. 反序列化报错：java.lang.ClassCastException**
```java
// 原因：存储和读取的类型不一致
// 解决方案：使用 GenericJackson2JsonRedisSerializer 或指定类型

// 明确指定类型
public <T> T getObject(String key, Class<T> clazz) {
    Object value = redisTemplate.opsForValue().get(key);
    return objectMapper.convertValue(value, clazz);
}
```

### **2. 存储后 Redis 中还是乱码**
检查序列化配置：
```java
// 确保设置了序列化器
template.setKeySerializer(new StringRedisSerializer());    // ✅
template.setValueSerializer(new Jackson2JsonRedisSerializer<>(Object.class));  // ✅
```

### **3. 存储对象缺少类型信息**
```java
// 在 ObjectMapper 中启用默认类型
objectMapper.activateDefaultTyping(
    LaissezFaireSubTypeValidator.instance,
    ObjectMapper.DefaultTyping.NON_FINAL
);
```

### **4. 性能优化配置**
```java
@Bean
public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
    RedisTemplate<String, Object> template = new RedisTemplate<>();
    template.setConnectionFactory(factory);
    
    // 开启事务支持（根据需求）
    template.setEnableTransactionSupport(true);
    
    // 禁用连接池（某些场景）
    // template.setEnableDefaultSerializer(false);
    
    // 设置序列化
    template.setKeySerializer(new StringRedisSerializer());
    template.setValueSerializer(new GenericJackson2JsonRedisSerializer());
    
    return template;
}
```

---

## 六、**最佳实践建议**

### **1. Key 的命名规范**
```java
// 使用冒号分隔，形成层级结构
String userKey = "user:1001:info";
String orderKey = "order:20240101:1001";
String cacheKey = "cache:product:list:page1";
```

### **2. 统一工具类**
```java
@Component
public class RedisUtil {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    // 设置缓存
    public void set(String key, Object value, long time) {
        redisTemplate.opsForValue().set(key, value, time, TimeUnit.SECONDS);
    }
    
    // 获取缓存
    public <T> T get(String key, Class<T> clazz) {
        Object value = redisTemplate.opsForValue().get(key);
        return value == null ? null : clazz.cast(value);
    }
    
    // 删除缓存
    public Boolean delete(String key) {
        return redisTemplate.delete(key);
    }
    
    // 设置过期时间
    public Boolean expire(String key, long time) {
        return redisTemplate.expire(key, time, TimeUnit.SECONDS);
    }
}
```

### **3. 使用缓存注解（Spring Cache）**
```java
@Configuration
@EnableCaching
public class CacheConfig {
    
    @Bean
    public CacheManager cacheManager(RedisConnectionFactory factory) {
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
            .entryTtl(Duration.ofMinutes(10))  // 默认过期时间
            .serializeKeysWith(RedisSerializationContext.SerializationPair
                .fromSerializer(new StringRedisSerializer()))
            .serializeValuesWith(RedisSerializationContext.SerializationPair
                .fromSerializer(new GenericJackson2JsonRedisSerializer()))
            .disableCachingNullValues();
        
        return RedisCacheManager.builder(factory)
            .cacheDefaults(config)
            .build();
    }
}

// 在Service中使用
@Service
public class UserService {
    
    @Cacheable(value = "user", key = "#id")
    public User getUserById(Long id) {
        // 从数据库查询
        return userRepository.findById(id);
    }
    
    @CachePut(value = "user", key = "#user.id")
    public User updateUser(User user) {
        // 更新数据库
        return userRepository.save(user);
    }
    
    @CacheEvict(value = "user", key = "#id")
    public void deleteUser(Long id) {
        userRepository.deleteById(id);
    }
}
```

---

## **总结**

1. **Key 一律用 `StringRedisSerializer`** - 保证可读性
2. **Value 推荐用 `GenericJackson2JsonRedisSerializer`** - 支持对象存储
3. **配置一次，全局生效** - 通过 @Configuration 配置类
4. **区分使用场景** - 简单字符串用 StringRedisTemplate，对象用 RedisTemplate

**你的代码修正后：**
```java
@Service
public class YourService {
    
    // 使用配置好的 RedisTemplate
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    public void test() {
        // 写入数据（键值都可读）
        redisTemplate.opsForValue().set("name", "zhangsan");
        
        // 获取数据
        Object name = redisTemplate.opsForValue().get("name");
        System.out.println("name=" + name);
        
        // 存储对象
        User user = new User("张三", 25);
        redisTemplate.opsForValue().set("user:1001", user);
    }
}
```

# Redis企业实战

## 短信登录

# 什么是缓存

**缓存**就是数据交换的缓冲区（Cache），是存储数据的。临时地方，一般读写性能较高。

### 缓存穿透

**缓存穿透**是指客户端请求的数据砸死缓存中和数据库中都不存在，这样缓存永远不会生效，这些请求就会达到数据库。

常见的解决方案两种：

缓存空对象：

优点：实现简单，维护方便

缺点：额外的内存消耗，可能短时期的不一致

布隆过滤：

优点：内存占用少，没有多余的key

缺点：实现复杂。存在误判可能

我来详细解释一下**Redis缓存穿透**以及解决方案。

## 什么是缓存穿透？

**缓存穿透**是指查询一个**根本不存在的数据**，这个数据在缓存和数据库中都没有。由于缓存不命中，请求会直接穿透到数据库，如果有大量这样的请求，可能会给数据库带来巨大压力甚至导致宕机。

## 典型场景

```java
// 伪代码示例
public String getData(String key) {
    // 1. 先查缓存
    String data = redis.get(key);
    
    if (data != null) {
        return data;  // 缓存命中
    }
    
    // 2. 缓存未命中，查询数据库
    data = db.query(key);
    
    if (data != null) {
        // 3. 数据库有数据，写入缓存
        redis.set(key, data, ttl);
    }
    
    return data;  // 如果数据不存在，每次都会查数据库
}
```

## 产生原因

1. **恶意攻击**：攻击者故意请求不存在的数据
2. **业务误操作**：错误地请求了无效的ID
3. **数据过期**：数据刚好在缓存中过期，同时被大量请求

## 解决方案

### 方案1：缓存空对象（Bloom Filter的简化版）
```java
public String getData(String key) {
    // 1. 先查缓存
    String data = redis.get(key);
    
    if (data != null) {
        // 注意：这里需要区分是真实数据还是空标记
        if ("NULL_OBJECT".equals(data)) {
            return null;  // 知道是不存在的数据
        }
        return data;
    }
    
    // 2. 查询数据库
    data = db.query(key);
    
    if (data != null) {
        // 3. 数据库有数据，写入缓存
        redis.set(key, data, ttl);
    } else {
        // 4. 数据库没有，缓存空对象（设置较短的过期时间）
        redis.set(key, "NULL_OBJECT", 300);  // 5分钟
    }
    
    return data;
}
```

### 方案2：布隆过滤器（Bloom Filter）
**推荐方案**，适合大规模数据场景。

```java
// 使用Guava的BloomFilter
import com.google.common.hash.BloomFilter;
import com.google.common.hash.Funnels;

public class BloomFilterDemo {
    private BloomFilter<String> bloomFilter;
    
    public void init() {
        // 预期插入100万个元素，误判率0.1%
        bloomFilter = BloomFilter.create(
            Funnels.stringFunnel(Charset.defaultCharset()),
            1000000,
            0.001
        );
        
        // 初始化时加载所有存在的key
        List<String> allKeys = db.getAllKeys();
        for (String key : allKeys) {
            bloomFilter.put(key);
        }
    }
    
    public String getData(String key) {
        // 1. 先检查布隆过滤器
        if (!bloomFilter.mightContain(key)) {
            return null;  // 肯定不存在
        }
        
        // 2. 查询缓存和数据库（同前）
        // ...
    }
}
```

### 方案3：接口层校验
```java
public String getData(String key) {
    // 1. 参数校验
    if (!isValidKey(key)) {
        return null;
    }
    
    // 2. 检查ID格式（如必须是数字）
    if (!key.matches("\\d+")) {
        return null;
    }
    
    // 3. 检查ID范围
    long id = Long.parseLong(key);
    if (id <= 0 || id > MAX_ID) {
        return null;
    }
    
    // 后续逻辑...
}
```

### 方案4：互斥锁（应对瞬时大量请求）
```java
public String getDataWithLock(String key) {
    String data = redis.get(key);
    if (data != null) {
        return data;
    }
    
    // 尝试获取分布式锁
    String lockKey = "lock:" + key;
    boolean locked = redis.setnx(lockKey, "1", 10);  // 10秒过期
    
    if (locked) {
        try {
            // 查询数据库
            data = db.query(key);
            
            if (data != null) {
                redis.set(key, data, ttl);
            } else {
                // 缓存空值
                redis.set(key, "NULL_OBJECT", 300);
            }
        } finally {
            // 释放锁
            redis.del(lockKey);
        }
    } else {
        // 等待并重试
        Thread.sleep(100);
        return getDataWithLock(key);
    }
    
    return data;
}
```

## 完整的最佳实践方案

```java
@Component
public class CacheService {
    
    @Resource
    private RedisTemplate redisTemplate;
    
    @Resource
    private BloomFilter bloomFilter;
    
    // 缓存空值的TTL（较短）
    private static final int NULL_CACHE_TTL = 300;
    
    // 正常数据的TTL
    private static final int NORMAL_CACHE_TTL = 3600;
    
    public Object getData(String key) {
        // 1. 参数校验
        if (StringUtils.isBlank(key)) {
            return null;
        }
        
        // 2. 布隆过滤器检查
        if (!bloomFilter.mightContain(key)) {
            // 可以记录日志，用于监控
            log.warn("BloomFilter reject key: {}", key);
            return null;
        }
        
        // 3. 查询缓存
        Object value = redisTemplate.opsForValue().get(key);
        
        if (value != null) {
            if (isNullValue(value)) {
                return null;  // 缓存了空值
            }
            return value;
        }
        
        // 4. 获取分布式锁，防止缓存击穿
        String lockKey = "lock:" + key;
        RLock lock = redissonClient.getLock(lockKey);
        
        try {
            // 尝试加锁，等待100ms，锁有效期30秒
            boolean locked = lock.tryLock(100, 30000, TimeUnit.MILLISECONDS);
            
            if (locked) {
                // 双重检查
                value = redisTemplate.opsForValue().get(key);
                if (value != null) {
                    return isNullValue(value) ? null : value;
                }
                
                // 查询数据库
                value = queryFromDatabase(key);
                
                if (value != null) {
                    // 缓存正常数据
                    redisTemplate.opsForValue().set(
                        key, 
                        value, 
                        NORMAL_CACHE_TTL + randomDelta(),  // 加随机值避免雪崩
                        TimeUnit.SECONDS
                    );
                    
                    // 更新布隆过滤器
                    bloomFilter.put(key);
                } else {
                    // 缓存空值（短期）
                    redisTemplate.opsForValue().set(
                        key, 
                        NULL_VALUE,
                        NULL_CACHE_TTL,
                        TimeUnit.SECONDS
                    );
                }
                
                return value;
            } else {
                // 获取锁失败，等待后重试
                Thread.sleep(50);
                return getData(key);
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            return null;
        } finally {
            if (lock.isHeldByCurrentThread()) {
                lock.unlock();
            }
        }
    }
    
    // 添加随机过期时间，避免缓存雪崩
    private int randomDelta() {
        return new Random().nextInt(300);  // 0-300秒随机
    }
}
```

## 监控和告警

```yaml
# 建议监控以下指标：
监控指标:
  - redis.cache.miss.rate      # 缓存未命中率
  - db.qps                     # 数据库查询QPS
  - bloom.filter.reject.count  # 布隆过滤器拦截次数
  - null.cache.count          # 空值缓存数量
  
告警规则:
  - 缓存未命中率 > 20% 持续5分钟
  - 数据库QPS突增300%
  - 布隆过滤器拦截次数异常增加
```

## 总结

| 方案       | 优点             | 缺点                 | 适用场景             |
| ---------- | ---------------- | -------------------- | -------------------- |
| 缓存空值   | 实现简单         | 内存浪费，数据不一致 | 数据量小，key有限    |
| 布隆过滤器 | 内存效率高       | 有误判率，不支持删除 | 大规模数据，只读场景 |
| 接口校验   | 完全避免无效请求 | 校验规则可能不完善   | 参数有明显规律       |
| 互斥锁     | 防止瞬时大量请求 | 实现复杂，可能阻塞   | 高并发场景           |

**推荐组合方案**：布隆过滤器 + 缓存空值 + 接口校验 + 监控告警

# Redis缓存雪崩详解及解决方案

## 什么是缓存雪崩？

**缓存雪崩**是指**大量缓存数据在同一时间过期**，导致所有请求直接打到数据库，造成数据库瞬时压力激增甚至宕机。

## 典型场景

```java
// 错误的实现：大量key同时过期
public void initCache() {
    List<Product> products = db.getAllProducts();
    
    for (Product product : products) {
        // 所有key设置相同的过期时间（比如1小时）
        redis.set("product:" + product.getId(), 
                  product, 
                  3600);  // 问题：1小时后同时失效
    }
}
```

## 雪崩的严重性

```
正常情况：
请求 → Redis（命中90%） → 数据库（10%）

雪崩发生：
请求 → Redis（全部失效） → 数据库（100%！）
                        ↓
                    数据库压力剧增
                        ↓
                    数据库宕机
                        ↓
                    整个系统瘫痪
```

## 解决方案

### 方案1：过期时间随机化（最常用）
```java
public class CacheService {
    
    // 基础过期时间
    private static final int BASE_TTL = 3600; // 1小时
    
    // 随机偏移范围
    private static final int RANDOM_RANGE = 300; // 5分钟
    
    public void setData(String key, Object value) {
        // 添加随机过期时间，避免同时失效
        int randomTTL = BASE_TTL + getRandomOffset();
        
        redisTemplate.opsForValue().set(
            key, 
            value, 
            randomTTL, 
            TimeUnit.SECONDS
        );
    }
    
    private int getRandomOffset() {
        // 生成 -RANDOM_RANGE 到 +RANDOM_RANGE 的随机数
        return ThreadLocalRandom.current()
                .nextInt(-RANDOM_RANGE, RANDOM_RANGE + 1);
    }
}
```

### 方案2：热点数据永不过期 + 异步刷新
```java
@Component
public class HotDataCacheManager {
    
    @Resource
    private RedisTemplate redisTemplate;
    
    @Resource
    private ThreadPoolExecutor cacheRefreshExecutor;
    
    // 热点数据缓存（永不过期）
    public void cacheHotData(String key, Object value) {
        // 1. 设置永不过期
        redisTemplate.opsForValue().set(key, value);
        
        // 2. 启动异步刷新任务
        scheduleCacheRefresh(key);
    }
    
    private void scheduleCacheRefresh(String key) {
        // 在过期前刷新缓存
        cacheRefreshExecutor.execute(() -> {
            try {
                // 在过期前30分钟开始刷新
                Thread.sleep((BASE_TTL - 1800) * 1000L);
                
                // 异步加载新数据
                Object newData = loadDataFromDB(key);
                if (newData != null) {
                    redisTemplate.opsForValue().set(key, newData);
                    
                    // 重新调度下一次刷新
                    scheduleCacheRefresh(key);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });
    }
}
```

### 方案3：多级缓存架构
```java
@Component
public class MultiLevelCache {
    
    // 一级缓存：本地缓存（Caffeine）
    private Cache<String, Object> localCache = Caffeine.newBuilder()
            .maximumSize(10000)
            .expireAfterWrite(10, TimeUnit.MINUTES)
            .build();
    
    // 二级缓存：Redis
    @Resource
    private RedisTemplate redisTemplate;
    
    public Object getData(String key) {
        // 1. 查本地缓存
        Object value = localCache.getIfPresent(key);
        if (value != null) {
            return value;
        }
        
        // 2. 查Redis
        value = redisTemplate.opsForValue().get(key);
        if (value != null) {
            // 回写到本地缓存
            localCache.put(key, value);
            return value;
        }
        
        // 3. 查数据库（加锁防止缓存击穿）
        value = getFromDBWithLock(key);
        
        if (value != null) {
            // 写入多级缓存
            localCache.put(key, value);
            redisTemplate.opsForValue().set(
                key, 
                value, 
                getRandomTTL(), 
                TimeUnit.SECONDS
            );
        }
        
        return value;
    }
}
```

### 方案4：熔断降级与限流
```java
@Component
public class CircuitBreakerCacheService {
    
    @Resource
    private RedisTemplate redisTemplate;
    
    // 使用Resilience4j熔断器
    private CircuitBreaker circuitBreaker;
    
    // 使用Sentinel限流
    @SentinelResource(
        value = "getData", 
        fallback = "getDataFallback",
        blockHandler = "handleFlowLimit"
    )
    public Object getData(String key) {
        return circuitBreaker.executeSupplier(() -> {
            // 正常的缓存查询逻辑
            Object value = redisTemplate.opsForValue().get(key);
            if (value == null) {
                value = queryFromDatabase(key);
            }
            return value;
        });
    }
    
    // 降级方法：返回默认值或缓存旧数据
    public Object getDataFallback(String key, Throwable t) {
        log.warn("触发降级，key: {}", key, t);
        
        // 1. 返回默认值
        // return getDefaultData();
        
        // 2. 返回过期的缓存数据（如果有）
        Object staleData = getStaleDataFromBackup(key);
        if (staleData != null) {
            return staleData;
        }
        
        // 3. 返回友好的错误提示
        return new ErrorResponse("系统繁忙，请稍后重试");
    }
    
    // 限流处理
    public Object handleFlowLimit(String key, BlockException ex) {
        log.warn("触发限流，key: {}", key);
        return new ErrorResponse("请求过于频繁，请稍后重试");
    }
}
```

### 方案5：缓存预热与定时刷新
```java
@Component
public class CacheWarmUpService {
    
    @Resource
    private RedisTemplate redisTemplate;
    
    // 应用启动时预热
    @PostConstruct
    public void warmUpOnStartup() {
        log.info("开始缓存预热...");
        
        // 预热热点数据
        warmUpHotData();
        
        // 预热常用查询
        warmUpFrequentQueries();
    }
    
    // 定时刷新缓存（错峰执行）
    @Scheduled(cron = "0 0 3 * * ?")  // 每天凌晨3点
    public void scheduledRefresh() {
        log.info("定时刷新缓存开始...");
        
        // 分批次刷新，避免一次性刷新所有
        refreshInBatches(100);  // 每批100个key
    }
    
    private void refreshInBatches(int batchSize) {
        List<String> allKeys = getAllCacheKeys();
        
        for (int i = 0; i < allKeys.size(); i += batchSize) {
            List<String> batch = allKeys.subList(i, 
                Math.min(i + batchSize, allKeys.size()));
            
            // 分批异步刷新
            refreshBatchAsync(batch);
            
            // 批次间延时
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }
}
```

### 方案6：数据库保护机制
```java
@Component
public class DatabaseProtection {
    
    // 使用HikariCP连接池配置
    @Bean
    public DataSource dataSource() {
        HikariConfig config = new HikariConfig();
        config.setMaximumPoolSize(50);      // 最大连接数
        config.setMinimumIdle(10);          // 最小空闲连接
        config.setConnectionTimeout(3000);  // 连接超时3秒
        config.setIdleTimeout(600000);      // 空闲连接超时10分钟
        config.setMaxLifetime(1800000);     // 连接最大生命周期30分钟
        
        // 保护配置
        config.addDataSourceProperty("prepStmtCacheSize", 250);
        config.addDataSourceProperty("prepStmtCacheSqlLimit", 2048);
        config.addDataSourceProperty("useServerPrepStmts", true);
        config.addDataSourceProperty("cachePrepStmts", true);
        
        return new HikariDataSource(config);
    }
    
    // 使用MyBatis拦截器进行慢SQL监控
    @Intercepts({
        @Signature(type = Executor.class, method = "query", 
                   args = {MappedStatement.class, Object.class, 
                           RowBounds.class, ResultHandler.class})
    })
    public class SlowSqlInterceptor implements Interceptor {
        
        private static final long SLOW_SQL_THRESHOLD = 1000; // 1秒
        
        @Override
        public Object intercept(Invocation invocation) throws Throwable {
            long start = System.currentTimeMillis();
            
            try {
                return invocation.proceed();
            } finally {
                long duration = System.currentTimeMillis() - start;
                if (duration > SLOW_SQL_THRESHOLD) {
                    log.warn("慢SQL检测，耗时: {}ms", duration);
                    // 发送告警、记录日志等
                }
            }
        }
    }
}
```

## 完整的最佳实践方案

```java
@Component
public class ComprehensiveCacheSolution {
    
    // 1. 多级缓存
    private Cache<String, Object> localCache = Caffeine.newBuilder()
            .maximumSize(5000)
            .expireAfterWrite(5, TimeUnit.MINUTES)
            .recordStats()  // 记录统计信息
            .build();
    
    @Resource
    private RedisTemplate<String, Object> redisTemplate;
    
    // 2. 熔断器
    private CircuitBreaker circuitBreaker;
    
    // 3. 线程池（用于异步刷新）
    private ExecutorService refreshExecutor = Executors.newFixedThreadPool(10);
    
    @PostConstruct
    public void init() {
        // 初始化熔断器
        CircuitBreakerConfig config = CircuitBreakerConfig.custom()
                .failureRateThreshold(50)  // 失败率阈值50%
                .waitDurationInOpenState(Duration.ofSeconds(30))  // 打开状态等待30秒
                .slidingWindowSize(10)  // 滑动窗口大小10
                .build();
        
        circuitBreaker = CircuitBreaker.of("cache-circuit-breaker", config);
        
        // 启动缓存预热
        warmUpCache();
    }
    
    public Object getData(String key) {
        // 1. 本地缓存
        Object value = localCache.getIfPresent(key);
        if (value != null) {
            return value;
        }
        
        // 2. Redis缓存（带熔断保护）
        try {
            value = circuitBreaker.executeSupplier(() -> {
                Object val = redisTemplate.opsForValue().get(key);
                
                if (val == null) {
                    // 双重检查锁防止缓存击穿
                    val = getFromDbWithLock(key);
                    
                    if (val != null) {
                        // 异步刷新（预刷新）
                        asyncRefreshBeforeExpire(key);
                    }
                } else {
                    // 异步延长缓存时间
                    asyncExtendCache(key);
                }
                
                return val;
            });
        } catch (Exception e) {
            log.error("缓存获取异常，key: {}", key, e);
            value = getFallbackData(key);
        }
        
        // 3. 回填本地缓存
        if (value != null) {
            localCache.put(key, value);
        }
        
        return value;
    }
    
    private void asyncRefreshBeforeExpire(String key) {
        refreshExecutor.submit(() -> {
            try {
                // 在缓存过期前30%的时间开始刷新
                Long ttl = redisTemplate.getExpire(key);
                if (ttl != null && ttl > 0) {
                    long refreshTime = (long) (ttl * 0.7 * 1000);
                    Thread.sleep(refreshTime);
                    
                    // 刷新缓存
                    Object newData = loadDataFromDB(key);
                    if (newData != null) {
                        redisTemplate.opsForValue().set(
                            key, 
                            newData, 
                            getRandomTTL(), 
                            TimeUnit.SECONDS
                        );
                    }
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });
    }
    
    private int getRandomTTL() {
        // 基础1小时 ± 随机5分钟
        int base = 3600;
        int random = ThreadLocalRandom.current().nextInt(-300, 301);
        return base + random;
    }
}
```

## 监控告警体系

```yaml
监控指标:
  - redis.hit.rate: 缓存命中率
  - redis.expired.keys: 过期key数量
  - db.connection.active: 数据库活跃连接数
  - db.qps: 数据库查询QPS
  - circuit.breaker.state: 熔断器状态
  - cache.refresh.latency: 缓存刷新延迟

告警规则:
  - 缓存命中率 < 80% 持续5分钟
  - 数据库连接数 > 80% 最大值
  - 单key查询QPS > 1000
  - 熔断器开启状态持续30秒

应急预案:
  1. 手动触发缓存预热
  2. 临时调整过期时间
  3. 启用降级策略
  4. 扩容Redis集群
```

## 总结对比

| 方案             | 核心思想           | 优点             | 适用场景         |
| ---------------- | ------------------ | ---------------- | ---------------- |
| 随机过期时间     | 分散失效时间       | 简单有效，成本低 | 大多数场景       |
| 热点数据永不过期 | 避免关键数据失效   | 保证热点数据可用 | 核心业务数据     |
| 多级缓存         | 分级缓存，逐级回源 | 减少Redis压力    | 高并发场景       |
| 熔断降级         | 故障隔离，快速失败 | 保护下游系统     | 依赖复杂系统     |
| 缓存预热         | 主动加载缓存       | 避免冷启动问题   | 系统启动、大促前 |
| 数据库保护       | 限制数据库压力     | 防止数据库宕机   | 所有数据库访问   |

**推荐策略组合**：
1. **预防为主**：随机过期时间 + 缓存预热
2. **容错为辅**：熔断降级 + 多级缓存
3. **监控告警**：实时监控 + 自动扩容
4. **应急预案**：手动干预 + 快速恢复

通过这种多层次、立体化的防护体系，可以有效防止缓存雪崩的发生，即使发生也能将影响降到最低。

# Redis缓存击穿详解及解决方案

## 什么是缓存击穿？

**缓存击穿**是指一个**热点key突然过期**，此时有**大量并发请求**同时访问这个key，导致所有请求都穿透到数据库，造成数据库瞬时压力巨大。

## 与缓存穿透、雪崩的区别

| 现象         | 核心问题         | 影响范围        | 触发条件                |
| ------------ | ---------------- | --------------- | ----------------------- |
| **缓存穿透** | 查询不存在的数据 | 多个不存在的key | 恶意请求或误操作        |
| **缓存雪崩** | 大量key同时过期  | 大量key同时失效 | 批量设置相同过期时间    |
| **缓存击穿** | 热点key突然过期  | 单个热点key失效 | 高并发访问的热点key过期 |

## 典型场景

```java
// 热点数据（如明星绯闻、秒杀商品）
public Object getHotNews(String newsId) {
    // 热点新闻缓存1小时
    String cacheKey = "hot_news:" + newsId;
    
    // 1小时后缓存失效
    Object news = redis.get(cacheKey);
    
    if (news == null) {
        // 大量请求同时到达这里！！！
        news = db.query("SELECT * FROM news WHERE id = ?", newsId);
        
        // 重新缓存
        redis.setex(cacheKey, 3600, news);
    }
    
    return news;
}
```

## 核心解决方案

### 方案1：互斥锁（Mutex Lock） - 最常用
```java
@Component
public class CacheBreakdownSolution {
    
    @Resource
    private RedisTemplate<String, Object> redisTemplate;
    
    // 锁的过期时间（防止死锁）
    private static final long LOCK_EXPIRE = 10; // 秒
    private static final long WAIT_TIME = 100;  // 毫秒
    private static final int RETRY_TIMES = 3;   // 重试次数
    
    /**
     * 方案1.1：基于SETNX的分布式锁
     */
    public Object getDataWithLockV1(String key) {
        // 1. 先查缓存
        Object value = redisTemplate.opsForValue().get(key);
        if (value != null) {
            return value;
        }
        
        // 2. 尝试获取锁
        String lockKey = "lock:" + key;
        boolean locked = false;
        
        try {
            // 使用SETNX获取锁
            locked = redisTemplate.opsForValue()
                    .setIfAbsent(lockKey, "1", LOCK_EXPIRE, TimeUnit.SECONDS);
            
            if (locked) {
                // 获取锁成功，再次检查缓存（双重检查）
                value = redisTemplate.opsForValue().get(key);
                if (value != null) {
                    return value;
                }
                
                // 查询数据库
                value = queryFromDatabase(key);
                
                if (value != null) {
                    // 写入缓存，设置随机过期时间避免雪崩
                    int randomTTL = 3600 + new Random().nextInt(600); // 1小时±10分钟
                    redisTemplate.opsForValue().set(
                        key, value, randomTTL, TimeUnit.SECONDS
                    );
                } else {
                    // 数据库不存在，缓存空值防止穿透
                    redisTemplate.opsForValue().set(
                        key, "", 300, TimeUnit.SECONDS
                    );
                }
                
                return value;
            } else {
                // 获取锁失败，等待后重试
                Thread.sleep(WAIT_TIME);
                return getDataWithLockV1(key); // 递归重试
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            return null;
        } finally {
            // 释放锁
            if (locked) {
                redisTemplate.delete(lockKey);
            }
        }
    }
    
    /**
     * 方案1.2：Redisson分布式锁（推荐）
     */
    @Resource
    private RedissonClient redissonClient;
    
    public Object getDataWithRedisson(String key) {
        // 1. 先查缓存
        Object value = redisTemplate.opsForValue().get(key);
        if (value != null) {
            return value;
        }
        
        // 2. 使用Redisson分布式锁
        String lockKey = "lock:" + key;
        RLock lock = redissonClient.getLock(lockKey);
        
        try {
            // 尝试获取锁，最多等待100ms，锁持有30秒
            boolean locked = lock.tryLock(100, 30000, TimeUnit.MILLISECONDS);
            
            if (locked) {
                try {
                    // 双重检查
                    value = redisTemplate.opsForValue().get(key);
                    if (value != null) {
                        return value;
                    }
                    
                    // 查询数据库
                    value = queryFromDatabase(key);
                    
                    if (value != null) {
                        // 异步更新缓存时间
                        asyncUpdateCacheExpire(key);
                        
                        redisTemplate.opsForValue().set(
                            key, value, getRandomTTL(), TimeUnit.SECONDS
                        );
                    }
                    
                    return value;
                } finally {
                    lock.unlock();
                }
            } else {
                // 获取锁失败，使用降级策略
                return getFallbackData(key);
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            return getFallbackData(key);
        }
    }
}
```

### 方案2：逻辑过期（Logical Expiration）
```java
@Component
public class LogicalExpirationCache {
    
    @Data
    @AllArgsConstructor
    private static class CacheData {
        private Object data;          // 实际数据
        private long expireTime;      // 逻辑过期时间
    }
    
    /**
     * 设置缓存（带逻辑过期时间）
     */
    public void setWithLogicalExpire(String key, Object value, long ttlSeconds) {
        CacheData cacheData = new CacheData(
            value, 
            System.currentTimeMillis() + ttlSeconds * 1000
        );
        
        redisTemplate.opsForValue().set(
            key, 
            JSON.toJSONString(cacheData)  // 序列化存储
        );
    }
    
    /**
     * 获取数据（逻辑过期方案）
     */
    public Object getWithLogicalExpire(String key) {
        // 1. 从缓存获取数据
        String json = (String) redisTemplate.opsForValue().get(key);
        if (json == null) {
            // 缓存不存在，重建缓存
            return rebuildCache(key);
        }
        
        // 2. 解析缓存数据
        CacheData cacheData = JSON.parseObject(json, CacheData.class);
        
        // 3. 判断是否逻辑过期
        long current = System.currentTimeMillis();
        if (current <= cacheData.getExpireTime()) {
            // 未过期，直接返回
            return cacheData.getData();
        } else {
            // 已过期，异步重建
            asyncRebuildCache(key);
            
            // 返回旧数据（保证可用性）
            return cacheData.getData();
        }
    }
    
    /**
     * 异步重建缓存
     */
    private void asyncRebuildCache(String key) {
        CompletableFuture.runAsync(() -> {
            // 获取互斥锁，防止重复重建
            String lockKey = "rebuild_lock:" + key;
            boolean locked = redisTemplate.opsForValue()
                    .setIfAbsent(lockKey, "1", 10, TimeUnit.SECONDS);
            
            if (locked) {
                try {
                    // 再次检查是否已有其他线程重建
                    String currentJson = (String) redisTemplate.opsForValue().get(key);
                    if (currentJson != null) {
                        CacheData currentData = JSON.parseObject(currentJson, CacheData.class);
                        if (System.currentTimeMillis() <= currentData.getExpireTime()) {
                            return; // 已有其他线程更新
                        }
                    }
                    
                    // 查询数据库
                    Object newData = queryFromDatabase(key);
                    if (newData != null) {
                        // 更新缓存
                        setWithLogicalExpire(key, newData, 3600);
                    }
                } finally {
                    redisTemplate.delete(lockKey);
                }
            }
        });
    }
}
```

### 方案3：预热与续期（Warm-up & Renewal）
```java
@Component
public class CacheWarmUpAndRenewal {
    
    @Resource
    private RedisTemplate<String, Object> redisTemplate;
    
    // 热点key监控
    private Map<String, AtomicInteger> hotKeyStats = new ConcurrentHashMap<>();
    private static final int HOT_THRESHOLD = 1000; // 访问阈值
    
    /**
     * 带热点检测的缓存获取
     */
    public Object getDataWithHotSpotDetection(String key) {
        // 1. 记录访问频率
        recordAccess(key);
        
        // 2. 检查缓存
        Object value = redisTemplate.opsForValue().get(key);
        
        if (value == null) {
            value = getDataWithLock(key); // 使用互斥锁方案
        } else {
            // 3. 如果是热点key，异步续期
            if (isHotKey(key)) {
                asyncRenewCache(key);
            }
        }
        
        return value;
    }
    
    /**
     * 记录key访问频率
     */
    private void recordAccess(String key) {
        hotKeyStats.computeIfAbsent(key, k -> new AtomicInteger(0))
                  .incrementAndGet();
        
        // 定期清理过期的统计
        ScheduledExecutorService executor = Executors.newSingleThreadScheduledExecutor();
        executor.scheduleAtFixedRate(() -> {
            hotKeyStats.entrySet().removeIf(entry -> 
                entry.getValue().get() < HOT_THRESHOLD / 2
            );
        }, 1, 1, TimeUnit.HOURS);
    }
    
    /**
     * 判断是否为热点key
     */
    private boolean isHotKey(String key) {
        AtomicInteger count = hotKeyStats.get(key);
        return count != null && count.get() >= HOT_THRESHOLD;
    }
    
    /**
     * 异步续期缓存
     */
    private void asyncRenewCache(String key) {
        CompletableFuture.runAsync(() -> {
            // 在缓存过期前30%的时间开始续期
            Long ttl = redisTemplate.getExpire(key);
            
            if (ttl != null && ttl > 0) {
                long renewTime = (long) (ttl * 0.7 * 1000);
                
                try {
                    Thread.sleep(renewTime);
                    
                    // 查询最新数据
                    Object newData = queryFromDatabase(key);
                    if (newData != null) {
                        redisTemplate.opsForValue().set(
                            key, 
                            newData, 
                            getRandomTTL(), 
                            TimeUnit.SECONDS
                        );
                    }
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            }
        });
    }
    
    /**
     * 热点key预热
     */
    @Scheduled(cron = "0 0 6 * * ?") // 每天6点预热
    public void preheatHotKeys() {
        List<String> hotKeys = detectHotKeysFromLogs();
        
        for (String key : hotKeys) {
            CompletableFuture.runAsync(() -> {
                Object data = queryFromDatabase(extractIdFromKey(key));
                if (data != null) {
                    // 设置较长的过期时间
                    redisTemplate.opsForValue().set(
                        key, data, 7200, TimeUnit.SECONDS // 2小时
                    );
                }
            });
        }
    }
}
```

### 方案4：多级缓存 + 本地锁
```java
@Component
public class MultiLevelCacheWithLocalLock {
    
    // 一级缓存：本地缓存（Guava Cache）
    private LoadingCache<String, Object> localCache = CacheBuilder.newBuilder()
            .maximumSize(10000)
            .expireAfterWrite(5, TimeUnit.MINUTES)
            .refreshAfterWrite(4, TimeUnit.MINUTES) // 主动刷新
            .build(new CacheLoader<String, Object>() {
                @Override
                public Object load(String key) throws Exception {
                    return loadFromRedis(key);
                }
            });
    
    // 本地锁（解决单机重复查询）
    private Map<String, Object> localLocks = new ConcurrentHashMap<>();
    
    @Resource
    private RedisTemplate<String, Object> redisTemplate;
    
    /**
     * 多级缓存方案
     */
    public Object getDataMultiLevel(String key) {
        try {
            // 1. 查本地缓存
            return localCache.get(key);
        } catch (ExecutionException e) {
            // 2. 本地缓存未命中，查Redis
            return getFromRedisWithLocalLock(key);
        }
    }
    
    /**
     * 带本地锁的Redis查询
     */
    private Object getFromRedisWithLocalLock(String key) {
        // 获取本地锁对象
        Object localLock = localLocks.computeIfAbsent(key, k -> new Object());
        
        synchronized (localLock) {
            try {
                // 再次检查本地缓存（双重检查）
                try {
                    return localCache.getIfPresent(key);
                } catch (Exception e) {
                    // 忽略异常
                }
                
                // 查Redis
                Object value = redisTemplate.opsForValue().get(key);
                
                if (value == null) {
                    // Redis未命中，查数据库（带分布式锁）
                    value = getFromDbWithDistributedLock(key);
                    
                    if (value != null) {
                        // 更新多级缓存
                        updateMultiLevelCache(key, value);
                    }
                } else {
                    // Redis命中，更新本地缓存
                    localCache.put(key, value);
                }
                
                return value;
            } finally {
                // 清理本地锁（延迟清理）
                localLocks.remove(key);
            }
        }
    }
    
    /**
     * 更新多级缓存
     */
    private void updateMultiLevelCache(String key, Object value) {
        // 1. 更新Redis（异步）
        CompletableFuture.runAsync(() -> {
            redisTemplate.opsForValue().set(
                key, value, getRandomTTL(), TimeUnit.SECONDS
            );
        });
        
        // 2. 更新本地缓存
        localCache.put(key, value);
    }
}
```

### 方案5：请求合并（Request Merging）
```java
@Component
public class RequestMergingCache {
    
    // 请求合并队列
    private Map<String, CompletableFuture<Object>> mergingMap = new ConcurrentHashMap<>();
    
    @Resource
    private RedisTemplate<String, Object> redisTemplate;
    
    // 合并时间窗口（毫秒）
    private static final long MERGE_WINDOW = 10;
    
    /**
     * 请求合并方案
     */
    public CompletableFuture<Object> getDataWithMerging(String key) {
        // 1. 先查缓存
        Object value = redisTemplate.opsForValue().get(key);
        if (value != null) {
            return CompletableFuture.completedFuture(value);
        }
        
        // 2. 检查是否有正在进行的相同请求
        return mergingMap.computeIfAbsent(key, k -> {
            // 创建异步任务
            CompletableFuture<Object> future = CompletableFuture.supplyAsync(() -> {
                try {
                    // 等待时间窗口，合并请求
                    Thread.sleep(MERGE_WINDOW);
                    
                    // 查询数据库（带锁）
                    return getFromDbWithLock(key);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    throw new RuntimeException(e);
                }
            });
            
            // 任务完成后清理
            future.whenComplete((result, error) -> {
                mergingMap.remove(key);
                
                if (result != null) {
                    // 更新缓存
                    redisTemplate.opsForValue().set(
                        key, result, getRandomTTL(), TimeUnit.SECONDS
                    );
                }
            });
            
            return future;
        });
    }
    
    /**
     * 批量获取（优化版）
     */
    public Map<String, Object> batchGet(List<String> keys) {
        // 1. 批量查缓存
        List<Object> values = redisTemplate.opsForValue().multiGet(keys);
        Map<String, Object> result = new HashMap<>();
        List<String> missingKeys = new ArrayList<>();
        
        // 2. 找出未命中的key
        for (int i = 0; i < keys.size(); i++) {
            if (values.get(i) != null) {
                result.put(keys.get(i), values.get(i));
            } else {
                missingKeys.add(keys.get(i));
            }
        }
        
        // 3. 批量查询缺失的数据
        if (!missingKeys.isEmpty()) {
            Map<String, Object> missingData = batchQueryFromDb(missingKeys);
            result.putAll(missingData);
            
            // 批量更新缓存
            batchUpdateCache(missingData);
        }
        
        return result;
    }
}
```

## 完整的最佳实践方案

```java
@Component
@Slf4j
public class ComprehensiveCacheBreakdownSolution {
    
    // 本地缓存
    private Cache<String, Object> localCache = Caffeine.newBuilder()
            .maximumSize(5000)
            .expireAfterWrite(3, TimeUnit.MINUTES)
            .recordStats()
            .build();
    
    @Resource
    private RedisTemplate<String, Object> redisTemplate;
    
    @Resource
    private RedissonClient redissonClient;
    
    // 监控指标
    private MeterRegistry meterRegistry;
    
    // 热点key检测器
    private HotKeyDetector hotKeyDetector;
    
    /**
     * 综合解决方案
     */
    public Object getDataComprehensive(String key) {
        // 1. 本地缓存
        Object value = localCache.getIfPresent(key);
        if (value != null) {
            meterRegistry.counter("cache.local.hit").increment();
            return value;
        }
        
        // 2. Redis缓存（带逻辑过期检查）
        value = getFromRedisWithLogicalExpire(key);
        
        if (value == null) {
            // 3. 缓存击穿保护
            value = handleCacheBreakdown(key);
        } else {
            // 4. 如果是热点key，异步续期
            if (hotKeyDetector.isHotKey(key)) {
                asyncRenewHotKey(key);
            }
            
            // 5. 更新本地缓存
            localCache.put(key, value);
        }
        
        return value;
    }
    
    /**
     * 处理缓存击穿
     */
    private Object handleCacheBreakdown(String key) {
        // 方案选择：根据key类型和访问频率选择不同策略
        if (isSuperHotKey(key)) {
            // 超级热点：逻辑过期 + 异步刷新
            return getWithLogicalExpireAndAsyncRefresh(key);
        } else if (isHighConcurrentKey(key)) {
            // 高并发：分布式锁
            return getWithDistributedLock(key);
        } else {
            // 普通场景：互斥锁
            return getWithMutexLock(key);
        }
    }
    
    /**
     * 分布式锁方案
     */
    private Object getWithDistributedLock(String key) {
        String lockKey = "lock:" + key;
        RLock lock = redissonClient.getLock(lockKey);
        
        try {
            // 尝试获取锁（非阻塞）
            if (lock.tryLock()) {
                try {
                    // 双重检查
                    Object value = redisTemplate.opsForValue().get(key);
                    if (value != null) return value;
                    
                    // 查询数据库
                    value = queryFromDatabase(key);
                    
                    if (value != null) {
                        // 设置缓存
                        redisTemplate.opsForValue().set(
                            key, value, getRandomTTL(), TimeUnit.SECONDS
                        );
                        
                        // 更新本地缓存
                        localCache.put(key, value);
                    }
                    
                    return value;
                } finally {
                    lock.unlock();
                }
            } else {
                // 获取锁失败，快速失败或降级
                return getFallbackOrRetry(key);
            }
        } catch (Exception e) {
            log.error("分布式锁异常，key: {}", key, e);
            return getFallbackData(key);
        }
    }
    
    /**
     * 降级策略
     */
    private Object getFallbackOrRetry(String key) {
        // 策略1：快速返回降级数据
        if (useFastFail()) {
            return getFallbackData(key);
        }
        
        // 策略2：有限重试
        for (int i = 0; i < 3; i++) {
            try {
                Thread.sleep(50 * (i + 1)); // 递增等待
                
                Object value = redisTemplate.opsForValue().get(key);
                if (value != null) {
                    return value;
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                break;
            }
        }
        
        return getFallbackData(key);
    }
    
    /**
     * 监控与告警
     */
    @EventListener
    public void handleCacheEvent(CacheBreakdownEvent event) {
        log.warn("缓存击穿风险，key: {}, 并发数: {}", 
                event.getKey(), event.getConcurrency());
        
        // 发送告警
        sendAlert(event);
        
        // 自动预热该key
        if (event.getConcurrency() > 1000) {
            preheatKey(event.getKey());
        }
    }
}
```

## 场景选择指南

| 场景         | 推荐方案               | 理由                   |
| ------------ | ---------------------- | ---------------------- |
| **一般业务** | 互斥锁（SETNX）        | 简单有效，成本低       |
| **秒杀场景** | 逻辑过期 + 异步刷新    | 避免锁竞争，保证高并发 |
| **热点数据** | 预热 + 续期 + 多级缓存 | 提前加载，减少击穿概率 |
| **查询聚合** | 请求合并               | 减少重复查询，提升效率 |
| **金融交易** | 分布式锁（Redisson）   | 强一致性，安全可靠     |

## 监控指标

```yaml
监控项:
  - cache.breakdown.count: 缓存击穿次数
  - lock.acquire.time: 锁获取时间
  - db.query.concurrency: 数据库并发查询数
  - hotkey.access.frequency: 热点key访问频率
  - cache.rebuild.latency: 缓存重建延迟

告警规则:
  - 单key数据库查询QPS > 500
  - 锁获取失败率 > 20%
  - 缓存击穿次数 > 10次/分钟
  - 缓存重建延迟 > 5秒
```

## 总结

**缓存击穿的核心解决思路**：

1. **加锁排队**：互斥锁防止大量请求同时查询数据库
2. **逻辑过期**：异步更新，不阻塞读请求
3. **永不过期**：后台刷新，保证数据最新
4. **多级缓存**：本地缓存减少Redis压力
5. **请求合并**：减少重复查询

**最佳实践组合**：
- **预防为主**：热点key预热 + 多级缓存
- **过程控制**：互斥锁 + 逻辑过期
- **降级保障**：快速失败 + 熔断降级
- **监控预警**：实时监控 + 自动扩容

**关键点**：缓存击穿的重点不是防止缓存失效，而是**防止大量并发请求在缓存失效时同时访问数据库**。通过合理的锁策略和异步更新机制，可以显著降低对数据库的冲击。

# 全局ID生成器

全局ID生成器，是一种在分布式系统下用来生成全局唯一ID的工具，一般要满足：唯一性，高可用，递增性，安全性，高性能。

**ID的组成部分：**

- 符号位：1bit，永远为0
- 时间戳：31bit，以秒为单位
- 序列号：32bit，秒内的计数器，支持每秒产生2^32个不同ID

```JAVA
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.stereotype.Component;

import java.time.LocalDateTime;
import java.time.ZoneOffset;
import java.time.format.DateTimeFormatter;

@Component
public class RedisIdWorker {
    /**
     * 开始时间戳 (2022-01-01 00:00:00)
     */
    private static final long BEGIN_TIMESTAMP = 1640995200L;
    
    /**
     * 序列号的位数
     */
    private static final int COUNT_BITS = 32;
    
    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    /**
     * 生成下一个ID
     * @param keyPrefix 业务前缀
     * @return 生成的唯一ID
     */
    public long nextId(String keyPrefix) {
        // 1. 生成时间戳
        LocalDateTime now = LocalDateTime.now();
        long nowSecond = now.toEpochSecond(ZoneOffset.UTC);
        long timestamp = nowSecond - BEGIN_TIMESTAMP;
        
        // 2. 生成序列号
        // 获取当前日期，精确到天
        String date = now.format(DateTimeFormatter.ofPattern("yyyy:MM:dd"));
        // 自增长，设置过期时间避免Redis中key无限增长
        Long count = stringRedisTemplate.opsForValue().increment("icr:" + keyPrefix + ":" + date);
        
        // 如果是当天的第一个ID，设置过期时间（24小时后过期）
        if (count != null && count == 1) {
            // 设置24小时后过期
            stringRedisTemplate.expire("icr:" + keyPrefix + ":" + date, java.time.Duration.ofDays(1));
        }
        
        // 3. 拼接并返回
        return timestamp << COUNT_BITS | (count != null ? count : 0);
    }
    
    /**
     * 测试主方法
     */
    public static void main(String[] args) {
        // 测试时间戳计算
        LocalDateTime beginTime = LocalDateTime.of(2022, 1, 1, 0, 0, 0);
        long beginTimestamp = beginTime.toEpochSecond(ZoneOffset.UTC);
        System.out.println("开始时间戳: " + beginTimestamp);
        
        // 测试ID生成逻辑（需要Redis环境）
        // RedisIdWorker worker = new RedisIdWorker();
        // for (int i = 0; i < 10; i++) {
        //     System.out.println("生成的ID: " + worker.nextId("order"));
        // }
    }
}
```

