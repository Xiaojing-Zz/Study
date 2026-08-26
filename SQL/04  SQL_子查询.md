---
date: 2026-08-24
---
# SQL 子查询

---
## 一、子查询的概念

子查询是指一条语句嵌套一条或者多条查询语句。

- 外侧的查询叫做**主查询**，里层的查询叫做**子查询**
- 里层的子查询需要加上 `()`
- 嵌套的形式可以是**层层嵌套**，也可以是**平行嵌套**

```sql
-- 层层嵌套
SELECT (SELECT (SELECT ...))

-- 平行嵌套
SELECT (SELECT ...), (SELECT ...), (SELECT ...)
```

---

## 二、子查询的分类

### 1. 相关子查询 —— 要的是**关系**

里层的查询与外层的查询有一定的关系。

- 子查询会牵扯主查询的内容，所以**不能单独运行**
- 主查询要的是与子查询的**关系**

### 2. 非相关子查询 —— 要的是**结果**

里层的子查询与外层的主查询没有任何关系。

- 不会牵扯主查询的任何内容，所以子查询**能单独运行**
- 主查询要的是子查询的**结果**

---

## 三、非相关子查询按返回结果分类

| 分类 | 示例 | 用途 |
|------|------|------|
| (1) 单行单列 | `SELECT AVG(sal) FROM emp` | `=` `>` `<` 比较 |
| (2) 单行多列 | `SELECT * FROM emp WHERE empno=7566` | `(col1,col2) = (子查询)` |
| (3) 单列多行 | `SELECT ename FROM emp` | `IN` `ANY` `ALL` |
| (4) 多列多行 | `SELECT * FROM emp` | 放在 `FROM` 后面当表 |

---

## 四、子查询解题三步法

> **第一步**：写主查询，子查询部分用**汉语**表达清楚
> **第二步**：单独写子查询，实现括号里的汉语
> **第三步**：合并

以"查询和刘亦菲在同一个部门的员工信息"为例：

```sql
-- 第一步：主查询
SELECT * FROM emp WHERE deptno = (刘亦菲的部门编号)

-- 第二步：子查询
SELECT deptno FROM emp WHERE ename='刘亦菲'

-- 第三步：合并
SELECT * FROM emp
WHERE deptno = (SELECT deptno FROM emp WHERE ename='刘亦菲')
```

---

## 五、(1) 单行单列子查询

### 例题1：查询和刘亦菲同一个部门的员工信息

```sql
SELECT * FROM emp
WHERE deptno = (SELECT deptno FROM emp WHERE ename='刘亦菲')
```

### 例题2：查询部门和ALLEN相同但岗位不同的员工信息

```sql
-- 主查询
SELECT * FROM emp
WHERE deptno = (ALLEN的部门编号) AND job <> (ALLEN的岗位)

-- 子查询
SELECT deptno FROM emp WHERE ename='ALLEN'
SELECT job FROM emp WHERE ename='ALLEN'

-- 合并
SELECT * FROM emp
WHERE deptno = (SELECT deptno FROM emp WHERE ename='ALLEN')
  AND job <> (SELECT job FROM emp WHERE ename='ALLEN')
```

---

## 六、(2) 单行多列子查询

子查询返回多个列，主查询条件列的**个数、顺序、属性**要和子查询结果一致。

### 例题：查询和 WARD 部门及岗位相同的员工信息

```sql
-- 主查询：用 (col1,col2) 接收
SELECT * FROM emp
WHERE (deptno, job) = (WARD的部门和岗位)

-- 子查询
SELECT deptno, job FROM emp WHERE ename='WARD'

-- 合并
SELECT * FROM emp
WHERE (deptno, job) = (SELECT deptno, job FROM emp WHERE ename='WARD')
```

### 练习：查询部门岗位以及薪资都和SCOTT相同的员工信息

```sql
SELECT * FROM emp
WHERE (deptno, job, sal) = (SELECT deptno, job, sal FROM emp WHERE ename='SCOTT')
```

---

## 七、(3) 单列多行子查询 —— `IN` / `ANY` / `ALL`

子查询返回多行单列，结果看成一个**集合**。

### `IN` —— 等于集合中的任意一个

#### 例题：查询和 KING 或刘亦菲部门相同的员工信息

```sql
SELECT * FROM emp
WHERE deptno IN (SELECT deptno FROM emp WHERE ename='KING' OR ename='刘亦菲')
```

`IN` 等价于 `=ANY`：

```sql
SELECT * FROM emp
WHERE deptno =ANY (SELECT deptno FROM emp WHERE ename='KING' OR ename='刘亦菲')
```

### `NOT IN` —— 不等于集合中的任何一个

#### 例题：查询不是领导的人

```sql
SELECT * FROM emp
WHERE empno NOT IN (SELECT DISTINCT mgr FROM emp WHERE mgr IS NOT NULL)
```

