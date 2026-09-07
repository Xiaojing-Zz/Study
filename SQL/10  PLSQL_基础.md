---
date: 2026-08-31
---

# PL/SQL 基础笔记

> PL/SQL = **P**rocedural **L**anguage / **SQL** —— Oracle 的核心过程化查询语言
>
> SQL 的进阶，学习程序块：**变量、变量类型、判断、循环**

---

## 一、SQL 复习

### 1.1 子查询 —— 查找每个部门中薪资最高的员工姓名

```sql
SELECT ename
  FROM emp
 WHERE sal IN (SELECT MAX(sal) FROM emp GROUP BY deptno);
```

### 1.2 数学函数对比

```sql
SELECT CEIL(-97.342)    ,  -- 向上取整  → -97
       FLOOR(-97.342)   ,  -- 向下取整  → -98
       ROUND(-97.342)   ,  -- 四舍五入  → -97
       TRUNC(-97.342)      -- 截断取整  → -97
  FROM dual;
```

| 函数 | 说明 | 结果 |
|------|------|------|
| `CEIL` | 向上取整（往正无穷方向） | -97 |
| `FLOOR` | 向下取整（往负无穷方向） | -98 |
| `ROUND` | 四舍五入 | -97 |
| `TRUNC` | 直接截断小数部分 | -97 |

### 1.3 自连接

```sql
-- 不等自连接：name 不同的所有组合
SELECT * FROM team a, team b WHERE a.name <> b.name;

-- 大于自连接：按 name 排序去重组合
SELECT * FROM team a, team b WHERE a.name > b.name;

-- 等价的 INNER JOIN 写法
SELECT * FROM team a INNER JOIN team b ON a.name <> b.name;
```

### 1.4 分组 + HAVING 综合练习

```sql
SELECT a.tid, COUNT(1)
  FROM testdb a        -- 原表，用来展示
  LEFT JOIN testdb b   -- 关联条件表
    ON a.m = b.m
   AND a.d > b.d
 WHERE b.tid = 101
 GROUP BY a.tid
HAVING COUNT(1) = 3;
```

---

## 二、PL/SQL 概述

### 2.1 什么是 PL/SQL

- **P** = Procedural（过程化）
- **L** = Language（语言）
- **SQL** = 结构化查询语言
- 是 **Oracle 的核心语言**，SQL 的超集

### 2.2 程序块的分类

| 类型 | 特点 | 示例 |
|------|------|------|
| **有名块** | 有名字，可永久保存到数据库中，随时调用 | 函数、存储过程 |
| **无名块** | 没有名字，临时执行，不能保存到数据库 | 匿名块 |

---

## 三、无名块（匿名块）结构

### 3.1 基本语法

```sql
DECLARE          ---------- 声明部分（可选）
  -- 声明变量、变量类型、游标……

BEGIN            ---------- 执行部分（必须）—— 程序块的核心
  -- DML 语句：INSERT / DELETE / UPDATE（直接使用）
  -- TCL 语句：COMMIT / ROLLBACK（直接使用）
  -- SELECT INTO 语句（可以使用）
  -- DDL 语句：不能直接使用，需通过动态 SQL 实现
  -- 判断、循环

EXCEPTION        ---------- 异常部分（可选）

END;             ---------- 结束部分（必须）
```

### 3.2 结构说明

| 部分 | 关键字 | 是否必须 | 说明 |
|------|--------|----------|------|
| 声明部分 | `DECLARE` | 可选 | 声明变量、类型、游标等 |
| 执行部分 | `BEGIN ... END` | **必须** | 程序核心，放置 SQL 和流程控制 |
| 异常部分 | `EXCEPTION` | 可选 | 捕获和处理运行时错误 |

---

## 四、变量

> 变量：用来接收可以变动的值，目前只能接收**一个值**

### 4.1 声明变量的语法

```sql
DECLARE
  变量名1  变量类型;    -- 每条语句以分号结尾
  变量名2  变量类型;
  -- ……
BEGIN
  -- ……
END;
```

> **命名建议**：使用有意义的名称，如 `v_ename`、`v_sal`（前缀 `v_` 表示变量）

### 4.2 基本示例 —— 打印输出"你好智云"

```sql
DECLARE
  v_a   VARCHAR2(30);
  v_a1  VARCHAR2(30);
BEGIN
  v_a := '你好智云';
  DBMS_OUTPUT.PUT_LINE(v_a);   -- 打印输出
END;
```

> **注意**：变量可以被重新赋值，后赋的值会覆盖前面的值。

```sql
DECLARE
  v_a   VARCHAR2(30);
BEGIN
  v_a := '你好智云';
  v_a := '李许铭';            -- 覆盖了上一行的值
  DBMS_OUTPUT.PUT_LINE(v_a);  -- 输出：李许铭
END;
```

> **注意**：未赋值的变量默认为 `NULL`。

```sql
DECLARE
  v_a   VARCHAR2(30);   -- 未赋值
BEGIN
  DBMS_OUTPUT.PUT_LINE(v_a);  -- 输出：NULL（空）
END;
```

### 练习：打印输出 hello world

```sql
DECLARE
  v_a  VARCHAR2(20);
BEGIN
  v_a := 'hello world';
  DBMS_OUTPUT.PUT_LINE(v_a);
END;
```

---

## 五、给变量赋值的三种方式

### 方式一：在 BEGIN 中用 `:=` 赋值

