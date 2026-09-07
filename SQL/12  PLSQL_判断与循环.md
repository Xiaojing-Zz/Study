---
title: PL/SQL 判断与循环
date: 2026-09-02
tags: [PL/SQL, Oracle, 数据库, 条件判断, 循环, IF, LOOP, WHILE, FOR]
category: 数据库笔记
---

# PL/SQL 判断与循环

---

## 一、IF 判断（分支判断）

### 1. 语法

```sql
DECLARE
BEGIN
  IF 条件1 THEN
    结果1;  -- 满足条件1时执行
  ELSIF 条件2 THEN
    结果2;  -- 满足条件2时执行
  ELSIF 条件3 THEN
    结果3;  -- 满足条件3时执行
  ELSE
    其他结果;  -- 以上条件都不满足时执行
  END IF;
END;
```

> **注意：** 当满足其中任意一个条件后，执行对应结果，然后跳到 `END IF` 结束判断，后续 `ELSIF` 不再判断。

---

### 2. 基础示例：根据数值范围输出

传入一个数值，判断其所在范围：

```sql
DECLARE
  n NUMBER := '&请输入';
BEGIN
  IF n BETWEEN 1 AND 10 THEN
    DBMS_OUTPUT.PUT_LINE('小于等于10');
  ELSIF n BETWEEN 11 AND 20 THEN
    DBMS_OUTPUT.PUT_LINE('小于等于20');
  ELSIF n BETWEEN 21 AND 30 THEN
    DBMS_OUTPUT.PUT_LINE('小于等于30');
  ELSE
    DBMS_OUTPUT.PUT_LINE('大于30');
  END IF;
END;
```

---

### 3. 练习：根据员工工资范围输出

传入员工编号，根据工资范围打印输出：

```sql
DECLARE
  v_empno NUMBER := '&请输入';
  v_sal   NUMBER;
BEGIN
  SELECT sal INTO v_sal FROM emp WHERE empno = v_empno;

  IF v_sal BETWEEN 1 AND 1000 THEN
    DBMS_OUTPUT.PUT_LINE('小于等于1000');
  ELSIF v_sal BETWEEN 1001 AND 2000 THEN
    DBMS_OUTPUT.PUT_LINE('小于等于2000');
  ELSE
    DBMS_OUTPUT.PUT_LINE('大于2000');
  END IF;
END;
```

**使用 `%TYPE` 声明变量：**

```sql
DECLARE
  v_empno emp.empno%TYPE := '&员工编码';
  v_sal   NUMBER;
BEGIN
  SELECT sal INTO v_sal FROM emp WHERE empno = v_empno;

  IF v_sal BETWEEN 1 AND 1000 THEN
    DBMS_OUTPUT.PUT_LINE('小于等于1000');
  ELSIF v_sal BETWEEN 1001 AND 2000 THEN
    DBMS_OUTPUT.PUT_LINE('小于等于2000');
  END IF;
END;
```

**使用动态 SQL（EXECUTE IMMEDIATE）：**

```sql
DECLARE
  v_empno NUMBER := '&请输入员工编号';
  v_sql   VARCHAR2(400);
  v_sal   NUMBER;
BEGIN
  v_sql := 'SELECT sal FROM emp WHERE empno = ' || v_empno;

  EXECUTE IMMEDIATE v_sql INTO v_sal;

  IF v_sal BETWEEN 1 AND 1000 THEN
    DBMS_OUTPUT.PUT_LINE('工资小于1000');
  ELSIF v_sal BETWEEN 1001 AND 2000 THEN
    DBMS_OUTPUT.PUT_LINE('工资小于2000');
  ELSIF v_sal BETWEEN 2001 AND 3000 THEN
    DBMS_OUTPUT.PUT_LINE('工资小于3000');
  ELSE
    DBMS_OUTPUT.PUT_LINE('工资大于3000');
  END IF;
END;
```

---

### 4. 练习：根据部门平均工资调薪

传入员工编号，工资低于部门平均工资则涨薪500，高于则减薪500，打印涨薪前后的薪资。

**写法一：两次查询**

```sql
DECLARE
  v_empno NUMBER := '&请输入';
  v_sal   NUMBER;       -- 修改前的工资
  v_deptno NUMBER;
  v_avg   NUMBER;
BEGIN
  -- 查询员工工资和部门号
  SELECT sal, deptno INTO v_sal, v_deptno FROM emp WHERE empno = v_empno;
  -- 查询部门平均工资
  SELECT AVG(sal) INTO v_avg FROM emp WHERE deptno = v_deptno;

  DBMS_OUTPUT.PUT_LINE('修改前：' || v_sal);

  IF v_sal < v_avg THEN
    UPDATE emp SET sal = sal + 500 WHERE empno = v_empno RETURNING sal INTO v_sal;
  ELSIF v_sal > v_avg THEN
    UPDATE emp SET sal = sal - 500 WHERE empno = v_empno RETURNING sal INTO v_sal;
  ELSE
    NULL;  -- 什么都不做
  END IF;

  DBMS_OUTPUT.PUT_LINE('修改后：' || v_sal);
END;
```

