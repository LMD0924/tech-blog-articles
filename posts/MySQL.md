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

```sql
select employee_id,last_name,department_name from employee e join departments d using (department_id);
```

## 函数