> **重要：** 使用 `NOT IN` 时，子查询结果中**不能有空值（NULL）**。因为 `NOT IN` 遇到 NULL 会导致整个条件结果为 UNKNOWN，从而查不出任何数据。

---

## 八、(4) 多列多行子查询 —— 放在 FROM 后面当表

### 例题：查询每个部门的平均工资中最大的值

```sql
-- 子查询作为一张临时表
SELECT MAX(a)
FROM (SELECT deptno, AVG(sal) a FROM emp GROUP BY deptno)
```

---

## 九、相关子查询

### 例题：查询员工的姓名、岗位、薪资、部门编号以及部门名称和部门地址

```sql
SELECT ename, job, sal, deptno,
       (SELECT dname FROM dept WHERE dept.deptno = emp.deptno),  -- 和主查询部门编号相同的部门名称
       (SELECT loc FROM dept WHERE dept.deptno = emp.deptno)     -- 和主查询部门编号相同的部门地址
FROM emp
```

### 练习1：查询员工姓名、岗位、薪资、部门编号以及岗位平均薪资

```sql
SELECT ename, job, sal, deptno,
       (SELECT AVG(sal) FROM emp b WHERE b.job = a.job) AS avg_sal
FROM emp a
```

### 练习2：查询员工姓名、岗位、薪资、部门编号以及部门平均工资，要求工资高于自己部门的平均工资

```sql
SELECT ename, job, sal, deptno,
       (SELECT AVG(sal) FROM emp b WHERE b.deptno = a.deptno) AS dept_avg_sal
FROM emp a
WHERE sal > (SELECT AVG(sal) FROM emp WHERE a.deptno = deptno)
```

---

## 十、子查询的位置与注意事项

### 子查询可以放在哪里

| 位置 | 说明 |
|------|------|
| `SELECT` 后面 | 必须是**单行单列**或相关子查询（标量子查询） |
| `WHERE` 后面 | 各种类型都可以 |
| `FROM` 后面 | 多列多行子查询当临时表 |
| `HAVING` 后面 | 聚合条件中使用 |
| `GROUP BY` 后面 | **不能放子查询** |

### SELECT 后面的子查询规则

```sql
-- 合法：单行单列
SELECT ename, (SELECT MAX(sal) FROM emp) FROM emp

-- 合法：相关子查询（返回单行单列）
SELECT ename, job, (SELECT MAX(sal) FROM emp b WHERE a.job = b.job) FROM emp a

-- 不合法：多列
SELECT ename, (SELECT job, sal FROM emp) FROM emp   -- 报错！

-- 不合法：多行
SELECT ename, (SELECT sal FROM emp) FROM emp   -- 报错！
```

---

## 十一、经典例题汇总

### 基础练习

**1. 查询SMITH所在部门的所有员工信息**

```sql
SELECT * FROM emp
WHERE deptno = (SELECT deptno FROM emp WHERE ename='SMITH')
```

**2. 查询BLAKE带领的员工有哪些**

```sql
SELECT * FROM emp
WHERE mgr = (SELECT empno FROM emp WHERE ename='BLAKE')
```

**3. 查询BLAKE的领导手下有哪些员工**

```sql
SELECT * FROM emp
WHERE mgr = (SELECT mgr FROM emp WHERE ename='BLAKE')
```

**4. 查询与SMITH同部门且薪资相等的员工**

```sql
SELECT * FROM emp
WHERE (deptno, sal) = (SELECT deptno, sal FROM emp WHERE ename='SMITH')
```

**5. 查询与SMITH同部门同薪资或与JAMES同部门同薪资的员工**

```sql
SELECT * FROM emp
WHERE (deptno, sal) IN (
    SELECT deptno, sal FROM emp WHERE ename='SMITH' OR ename='JAMES'
)
```

**6. 查询公司内薪资最高的员工**

```sql
SELECT * FROM emp
WHERE sal = (SELECT MAX(sal) FROM emp)
```

**7. 查询公司内各部门薪资最高的员工**

```sql
-- 相关子查询写法
SELECT * FROM emp a
WHERE sal = (SELECT MAX(sal) FROM emp b WHERE b.deptno = a.deptno)

-- 显示部门最高工资列
SELECT a.*, (SELECT MAX(sal) FROM emp b WHERE b.deptno = a.deptno) AS max_sal
FROM emp a
WHERE sal = (SELECT MAX(sal) FROM emp b WHERE b.deptno = a.deptno)
```

**8. 查询公司内哪个部门的平均工资高于整个公司的平均工资**

```sql
-- 方法一：相关子查询
SELECT DISTINCT deptno,
       (SELECT AVG(sal) FROM emp WHERE deptno = e.deptno),
       (SELECT AVG(sal) FROM emp)
FROM emp e
WHERE (SELECT AVG(sal) FROM emp WHERE deptno = e.deptno)
    > (SELECT AVG(sal) FROM emp)

-- 方法二：GROUP BY + HAVING（更简洁）
SELECT deptno, AVG(sal)
FROM emp
GROUP BY deptno
HAVING AVG(sal) > (SELECT AVG(sal) FROM emp)
```

