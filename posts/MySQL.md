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
