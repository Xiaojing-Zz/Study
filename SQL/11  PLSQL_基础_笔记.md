---
date: 2026-09-01
tags:
  - "#PL/SQL"
  - Oracle
  - 动态SQL
  - RETURNING
---

# PL/SQL 基础笔记

---
## 一、变量声明

### 1. 基本语法

```sql
DECLARE
  变量名 数据类型 [:= 初始值];
BEGIN
  -- 逻辑代码
END;
```

### 2. 手动输入 `&`

使用 `&` 可以在运行时手动输入变量值，建议都加上单引号。

**注意：** 有多个输入时，提示词不能一样。

### 3. 例题：传入员工编号，打印输出该员工的姓名

```sql
DECLARE
  V_empno  NUMBER := '&请输入员工编号';
  V_ename  VARCHAR2(20);
BEGIN
  SELECT ename INTO V_ename FROM emp WHERE empno = V_empno;
  DBMS_OUTPUT.PUT_LINE(V_ename);
END;
```

---

## 二、打印输出

### 1. 换行输出 `DBMS_OUTPUT.PUT_LINE`

```sql
DBMS_OUTPUT.PUT_LINE(参数);  -- 打印结束后回车换行，另起一行
```

### 2. 不换行输出 `DBMS_OUTPUT.PUT`

```sql
DBMS_OUTPUT.PUT(参数);  -- 打印结束后不换行
```

**注意：** 不换行输出后，最后一定要有一个 `PUT_LINE` 来触发换行。

```sql
DECLARE
BEGIN
  DBMS_OUTPUT.PUT('李许铭');
  DBMS_OUTPUT.PUT('康志存');
  DBMS_OUTPUT.PUT_LINE('');  -- 回车换行
END;
```

### 3. 练习：传入员工编号，打印部门编号及部门平均工资（要求用两个不换行输出）

```sql
DECLARE
  V_empno  NUMBER := '&请输入';
  V_deptno NUMBER;
  V_avg    NUMBER;
BEGIN
  SELECT deptno,
         (SELECT AVG(sal) FROM emp b WHERE b.deptno = a.deptno)
    INTO V_deptno, V_avg
    FROM emp a
   WHERE empno = V_empno;

  DBMS_OUTPUT.PUT(V_deptno);
  DBMS_OUTPUT.PUT(V_avg);
  DBMS_OUTPUT.PUT_LINE('');
END;
```

**另一种写法：分两步查询**

```sql
DECLARE
  V_empno  CHAR(20) := '&输入编号';
  V_deptno NUMBER;
  V_avg    NUMBER;
BEGIN
  SELECT deptno INTO V_deptno FROM emp WHERE empno = V_empno;
  SELECT AVG(sal) INTO V_avg FROM emp WHERE deptno = V_deptno;

  DBMS_OUTPUT.PUT(V_deptno);
  DBMS_OUTPUT.PUT_LINE(V_avg);
END;
```

---

## 三、变量类型

### 1. 引用型变量

#### （1）`%type` —— 引用某张表的某个字段属性

引用某张表的某字段的属性，或者引用已经存在的变量属性。

**好处：**
- 可以不用知道引用的列的具体属性
- 字段属性发生改变时，变量类型自动跟随该表

**语法：**

```sql
DECLARE
  变量1  表名.列名%TYPE;   -- 引用某张表的某字段的属性
  变量2  变量1%TYPE;       -- 引用已经存在的变量属性
BEGIN
  -- ...
END;
```

**练习：** 打印输出 7566 的员工姓名、岗位、薪资、部门编号、部门名称及工资等级

```sql
DECLARE
  V_ename  emp.ename%TYPE;
  V_job    emp.job%TYPE;
  V_sal    emp.sal%TYPE;
  V_deptno emp.deptno%TYPE;
  V_dname  dept.dname%TYPE;
  V_grade  salgrade.grade%TYPE;
BEGIN
  SELECT ename, job, sal, emp.deptno, dname, grade
    INTO V_ename, V_job, V_sal, V_deptno, V_dname, V_grade
    FROM emp
    LEFT JOIN dept ON emp.deptno = dept.deptno
    LEFT JOIN salgrade ON sal BETWEEN losal AND hisal
   WHERE empno = 7566;

  DBMS_OUTPUT.PUT_LINE(V_ename || V_job || V_sal || V_deptno || V_dname || V_grade);
END;
```

---

#### （2）`%ROWTYPE` —— 引用某张表一行的所有字段属性

**语法：**

```sql
DECLARE
  变量  表名%ROWTYPE;   -- 复合类型，包含该表所有字段
BEGIN
  -- 使用：变量.列名
END;
```

**例题：** 打印输出 7566 的员工姓名、岗位、薪资、部门编号及部门名称

