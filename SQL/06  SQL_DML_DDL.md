---
date: 2026-08-26
---
# SQL 笔记 —— DML 与 DDL

---

## 一、DML（数据操纵语言）

> **定义：** 针对 **"数据"** 进行 **增、删、改** 操作的语言。

| 关键字      | 作用   |
| -------- | ---- |
| `INSERT` | 增加数据 |
| `UPDATE` | 更新数据 |
| `DELETE` | 删除数据 |

---

## 二、字段属性

> 每张表在创建每个字段时，会给每个字段一个 **属性**，该属性决定了该字段存入的数据类型。

### 1. 数值型字段属性

| 属性                | 说明                                                     |
| ----------------- | ------------------------------------------------------ |
| `NUMBER([p][,s])` | 常用数值型字段属性，该属性下只能存储数值型数据。`p` = 总长度（含小数位），`s` = 精度（小数位数） |
| `NUMBER(7, 2)`    | 总长 7，小数 2 位，整数最多 5 位                                   |
| `NUMBER(7)`       | 只保留整数                                                  |
| `NUMBER`          | 默认长度 38                                                |
| `INT`             | 存储整数，等价于 `NUMBER(38)`                                  |

**`NUMBER(7, 2)` 示例：**

| 输入值       | 存储结果       | 说明                    |
| --------- | ---------- | --------------------- |
| 12345678  | 报错         | 超过总长度 7               |
| 1234567   | 1234567.00 | 整数部分 5 位 + 小数 2 位 = 7 |
| 12345     | 12345.00   | 整数部分 5 位 + 小数 2 位 = 7 |
| 12345.678 | 12345.68   | 四舍五入保留 2 位小数          |

### 2. 字符型字段属性

| 属性            | 说明                                                                 |
| ------------- | ------------------------------------------------------------------ |
| `CHAR(n)`     | **固定长度**字符串。`n` 表示总长度，存入的数据长度不能超过 `n`,不足 `n` 位则右侧空格填充，始终以 `n` 长度存储 |
| `VARCHAR(n)`  | **可变长度**字符串。不足 `n` 位则按实际长度存储，最长 2000                               |
| `VARCHAR2(n)` | Oracle 专用可变长度字符串，最长 4000                                           |

**示例：**

```sql
CHAR(10)    -- '李许铭' → 占 10 个长度（空格填充）
CHAR(10)    -- 'a'      → 占 10 个长度（空格填充）

VARCHAR(10) -- '李许铭' → 占 6 个长度（实际存储）
```

### 3. 日期型字段属性

| 属性 | 说明 |
|------|------|
| `DATE` | 包含 世纪、年月日、时分秒 |
| `TIMESTAMP` | 时间戳类型，比 `DATE` 多出 **毫秒** 部分 |

### 4. 字段属性默认值与省略规则

| 字段类型 | 能否省略括号 | 省略后的默认值 | 说明 |
|----------|:---:|------|------|
| `NUMBER(p, s)` | ✅ | `NUMBER` → 默认 38 位精度 | 省略 `s` 则为纯整数 |
| `INT` | ✅ | 本身就是简写 | 等价于 `NUMBER(38)` |
| `CHAR(n)` | ❌ | — | `n` 必须指定，没有默认值 |
| `VARCHAR(n)` | ❌ | — | `n` 必须指定，没有默认值 |
| `VARCHAR2(n)` | ❌ | — | `n` 必须指定，没有默认值 |
| `DATE` | ✅ | 本身就是完整类型 | 不需要括号，也没有参数 |

---

## 三、INSERT —— 增加数据

### 基本语法

```sql
INSERT INTO 表名 [(列名1, 列名2, 列名3, ...)]
       VALUES (值1, 值2, 值3, ...);
```

> **值** 的个数、顺序、属性 要和 **列** 一致。

### 示例

