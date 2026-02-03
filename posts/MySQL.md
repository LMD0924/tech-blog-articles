# MySQL

### 1.去重

**distinct**

```sql
select distinct department_id from employees;
```

### 2.空值参与运算：结果一定也是空

### 3.着重号``:当命名的名字与关键字发生冲突时可使用着重号

```sql
select * from `order`;
```

### 4.显示表结构

**describe/desc**

```sql
describe user;
desc user;
```

### 5.安全等于<=>

```sql
select 1<=>null,null<=>null from dual;
#结果：0    1

#作用：可以查询表中为null的数据
```

### 6.in (set) \ not |in (set)     like模糊查询

```sql
#练习：查询部门为10,20,30部门的员工信息
SELECT last_name,salary,department_id
FROM employees
#where department_id = 10 or department_id = 20 or department_id = 30;
WHERE department_id IN (10,20,30);


#练习：查询工资不是6000,7000,8000的员工信息
SELECT last_name,salary,department_id
FROM employees
WHERE salary NOT IN (6000,7000,8000);
```

```sql
# ⑥ LIKE :模糊查询
#练习：查询last_name中包含字符'a'的员工信息

# % : 代表不确定个数的字符（0个，1个，或多个）

SELECT last_name
FROM employees
WHERE last_name LIKE '%a%';
```

### 7.使用order by 对查询到的数据进行排序操作

升序：asc   降序:desc   默认升序

```sql
select employee_id from employees order by salary;
```

注意：列的别名只能再order by中使用，不能再where中使用

格式：where需要声明再from后，order by之前

### 8.分页 limit

```sql
#需求：每页显示pageSize条记录，此时显示第pageNo页
#公式：limit (pageNo-1)*pageSize,pageSize;
```

MySQL8.0新特性：limit 显示条数 offset 偏移量;

## 多表查询

如果给表起了别名，一旦在select或where中使用表名的话，必须使用表的别名，不能使用表的原名。

1.**内连接**：合并具有同一列的两个以上的表的行，结果集中不包含一个表与另一个表不匹配的行

```sql
select last_name,department_name from employee e inner join department d on e.department_id=d.department_id;
```

2.**内连接**：合并具有同一列的两个以上的表的行，记过集中除了包含一个表与另一个表匹配的行之外，还查询到了左表或右表中不匹配的行

分类：左外连接，右外连接，满外连接

左外连接

```sql
select last_name,department_name from employees e left outer join departments d on e.department_id=d.department_id;
```

右外连接

```sql
select last_name,department_name from employees e right outer join departments d on e.department_id=d.department_id;
```

### 1.union和union all的使用

union:会执行去重操作

union all：不会执行去重操作

结论：如果明确知道合并数据后的结果数据不存在重复的数据，或者不需要去重复的数据，则尽量使用union all语句，以提高数据的查询效率.

![image-20251230194120401](E:\GitHub\tech-blog-articles\posts\assets\image-20251230194120401.png)

**中图：内连接**
```sql
SELECT employee_id, department_name
FROM employees e JOIN departments d
ON e.`department_id` = d.`department_id`;
```

**左上图：左外连接**
```sql
SELECT employee_id, department_name
FROM employees e LEFT JOIN departments d
ON e.`department_id` = d.`department_id`;
```

**右上图：右外连接**
```sql
SELECT employee_id, department_name
FROM employees e RIGHT JOIN departments d
ON e.`department_id` = d.`department_id`;
```

**左中图：只存在于左表的数据**
```sql
SELECT employee_id, department_name
FROM employees e LEFT JOIN departments d
ON e.`department_id` = d.`department_id`
WHERE d.`department_id` IS NULL;
```

**右中图：只存在于右表的数据**
```sql
SELECT employee_id, department_name
FROM employees e RIGHT JOIN departments d
ON e.`department_id` = d.`department_id`
WHERE e.`department_id` IS NULL;
```

**左下图：满外连接（方式1：左上图 UNION ALL 右中图）**
```sql
SELECT employee_id,department_name
FROM employees e LEFT JOIN departments d
ON e.`department_id` = d.`department_id`
UNION ALL
SELECT employee_id,department_name
FROM employees e RIGHT JOIN departments d
ON e.`department_id` = d.`department_id`
WHERE e.`department_id` IS NULL;
```

**右下图：左中图 UNION ALL 右中图**
```sql
SELECT employee_id,department_name
FROM employees e LEFT JOIN departments d
ON e.`department_id` = d.`department_id`
WHERE d.`department_id` IS NULL
UNION ALL
SELECT employee_id,department_name
FROM employees e RIGHT JOIN departments d
ON e.`department_id` = d.`department_id`
WHERE e.`department_id` IS NULL;
```

**解释：**
1. **左中图**：左表独有的数据（右表对应为NULL）
2. **右中图**：右表独有的数据（左表对应为NULL）
3. **左下图**：左外连接 + 右表独有的数据 = 所有数据（满外连接）
4. **右下图**：左表独有的数据 + 右表独有的数据 = 两个表互不匹配的数据

### 2.自然连接natural join

他会帮你自动查询两张连接表中所有相同的字段，然后进行等值连接

```sql
select employee_id,last_name,department_name from employees e natural join departments d;
```

### 3.using

