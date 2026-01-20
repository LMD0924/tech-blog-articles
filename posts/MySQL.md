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

