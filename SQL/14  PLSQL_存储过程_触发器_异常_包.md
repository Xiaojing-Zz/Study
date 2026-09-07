---
title: PL/SQL 存储过程、触发器、异常、包
date: 2026-09-04
tags:
  - PL/SQL
  - Oracle
  - 存储过程
  - 触发器
  - 异常处理
  - 包
  - 数据库
---

# PL/SQL 存储过程、触发器、异常、包

---

## 一、存储过程（Procedure）

### 1.1 定义

将要执行的**过程或任务**存储成**有名块**，随时拿来调用。

### 1.2 存储过程 vs 函数

| 对比项 | 函数（Function） | 存储过程（Procedure） |
|--------|------------------|----------------------|
| 返回值 | 有且只有**一个**返回值 | **没有**返回值（可通过 `out` 参数传出） |
| 效率 | 普通 | **比普通程序块效率高** |
| 调用方式 | `SELECT` 语句中调用 | `CALL` 或程序块中调用 |

### 1.3 创建语法

```sql
CREATE [OR REPLACE] PROCEDURE 存储过程名(形参1 形参类型, 形参2 形参类型...)
-- 注意：形参不能写长度
IS
BEGIN
  -- 过程体
END;
```

### 1.4 基础示例

> **例题：** 创建存储过程，传入员工编号，打印输出该员工的姓名。

```sql
CREATE OR REPLACE PROCEDURE sp_98(v_empno IN NUMBER)
IS
  v_ename VARCHAR2(20);
BEGIN
  SELECT ename INTO v_ename FROM emp WHERE empno = v_empno;
  DBMS_OUTPUT.PUT_LINE('员工姓名：' || v_ename);
END;
```

### 1.5 存储过程的调用

#### 方式一：`CALL` 调用

```sql
CALL sp_98(7566);
```

#### 方式二：在程序块中调用

```sql
DECLARE
  v_ename VARCHAR2(20); -- 变量属于无名块
BEGIN
  sp_98(7566); -- 调用存储过程
  DBMS_OUTPUT.PUT_LINE(v_ename); -- 没有值（除非通过 out 参数传出）
END;
```

---

### 1.6 三种形参类型

| 类型 | 说明 | 默认 |
|------|------|------|
| `IN` | 输入型形参 | **默认**就是输入型 |
| `OUT` | 输出型形参 | — |
| `IN OUT` | 输入输出型形参 | — |

---

### 1.7 `OUT` 输出型形参

> **例题：** 传入一个员工编号，传出一个员工姓名。

```sql
CREATE OR REPLACE PROCEDURE sp_98(
  v_empno  IN  NUMBER,
  v_ename  OUT VARCHAR2
)
IS
BEGIN
  SELECT ename INTO v_ename FROM emp WHERE empno = v_empno;
END;
```

**调用（不能用 `CALL`，必须在程序块中）：**

```sql
DECLARE
  v_a VARCHAR2(20); -- 无名块中声明
BEGIN
  sp_98(7566, v_a);
  -- v_a 可以打印、可以判断比较
  DBMS_OUTPUT.PUT_LINE(v_a);
END;
```

> **重点：** 存储过程用 `OUT` 输出型形参**可以有返回值**，且能返回**多个值**（函数只能一个）。

---

### 1.8 多个 `OUT` 参数示例

> **练习：** 传入一个部门编号，打印输出该部门员工的姓名和岗位。

```sql
CREATE OR REPLACE PROCEDURE sp_98(
  v_deptno  IN  NUMBER,
  v_ename   OUT VARCHAR2,
  v_job     OUT VARCHAR2
)
IS
BEGIN
  FOR i IN (SELECT ename, job FROM emp WHERE deptno = v_deptno)
  LOOP
    v_ename := i.ename;
    v_job   := i.job;
  END LOOP;
END;
```

**调用：**

```sql
DECLARE
  v_a VARCHAR2(20);
  v_b VARCHAR2(20);
BEGIN
  sp_98(10, v_a, v_b);
  DBMS_OUTPUT.PUT_LINE(v_a || v_b);
END;
```

> **注意：** 以上方式只会输出**最后一条**记录。要输出多条记录，需使用**集合类型**。

---

### 1.9 使用集合类型返回多行数据

#### 步骤一：定义行类型

```sql
CREATE OR REPLACE TYPE emp_row AS OBJECT(
  ename VARCHAR2(10),
  job   VARCHAR2(9)
);
/
```

#### 步骤二：定义集合类型