在 MySQL 中，`USING` 主要用于 **JOIN 操作**中，用于指定连接条件，特别是当两个表中连接字段**名称相同**时。

```sql
select employee_id,last_name,department_name from employee e join departments d using (department_id);
```



## 函数

### 数值函数

CEIL(x),CEILING(x)返回大于或等于某个值的最小整数

FLOOR(x)返回小于或等于某个值的最大整数

LEAST(e1,e2,e3……)返回列表中最小的值

GREATEST()返回列表中最大的值

MOD()返回余数

ROUND()四舍五入

TRUNCATE(x,y)截断，全舍去



### 字符串函数

字符串的索引是从1开始的



### 日期时间函数



### 流程控制函数

IF（value,value1,value2)如果value的值为true，返回value1，否则返回value2

IFNULL(value1，value2)如果value1不为null，返回value1，否则返回value2

CASE WHEN 条件1 THEN 结果二 WHEN 条件2 THEN 结果2……[ELSE RESULTN]END 相当于if else

CASE expr WHEN 常量值1 THEMN值1 WHEN 常量值1 THEN 值……ELSE  END   相当于switch case



### 加密与解密的函数

加密：MD5(str),SHA(str)



### 聚合函数

AVG()，SUM(),MAX(),MIN(),COUNT()



**GROUP BY**

SELECT中出现的非组函数的字段必须声明在GROUP BY中，反之，GROUP BY中声明的字段可以不出现在SELECT中。

**HAVING的使用**

作用：用来过滤数据的

```sql
SELECT department_id,MAX(salary)
FROM employees
GOURP BY department_id
HAVING MAX(salary)>1000;
```

要求1：如果过滤条件中使用聚合函数，则必须使用HAVING来替换WHERE。否则报错。

要求2：HAVING必须声明在GROUP BY 后面。



## 子查询

需求：谁的工资比Abel的高？

方式1：简单查询

```sql
select salary
from employees
where last_name = 'Abel';

select last_name,salary
from employees
where salary>10000;
```

方式2：子连接

```sql
select e2.last_name,e2.salary
from employees e1,employees e2
where e2.salary>e1.salary
and e1.last_name='Abel';
```

方式3：子查询(单行子查询)

```sql
select last_name,salary
from employees
where salary>(
	select salary
    from employees
    where last_name='Abel'
);
```

#### **单行子查询**

内查询返回一条数据

单行操作符：=     !=	>	 >=	<	<=

#### 多行子查询

内查询返回多行数据

多行比较操作符：

IN	等于列表中的任意一个

ANY	需要和单行比较符一起使用，和子查询返回的某一个值比较

ALL	需要和单行比较符一起使用，和子查询返回的所有值比较

SOME	 和any一样

需求：返回其他job_id中比job_id为IT_PROG部门任一工资低的员工的信息

```sql
select epmployee_id,last_name,salary
from employees
where job_id<>'IT_PROG'
and salary< any (
	select salary
    from employees
    where job_id='IT_PROG'
);
```

#### 相关子查询

如果子查询的执行依赖于外部查询，通常情况下都是因为子查询中的表用到了外部的表，并进行了条件关联，因此每执行一次外部查询，子查询都要重新计算一次，这样的子查询就称为关联子查询

在SELECT中，除了GROUP BY和LIMIT外，其他位置都可以声明子查询！

### 创建和管理表

方式1：创建数据库

```sql
CREATE DATABASE 数据库名;
```

方式2：创建数据库并指定字符集

```sql
CREATE DATABASE 数据库名 CHARACTER SET 字符集;
```

方式3：判断数据库是否已经存在，不存在则创建数据库（推荐）

```sql
CREATE DATABASE IF NOT EXISTS 数据库名;
```

查看当前连接中的数据库都有哪些

```sql
SHOW DATABASE;
```

切换数据库，查看当前数据库中保存的数据表

```sql
USE 数据库名；
SHOW TABLES;
#查看当前使用的数据库
SELECT DATABASE() FROM DUAL;
#查看指定数据库下保存的数据表
SHOW TABLES FROM mysql;
```

更改数据库字符集

```SQL
ALTER DATABASE 数据库名 CHARACTER SET 'utf8';
```

删除数据库

```SQL
DROP DATABASE 数据库名;
#或者
DROP DATABASE IF EXISTS 数据库名;
```

### 创建数据表

方式1：

```SQL
CREATE TABLE IF NOT EXISTS 表名(
字段名 类型,
);
#查看表结构
DESC 表名;
SHOW CREATE TABLE 表名;
```

### 修改表——ALTER  TABLE

#### 添加一个字段

```SQL
ALTER TABLE 表名
ADD 字段 类型;#默认添加到表中的最后一个字段的位置

ALTER TABLE 表名
ADD 字段 类型 FRIST;#添加到第一个

ALTER TABLE 表名
ADD 字段1 类型 AFTER 字段2;#添加到字段2的前面
```



#### 修改一个字段

```sql
ALTER TABLE 表名
MODIFY 字段名 类型 DEFAULT 'AAA';
#MODIFGY修改字段的信息（长度，默认值）
#DEFAULT设置默认值
```



#### 重命名字段

```sql
ALTER TABLE 表名
CHANGE 旧的字段名 新的字段名;
```



#### 删除字段

```sql
ALTER TABLE 表名
DROP COLUMN 字段名 类型;
```