```sql
DECLARE
  V_emp   emp%ROWTYPE;          -- 复合类型，包含 emp 表所有字段
  V_dname dept.dname%TYPE;      -- 单独引用
BEGIN
  SELECT ename, job, sal, emp.deptno, dname
    INTO V_emp.ename, V_emp.job, V_emp.sal, V_emp.deptno, V_dname
    FROM emp
    LEFT JOIN dept ON emp.deptno = dept.deptno
   WHERE empno = 7566;

  DBMS_OUTPUT.PUT_LINE(V_emp.ename || V_emp.job || V_emp.sal || V_emp.deptno || V_dname);
END;
```

**练习：** 传入员工编号，打印姓名、岗位、薪资、部门编号、部门地址及部门平均工资

```sql
DECLARE
  V_empno emp.empno%TYPE := '&请输入';
  V_emp   emp%ROWTYPE;
  V_dname dept.dname%TYPE;
  V_avg   emp.sal%TYPE;
BEGIN
  SELECT ename, job, sal, a.deptno, dname,
         (SELECT AVG(sal) FROM emp b WHERE b.deptno = a.deptno)
    INTO V_emp.ename, V_emp.job, V_emp.sal, V_emp.deptno, V_dname, V_avg
    FROM emp a
    LEFT JOIN dept ON a.deptno = dept.deptno
   WHERE empno = V_empno;

  DBMS_OUTPUT.PUT_LINE(V_emp.ename || V_emp.job || V_emp.sal || V_emp.deptno || V_dname || V_avg);
END;
```

---

### 2. 记录型变量 `RECORD`

自定义复合类型，比 `%ROWTYPE` 声明复杂但更灵活。

**语法：**

```sql
DECLARE
  TYPE 类型名 IS RECORD (
    变量1  数据类型,
    变量2  数据类型,
    ...
  );
  变量名  类型名;   -- 复合变量，使用：变量名.变量1, 变量名.变量2 ...
BEGIN
  -- ...
END;
```

**例题：** 打印输出 7566 的员工姓名、岗位、薪资、部门编号及部门名称

```sql
DECLARE
  TYPE c1 IS RECORD (
    v_ename  VARCHAR2(20),
    v_job    emp.job%TYPE,
    v_sal    NUMBER,
    v_deptno NUMBER,
    v_dname  VARCHAR2(20)
  );
  V_a  c1;
BEGIN
  SELECT ename, job, sal, emp.deptno, dname
    INTO V_a
    FROM emp
    LEFT JOIN dept ON emp.deptno = dept.deptno
   WHERE empno = 7566;

  DBMS_OUTPUT.PUT_LINE(V_a.v_ename || V_a.v_job || V_a.v_sal || V_a.v_deptno || V_a.v_dname);
END;
```

**练习：** 传入员工编号，打印姓名、岗位、薪资、入职日期、岗位平均薪资及部门名称

```sql
DECLARE
  V_empno NUMBER := '&请输入';
  TYPE c1 IS RECORD (
    V_ename   VARCHAR2(20),
    v_job     VARCHAR2(20),
    V_sal     NUMBER,
    V_hiredate DATE,
    V_avg     NUMBER,
    v_dname   VARCHAR2(20)
  );
  a  c1;
BEGIN
  SELECT ename, job, sal, hiredate,
         (SELECT AVG(sal) FROM emp b WHERE b.job = a.job),
         dname
    INTO a.V_ename, a.V_job, a.v_sal, a.V_hiredate, a.v_avg, a.v_dname
    FROM emp a
    LEFT JOIN dept ON a.deptno = dept.deptno
   WHERE empno = V_empno;

  DBMS_OUTPUT.PUT_LINE(a.V_ename || a.V_job || a.v_sal || a.V_hiredate || a.v_avg || a.v_dname);
END;
```

### 3. `RECORD` 与 `%ROWTYPE` 对比

| 特性   | `RECORD`         | `%ROWTYPE`      |
| ---- | ---------------- | --------------- |
| 声明方式 | 较复杂，需自定义字段       | 简单，直接引用表        |
| 灵活性  | **高**，可自由选择字段和类型 | **低**，固定为表的所有字段 |
| 适用场景 | 需要自定义字段组合时       | 需要整行数据时         |

---

## 四、程序块中的 DML 语句

### 1. `RETURNING ... INTO` 子句

将 DML 语句（INSERT / DELETE / UPDATE）影响到的列值交给变量。

**语法：**

```sql
DML语句 RETURNING 列名1, 列名2, ... INTO 变量1, 变量2, ...;
```

### 2. INSERT + RETURNING

**例题：** 向 emp 表中插入一行数据，同时打印输出插入的值

```sql
DECLARE
  V_empno NUMBER;
  V_ename emp.ename%TYPE;
  V_job   VARCHAR2(20);
BEGIN
  INSERT INTO emp (empno, ename)
  VALUES (9988, '武梦杰')
  RETURNING empno, ename, job INTO V_empno, V_ename, V_job;

  DBMS_OUTPUT.PUT_LINE(V_empno || V_ename || V_job);
END;
```

### 3. DELETE + RETURNING

**例题：** 删除 emp 表中某行数据，同时打印输出被删除的人名

```sql
DECLARE
  V_ename emp.ename%TYPE;
  V_job   VARCHAR2(20);
BEGIN
  DELETE FROM emp WHERE empno = 7566
  RETURNING ename, job INTO V_ename, V_job;

  DBMS_OUTPUT.PUT_LINE(V_ename || V_job);
END;
```