```sql
CREATE OR REPLACE TYPE emp_table IS TABLE OF emp_row;
/
```

#### 步骤三：创建存储过程

```sql
CREATE OR REPLACE PROCEDURE proc_get_dept_emp_list(
  p_deptno   IN  emp.deptno%TYPE,
  p_emp_list OUT emp_table
)
IS
BEGIN
  SELECT emp_row(ename, job)
  BULK COLLECT INTO p_emp_list
  FROM emp
  WHERE deptno = p_deptno;
END;
/
```

#### 调用

```sql
DECLARE
  v_list emp_table;
BEGIN
  proc_get_dept_emp_list(10, v_list);
  FOR i IN 1..v_list.COUNT LOOP
    DBMS_OUTPUT.PUT_LINE('姓名:' || v_list(i).ename || ' 岗位:' || v_list(i).job);
  END LOOP;
END;
/
```

---

### 1.10 `IN OUT` 输入输出型形参

> **例题：** 传入一个员工编号，传出员工姓名。

```sql
CREATE OR REPLACE PROCEDURE sp_98(v_emp IN OUT emp%rowtype)
IS
BEGIN
  SELECT ename INTO v_emp.ename FROM emp WHERE empno = v_emp.empno;
END;
```

**调用：**

```sql
DECLARE
  v_a emp%rowtype;
BEGIN
  v_a.empno := 7566;
  sp_98(v_a);
  DBMS_OUTPUT.PUT_LINE(v_a.ename);
END;
```

---

### 1.11 综合练习

#### 练习 1：创建函数 —— 传入部门编号，返回部门平均工资

```sql
CREATE OR REPLACE FUNCTION fu_98(v_deptno NUMBER)
RETURN NUMBER
IS
  v_avg NUMBER;
BEGIN
  SELECT AVG(sal) INTO v_avg FROM emp WHERE deptno = v_deptno;
  RETURN v_avg;
END;
```

```sql
SELECT fu_98(10) FROM dual;
```

#### 练习 2：创建存储过程 —— 传入员工编号，根据部门平均工资涨降薪

> 要求用到练习 1 的函数。

- 如果员工工资 **小于** 部门平均工资 → **涨薪 500**
- 如果员工工资 **大于** 部门平均工资 → **降薪 500**
- 如果**等于** → 不变

```sql
CREATE OR REPLACE PROCEDURE sp_98(v_empno NUMBER)
IS
  v_deptno NUMBER;
  v_sal    NUMBER;
BEGIN
  SELECT deptno, sal INTO v_deptno, v_sal FROM emp WHERE empno = v_empno;

  IF v_sal > fu_98(v_deptno) THEN
    UPDATE emp SET sal = sal - 500 WHERE empno = v_empno;
  ELSIF v_sal < fu_98(v_deptno) THEN
    UPDATE emp SET sal = sal + 500 WHERE empno = v_empno;
  ELSE
    NULL;
  END IF;
END;
```

```sql
CALL sp_98(7566);
SELECT * FROM emp;
```

---

## 二、触发器（Trigger）

### 2.1 定义

由 **DML 语句**引起的一系列**触发事件**。

### 2.2 创建语法

```sql
CREATE [OR REPLACE] TRIGGER 触发器名
BEFORE | AFTER
UPDATE OR INSERT OR DELETE   -- 随意组合
ON 表名                      -- 针对某张表
[FOR EACH ROW]              -- 行级触发器（不写则为表级触发器）
BEGIN
  -- 触发的事件
END;
```

### 2.3 行级触发器 vs 表级触发器

| 类型 | 说明 |
|------|------|
| **行级触发器**（有 `FOR EACH ROW`） | DML 语句影响了多少行数据，触发器就触发**多少次** |
| **表级触发器**（无 `FOR EACH ROW`） | 无论 DML 影响多少行，**只触发一次** |

---

### 2.4 基础示例

> **例题：** 当对 emp 表增、删、改时，打印输出"修改了数据"。

```sql
CREATE OR REPLACE TRIGGER t_98
BEFORE
UPDATE OR INSERT OR DELETE
ON emp
BEGIN
  DBMS_OUTPUT.PUT_LINE('修改了数据');
END;
```

---

### 2.5 触发器的三种判断属性

| 属性 | 说明 |
|------|------|
| `DELETING` | 判断当前操作是否为 `DELETE`，是则返回 `TRUE` |
| `INSERTING` | 判断当前操作是否为 `INSERT`，是则返回 `TRUE` |
| `UPDATING` | 判断当前操作是否为 `UPDATE`，是则返回 `TRUE` |