### 重命名表

```SQL
RENAME TABLE 旧的表名
TO 新的表名;
```



### 删除表

```sql
DROP TABLE IF EXISTS 表名;
```



### 清空表

```sql
TRUNCATE TABLE 表名;
```

### MySQL8的新特性：计算列

```SQL
CREATE TABLE test1(
	a INT,
    B INT,
    C INT GENERATED ALWAYS AS (a+b) VIRTUAL
);
#字段c为计算列，是由字段a和b的值相加得来的
```

### MySQL中的数据类型

整数类型

浮点类型：计算会出现精度不精确

定点数类型：DECIMAL  底层利用字符串存储数据的，比较精确

位类型：BIT

日期时间类型

文本字符串类型

枚举类型：ENUM

集合类型：SET

二进制字符串类型：BINARY	VARBINARY	TINYBLOB	BLOB	MEDIUMBLOB	LONBLOB

JSON类型

空间数据类型

### MySQL 中 ZEROFILL

**ZEROFILL** 是 MySQL 中的一个字段属性，用于在**显示数值时**用前导零填充到指定宽度。

```sql
-- 创建测试表
CREATE TABLE zerofill_demo (
    num_normal INT(5),
    num_zerofill INT(5) ZEROFILL,
    price DECIMAL(6,2) ZEROFILL
);

-- 插入数据
INSERT INTO zerofill_demo VALUES 
(123, 123, 45.67),
(1, 1, 1234.56),
(99999, 99999, 0.5),
(100000, 100000, 9999.99);  -- 超出宽度

-- 查询数据
SELECT * FROM zerofill_demo;
```

```tex
num_normal | num_zerofill | price
-----------|--------------|--------
123        | 00123        | 045.67
1          | 00001        | 1234.56
99999      | 99999        | 0000.50
100000     | 100000       | 9999.99  -- 超出宽度时正常显示
```



### 约束constraint

#### 查看表中约束：

```SQL
SELECT * FROM informatio_schema.table_constraints
WHERE table_name='表名';
```

#### 1.NOT NULL 	非空约束

限制某个字段/某列的值不为null

1.1在创建表时添加约束

```SQL
CREATE TABLE test1(
	id int NOT NULL,
    last_name varchar(25) NOT NULL,
    email varchar(25),
    salary DECIMAL(10,2)
);
```

1.2在修改表时添加/删除约束

```SQL
#添加约束
ALTER TABLE test1
MODIFY email VARCHAR(25) NOT NULL;
#删除约束
ALTER TABLE test1
MODIFY email VARCHAR(25) NULL;
```

### 2.unique 唯一性约束

2.1在创建表时添加约束

```sql
CREATE TABLE test2(
	id int UNIQUE,#列级约束
    last_name varchar(25) UNIQUE,
    email varchar(25) UNIQUE,
    salary DECIMAL(10,2)
    #表级约束  所有字段声明完，写在最后
    CONSTRAINT uk_test2_email UNIQUE(email)
);
```

2.2在修改表时添加约束

```SQL
#添加约束
ALTER TABLE 表名
ADD CONSTRAINT 约束名 UNIQUE (字段名);

ALTER TABLE 表名
MODIFY 字段名 类型 UNIQUE;
```

2.3删除唯一性约束

添加唯一性约束的列上也会自动创建唯一索引

删除唯一约束只能通过删除唯一索引的方式删除

删除时需要指定唯一索引名，唯一索引名就和唯一约束名一样

如果创建唯一约束时未指定名称，如果时单列，就默认和列名相同；如果时组合列，那么默认和（）中排在第一个的列名相同，也可以自定义名；

```SQL
ALTER TABLE 表名
DROP INDEX 约束名;
```

#### 3.primary key 约束

用来唯一标识表中的一行记录

一个表只能添加一个主键约束

MySQL的主键名总是PRIMARY,就算自己命名主键约束名也没用。

**auto_increment自增列  **

①一个表中最多有一个自增长列**

②自增长列约束的列必须是键列（主键列，唯一键列）

③自增约束的列的数据类型必须是整数类型

⑤如果自增列指定了0和null，会在当前的最大值的基础上自增，如果手动指定了具体的值，直接赋值为具体值。

#### 4.foreign key 外键约束

```SQL
#先创建主表
CREATE TABLE 主表名(
	id int primary key,
    name varchar(15)
);
#创建从表
CREATE TABLE 从表名(
	id int primary key auto_increment,
    name varchar(15),
    主表名_id int,
    CONSTRAINT 约束名 FOREIGN KEY (主表名_id) REFERENCES 主表名(id);
);
```

#### 5.CHECK约束

```SQL
CREATE TABLE 表名(
	id int primary key,
    name varchar(15),
    salary DECIMAL(10,2) CHECK(salary>2000)
);
```

#### 6.DEFAULT默认值约束

### 视图

①.视图可以看做是一个虚拟表，本身不存储数据，视图的本质，就可以看做是存储起来的select语句

②视图中select语句中涉及到的表称为基表

③针对视图做DML操作，会影响到对应的基表数据

④视图本身删除，不会导致基表中数据的删除

#### 创建视图

```SQL
CREATE VIEW 视图名
AS select * 
from 表名;
```

#### 查看视图详细信息

```SQL
SHOW CREATE VIEW 视图名;
```

