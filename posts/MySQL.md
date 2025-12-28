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