**写法二：子查询一次获取（推荐）**

```sql
DECLARE
  v_empno NUMBER := '&请输入';
  v_sal   NUMBER;
  v_avg   NUMBER;
BEGIN
  SELECT sal,
         (SELECT AVG(sal) FROM emp b WHERE b.deptno = a.deptno)
    INTO v_sal, v_avg
    FROM emp a
   WHERE empno = v_empno;

  DBMS_OUTPUT.PUT_LINE('修改前：' || v_sal);

  IF v_sal < v_avg THEN
    UPDATE emp SET sal = sal + 500 WHERE empno = v_empno RETURNING sal INTO v_sal;
  ELSIF v_sal > v_avg THEN
    UPDATE emp SET sal = sal - 500 WHERE empno = v_empno RETURNING sal INTO v_sal;
  ELSE
    NULL;
  END IF;

  DBMS_OUTPUT.PUT_LINE('修改后：' || v_sal);
END;
```

> **知识点：** `RETURNING sal INTO v_sal` 可以在 UPDATE 语句中直接将修改后的值赋给变量。

---

### 5. 练习：根据学生平均成绩判断优秀程度

传入学生姓名，根据平均成绩输出等级：

| 平均成绩 | 输出 |
|---------|------|
| 60 以下 | 啥也不是 |
| 60 ~ 80 | 工资6000 |
| 80 ~ 90 | 工资8000 |
| 90 以上 | 工资10000 |

```sql
DECLARE
  v_name  VARCHAR2(20) := '&请输入';
  v_score NUMBER;
BEGIN
  SELECT AVG(score)
    INTO v_score
    FROM student s
    LEFT JOIN sc ON s.sno = sc.sno
   WHERE sname = v_name;

  IF v_score < 60 THEN
    DBMS_OUTPUT.PUT_LINE('啥也不是');
  ELSIF v_score <= 80 THEN
    DBMS_OUTPUT.PUT_LINE('工资6000');
  ELSIF v_score <= 90 THEN
    DBMS_OUTPUT.PUT_LINE('工资8000');
  ELSIF v_score > 90 THEN
    DBMS_OUTPUT.PUT_LINE('工资10000');
  ELSE
    DBMS_OUTPUT.PUT_LINE('土豪不需要工作');
  END IF;
END;
```

---

## 二、循环

### 循环类型总览

| 类型 | 特点 | 说明 |
|------|------|------|
| `LOOP` | 无限循环（死循环） | 必须配合 `EXIT WHEN` 退出条件 |
| `WHILE` | 自带条件的循环 | 满足条件才能进入循环（门票） |
| `FOR` | 最常用、最方便的循环 | 自带变量，支持正序/反序/游标 |

---

### 1. LOOP 循环

#### 语法

```sql
DECLARE
BEGIN
  LOOP                    -- 循环开始
    -- 循环内容（必须）
    -- 控制次数（可选）
    EXIT WHEN 条件;       -- 满足条件退出循环
  END LOOP;               -- 结束循环
END;
```

#### 示例：打印10次"你好智云"

```sql
DECLARE
  n NUMBER := 1;
BEGIN
  LOOP
    DBMS_OUTPUT.PUT_LINE('你好智云');
    n := n + 1;
    EXIT WHEN n = 11;
  END LOOP;
END;
```

#### 练习：打印 1-100

```sql
DECLARE
  n NUMBER := 1;
BEGIN
  LOOP
    DBMS_OUTPUT.PUT_LINE(n);
    n := n + 1;
    EXIT WHEN n = 101;
  END LOOP;
END;
```

#### 练习：打印 1-100 之间的偶数（使用 IF）

```sql
DECLARE
  n NUMBER := 1;
BEGIN
  LOOP
    IF MOD(n, 2) = 0 THEN
      DBMS_OUTPUT.PUT_LINE(n);
    END IF;
    n := n + 1;
    EXIT WHEN n = 101;
  END LOOP;
END;
```

---

### 2. WHILE 循环

#### 语法