#### 修改视图

方式1：

```SQL
CREATE OR REPLACE VIEW 视图名
AS
查询语句;
```

方式2：

```SQL
ALTER VIEW 视图名
AS
查询语句;
```

#### 删除视图

```SQL
DROP VIEW IF EXISTS 视图名;
```



### 存储过程和存储函数

1.没有参数

2.IN类型（有参无返）

3.OUT类型（无参有返）

4.IN和OUT（有参有返）

5.INOUT（有参有返）

#### 创建存储过程

```sql
CREATE PROCEDURE 存储过程名(IN|OUT|INOUT 参数名 参数类型)
BEGIN
存储过程体
END;
```

```SQL
DELINITER 结束标识符
```

#### 存储过程的调用

```SQL
CALL 存储过程名();
```

### 存储函数

创建函数前执行此语句，保证函数的创建成功

```SQL
SET GLOBAL log_bin_trust_function_creators=1;
```



```SQL
CREATE FUNCTION 函数名(参数名 参数类型)
RETURNS 返回值类型
BEGIN
 函数体 #函数体中肯定有return语句
 END;
```

#### 调用存储函数

```SQL
SELECT 函数名(参数列表);
```



### 变量，流程控制与游标

#### 变量

在MySQL中，变量分为系统变量和用户自定义变量

系统变量：全局系统变量，会话系统变量

#### 查看系统变量

```SQL
#查询全局系统变量
SHOW GLOBAL VARIABLES;
#查询会话系统变量
SHOW SESSION VARIABLES;
#或
SHOW VARIABLES;#默认查询的是会话系统变量
```

查询部分系统变量

```SQL
SHOW GLOBAL|SESSION VARIABLES LIKE '%标识符%';
```

两个@表示系统变量，一个@表示用户自定义变量

#### 用户变量

用户变量：会话用户变量，局部用户变量

会话用户变量：使用@开头，作用域为当前会话

局部变量：只能使用在存储过程和存储函数中

##### 会话用户变量的声明和赋值

```SQL
#方式1：
SET @用户变量=值;
SET @用户变量:=值；
#方式2：
SELECT @用户变量:=表达式;
SELECT 表达式 INTO @用户变量;
```

##### 使用

```SQL
SELECT @变量名;
```

##### 局部变量

①使用DECLARE声明

②声明并使用在BEGIN……END中（使用在存储过程，存储函数中）

③DECLARE的方式声明的局部变量必须声明在BEGIN中的首行位置

举例

```SQL
DELIMITER //
CREATE PROCEDURE test_var()
BEGIN
  		#声明局部变量
  		DECLARE a INT DEFAULT 0;
  		DECLARE b INT;
  		DECLARE emp_name VARCHAR(25);
  		#赋值
  		SET a =1;
  		set b:=2;
  		SELECT last_name into emp_name from employees where employee_id =101;
  		#使用
  		select a,b,emp_name;
  END //
  DELIMITER;
```



### 流程控制

顺序结构，分支结构，循环结构

#### 分支结构

IF

```SQL
IF 表达式1 THEN 操作1
ELSEIF 表达式2 THEN 操作2
ELSE 操作N
END IF
```

### 形式1：简单 CASE 表达式

```sql
CASE case_value
    WHEN when_value1 THEN result1
    WHEN when_value2 THEN result2
    ...
    [ELSE else_result]
END
```

### **形式2：搜索 CASE 表达式**

```sql
CASE
    WHEN condition1 THEN result1
    WHEN condition2 THEN result2
    ...
    [ELSE else_result]
END
```

## 🔄 **1. WHILE 循环**

### **基本语法**
```sql
[label:] WHILE search_condition DO
    statement_list
END WHILE [label]
```

### **完整结构**
```sql
[label_name:] WHILE condition DO
    -- 循环体语句
    statement1;
    statement2;
    ...
    
    -- 控制语句
    ITERATE label_name;  -- 相当于 continue
    LEAVE label_name;    -- 相当于 break
END WHILE label_name;
```

## 🔁 **2. REPEAT 循环**

### **基本语法**
```sql
[label:] REPEAT
    statement_list
UNTIL search_condition
END REPEAT [label]
```

### **完整结构**
```sql
[label_name:] REPEAT
    -- 循环体语句
    statement1;
    statement2;
    ...
    
    -- 控制语句
    ITERATE label_name;
    LEAVE label_name;
UNTIL condition  -- 条件为真时结束循环
END REPEAT label_name;
```

## 🔂 **3. LOOP 循环**

### **基本语法**
```sql
[label:] LOOP
    statement_list
END LOOP [label]
```

### **完整结构**
```sql
[label_name:] LOOP
    -- 循环体语句
    statement1;
    statement2;
    ...
    
    -- 必须使用LEAVE退出，否则无限循环
    IF condition THEN
        LEAVE label_name;
    END IF;
    
    -- 控制语句
    ITERATE label_name;#ITERATE = 跳过本次循环的剩余代码，直接开始下一次循环
END LOOP label_name;
```

## 📝 **4. 在存储过程中的完整示例结构**