> **例题：** 增删改分别打印不同的提示信息。

```sql
CREATE OR REPLACE TRIGGER t_98
BEFORE
UPDATE OR DELETE OR INSERT
ON emp
FOR EACH ROW
BEGIN
  IF UPDATING THEN
    DBMS_OUTPUT.PUT_LINE('修改了数据');
  ELSIF DELETING THEN
    DBMS_OUTPUT.PUT_LINE('删除了数据');
  ELSIF INSERTING THEN
    DBMS_OUTPUT.PUT_LINE('增加了数据');
  END IF;
END;
```

---

### 2.6 行级触发器的两个特殊属性

| 属性 | 说明 |
|------|------|
| `:OLD` | 代表**修改前**的数据 |
| `:NEW` | 代表**修改后**的数据 |

> **例题：** 修改员工工资时，判断涨降并打印前后工资。

```sql
CREATE OR REPLACE TRIGGER t_98
BEFORE
UPDATE
ON emp
FOR EACH ROW -- 必须是行级触发器才能使用 :OLD 和 :NEW
BEGIN
  IF :OLD.sal < :NEW.sal THEN
    DBMS_OUTPUT.PUT_LINE('涨工资 | 原工资:' || :OLD.sal || ' 新工资:' || :NEW.sal);
  ELSIF :OLD.sal > :NEW.sal THEN
    DBMS_OUTPUT.PUT_LINE('降工资 | 原工资:' || :OLD.sal || ' 新工资:' || :NEW.sal);
  ELSE
    DBMS_OUTPUT.PUT_LINE('工资不变 | 原工资:' || :OLD.sal || ' 新工资:' || :NEW.sql);
  END IF;
END;
```

---

## 三、异常处理（Exception）

### 3.1 语法

```sql
DECLARE
  -- 声明部分
BEGIN
  -- 执行部分

EXCEPTION
  -- 只能捕获异常，不能修改
  WHEN 异常名1 THEN
    -- 异常处理代码
  WHEN 异常名2 THEN
    -- 异常处理代码
  WHEN OTHERS THEN
    -- 兜底处理
END;
```

### 3.2 基础示例

```sql
DECLARE
BEGIN
  DBMS_OUTPUT.PUT_LINE(1/0);
EXCEPTION
  WHEN ZERO_DIVIDE THEN
    DBMS_OUTPUT.PUT_LINE('除数不能为0');
END;
```

### 3.3 多个预定义异常同时捕获

```sql
DECLARE
  v_name VARCHAR2(20);
BEGIN
  DBMS_OUTPUT.PUT_LINE(1/0);
  SELECT ename INTO v_name FROM emp;
EXCEPTION
  WHEN ZERO_DIVIDE THEN
    DBMS_OUTPUT.PUT_LINE('除数为0，请修改');
  WHEN TOO_MANY_ROWS THEN
    DBMS_OUTPUT.PUT_LINE('返回多行');
END;
```

### 3.4 未知异常处理（OTHERS）

```sql
DECLARE
BEGIN
  DBMS_OUTPUT.PUT_LINE(1/0);
EXCEPTION
  WHEN OTHERS THEN
    DBMS_OUTPUT.PUT_LINE(SQLCODE || '  ' || SQLERRM);
END;
```

- `SQLERRM` → 错误信息
- `SQLCODE` → 错误代码

### 3.5 异常的三种类型

| 类型 | 特点 |
|------|------|
| **预定义异常** | 有异常名称、异常代码、异常信息 |
| **非预定义异常** | 有异常代码、异常信息，无异常名 |
| **自定义异常** | 用户自定义 |

### 3.6 常见预定义异常速查表