**9. 查询公司内没有员工的部门**

```sql
-- 方法一：NOT IN
SELECT * FROM dept
WHERE deptno NOT IN (SELECT deptno FROM emp WHERE deptno IS NOT NULL)

-- 方法二：<> ALL
SELECT * FROM dept
WHERE deptno <>ALL (SELECT deptno FROM emp)
```

### SELECT 后的子查询

**1. 查询员工的姓名、岗位、薪资及部门名称和工作地点**

```sql
SELECT ename, job, sal,
       (SELECT dname FROM dept WHERE dept.deptno = emp.deptno),
       (SELECT loc FROM dept WHERE dept.deptno = emp.deptno)
FROM emp
```

**2. 查询10号部门与20号部门在平均薪资上相差了多少**

```sql
-- 用 dual 虚拟表，只返回一行
SELECT (SELECT AVG(sal) FROM emp WHERE deptno = 10)
     - (SELECT AVG(sal) FROM emp WHERE deptno = 20) AS diff
FROM dual
```

### 综合练习

**1. 查询每个部门的最大工资数，并显示该员工名字、部门名称、部门区域，按部门编号排序**

```sql
SELECT ename, deptno,
       (SELECT dname FROM dept WHERE dept.deptno = a.deptno),
       (SELECT loc FROM dept WHERE dept.deptno = a.deptno)
FROM emp a
WHERE sal = (SELECT MAX(sal) FROM emp b WHERE b.deptno = a.deptno)
ORDER BY deptno
```

**2. 查询平均工资比30号部门高的部门中员工的信息，并显示当前部门的平均工资**

```sql
SELECT a.*,
       (SELECT AVG(sal) FROM emp b WHERE b.deptno = a.deptno) AS dept_avg
FROM emp a
WHERE deptno IN (
    SELECT deptno FROM emp
    GROUP BY deptno
    HAVING AVG(sal) > (SELECT AVG(sal) FROM emp WHERE deptno = 30)
)
```

**3. 查询平均工资比30号部门高的部门中员工的信息，显示当前部门的平均工资和部门名称**

```sql
SELECT a.*,
       (SELECT AVG(sal) FROM emp b WHERE b.deptno = a.deptno) AS dept_avg,
       (SELECT dname FROM dept WHERE dept.deptno = a.deptno)
FROM emp a
WHERE deptno IN (
    SELECT deptno FROM emp
    GROUP BY deptno
    HAVING AVG(sal) > (SELECT AVG(sal) FROM emp WHERE deptno = 30)
)
```

**4. 查询工资高于自己所在部门平均工资的员工信息**

```sql
SELECT * FROM emp a
WHERE sal > (SELECT AVG(sal) FROM emp b WHERE b.deptno = a.deptno)
```

**5. 在第4题基础上显示部门名称和所在地区**

```sql
SELECT a.*,
       (SELECT dname FROM dept WHERE dept.deptno = a.deptno),
       (SELECT loc FROM dept WHERE dept.deptno = a.deptno)
FROM emp a
WHERE sal > (SELECT AVG(sal) FROM emp b WHERE b.deptno = a.deptno)
```

**6. 查询工资比SCOTT所在部门平均工资高的员工，且该员工所在部门的平均工资也要比NGAO所在部门的平均工资高**

```sql
SELECT * FROM emp a
WHERE sal >
      (SELECT AVG(sal) FROM emp
       WHERE deptno = (SELECT deptno FROM emp WHERE ename = 'SCOTT'))
  AND (SELECT AVG(sal) FROM emp b WHERE b.deptno = a.deptno) >
      (SELECT AVG(sal) FROM emp
       WHERE deptno = (SELECT deptno FROM emp WHERE ename = 'NGAO'))
```

**9. 查询工资相同的员工的工资和姓名**

```sql
-- 方法一：IN + GROUP BY HAVING（最简单）
SELECT ename, sal FROM emp
WHERE sal IN (SELECT sal FROM emp GROUP BY sal HAVING COUNT(*) > 1)

-- 方法二：EXISTS
SELECT ename, sal FROM emp e
WHERE EXISTS (SELECT 1 FROM emp WHERE sal = e.sal AND empno <> e.empno)

-- 方法三：相关子查询 COUNT
SELECT ename, sal FROM emp e
WHERE (SELECT COUNT(*) FROM emp WHERE sal = e.sal) > 1
```

**10. 列出薪金比SMITH多的所有雇员**

```sql
SELECT * FROM emp
WHERE sal > (SELECT sal FROM emp WHERE ename = 'SMITH')
```