### **存储过程内使用循环**
```sql
DELIMITER $$

CREATE PROCEDURE procedure_name()
BEGIN
    -- 变量声明
    DECLARE var_name INT DEFAULT 0;
    
    -- WHILE循环示例
    WHILE var_name < 10 DO
        -- 循环体
        SET var_name = var_name + 1;
    END WHILE;
    
    -- REPEAT循环示例
    REPEAT
        SET var_name = var_name - 1;
    UNTIL var_name <= 0
    END REPEAT;
    
    -- LOOP循环示例
    my_loop: LOOP
        SET var_name = var_name + 1;
        
        IF var_name >= 5 THEN
            LEAVE my_loop;
        END IF;
        
        IF var_name = 3 THEN
            ITERATE my_loop;
        END IF;
    END LOOP my_loop;
END$$

DELIMITER ;
```

## 🎯 **5. 标签和流程控制语句**

### **标签语法**
```sql
-- 循环标签
label_name: LOOP
    -- ...
END LOOP label_name;

-- 标签用于ITERATE和LEAVE
ITERATE label_name;  -- 跳转到标签处继续下一次循环
LEAVE label_name;    -- 跳出标签标识的循环或代码块
```

### **流程控制组合**
```sql
[label_name:] WHILE condition DO
    -- ...
    
    IF condition1 THEN
        ITERATE label_name;  -- 继续下次循环
    END IF;
    
    IF condition2 THEN
        LEAVE label_name;    -- 退出循环
    END IF;
END WHILE label_name;
```

## 💡 **6. 嵌套循环结构**

```sql
outer_loop: WHILE outer_condition DO
    -- 外层循环体
    
    inner_loop: WHILE inner_condition DO
        -- 内层循环体
        
        IF some_condition THEN
            LEAVE outer_loop;  -- 直接退出外层循环
        END IF;
        
        IF another_condition THEN
            ITERATE inner_loop;  -- 继续内层下次循环
        END IF;
    END WHILE inner_loop;
    
END WHILE outer_loop;
```

**LEAVE = 退出整个循环（相当于 break）**
**ITERATE = 跳过本次循环，继续下一次（相当于 continue）**

# MySQL 游标（Cursor）详解

## 📌 **什么是游标？**

**游标**是**数据库查询结果集的一个指针**，可以逐行遍历结果集。

## 🎯 **一句话理解**
> **"游标就是数据库里的迭代器，让你能一行一行地处理查询结果"**

## 🔄 **游标使用步骤（固定流程）**

```sql
-- 1. 声明游标
DECLARE cursor_name CURSOR FOR select_statement;

-- 2. 打开游标
OPEN cursor_name;

-- 3. 获取数据（循环）
FETCH cursor_name INTO variables;

-- 4. 处理数据
-- ... 业务逻辑 ...

-- 5. 关闭游标
CLOSE cursor_name;
```

## 📊 **完整的游标语法结构**

```sql
DELIMITER $$

CREATE PROCEDURE cursor_demo()
BEGIN
    -- ============ 第1步：声明变量 ============
    DECLARE done INT DEFAULT 0;          -- 结束标志
    DECLARE var1 INT;                    -- 接收字段1
    DECLARE var2 VARCHAR(50);            -- 接收字段2
    DECLARE var3 DECIMAL(10,2);          -- 接收字段3
    
    -- ============ 第2步：声明游标 ============
    DECLARE cursor_name CURSOR FOR 
        SELECT column1, column2, column3 
        FROM table_name 
        WHERE condition 
        ORDER BY column1;
    
    -- ============ 第3步：声明处理程序 ============
    -- 当没有更多行时，设置 done = 1
    DECLARE CONTINUE HANDLER FOR NOT FOUND 
        SET done = 1;
    
    -- ============ 第4步：打开游标 ============
    OPEN cursor_name;
    
    -- ============ 第5步：循环获取数据 ============
    read_loop: LOOP
        -- 获取一行数据到变量
        FETCH cursor_name INTO var1, var2, var3;
        
        -- 检查是否还有数据
        IF done = 1 THEN
            LEAVE read_loop;
        END IF;
        
        -- ============ 第6步：处理数据 ============
        -- 这里写你的业务逻辑
        -- 例如：插入到另一张表、计算、更新等
        
    END LOOP read_loop;
    
    -- ============ 第7步：关闭游标 ============
    CLOSE cursor_name;
    
END$$

DELIMITER ;
```

## 💡 **实际示例**

### **示例1：基本的游标使用**
```sql
DELIMITER $$

CREATE PROCEDURE basic_cursor_example()
BEGIN
    DECLARE v_id INT;
    DECLARE v_name VARCHAR(50);
    DECLARE v_salary DECIMAL(10,2);
    DECLARE done INT DEFAULT 0;
    
    -- 1. 声明游标
    DECLARE emp_cursor CURSOR FOR 
        SELECT employee_id, employee_name, salary 
        FROM employees 
        WHERE department_id = 1;
    
    -- 2. 声明处理程序
    DECLARE CONTINUE HANDLER FOR NOT FOUND 
        SET done = 1;
    
    -- 3. 打开游标
    OPEN emp_cursor;
    
    -- 4. 创建临时表存储结果
    DROP TEMPORARY TABLE IF EXISTS temp_results;
    CREATE TEMPORARY TABLE temp_results (
        emp_id INT,
        emp_name VARCHAR(50),
        emp_salary DECIMAL(10,2),
        bonus DECIMAL(10,2)
    );
    
    -- 5. 循环处理
    read_loop: LOOP
        FETCH emp_cursor INTO v_id, v_name, v_salary;
        
        IF done = 1 THEN
            LEAVE read_loop;
        END IF;
        
        -- 业务逻辑：计算奖金（工资的10%）
        INSERT INTO temp_results 
        VALUES (v_id, v_name, v_salary, v_salary * 0.1);
        
    END LOOP read_loop;
    
    -- 6. 关闭游标
    CLOSE emp_cursor;
    
    -- 7. 返回结果
    SELECT * FROM temp_results;
    
    -- 8. 清理临时表（可选）
    DROP TEMPORARY TABLE temp_results;
END$$

DELIMITER ;
```

