---
title: SQL 序列、开窗函数、行列转换与集合运算
tags:
  - SQL
  - Oracle
  - 笔记
  - 序列
  - 开窗函数
  - 行列转换
  - 集合运算
created: 2026-08-28
---

# SQL 序列、开窗函数、行列转换与集合运算

---

## 一、序列（Sequence）

> **序列**是 Oracle 提供的一组**能够自动增长的序号**，属于数据库对象之一，主要用于**为主键列提供数据**。

### 1.1 创建序列语法

```sql
CREATE SEQUENCE 序列名
  START WITH n            -- 初始值，默认 1
  INCREMENT BY n          -- 步长（增长幅度），默认 1，可以为负数
  MAXVALUE n | NOMAXVALUE -- 最大值，默认 10^27
  MINVALUE n | NOMINVALUE -- 最小值，默认 -10^27
  CYCLE | NOCYCLE         -- 循环/不循环，默认不循环
  CACHE n | NOCACHE       -- 缓存，默认缓存 20 个序号
```

### 1.2 示例

```sql
CREATE SEQUENCE seq_98
  START WITH 5
  INCREMENT BY 2
  MAXVALUE 100
  CYCLE       -- 循环
  CACHE 50;   -- 注意：缓存必须 < 最大值 / 步长
```

### 1.3 使用序列

| 属性 | 说明 |
|------|------|
| `currval` | 当前值 |
| `nextval` | 下一个值 |

```sql
SELECT seq_98.currval FROM dual;
SELECT seq_98.nextval FROM dual;
```

### 1.4 注意事项

> [!warning] 重要注意
> 1. **序列刚创建时没有值**
> 2. **第一次使用必须用 `nextval`**，返回的是初始值
> 3. **循环都是从 1 开始**

### 1.5 在插入数据时使用序列

```sql
CREATE TABLE zy98 (zno NUMBER, zname VARCHAR2(20));

INSERT INTO zy98 VALUES(seq_98.nextval, '李许铭');
INSERT INTO zy98 VALUES(seq_98.nextval, '康志存');
INSERT INTO zy98 VALUES(seq_98.nextval, '刘赫');
INSERT INTO zy98 VALUES(seq_98.nextval, '韩晓静');
INSERT INTO zy98 VALUES(seq_98.nextval, '武梦杰');

SELECT * FROM zy98;
```

---

## 二、开窗函数（分析函数 / 窗口函数）

> **开窗函数**即**分析函数**，搭配 `OVER()` 子句使用，能在不改变行数的前提下对数据进行分组分析。

### 2.1 基本语法

```sql
分析函数名() OVER(分析子句)
```

**分析子句**包含：
- `PARTITION BY` — 分组
- `ORDER BY` — 排序（ASC | DESC）
- `ROWS` — 窗口范围

### 2.2 三类分析函数

| 类别 | 函数 | 说明 |
|------|------|------|
| **聚合类** | `SUM` `AVG` `MIN` `MAX` `COUNT` | 聚合开窗 |
| **排序类** | `ROW_NUMBER()` `RANK()` `DENSE_RANK()` | 排序开窗 |
| **偏移类** | `LAG()` `LEAD()` | 偏移开窗 |

---

### 2.3 聚合开窗

> 按照一定规则分组，分析各组的情况，**每一行返回一个值**。

#### 查询员工姓名、岗位、薪资、部门及部门平均工资

```sql
SELECT ename, job, sal, deptno,
       AVG(sal) OVER(PARTITION BY deptno) AS avg_dept_sal
  FROM emp;
```

#### 查询工资高于自己岗位平均工资的员工

```sql
SELECT *
  FROM (SELECT ename, job, sal, emp.deptno, dname,
               AVG(sal) OVER(PARTITION BY job) AS avg_job_sal
          FROM emp
          LEFT JOIN dept ON emp.deptno = dept.deptno)
 WHERE sal > avg_job_sal;
```

> [!warning] `WHERE` 子句中**不能直接使用开窗函数或别名**，需用子查询包裹。

#### 按部门分组、按工资排序的累计薪资求和

```sql
SELECT ename, job, sal, deptno,
       SUM(sal) OVER(PARTITION BY deptno ORDER BY sal)
  FROM emp;
```

> **注意：** 聚合开窗时，`OVER` 里面可以**随意组合** `PARTITION BY` 和 `ORDER BY`。

---

### 2.4 排序开窗