| 异常名 | 错误代码 | 说明 |
|--------|---------|------|
| `CASE_NOT_FOUND` | ORA-06592 | CASE 语句中 WHEN 子句未包含必需条件，且无 ELSE |
| `COLLECTION_IS_NULL` | ORA-06531 | 给集合元素赋值前未初始化 |
| `CURSOR_ALREADY_OPEN` | ORA-06511 | 重新打开已打开的游标 |
| `DUP_VAL_ON_INDEX` | ORA-00001 | 唯一索引列上插入重复值 |
| `INVALID_CURSOR` | ORA-01001 | 操作不合法的游标 |
| `INVALID_NUMBER` | ORA-01722 | 字符转数字失败 |
| `NO_DATA_FOUND` | ORA-01403 | SELECT INTO 未返回行 |
| `TOO_MANY_ROWS` | ORA-01422 | SELECT INTO 返回超过一行 |
| `ZERO_DIVIDE` | ORA-01476 | 除以零 |
| `SUBSCRIPT_BEYOND_COUNT` | ORA-06533 | 元素下标超出嵌套表或 VARRAY 范围 |
| `SUBSCRIPT_OUTSIDE_LIMIT` | ORA-06532 | 元素下标为负 |
| `VALUE_ERROR` | ORA-06502 | 变量长度不够或非法字符串转数据 |
| `LOGIN_DENIED` | ORA-01017 | 用户名/密码不正确 |
| `NOT_LOGGED_ON` | ORA-01012 | 未连接数据库 |
| `PROGRAM_ERROR` | ORA-06510 | PL/SQL 内部问题 |
| `ROWTYPE_MISMATCH` | ORA-06504 | 宿主游标变量与 PL/SQL 游标变量返回类型不兼容 |
| `SELF_IS_NULL` | ORA-30625 | 在 NULL 实例上调用成员方法 |
| `STORAGE_ERROR` | — | 内存超出或被破坏 |
| `SYS_INVALID_ROWID` | ORA-01410 | 字符串转 ROWID 无效 |
| `TIMEOUT_ON_RESOURCE` | ORA-00051 | 等待资源超时 |
| `TRANSACTION_BACKED_OUT` | ORA-00006 | 死锁导致提交被退回 |

---

## 四、包（Package）

### 4.1 定义

包是**一系列函数和存储过程的集合**，由两部分组成：

1. **包头（Package）** —— 相当于标签，声明要装的函数和存储过程
2. **包体（Package Body）** —— 具体实现

### 4.2 创建包头

```sql
CREATE [OR REPLACE] PACKAGE 包名
IS
  -- 声明要装的函数和存储过程
END;
```

**示例：**

```sql
CREATE OR REPLACE PACKAGE lv
IS
  FUNCTION fu_98(v_deptno NUMBER) RETURN NUMBER;  -- 声明函数
  FUNCTION fu_1(v_deptno NUMBER)  RETURN NUMBER;  -- 声明函数
  PROCEDURE sp_98(v_empno NUMBER);                -- 声明存储过程
END;
```

### 4.3 创建包体

```sql
CREATE [OR REPLACE] PACKAGE BODY 包名  -- 和包头一致
IS
  -- 具体实现
END;
```

**示例：**

```sql
CREATE OR REPLACE PACKAGE BODY lv
IS

  -- fu_98 的实现
  FUNCTION fu_98(v_deptno NUMBER)
  RETURN NUMBER
  IS
    v_avg NUMBER;
  BEGIN
    SELECT AVG(sal) INTO v_avg FROM emp WHERE deptno = v_deptno;
    RETURN v_avg;
  END;

  -- fu_1 的实现
  FUNCTION fu_1(v_deptno NUMBER)
  RETURN NUMBER
  IS
    v_avg NUMBER;
  BEGIN
    SELECT AVG(sal) INTO v_avg FROM emp WHERE deptno = v_deptno;
    RETURN v_avg;
  END;

  -- sp_98 的实现
  PROCEDURE sp_98(v_empno NUMBER)
  IS
    v_deptno NUMBER;
    v_sal    NUMBER;
  BEGIN
    SELECT deptno, sal INTO v_deptno, v_sal FROM emp WHERE empno = v_empno;

    IF v_sal > fu_98(v_deptno) THEN
      UPDATE emp SET sal = sal - 500 WHERE empno = v_empno;
    ELSIF v_sal < fu_98(v_deptno) THEN
      UPDATE emp SET sal = sal + 500 WHERE empno = v_empno;
    ELSE
      NULL;
    END IF;
  END;

END;
```

### 4.4 调用包中的函数/存储过程

```sql
-- 调用包中的函数
SELECT lv.fu_98(10) FROM dual;
```

---

## 五、知识体系总览

```
PL/SQL 核心知识点
├── 变量 & 变量类型
├── IF 判断
├── 循环
├── 游标
├── 动态 SQL
├── 函数（Function）        —— 有且只有一个返回值
├── 存储过程（Procedure）    —— 无返回值，通过 OUT 参数传出
├── 触发器（Trigger）        —— DML 语句触发
├── 异常（Exception）        —— 预定义 / 非预定义 / 自定义
└── 包（Package）            —— 包头 + 包体
```