### **示例2：带参数的游标**
```sql
DELIMITER $$

CREATE PROCEDURE cursor_with_parameter(
    IN dept_id INT,
    IN min_salary DECIMAL(10,2)
)
BEGIN
    DECLARE v_emp_id INT;
    DECLARE v_emp_name VARCHAR(100);
    DECLARE v_salary DECIMAL(10,2);
    DECLARE v_count INT DEFAULT 0;
    DECLARE v_total_salary DECIMAL(10,2) DEFAULT 0;
    DECLARE done INT DEFAULT 0;
    
    -- 声明带参数的游标
    DECLARE emp_cursor CURSOR FOR 
        SELECT employee_id, employee_name, salary
        FROM employees
        WHERE department_id = dept_id
          AND salary >= min_salary
        ORDER BY salary DESC;
    
    DECLARE CONTINUE HANDLER FOR NOT FOUND 
        SET done = 1;
    
    OPEN emp_cursor;
    
    -- 输出表头
    SELECT '员工ID', '姓名', '工资' 
    UNION ALL
    
    read_loop: LOOP
        FETCH emp_cursor INTO v_emp_id, v_emp_name, v_salary;
        
        IF done = 1 THEN
            LEAVE read_loop;
        END IF;
        
        -- 输出每一行
        SELECT v_emp_id, v_emp_name, v_salary;
        
        -- 统计
        SET v_count = v_count + 1;
        SET v_total_salary = v_total_salary + v_salary;
        
    END LOOP read_loop;
    
    CLOSE emp_cursor;
    
    -- 输出统计信息
    SELECT 
        CONCAT('符合条件的员工数: ', v_count) AS summary,
        CONCAT('总工资: ', FORMAT(v_total_salary, 2)) AS total_salary,
        CONCAT('平均工资: ', FORMAT(v_total_salary / NULLIF(v_count, 0), 2)) AS avg_salary;
END$$

DELIMITER ;
```

### **示例3：嵌套游标（游标中的游标）**
```sql
DELIMITER $$

CREATE PROCEDURE nested_cursors()
BEGIN
    DECLARE v_dept_id INT;
    DECLARE v_dept_name VARCHAR(50);
    DECLARE v_emp_id INT;
    DECLARE v_emp_name VARCHAR(50);
    DECLARE done1 INT DEFAULT 0;
    DECLARE done2 INT DEFAULT 0;
    
    -- 外层游标：遍历部门
    DECLARE dept_cursor CURSOR FOR 
        SELECT department_id, department_name 
        FROM departments 
        ORDER BY department_id;
    
    DECLARE CONTINUE HANDLER FOR NOT FOUND 
        SET done1 = 1;
    
    OPEN dept_cursor;
    
    dept_loop: LOOP
        FETCH dept_cursor INTO v_dept_id, v_dept_name;
        
        IF done1 = 1 THEN
            LEAVE dept_loop;
        END IF;
        
        -- 输出部门信息
        SELECT CONCAT('=== 部门: ', v_dept_name, ' (ID: ', v_dept_id, ') ===') AS department_info;
        
        -- 重置内层游标的状态
        SET done2 = 0;
        
        -- 内层游标：遍历该部门的员工
        BEGIN
            DECLARE emp_cursor CURSOR FOR 
                SELECT employee_id, employee_name
                FROM employees
                WHERE department_id = v_dept_id
                ORDER BY employee_name;
            
            DECLARE CONTINUE HANDLER FOR NOT FOUND 
                SET done2 = 1;
            
            OPEN emp_cursor;
            
            emp_loop: LOOP
                FETCH emp_cursor INTO v_emp_id, v_emp_name;
                
                IF done2 = 1 THEN
                    LEAVE emp_loop;
                END IF;
                
                -- 输出员工信息
                SELECT CONCAT('  - 员工: ', v_emp_name, ' (ID: ', v_emp_id, ')') AS employee_info;
                
            END LOOP emp_loop;
            
            CLOSE emp_cursor;
        END;
        
    END LOOP dept_loop;
    
    CLOSE dept_cursor;
END$$

DELIMITER ;
```

## 🚀 **高级游标技巧**