```sql
-- 向 emp 表插入一行数据
INSERT INTO emp (empno, ename, job, sal, hiredate, deptno)
       VALUES (2778, '康志存', '男明星', 28000, SYSDATE, 10);

-- 查询结果
SELECT * FROM emp;

-- 没有插入的字段默认是空值

-- 不指定列名，按表结构顺序插入所有字段
INSERT INTO dept VALUES (70, '娱乐部门', '澳门');

-- 插入空字符串和空格（注意与 NULL 的区别）
INSERT INTO dept VALUES (80, '',  '澳门');
INSERT INTO dept VALUES (90, ' ', '香港');

-- 查询 dname 为 NULL 的记录
SELECT * FROM dept WHERE dname IS NULL;
```

### 快捷插入（子查询插入）

```sql
-- 语法：将查询语句的结果插入到表中
INSERT INTO 表名 [(列名1, 列名2, ...)] SELECT 语句;

-- 查询语句的结果也要和前面的列 个数、顺序、属性 一致

-- 示例：将 dept 表的 deptno 和 dname 插入到 emp 表
INSERT INTO emp (empno, ename) SELECT deptno, dname FROM dept;

-- 练习：将 30 号部门的员工信息插入到奖金表
INSERT INTO bonus SELECT ename, job, sal, comm FROM emp WHERE deptno = 30;
```

### ⚠️ 注意

1. **DML 语句需要提交才能生效**
2. **提交之后不能回滚**

---

## 四、DELETE —— 删除数据

### 语法

```sql
DELETE FROM 表名 [WHERE 条件];
```

### 示例

```sql
-- 不加 WHERE 条件 → 删除所有数据（慎用！）
DELETE FROM emp;

-- 按条件删除部分数据
DELETE FROM emp WHERE deptno = 10;

DELETE FROM emp WHERE ename = '康志存';

DELETE FROM emp WHERE sal = 2;
```

---

## 五、UPDATE —— 修改数据

### 语法

```sql
UPDATE 表名
   SET 列名1 = 值1, 列名2 = 值2, 列名3 = 值3, ...
 [WHERE 条件];
```

### 示例

```sql
-- 不加 WHERE → 修改所有行
UPDATE emp SET sal = 1, comm = 2;

-- 按条件修改
UPDATE emp SET sal = 9, comm = 8, job = '保安头子' WHERE ename = '李许铭';

-- 值可以是子查询（必须单行单列）
UPDATE emp SET sal = (SELECT MAX(sal) FROM emp) WHERE ename = '李许铭';

-- 值可以是计算表达式
UPDATE emp SET sal = sal * 1.1 WHERE ename = '李许铭';

-- 值可以是函数
UPDATE emp SET sal = LENGTH(ename) WHERE ename = '李许铭';
```

### ⚠️ 注意

1. **值** 可以是具体值、函数、计算表达式、或子查询（必须单行单列）
2. 列名和值 **不能写反**

---

## 六、MERGE INTO —— 同时增删改

> 可以同时完成 **增、删、改** 操作。

### 语法

```sql
MERGE INTO 表名1 a                    -- 目标表（被增删改的表）
     USING 表名2 b                    -- 数据源（提供数据的表）
        ON (匹配字段)                  -- 关系：必须是主键列或唯一列（不能有重复数据）
  WHEN MATCHED THEN                   -- 匹配成功时
       UPDATE SET 列名1 = 值1,
                  列名2 = 值2,
                  a.列名3 = b.列名3
              WHERE 条件1              -- ★ 以上 UPDATE 必须有
       DELETE WHERE 条件2              -- 可选，需同时满足 ON 和 UPDATE 的条件
  WHEN NOT MATCHED THEN               -- 未匹配时
       INSERT (a.列名1, a.列名2, ...)
       VALUES (b.列名1, b.列名2, ...);
```

### 示例

```sql
-- 先准备数据源
CREATE TABLE emp2 AS SELECT * FROM emp;   -- 快捷建表

MERGE INTO emp  a          -- 目标表
     USING emp1 b          -- 数据源
        ON (a.empno = b.empno)
  WHEN MATCHED THEN
       UPDATE SET sal = 1, comm = 2, a.job = b.job
              WHERE deptno = 30
       DELETE WHERE ename = '李许铭'
  WHEN NOT MATCHED THEN
       INSERT (a.empno, a.ename)
       VALUES (b.empno, b.ename);

SELECT * FROM emp;
```