```sql
DECLARE
  变量1  变量类型;
BEGIN
  变量1 := 值;
END;
```

**示例：**

```sql
DECLARE
  v_a  VARCHAR2(20);
BEGIN
  v_a := 'hello world';
  DBMS_OUTPUT.PUT_LINE(v_a);
END;
```

### 方式二：在 DECLARE 中用 `:=` 赋初始值

```sql
DECLARE
  变量1  变量类型 := 值;
BEGIN
  -- ……
END;
```

**示例：**

```sql
DECLARE
  v_a  VARCHAR2(20) := 'hello world';
BEGIN
  DBMS_OUTPUT.PUT_LINE(v_a);
END;
```

> 方式一和方式二**没有本质区别**，只是赋值的位置不同。

### 方式三：通过 SELECT INTO 语句赋值

```sql
DECLARE
  变量1  变量类型;
  变量2  变量类型;
BEGIN
  SELECT 列名1, 列名2, …
    INTO 变量1, 变量2, …
    FROM 表名
   WHERE …;
  -- 变量的个数、顺序、类型要和查询结果一致
  -- 此时只能处理一行数据
END;
```

**示例：打印 7566 号员工的姓名、岗位和薪资**

```sql
DECLARE
  v_ename  VARCHAR2(20);
  v_job    VARCHAR2(10);
  v_sal    NUMBER;
BEGIN
  SELECT ename, job, sal
    INTO v_ename, v_job, v_sal
    FROM emp
   WHERE empno = 7566;

  DBMS_OUTPUT.PUT_LINE(v_ename || ' ' || v_job || ' ' || v_sal);
END;
```

### SELECT INTO 的常见错误

| 错误场景 | 说明 |
|----------|------|
| 查询返回**多行** | `INTO` 只能接收一行，多行会报错 |
| 变量个数与列数不匹配 | 数量、顺序、类型必须一致 |
| 查询返回**零行** | 会抛出 `NO_DATA_FOUND` 异常 |

---

## 六、SELECT INTO 综合练习

### 练习 1：打印 7566 的姓名、岗位、薪资、部门编号、部门名称、部门地址、工资等级

```sql
DECLARE
  v_ename   VARCHAR2(20);
  v_job     VARCHAR2(20);
  v_sal     NUMBER;
  v_deptno  NUMBER;
  v_dname   VARCHAR2(20);
  v_loc     VARCHAR2(20);
  v_grade   NUMBER;
BEGIN
  SELECT ename, job, sal, a.deptno, dname, loc, grade
    INTO v_ename, v_job, v_sal, v_deptno, v_dname, v_loc, v_grade
    FROM emp a
    LEFT JOIN dept b      ON a.deptno = b.deptno
    LEFT JOIN salgrade c  ON a.sal BETWEEN c.losal AND c.hisal
   WHERE empno = 7566;

  DBMS_OUTPUT.PUT_LINE(
    v_ename || '  ' || v_job || '  ' || v_sal || '  ' ||
    v_deptno || '  ' || v_dname || '  ' || v_loc || '  ' || v_grade
  );
END;
```

### 练习 2：打印 7566 的姓名、岗位、薪资、部门编号、部门平均工资、部门名称

```sql
DECLARE
  v_ename   CHAR(10);
  v_job     CHAR(10);
  v_sal     NUMBER;
  v_deptno  NUMBER;
  v_avg     NUMBER;
  v_dname   CHAR(10);
BEGIN
  SELECT ename, job, sal, a.deptno,
         (SELECT AVG(sal) FROM emp b WHERE b.deptno = a.deptno),
         dname
    INTO v_ename, v_job, v_sal, v_deptno, v_avg, v_dname
    FROM emp a
    LEFT JOIN dept ON a.deptno = dept.deptno
   WHERE empno = 7566;

  DBMS_OUTPUT.PUT_LINE(
    v_ename || '  ' || v_job || '  ' || v_sal || '  ' ||
    v_deptno || '  ' || v_avg || '  ' || v_dname
  );
END;
```

### 关联子查询：与主查询相同部门的平均工资

```sql
-- 关联子查询写法
SELECT ename, job, sal, a.deptno,
       (SELECT AVG(sal)
          FROM emp c
         WHERE a.deptno = c.deptno   -- 子查询与主查询的关联条件
         GROUP BY deptno),
       dname
  FROM emp a
  JOIN dept b ON a.deptno = b.deptno
 WHERE empno = 7566;
```

### 开窗函数写法

```sql
SELECT ename, job, sal, deptno, v, dname
  FROM (
    SELECT empno, ename, job, sal, a.deptno,
           AVG(sal) OVER(PARTITION BY a.deptno) AS v,
           dname
      FROM emp a
      JOIN dept b ON a.deptno = b.deptno
  )
 WHERE empno = 7566;
```

---

## 七、重点总结

| 知识点 | 要点 |
|--------|------|
| 无名块结构 | `DECLARE → BEGIN → EXCEPTION → END;` |
| 必须有的部分 | `BEGIN ... END;` |
| 赋值符号 | `:=`（冒号等号） |
| 打印输出 | `DBMS_OUTPUT.PUT_LINE()` |
| SELECT INTO | 变量个数、顺序、类型必须与查询结果一致 |
| SELECT INTO 限制 | 只能处理**一行**数据 |
| DDL 语句 | 不能直接在 PL/SQL 中使用，需动态 SQL |
| DML / TCL 语句 | 可以直接在 PL/SQL 中使用 |