### **可滚动游标（MySQL 8.0+）**
```sql
DELIMITER $$

CREATE PROCEDURE scrollable_cursor()
BEGIN
    DECLARE v_id INT;
    DECLARE v_name VARCHAR(50);
    DECLARE done INT DEFAULT 0;
    
    -- 创建可滚动游标（需要MySQL 8.0+）
    -- 实际上MySQL存储过程中的游标默认是只向前的
    -- 但可以在应用程序中使用可滚动结果集
    
    -- 对于存储过程，通常这样模拟：
    DROP TEMPORARY TABLE IF EXISTS temp_cursor_data;
    
    -- 先把数据存到临时表
    CREATE TEMPORARY TABLE temp_cursor_data (
        row_num INT AUTO_INCREMENT PRIMARY KEY,
        id INT,
        name VARCHAR(50)
    );
    
    INSERT INTO temp_cursor_data (id, name)
    SELECT id, name FROM users ORDER BY id;
    
    -- 然后对临时表使用游标
    DECLARE cur CURSOR FOR 
        SELECT id, name FROM temp_cursor_data ORDER BY row_num;
    
    DECLARE CONTINUE HANDLER FOR NOT FOUND 
        SET done = 1;
    
    OPEN cur;
    
    -- 向前遍历
    SELECT '=== 向前遍历 ===';
    WHILE done = 0 DO
        FETCH cur INTO v_id, v_name;
        IF done = 0 THEN
            SELECT CONCAT('行: id=', v_id, ', name=', v_name);
        END IF;
    END WHILE;
    
    -- 注意：MySQL存储过程游标不能向后移动
    -- 如果需要双向移动，需要在应用程序中使用JDBC的可滚动结果集
    
    CLOSE cur;
    
    DROP TEMPORARY TABLE temp_cursor_data;
END$$

DELIMITER ;
```

### **带更新/删除的游标**
```sql
DELIMITER $$

CREATE PROCEDURE cursor_for_update()
BEGIN
    DECLARE v_order_id INT;
    DECLARE v_order_amount DECIMAL(10,2);
    DECLARE v_customer_id INT;
    DECLARE done INT DEFAULT 0;
    
    -- 使用 FOR UPDATE 锁定选中的行
    DECLARE order_cursor CURSOR FOR 
        SELECT order_id, amount, customer_id
        FROM orders
        WHERE status = 'PENDING'
          AND order_date < DATE_SUB(NOW(), INTERVAL 7 DAY)
        FOR UPDATE;  -- 锁定这些行，防止其他会话修改
    
    DECLARE CONTINUE HANDLER FOR NOT FOUND 
        SET done = 1;
    
    START TRANSACTION;
    
    OPEN order_cursor;
    
    process_loop: LOOP
        FETCH order_cursor INTO v_order_id, v_order_amount, v_customer_id;
        
        IF done = 1 THEN
            LEAVE process_loop;
        END IF;
        
        -- 自动取消过期订单
        UPDATE orders 
        SET status = 'CANCELLED', 
            cancelled_at = NOW(),
            cancel_reason = '超时未支付'
        WHERE order_id = v_order_id;
        
        -- 记录取消日志
        INSERT INTO order_cancel_logs 
            (order_id, customer_id, amount, cancel_time, reason)
        VALUES 
            (v_order_id, v_customer_id, v_order_amount, NOW(), '超时未支付');
        
        -- 可选：发送通知或退款逻辑
        
        SELECT CONCAT('已取消订单: ', v_order_id) AS message;
        
    END LOOP process_loop;
    
    CLOSE order_cursor;
    
    COMMIT;
END$$

DELIMITER ;
```

## 📝 **游标的最佳实践**

### **错误处理模板**
```sql
DELIMITER $$

CREATE PROCEDURE safe_cursor_template()
BEGIN
    DECLARE v_exit_handler INT DEFAULT 0;
    DECLARE v_error_msg VARCHAR(500);
    
    -- 主变量
    DECLARE v_id INT;
    DECLARE v_name VARCHAR(50);
    DECLARE done INT DEFAULT 0;
    
    -- 声明游标
    DECLARE cur CURSOR FOR 
        SELECT id, name FROM source_table;
    
    -- 错误处理：捕获所有异常
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        GET DIAGNOSTICS CONDITION 1
            v_error_msg = MESSAGE_TEXT;
        
        -- 关闭游标（如果打开了）
        IF v_exit_handler = 0 THEN
            CLOSE cur;
        END IF;
        
        -- 回滚事务（如果开始了）
        ROLLBACK;
        
        -- 记录错误
        INSERT INTO error_logs (error_message, error_time)
        VALUES (v_error_msg, NOW());
        
        -- 返回错误信息
        SELECT CONCAT('处理失败: ', v_error_msg) AS result;
    END;
    
    -- 正常处理：没有更多数据
    DECLARE CONTINUE HANDLER FOR NOT FOUND 
        SET done = 1;
    
    START TRANSACTION;
    
    OPEN cur;
    SET v_exit_handler = 1;  -- 标记游标已打开
    
    process_loop: LOOP
        FETCH cur INTO v_id, v_name;
        
        IF done = 1 THEN
            LEAVE process_loop;
        END IF;
        
        -- 业务逻辑
        -- 这里处理每一行数据
        
    END LOOP process_loop;
    
    CLOSE cur;
    SET v_exit_handler = 0;  -- 标记游标已关闭
    
    COMMIT;
    
    SELECT '处理完成' AS result;
    
END$$

DELIMITER ;
```