> `OVER` 里面**必须有 `ORDER BY`**。

#### 三种排序函数的区别

| 函数 | 是否考虑并列 | 是否跳过并列 | 示例序列 |
|------|:---:|:---:|------|
| `ROW_NUMBER()` | ❌ | — | 1, 2, 3, 4, 5, 6 |
| `RANK()` | ✅ | ✅ 跳过 | 1, 2, 3, 3, 5, 6 |
| `DENSE_RANK()` | ✅ | ❌ 不跳过 | 1, 2, 3, 3, 4, 5 |

#### 查询每个部门工资的前两名

```sql
-- 使用 ROW_NUMBER()
SELECT *
  FROM (SELECT emp.*,
               ROW_NUMBER() OVER(PARTITION BY deptno ORDER BY sal DESC) AS rn
          FROM emp)
 WHERE rn <= 2;

-- 使用 RANK()
SELECT *
  FROM (SELECT emp.*,
               RANK() OVER(PARTITION BY deptno ORDER BY sal DESC) AS rn
          FROM emp)
 WHERE rn <= 2;

-- 使用 DENSE_RANK()
SELECT *
  FROM (SELECT emp.*,
               DENSE_RANK() OVER(PARTITION BY deptno ORDER BY sal DESC) AS rn
          FROM emp)
 WHERE rn <= 2;
```

#### 查询员工信息及按部门分组、工资升序的累计求和

```sql
SELECT a.*, SUM(sal) OVER(PARTITION BY deptno ORDER BY r)
  FROM (SELECT emp.*,
               ROW_NUMBER() OVER(PARTITION BY deptno ORDER BY sal) AS r
          FROM emp) a;
```

---

### 2.5 偏移开窗

#### `LAG()` — 向前偏移取数

```sql
LAG(要分析的字段, 偏移量(默认1)[, 默认返回值(默认NULL)])
  OVER(PARTITION BY ... ORDER BY ...)
```

> [!info] 注意
> - 偏移量**不允许写负数**
> - 默认返回值的**数据类型要与分析字段一致**
> - `OVER` 里面**必须有 `ORDER BY`**

#### 示例：查询每个员工上一个月的薪水

```sql
SELECT zy981.*,
       LAG(zsal, 1) OVER(PARTITION BY zname ORDER BY zdate) AS 上月薪水
  FROM zy981;
```

#### 环比增长率计算

> **环比**：与上一期（上月/上周/上季/昨天）对比

$$
\text{环比增长率} = \frac{\text{本期数} - \text{上期数}}{\text{上期数}} \times 100\%
$$

```sql
SELECT zy981.*,
       LAG(zsal, 1, 0) OVER(PARTITION BY zname ORDER BY zdate) AS 上月薪水,
       TRUNC(
         (zsal - LAG(zsal, 1, 0) OVER(PARTITION BY zname ORDER BY zdate))
         / DECODE(LAG(zsal, 1, 0) OVER(PARTITION BY zname ORDER BY zdate),
                  0, 1,
                  LAG(zsal, 1, 0) OVER(PARTITION BY zname ORDER BY zdate))
         * 100, 2
       ) || '%' AS 环比增长率
  FROM zy981;
```

#### `LEAD()` — 向后偏移取数

> 参数含义与 `LAG()` 类似，方向相反。

```sql
SELECT zid, zname, zsal,
       LEAD(zsal, 1, 0) OVER(ORDER BY zdate) AS 下月工资
  FROM zy741;
```

#### 同比增长率

> **同比**：与上年同期对比

$$
\text{同比增长率} = \frac{\text{本期数} - \text{上年同期数}}{\text{上年同期数}} \times 100\%
$$

---

## 三、行列转换

### 3.1 行转列

> 将多行数据转换为多列显示。

#### 方法一：`DECODE`

```sql
SELECT sno,
       SUM(DECODE(cno, 'c001', score, 0)) AS c001,
       SUM(DECODE(cno, 'c002', score, 0)) AS c002,
       SUM(DECODE(cno, 'c003', score, 0)) AS c003,
       SUM(DECODE(cno, 'c007', score, 0)) AS c007,
       SUM(DECODE(cno, 'c010', score, 0)) AS c010
  FROM sc
 GROUP BY sno;
```

#### 方法二：`CASE WHEN`