```sql
DECLARE
BEGIN
  WHILE 条件              -- 满足条件才能进入循环（门票）
  LOOP
    -- 循环内容（必须）
    -- 控制次数（可选）
  END LOOP;
END;
```

#### 示例：打印10次"你好智云"

```sql
DECLARE
  n NUMBER := 1;
BEGIN
  WHILE n <= 10
  LOOP
    DBMS_OUTPUT.PUT_LINE('你好智云');
    n := n + 1;
  END LOOP;
END;
```

#### 练习：打印 100-1 之间的奇数（使用 IF）

```sql
DECLARE
  n NUMBER := 100;
BEGIN
  WHILE n > 0
  LOOP
    IF MOD(n, 2) = 1 THEN
      DBMS_OUTPUT.PUT_LINE(n);
    END IF;
    n := n - 1;
  END LOOP;
END;
```

---

### 3. FOR 循环

#### 语法

```sql
DECLARE
BEGIN
  FOR 变量名 IN x..y | (SELECT语句) | 游标
  LOOP
    -- 循环内容
  END LOOP;
END;
```

> **说明：** `x..y` 中 x 和 y 是正整数，x 必须小于 y。变量名是 FOR 循环自带的变量，无需声明。

#### 示例：打印10次"你好智云"

```sql
DECLARE
BEGIN
  FOR i IN 101..110
  LOOP
    DBMS_OUTPUT.PUT_LINE('你好智云');
  END LOOP;
END;
```

#### 练习：打印 1-100 之间的奇数

```sql
DECLARE
BEGIN
  FOR i IN 1..100
  LOOP
    IF MOD(i, 2) = 1 THEN
      DBMS_OUTPUT.PUT_LINE(i);
    END IF;
  END LOOP;
END;
```

#### FOR 循环翻转（REVERSE）

使用 `REVERSE` 关键字可以从大到小循环：

```sql
DECLARE
BEGIN
  FOR i IN REVERSE 1..100
  LOOP
    IF MOD(i, 2) = 1 THEN
      DBMS_OUTPUT.PUT_LINE(i);
    END IF;
  END LOOP;
END;
```

#### 练习：反向打印字符串（如"李许铭" → "铭许李"）

使用 `REVERSE` + `SUBSTR` + `LENGTH` 函数：

```sql
DECLARE
  v_a VARCHAR2(20) := '&请输入';
BEGIN
  FOR i IN REVERSE 1..LENGTH(v_a)
  LOOP
    DBMS_OUTPUT.PUT(SUBSTR(v_a, i, 1));
  END LOOP;
  DBMS_OUTPUT.PUT_LINE('');
END;
```

> **知识点：**
> - `LENGTH(str)` — 返回字符串长度
> - `SUBSTR(str, pos, len)` — 从第 pos 位开始截取 len 个字符
> - `DBMS_OUTPUT.PUT()` — 输出不换行
> - `DBMS_OUTPUT.PUT_LINE()` — 输出并换行

---

### 4. 循环嵌套

**规则：** 外层循环每循环一次，内层循环完整循环一遍。

#### 示例：打印 24 小时制时间

```sql
DECLARE
BEGIN
  FOR i IN 0..24          -- 外层：时针
  LOOP
    FOR j IN 0..59        -- 内层：分针
    LOOP
      DBMS_OUTPUT.PUT_LINE(i || '小时' || j || '分钟');
    END LOOP;
  END LOOP;
END;
```

#### 练习：打印九九乘法表

```
1*1=1
1*2=2  2*2=4
1*3=3  2*3=6  3*3=9
...
```

```sql
DECLARE
BEGIN
  FOR i IN 1..9           -- 外层：行数 1~9
  LOOP
    FOR j IN 1..i         -- 内层：列数 1~当前行号
    LOOP
      DBMS_OUTPUT.PUT(i || '*' || j || '=' || i * j || '    ');
    END LOOP;
    DBMS_OUTPUT.PUT_LINE('');
  END LOOP;
END;
```

> **关键点：** 内层循环条件为 `1..i`，保证第 n 行输出 n 列。

---

## 三、常用函数速查

| 函数 | 用途 | 示例 |
|------|------|------|
| `MOD(m, n)` | 取余数 | `MOD(10, 3)` → 1 |
| `LENGTH(str)` | 字符串长度 | `LENGTH('abc')` → 3 |
| `SUBSTR(str, pos, len)` | 截取子串 | `SUBSTR('abc', 2, 1)` → 'b' |
| `BETWEEN a AND b` | 范围判断（含边界） | `n BETWEEN 1 AND 10` |
| `RETURNING ... INTO` | UPDATE 后返回值 | `RETURNING sal INTO v_sal` |