**注意：** `RETURNING` 只能处理单行数据。

### 4. UPDATE + RETURNING

**例题：** 修改 emp 表中某员工薪资，打印输出修改后的薪资

```sql
DECLARE
  V_sal NUMBER;
BEGIN
  UPDATE emp SET sal = 1 WHERE empno = 7566
  RETURNING sal INTO V_sal;

  DBMS_OUTPUT.PUT_LINE(V_sal);
END;
```

---

## 五、程序块中的 DDL 语句 —— 动态 SQL

DDL 语句（CREATE / DROP / ALTER 等）不能在 PL/SQL 程序块中直接使用，需要通过**动态 SQL** 来实现。

### 1. 什么是动态 SQL

将 SQL 语句用单引号包裹后交给变量，再通过 `EXECUTE IMMEDIATE` 立即执行。

### 2. 语法

```sql
DECLARE
  变量名  VARCHAR2(300);    -- 1. 声明变量（字符类型长度尽量长）
  变量1, 变量2, ...;
BEGIN
  变量名 := 'SQL语句';     -- 2. 将 SQL 语句交给变量（定义动态 SQL）
  EXECUTE IMMEDIATE 变量名  -- 3. 立即执行动态 SQL
    [INTO 变量1, 变量2, ...];  -- 可选：将执行结果交给变量
END;
```

### 3. 例题：利用动态 SQL 实现 DDL 语句（删除表）

```sql
DECLARE
  V_sql VARCHAR2(400);
BEGIN
  V_sql := 'DROP TABLE "zy982"';   -- 定义动态 SQL
  EXECUTE IMMEDIATE V_sql;          -- 立即执行
END;
```

### 4. 例题：传入表名，打印输出该表的记录数

```sql
DECLARE
  V_table VARCHAR2(20) := '&请输入表名';
  V_count NUMBER;
  V_sql   VARCHAR2(400);
BEGIN
  V_sql := 'SELECT COUNT(1) FROM ' || V_table;   -- 拼接动态 SQL
  DBMS_OUTPUT.PUT_LINE(V_sql);                     -- 打印 SQL 语句
  EXECUTE IMMEDIATE V_sql INTO V_count;            -- 执行并将结果交给变量
  DBMS_OUTPUT.PUT_LINE(V_count);
END;
```

> 执行时 V_sql 的值为：`SELECT COUNT(1) FROM emp`

### 5. 练习：传入表名，打印该表中最大工资的员工姓名

```sql
DECLARE
  V_table VARCHAR2(20) := '&请输入表名';
  V_sql   VARCHAR2(200);
  V_ename VARCHAR2(30);
BEGIN
  V_sql := 'SELECT ename FROM ' || V_table ||
           ' WHERE sal = (SELECT MAX(sal) FROM ' || V_table || ')';

  EXECUTE IMMEDIATE V_sql INTO V_ename;
  DBMS_OUTPUT.PUT_LINE(V_ename);
END;
```

> 执行时 V_sql 的值为：
> `SELECT ename FROM emp WHERE sal = (SELECT MAX(sal) FROM emp)`

### 6. 练习：传入两个表名，打印平均成绩最小的学生姓名

```sql
DECLARE
  V_table1 VARCHAR2(20) := '&请输入表名1';  -- student 表
  V_table2 VARCHAR2(20) := '&请输入表名2';  -- sc 表
  V_sql    VARCHAR2(500);
  V_sname  VARCHAR2(20);
BEGIN
  V_sql := 'SELECT sname
             FROM (SELECT a.sno, a.sname, AVG(score) av,
                          ROW_NUMBER() OVER(ORDER BY AVG(score)) r
                     FROM ' || V_table1 || ' a
                     LEFT JOIN ' || V_table2 || ' b ON a.sno = b.sno
                    GROUP BY a.sno, a.sname) v
            WHERE r = 1';

  EXECUTE IMMEDIATE V_sql INTO V_sname;
  DBMS_OUTPUT.PUT_LINE(V_sname);
END;
```

---

## 六、知识点总结

| 知识点 | 要点 |
|--------|------|
| 变量声明 | `DECLARE` 块中声明，`:=` 赋值 |
| 手动输入 | `&提示词`，提示词需唯一且建议加单引号 |
| 换行输出 | `DBMS_OUTPUT.PUT_LINE()` |
| 不换行输出 | `DBMS_OUTPUT.PUT()`，最后必须跟一个 `PUT_LINE('')` |
| `%TYPE` | 引用某表某列的数据类型 |
| `%ROWTYPE` | 引用某表整行的所有字段（复合类型） |
| `RECORD` | 自定义复合类型，灵活但声明复杂 |
| `RETURNING INTO` | DML 语句中获取受影响行的列值（仅限单行） |
| 动态 SQL | `EXECUTE IMMEDIATE` 执行字符串形式的 SQL |
| `SELECT INTO` | 只能处理**单行**数据 |