---

## 七、DDL（数据定义语言）

> **定义：** 针对 **数据库对象** 进行 **增、删、改** 操作。
>
> 数据库对象：表、索引、视图、序列、注释、约束、函数、存储过程……

| 关键字 | 作用 |
|--------|------|
| `CREATE` | 创建数据库对象 |
| `DROP` | 删除数据库对象 |
| `ALTER` | 修改数据库对象 |
| `TRUNCATE` | 清空表数据 |

---

## 八、CREATE —— 创建表

### 基本语法

```sql
CREATE TABLE 表名 (
    列名1 字段属性,
    列名2 字段属性,
    列名3 字段属性,
    ...
);
```

### 示例

```sql
CREATE TABLE zhiyun98 (
    zno   NUMBER,
    zname VARCHAR2(20),
    zsr   DATE
);

SELECT * FROM zhiyun98;
```

### ⚠️ 注意

**DDL 语句执行成功即生效**（自动提交，不可回滚）。

### 命名规范

| 规则 | 说明 |
|------|------|
| 长度 | 不能超过 30 个字符 |
| 见名知意 | `zhiyun98` 优于 `a` |
| 字母开头 | `a1` ✅，`1a` ❌ |
| 特殊符号 | 除 `_` 外不建议使用（`to_date` ✅，`zhiyun(98)` ❌） |
| 关键字 | 不建议和关键字重名 |

### 快捷建表

```sql
-- 语法：将 SELECT 语句的结果创建成表
CREATE TABLE 表名 AS SELECT 语句;

-- 示例：复制表结构和数据
CREATE TABLE zhiyun981 AS SELECT * FROM zhiyun98;

SELECT * FROM zhiyun981;
```

### 练习

> 快捷建表，字段包含：员工姓名、岗位、薪资、部门编号、部门名称、部门地址、部门平均工资

```sql
CREATE TABLE emp98 AS
SELECT ename,
       job,
       sal,
       a.deptno,
       dname,
       loc,
       (SELECT AVG(sal) FROM emp b WHERE b.deptno = a.deptno) AS avg_sal  -- 相关子查询
  FROM emp a
  LEFT JOIN dept ON a.deptno = dept.deptno;
```

---

## 九、DROP —— 删除表

### 语法

```sql
DROP TABLE 表名;
```

### 示例

```sql
DROP TABLE zhiyun981;

SELECT * FROM zhiyun981;  -- 报错：表不存在
```

> ⛔ **慎重使用，甚至建议忘掉，不要使用！**

---

## 十、TRUNCATE —— 清空表数据

### 语法

```sql
TRUNCATE TABLE 表名;
```

### 示例

```sql
TRUNCATE TABLE emp2;   -- 清空所有数据
```

---

## 十一、TRUNCATE 与 DELETE 的区别

| 对比项 | DELETE | TRUNCATE |
|--------|--------|----------|
| 语句类型 | DML | DDL |
| 回滚日志 | ✅ 有，能回滚 | ❌ 没有，不能回滚 |
| 效率 | 较低 | **较高** |
| 释放空间 | ❌ 不会 | ✅ 会释放 |

---

## 十二、快捷建表语法汇总

```sql
-- 复制整张表（结构 + 数据）
CREATE TABLE 新表名 AS SELECT * FROM 原表名;

-- 复制部分列
CREATE TABLE 新表名 AS SELECT 列1, 列2 FROM 原表名;

-- 带条件复制
CREATE TABLE 新表名 AS SELECT * FROM 原表名 WHERE 条件;

-- 带计算列（自创字段需取别名）
CREATE TABLE 新表名 AS
SELECT ename, sal, sal + comm AS income FROM emp;
```

---

## 十三、FOR UPDATE —— 行级编辑

```sql
SELECT * FROM emp FOR UPDATE;   -- 可在结果集中直接编辑数据
```