### **性能优化技巧**
```sql
DELIMITER $$

CREATE PROCEDURE optimized_cursor()
BEGIN
    DECLARE v_batch_size INT DEFAULT 1000;
    DECLARE v_processed INT DEFAULT 0;
    DECLARE v_total INT DEFAULT 0;
    DECLARE done INT DEFAULT 0;
    
    -- 1. 先获取总行数
    SELECT COUNT(*) INTO v_total FROM large_table;
    SELECT CONCAT('总记录数: ', v_total) AS info;
    
    -- 2. 分批处理（避免内存问题）
    DROP TEMPORARY TABLE IF EXISTS temp_batch;
    CREATE TEMPORARY TABLE temp_batch (
        batch_id INT AUTO_INCREMENT PRIMARY KEY,
        record_id INT
    );
    
    -- 3. 只存储ID，减少内存占用
    INSERT INTO temp_batch (record_id)
    SELECT id FROM large_table ORDER BY id;
    
    -- 4. 使用游标处理批次
    DECLARE batch_cursor CURSOR FOR 
        SELECT record_id FROM temp_batch ORDER BY batch_id;
    
    DECLARE CONTINUE HANDLER FOR NOT FOUND 
        SET done = 1;
    
    OPEN batch_cursor;
    
    batch_loop: LOOP
        FETCH batch_cursor INTO @current_id;
        
        IF done = 1 THEN
            LEAVE batch_loop;
        END IF;
        
        -- 处理单条记录（通过ID获取完整数据）
        CALL process_single_record(@current_id);
        
        SET v_processed = v_processed + 1;
        
        -- 显示进度
        IF v_processed % v_batch_size = 0 THEN
            SELECT CONCAT(
                '进度: ', v_processed, '/', v_total, ' (',
                ROUND(v_processed / v_total * 100, 2), '%)'
            ) AS progress;
        END IF;
        
    END LOOP batch_loop;
    
    CLOSE batch_cursor;
    
    SELECT CONCAT('处理完成，共处理 ', v_processed, ' 条记录') AS result;
    
    DROP TEMPORARY TABLE temp_batch;
END$$

DELIMITER ;
```

## ⚠️ **游标的限制和注意事项**

### **限制**
```sql
-- 1. 游标是只向前的（FORWARD_ONLY）
--    不能向后移动，不能随机访问

-- 2. 游标是只读的（READ_ONLY）
--    不能通过游标直接更新数据（需要用UPDATE语句）

-- 3. 敏感度
--    ASENSITIVE: 可能反映其他会话的更改（默认）
--    INSENSITIVE: 不反映其他会话的更改（创建临时副本）

-- 4. 声明顺序（重要！）
DECLARE var1 INT;                    -- 变量声明
DECLARE cur CURSOR FOR SELECT ...;   -- 然后游标声明
DECLARE CONTINUE HANDLER ...;        -- 最后处理程序声明
-- 顺序错会导致语法错误！

-- 5. 作用域
--    游标在声明的BEGIN...END块内有效
```

### **常见错误**
```sql
-- ❌ 错误1：处理程序声明在游标之前
DECLARE CONTINUE HANDLER FOR NOT FOUND ...;  -- 错！
DECLARE cur CURSOR FOR SELECT ...;           -- 应该在游标之后

-- ✅ 正确顺序：
DECLARE cur CURSOR FOR SELECT ...;
DECLARE CONTINUE HANDLER FOR NOT FOUND ...;

-- ❌ 错误2：重复打开已打开的游标
OPEN cur;
OPEN cur;  -- 错误！需要先关闭

-- ✅ 正确：
OPEN cur;
-- 使用游标...
CLOSE cur;
-- 需要时重新打开
OPEN cur;

-- ❌ 错误3：忘记检查结束条件（无限循环）
LOOP
    FETCH cur INTO var;  -- 没有检查NOT FOUND
    -- 处理数据...
END LOOP;

-- ✅ 正确：
LOOP
    FETCH cur INTO var;
    IF done THEN LEAVE; END IF;
    -- 处理数据...
END LOOP;
```

## 🎯 **游标 vs 普通查询**

| 场景             | 使用游标       | 使用普通查询 |
| ---------------- | -------------- | ------------ |
| **逐行处理**     | ✅ 适合         | ❌ 不适合     |
| **复杂业务逻辑** | ✅ 适合         | ❌ 不适合     |
| **大数据量**     | ❌ 不适合（慢） | ✅ 适合       |
| **简单统计**     | ❌ 不适合       | ✅ 适合       |
| **需要事务控制** | ✅ 适合         | ✅ 适合       |
| **性能要求高**   | ❌ 不适合       | ✅ 适合       |

## 💎 **总结**

### **什么时候用游标？**
1. **需要逐行处理**：每行数据需要不同的业务逻辑
2. **复杂计算**：需要基于前一行结果计算下一行
3. **数据转换**：需要把数据从一种格式转换成另一种
4. **级联操作**：处理一行数据需要触发多个操作

### **什么时候不用游标？**
1. **简单查询**：直接用 SELECT
2. **批量更新**：用 UPDATE ... WHERE
3. **数据统计**：用 GROUP BY + 聚合函数
4. **性能敏感**：游标通常比集合操作慢

### **游标使用口诀**
```
一声明变量，二声明游标
三声明处理，四打开它
循环取数据，记得查结束
处理完关闭，错误要捕获
```

**记住**：游标是强大的工具，但应该谨慎使用。在大多数情况下，集合操作（SQL语句）比游标更高效！

