---
date: 2026-08-25
aliases:
  - 表连接
---
# SQL 连接查询（表连接）

---

## 一、基本概念

### 定义

表连接是指将两张表或多张表**横向拼接**到一起，形成一个临时的大表。

### 笛卡尔积

两张表的数据**相乘**，产生所有可能的行组合。

```sql
SELECT * FROM emp, dept;   -- 笛卡尔积：17 × 6 = 102 行
```

> ⚠️ 笛卡尔积会产生大量错误数据，工作中应尽量避免。

---

## 二、表连接分类

| 类型 | 关键字 | 说明 |
|------|--------|------|
| 内连接 | `INNER JOIN` | 只显示匹配成功的数据 |
| 左外连接 | `LEFT [OUTER] JOIN` | 左表全部显示，右表不匹配填 NULL |
| 右外连接 | `RIGHT [OUTER] JOIN` | 右表全部显示，左表不匹配填 NULL |
| 全外连接 | `FULL [OUTER] JOIN` | 两表全部显示，不匹配填 NULL |
| 交叉连接 | `CROSS JOIN` | 类似笛卡尔积，无关联条件 |
| 自然连接 | `NATURAL JOIN` | 自动匹配相同列名 |
| 自连接 | — | 自己表和自己表相连（特殊形式） |
| 不等值连接 | — | 关联条件不用等号（特殊形式） |
![[sql joins韦恩图.png]]

---

## 三、内连接 INNER JOIN

### 语法

```sql
SELECT  * | 列名 | 常量 | 函数 | 计算 | (子查询)
  FROM  表名1 | (子查询)
 INNER JOIN  表名2 | (子查询)
    ON  关联条件 ... AND | OR
 INNER JOIN  表名3 | (子查询)
    ON  关联条件 ...
  ...
 WHERE   分组前过滤条件          -- 不能使用聚合函数
 GROUP BY  分组内容
 HAVING  分组后过滤条件          -- 可以使用聚合函数
 ORDER BY  排序内容 ASC | DESC;
```

### 特点

按照关联条件匹配两张表，**只显示匹配成功**的数据，匹配不成功的忽略不显示。

### 示例

```sql
-- 将 emp 表和 dept 表用内连接连接到一起
SELECT *
  FROM emp
 INNER JOIN dept
    ON emp.deptno = dept.deptno;    -- 关联条件是固定的

-- 查询员工姓名、岗位、薪资、部门编号、部门名称及部门地址（地址为纽约）
SELECT ename, job, sal, dept.deptno, dname, loc
  FROM emp
 INNER JOIN dept
    ON emp.deptno = dept.deptno
 WHERE loc = 'NEW YORK';
```

---

## 四、外连接

### 4.1 左外连接 LEFT JOIN

#### 语法

```sql
FROM 表名1  LEFT JOIN  表名2  ON 关联条件 ...
```

#### 特点

- **左表为主**，左表数据全部显示
- 匹配不成功的，右表以 NULL 填充
- 右表匹配不成功的忽略不显示

#### 示例

```sql
SELECT *
  FROM emp
  LEFT JOIN dept
    ON emp.deptno = dept.deptno;   -- 结果 17 行（以 emp 为主）

-- 查询没有员工的部门信息
SELECT *
  FROM dept
  LEFT JOIN emp
    ON dept.deptno = emp.deptno
 WHERE empno IS NULL;               -- 右表的主键列为空
```

> **技巧**：查询"没有 XX 的 YY"时，用 `LEFT JOIN`，`WHERE 右表主键 IS NULL`。

---

### 4.2 右外连接 RIGHT JOIN

#### 语法

```sql
FROM 表名1  RIGHT JOIN  表名2  ON 关联条件
```

#### 特点

- **右表为主**，右表数据全部显示
- 匹配不成功的，左表以 NULL 填充

#### 示例

```sql
SELECT *
  FROM emp        -- 17 行
 RIGHT JOIN dept  -- 6 行
    ON emp.deptno = dept.deptno;   -- 结果 20 行（以 dept 为主）
```

---

### 4.3 全外连接 FULL JOIN

#### 语法

```sql
FROM 表名1  FULL JOIN  表名2  ON 关联条件
```