```sql
SELECT sno,
       SUM(CASE WHEN cno = 'c001' THEN score ELSE 0 END) AS c001,
       SUM(CASE WHEN cno = 'c007' THEN score ELSE 0 END) AS c007,
       SUM(CASE WHEN cno = 'c002' THEN score ELSE 0 END) AS c002,
       SUM(CASE WHEN cno = 'c003' THEN score ELSE 0 END) AS c003,
       SUM(CASE WHEN cno = 'c010' THEN score ELSE 0 END) AS c010
  FROM sc
 GROUP BY sno;
```

#### 方法三：`PIVOT`（行转列专用函数）

```sql
SELECT *
  FROM sc
  PIVOT(SUM(score) FOR cno IN(
    'c010' AS a,
    'c001' AS b,
    'c002' AS c,
    'c003' AS d,
    'c007' AS e
  ));
```

> [!warning] `PIVOT` 的聚合函数**不能用 `COUNT`**（对转换列对应的字段）。

---

### 3.2 列转行

> 将多列数据还原为多行。使用专用函数 `UNPIVOT`。

```sql
SELECT *
  FROM lzh
  UNPIVOT(成绩 FOR 课程 IN(c010, c001, c002, c003, c007));
```

**参数说明：**

| 参数 | 说明 |
|------|------|
| `列名1`（成绩） | 用来**接收数据**的列名 |
| `列名2`（课程） | 用来**接收字段名**的列名 |
| `IN(...)` | 对列名2 接收的字段名进行排序 |

---

## 四、集合运算

> 集合运算是指 **SQL 语句之间的运算**。

### 4.1 四种集合运算

| 运算 | 关键字 | 说明 |
|------|--------|------|
| **并集** | `UNION` / `UNION ALL` | 合并两个结果集 |
| **交集** | `INTERSECT` | 取两个结果集的公共部分 |
| **差集** | `MINUS` | 取属于 A 但不属于 B 的部分 |

### 4.2 `UNION` 与 `UNION ALL` 的区别

```sql
-- UNION：去重
SELECT * FROM emp WHERE deptno = 10   -- 5 条
UNION
SELECT * FROM emp WHERE sal > 2000;   -- 4 条
-- 结果：7 条（5 + 4 - 2 重复）

-- UNION ALL：不去重
SELECT * FROM emp WHERE deptno = 10   -- 5 条
UNION ALL
SELECT * FROM emp WHERE sal > 2000;   -- 4 条
-- 结果：9 条（5 + 4）
```

### 4.3 交集 `INTERSECT`

```sql
SELECT * FROM emp WHERE deptno = 10
INTERSECT
SELECT * FROM emp WHERE sal > 2000;
-- 结果：2 条
```

### 4.4 差集 `MINUS`

```sql
-- A - B
SELECT * FROM emp WHERE deptno = 10   -- 5 条
MINUS
SELECT * FROM emp WHERE sal > 2000;   -- 4 条
-- 结果：3 条（5 - 2）

-- B - A（结果不同！）
SELECT * FROM emp WHERE sal > 2000    -- 4 条
MINUS
SELECT * FROM emp WHERE deptno = 10;  -- 5 条
-- 结果：2 条（4 - 2）
```

### 4.5 注意事项

> [!warning] 集合运算注意
> 1. **只有差集有上下之分**（`A MINUS B` ≠ `B MINUS A`）
> 2. **执行顺序从上到下**，括号内先执行
> 3. **除 `UNION ALL` 外**，其余运算都会按**第一列升序排列**
> 4. 各语句的**列个数、顺序、数据类型必须一致**
> 5. **列名可以不一致**，最终展示**第一个语句的字段名**

#### 使用集合运算创建表

```sql
CREATE TABLE result AS
  SELECT * FROM emp WHERE deptno = 10
  UNION
  SELECT * FROM emp1 WHERE sal < 1000;

SELECT * FROM result;
```

---

> [!tip] 总结速记
> - **序列**：自动编号，`nextval`/`currval`，首次必用 `nextval`
> - **聚合开窗**：每行一个聚合值，`WHERE` 不能直接用开窗函数
> - **排序开窗**：`ROW_NUMBER` 不并列 > `RANK` 跳号 > `DENSE_RANK` 不跳号
> - **偏移开窗**：`LAG` 往前看，`LEAD` 往后看
> - **行转列**：`DECODE` / `CASE WHEN` / `PIVOT`
> - **列转行**：`UNPIVOT`
> - **集合运算**：`UNION` 去重，`UNION ALL` 不去重，`MINUS` 有方向