#### 特点

两张表数据**全部显示**，匹配不成功时对应的表以 NULL 填充。

#### 示例

```sql
SELECT *
  FROM emp        -- 17 行
  FULL JOIN dept  -- 6 行
    ON emp.deptno = dept.deptno;   -- 结果 20 行
```

---

## 五、交叉连接 CROSS JOIN

类似笛卡尔积，**没有关联条件**。

```sql
SELECT *
  FROM emp
 CROSS JOIN dept;    -- 17 × 6 = 102 行
```

---

## 六、特殊连接

### 6.1 自然连接 NATURAL JOIN

**没有关联条件**，会自动寻找相同列名进行匹配，只显示匹配成功的数据，并**去除重复列**。

```sql
SELECT *
  FROM emp
 NATURAL JOIN dept;
```

### 6.2 自连接

没有新关键字，是一种**特殊形式**——自己表和自己表相连。通过**表别名**区分。

```sql
-- 查询员工信息以及对应的领导信息
SELECT *
  FROM emp a         -- 员工信息
 INNER JOIN emp b    -- 领导信息
    ON a.mgr = b.empno;

-- 使用左连接（包含没有领导的员工，如 KING）
SELECT *
  FROM emp a
  LEFT JOIN emp b
    ON a.mgr = b.empno;
```

#### 练习：查询 emp 表中所有的领导信息

```sql
-- 方法一：左连接 + 排除无下属的
SELECT DISTINCT a.*
  FROM emp a          -- 领导信息
  LEFT JOIN emp b     -- 员工信息
    ON a.empno = b.mgr
 WHERE b.empno IS NOT NULL;

-- 方法二：内连接（更简洁）
SELECT *
  FROM emp a          -- 领导信息
 INNER JOIN emp b     -- 员工信息
    ON a.empno = b.mgr;
```

### 6.3 不等值连接

关联条件**不用等号**（如 `BETWEEN`、`>`、`<` 等），也是一种特殊形式。

```sql
-- 查询员工信息以及对应的工资等级
SELECT *
  FROM emp
  LEFT JOIN salgrade
    ON sal BETWEEN losal AND hisal;
```

---

## 七、WHERE 与 ON 的区别

| 对比项  | ON                      | WHERE                     |
| ---- | ----------------------- | ------------------------- |
| 作用   | **关联条件**，决定如何连接两张表      | **过滤条件**，对连接后的临时大表进行过滤    |
| 执行顺序 | `FROM → ON → JOIN`      | `WHERE`（在 JOIN 之后执行）      |
| 内连接时 | 结果**没有区别**              | 结果**没有区别**                |
| 左连接时 | 放在 `ON` 后面，左表数据**全部显示** | 放在 `WHERE` 后面，不符合条件的会被过滤掉 |

### SQL 执行顺序

```
FROM → ON → JOIN → WHERE → GROUP BY → HAVING → SELECT → DISTINCT → ORDER BY
```

---

## 八、综合练习

### 1. 查询和 SCOTT 相同部门的员工姓名和雇用日期（两种方法）

```sql
-- 方法一：子查询
SELECT ename, hiredate
  FROM emp
 WHERE deptno = (SELECT deptno FROM emp WHERE ename = 'SCOTT');

-- 方法二：表连接（自连接）
SELECT a.ename, a.hiredate
  FROM emp a
  LEFT JOIN emp b
    ON a.deptno = b.deptno
 WHERE b.ename = 'SCOTT';
```

### 2. 查询和姓名中包含字母 U 的员工在相同部门的员工的员工号和姓名（两种方法）

```sql
-- 方法一：子查询
SELECT ename, empno
  FROM emp
 WHERE deptno IN (SELECT deptno FROM emp WHERE ename LIKE '%U%');

-- 方法二：表连接
SELECT a.empno, a.ename
  FROM emp a
  LEFT JOIN emp b
    ON a.deptno = b.deptno
 WHERE b.ename LIKE '%U%';
```

### 3. 查询在 loc 为 NEW YORK 的部门工作的员工姓名、部门名称和岗位（两种方法）

```sql
-- 方法一：表连接
SELECT ename, dname, job
  FROM emp a
  LEFT JOIN dept b
    ON a.deptno = b.deptno
 WHERE b.loc = 'NEW YORK';

-- 方法二：子查询
SELECT ename,
       (SELECT dname FROM dept WHERE loc = 'NEW YORK'),
       job
  FROM emp
 WHERE deptno = (SELECT deptno FROM dept WHERE loc = 'NEW YORK');
```

### 4. 查询工资比 SMITH 高且工作地点在 CHICAGO 的员工姓名、工资、部门名称

```sql
-- 方法一：多表连接
SELECT a.ename, a.sal, b.dname
  FROM emp a
  LEFT JOIN dept b
    ON a.deptno = b.deptno
 INNER JOIN emp c       -- 表示 SMITH
    ON a.sal > c.sal
 WHERE b.loc = 'CHICAGO'
   AND c.ename = 'SMITH';

-- 方法二：子查询
SELECT a.ename, a.sal, b.dname
  FROM emp a
  LEFT JOIN dept b
    ON a.deptno = b.deptno
 WHERE (SELECT sal FROM emp WHERE ename = 'SMITH') < a.sal
   AND b.loc = 'CHICAGO';
```

### 5. 查询部门人数大于所有部门平均人数的部门编号、部门名称、部门人数

```sql
SELECT dept.deptno, dept.dname, COUNT(empno)
  FROM emp
  LEFT JOIN dept
    ON emp.deptno = dept.deptno
 GROUP BY dept.deptno, dept.dname
HAVING COUNT(empno) > (
       (SELECT COUNT(1) FROM emp) / (SELECT COUNT(1) FROM dept));
```

### 6. 查询比自己部门平均工资高的员工姓名、工资、部门编号、部门平均工资

```sql
SELECT a.ename, a.sal, a.deptno, b.v AS 平均工资
  FROM emp a
 INNER JOIN (SELECT deptno, AVG(sal) v FROM emp GROUP BY deptno) b
    ON a.deptno = b.deptno
 WHERE a.sal > b.v;
```

### 7. 列出至少有三个员工的所有部门和部门信息

```sql
SELECT a.deptno, COUNT(b.empno)
  FROM dept a
  LEFT JOIN emp b
    ON a.deptno = b.deptno
 GROUP BY a.deptno
HAVING COUNT(b.empno) >= 3;
```

### 8. 列出受雇日期早于直接上级的所有员工的编号、姓名、部门名称

```sql
SELECT a.empno, a.ename, c.dname
  FROM emp a          -- 员工
  LEFT JOIN emp b     -- 领导
    ON a.mgr = b.empno
  LEFT JOIN dept c    -- 部门
    ON a.deptno = c.deptno
 WHERE a.hiredate < b.hiredate;
```

### 9. 列出职位为 CLERK 的员工姓名及其所在部门名称、部门人数

```sql
SELECT b.dname, COUNT(empno)
  FROM emp a
  LEFT JOIN dept b
    ON a.deptno = b.deptno
 WHERE a.job = 'CLERK'
 GROUP BY b.dname;
```

### 10. 列出和 SCOTT 从事相同工作的所有员工及部门名称

```sql
SELECT a.ename, c.dname
  FROM emp a
  LEFT JOIN emp b
    ON a.job = b.job
  LEFT JOIN dept c
    ON a.deptno = c.deptno
 WHERE b.ename = 'SCOTT';
```

---

## 九、核心要点速记

1. **笛卡尔积**：两表数据相乘，应尽量避免
2. **内连接**：只显示匹配成功的数据
3. **左连接**：左表全部显示，右表不匹配填 NULL
4. **右连接**：右表全部显示，左表不匹配填 NULL
5. **全连接**：两表全部显示，不匹配填 NULL
6. **自连接**：自己连自己，用别名区分
7. **不等值连接**：关联条件用 `BETWEEN`、`>` 等非等号运算符
8. **WHERE vs ON**：内连接无区别；左连接时放 `ON` 保证左表全部显示
9. **执行顺序**：`FROM → ON → JOIN → WHERE → GROUP BY → HAVING → SELECT → DISTINCT → ORDER BY`
