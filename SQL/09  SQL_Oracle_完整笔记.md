---
title: Oracle SQL 分类速查手册
tags:
  - SQL
  - Oracle
  - 数据库
  - DQL
  - DML
  - DDL
  - DCL
  - TCL
created: 2026-08-29
---

# Oracle SQL 分类速查手册

---

## 目录

- [一、SQL 语言总览](#一sql-语言总览)
- [二、DQL — 数据查询语言](#二dql--数据查询语言)
- [三、DML — 数据操纵语言](#三dml--数据操纵语言)
- [四、DDL — 数据定义语言](#四ddl--数据定义语言)
- [五、DCL — 数据控制语言](#五dcl--数据控制语言)
- [六、TCL — 事务控制语言](#六tcl--事务控制语言)
- [附录一：常用字段属性](#附录一常用字段属性)
- [附录二：执行顺序速记](#附录二执行顺序速记)

---

## 一、SQL 语言总览

|   缩写    |              全称              |   中文   |           核心关键字            |      用途       |
| :-----: | :--------------------------: | :----: | :------------------------: | :-----------: |
| **DQL** |     Data Query Language      | 数据查询语言 |          `SELECT`          |     查询数据      |
| **DML** |  Data Manipulation Language  | 数据操纵语言 | `INSERT` `UPDATE` `DELETE` |    增、改、删数据    |
| **DDL** |   Data Definition Language   | 数据定义语言 |  `CREATE` `ALTER` `DROP`   | 创建/修改/删除数据库对象 |
| **DCL** |    Data Control Language     | 数据控制语言 |      `GRANT` `REVOKE`      |     权限管理      |
| **TCL** | Transaction Control Language | 事务控制语言 |    `COMMIT` `ROLLBACK`     |    事务提交与回滚    |

### Oracle 基础配置

| 项目 | 说明 |
|------|------|
| 预置管理员用户 | `system`（高权限） |
| 预置普通用户 | `SCOTT`，密码 `tiger`（学习用） |
| 全局数据库服务名 | `orcl` |
| 连接串格式 | `IP:端口号/实例名` → `ip:1521/orcl` |
| 常用工具 | PL/SQL Developer（仅Oracle）、DBeaver（推荐，支持多种数据库） |

### SCOTT 用户自带表

| 表名 | 说明 |
|------|------|
| `emp` | 员工信息表（empno, ename, job, mgr, hiredate, sal, comm, deptno） |
| `dept` | 部门信息表（deptno, dname, loc） |
| `salgrade` | 工资等级表 |
| `bonus` | 奖金表 |

---

## 二、DQL — 数据查询语言

### 2.1 完整查询语法

```sql
SELECT   列名 | * | 常量 | 计算 | 函数 | (子查询)    -- 展示内容
  FROM   表名 | (子查询)                             -- 数据源
  [INNER | LEFT | RIGHT | FULL] JOIN 表名 ON 条件    -- 连接
 WHERE   行过滤条件                                   -- 分组前过滤（不能用聚合函数）
 GROUP BY 分组内容                                    -- 分组
 HAVING  分组后过滤条件                                -- 分组后过滤（可以用聚合函数）
 ORDER BY 排序字段 ASC | DESC;                        -- 排序（永远最后执行）
```

**执行顺序：** `FROM` → `JOIN` → `WHERE` → `GROUP BY` → `HAVING` → `SELECT` → `ORDER BY`

---

### 2.2 基础查询

#### 查询特定字段

```sql
SELECT ename, job, sal FROM emp;
```

#### 查询所有字段

```sql
-- 方法1：*（效率低，不推荐）
SELECT * FROM emp;

-- 方法2：显式列出所有字段（推荐）
SELECT empno, ename, job, mgr, sal, comm, hiredate, deptno FROM emp;
```

#### 连接符 `||`

```sql
-- 将两侧字符拼接成一个字符
SELECT 'abc' || '1223' FROM dual;        -- abc1223
SELECT ename || job, sal FROM emp;        -- 刘亦菲保洁
SELECT ename || job || sal FROM emp;
```

> `dual` 是一张空表，用于补全语法

#### 别名

```sql
-- 列别名（三种写法）
SELECT ename AS "姓名",
       job    "岗位",
       sal     工资            -- 常用写法
  FROM emp;

-- 表别名
SELECT e.ename, e.job, e.sal FROM emp e;
```

**别名注意事项：**
1. 别名只在当前语句有效，不是改名
2. 不建议使用中文
3. 不建议使用数字和特殊符号，如使用则 `""` 不能省略
4. 一旦取了表别名，不能再使用原表名
5. `*` 与其他字段混用时，`*` 前必须加表名：`SELECT emp.*, ename FROM emp;`

#### 数值计算

```sql
-- 运算符：+ - * / ()
SELECT ename, sal, comm, sal/30 FROM emp;
SELECT ename, sal, comm, (sal + comm) * 12 FROM emp;
```

> **NULL 不参与计算**，任何数值与 NULL 进行加减乘除，结果仍为 NULL

---

### 2.3 条件查询（WHERE）

```sql
SELECT * | 列名 FROM 表名 WHERE 过滤条件;
```

#### 比较运算

| 运算符 | 说明 |
|--------|------|
| `=` | 等于 |
| `<>` 或 `!=` | 不等于 |
| `<` `>` | 小于 / 大于 |
| `<=` `>=` | 小于等于 / 大于等于 |

```sql
SELECT * FROM emp WHERE sal > 1000;
SELECT * FROM emp WHERE ename = 'SCOTT';   -- 数据区分大小写
SELECT * FROM emp WHERE hiredate < TO_DATE('1982/1/1', 'yyyy/mm/dd');
```

#### NULL 值判断

```sql
SELECT * FROM emp WHERE comm IS NOT NULL;  -- 有奖金的员工
SELECT * FROM emp WHERE comm IS NULL;      -- 没有奖金的员工
SELECT * FROM emp WHERE mgr IS NULL;       -- 最大的领导（没有上级）
```

#### 模糊查询 LIKE

| 通配符 | 说明 |
|--------|------|
| `%` | 占 0 位或多位 |
| `_` | 占 1 位 |

```sql
SELECT * FROM emp WHERE ename LIKE 'S%';      -- 以S开头
SELECT * FROM emp WHERE ename NOT LIKE 'S%';  -- 不以S开头
SELECT * FROM emp WHERE ename LIKE '%T_';     -- 倒数第二位是T
SELECT * FROM emp WHERE ename LIKE '_____' ;  -- 姓名总共5位
SELECT * FROM emp WHERE ename LIKE '%A%';     -- 包含A
```

#### 范围查询 BETWEEN AND

```sql
-- 包含边界值，小值写前面
SELECT * FROM emp WHERE sal BETWEEN 1000 AND 3000;

SELECT * FROM emp
 WHERE hiredate BETWEEN TO_DATE('1980/1/1', 'yyyy/mm/dd')
                    AND TO_DATE('1981/5/31', 'yyyy/mm/dd');
```

#### 包含查询 IN

```sql
SELECT * FROM emp WHERE deptno IN (10, 30);
SELECT * FROM emp WHERE deptno NOT IN (10, 30);
SELECT * FROM emp WHERE job IN ('SALESMAN', 'MANAGER');
```

#### ANY 和 ALL

```sql
SELECT * FROM emp WHERE sal > ANY(1000, 3000);  -- 大于任意一个（即大于最小值1000）
SELECT * FROM emp WHERE sal > ALL(1000, 3000);  -- 大于所有（即大于最大值3000）
```

#### 条件连接 AND / OR

| 运算符 | 说明 | 优先级 |
|--------|------|--------|
| `AND` | 并且（同时满足） | **高** |
| `OR` | 或者（满足其一） | 低 |

```sql
-- AND 优先于 OR，加 () 可改变优先级
SELECT * FROM emp WHERE deptno = 10 AND job = 'MANAGER';
SELECT * FROM emp WHERE deptno = 10 OR deptno = 20;
SELECT * FROM emp WHERE deptno IN (10, 20);   -- 等价写法

SELECT * FROM emp
 WHERE (deptno = 10 AND job = 'MANAGER')
    OR (deptno = 20 AND job = 'ANALYST');
```

---

### 2.4 排序查询（ORDER BY）

```sql
SELECT * FROM emp ORDER BY sal ASC;           -- 升序（默认）
SELECT * FROM emp ORDER BY sal DESC;          -- 降序
SELECT * FROM emp ORDER BY sal DESC NULLS LAST;  -- 降序，空值排最后
SELECT * FROM emp ORDER BY deptno ASC, sal DESC; -- 多列排序
```

**注意事项：**
- 空值最大：排序时 NULL 被视为最大值
- `NULLS FIRST` / `NULLS LAST`：强制空值排最前/最后
- `ORDER BY` 后面可以使用别名
- `WHERE` 后面不能使用别名

---

### 2.5 分组查询（GROUP BY + HAVING）

#### 聚合函数

| 函数 | 作用 | 说明 |
|------|------|------|
| `SUM(列名)` | 求和 | 忽略 NULL |
| `AVG(列名)` | 求平均 | 忽略 NULL |
| `MIN(列名)` | 最小值 | 忽略 NULL |
| `MAX(列名)` | 最大值 | 忽略 NULL |
| `COUNT(列名)` | 计数 | 统计该列非空值个数 |
| `COUNT(*)` | 计数 | 统计总行数 |
| `COUNT(1)` | 计数 | 统计总行数（效率同 `COUNT(*)`） |

#### 分组查询示例

```sql
-- 每个部门的平均工资
SELECT deptno, AVG(sal) FROM emp GROUP BY deptno;

-- 每个部门每个岗位的平均工资
SELECT deptno, job, AVG(sal) FROM emp GROUP BY deptno, job;

-- 完整分组查询：WHERE + GROUP BY + HAVING + ORDER BY
SELECT deptno, job, AVG(sal)
  FROM emp
 WHERE sal > 1000                  -- 分组前过滤
 GROUP BY deptno, job
 HAVING AVG(sal) > 2000            -- 分组后过滤
 ORDER BY AVG(sal) DESC;
```

#### WHERE 与 HAVING 的区别

| 特性 | WHERE | HAVING |
|------|-------|--------|
| 过滤时机 | 分组前 | 分组后 |
| 使用聚合函数 | ❌ 不能 | ✅ 可以 |
| 执行顺序 | 先执行 | 后执行 |

#### SELECT 与 GROUP BY 的规则

> SELECT 后面只能展示 GROUP BY 后面有的字段（或聚合函数）

```sql
-- ❌ 错误：ename 不在 GROUP BY 中
SELECT ename, deptno, AVG(sal) FROM emp GROUP BY deptno;

-- ✅ 正确
SELECT deptno, AVG(sal) FROM emp GROUP BY deptno;
```

---

### 2.6 子查询

> 子查询是指一条语句嵌套一条或多条查询语句。里层的子查询需要加 `()`。

#### 分类

| 分类 | 说明 | 示例 |
|------|------|------|
| **相关子查询** | 里层与外层有关系，不能单独运行 | `WHERE sal > (SELECT AVG(sal) FROM emp WHERE deptno = e.deptno)` |
| **非相关子查询** | 里层与外层无关系，能单独运行 | `WHERE deptno = (SELECT deptno FROM emp WHERE ename='SCOTT')` |

#### 非相关子查询按返回结果分类

| 分类   | 示例                                   | 用途                    |
| ---- | ------------------------------------ | --------------------- |
| 单行单列 | `SELECT AVG(sal) FROM emp`           | `=` `>` `<` 比较        |
| 单行多列 | `SELECT * FROM emp WHERE empno=7566` | `(col1,col2) = (子查询)` |
| 单列多行 | `SELECT ename FROM emp`              | `IN` `ANY` `ALL`      |
| 多列多行 | `SELECT * FROM emp`                  | 放在 `FROM` 后面当表        |

#### 子查询解题三步法

> 第一步：写主查询，子查询部分用**汉语**表达清楚
> 第二步：单独写子查询，实现括号里的汉语
> 第三步：合并

```sql
-- 查询和刘亦菲在同一个部门的员工信息

-- 第一步：主查询
SELECT * FROM emp WHERE deptno = (刘亦菲的部门编号)

-- 第二步：子查询
SELECT deptno FROM emp WHERE ename='刘亦菲'

-- 第三步：合并
SELECT * FROM emp
 WHERE deptno = (SELECT deptno FROM emp WHERE ename='刘亦菲')
```

#### 子查询示例

```sql
-- 查询部门和ALLEN相同但岗位不同的员工
SELECT * FROM emp
 WHERE deptno = (SELECT deptno FROM emp WHERE ename='ALLEN')
   AND job <> (SELECT job FROM emp WHERE ename='ALLEN');

-- 子查询放在 FROM 后面当表（派生表）
SELECT * FROM (SELECT deptno, AVG(sal) avg_sal FROM emp GROUP BY deptno);

-- 子查询放在 SELECT 后面（标量子查询）
SELECT ename, sal, (SELECT AVG(sal) FROM emp) avg_sal FROM emp;
```

---

### 2.7 连接查询（表连接）

> 将两张表或多张表横向拼接到一起，形成一个临时的大表。

#### 连接分类

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

#### 笛卡尔积

```sql
SELECT * FROM emp, dept;   -- 笛卡尔积：17 × 6 = 102 行
```

> 笛卡尔积会产生大量错误数据，工作中应尽量避免

#### 内连接 INNER JOIN

```sql
-- 基本内连接
SELECT *
  FROM emp
 INNER JOIN dept
    ON emp.deptno = dept.deptno;

-- 内连接 + 条件过滤
SELECT ename, job, sal, dept.deptno, dname, loc
  FROM emp
 INNER JOIN dept
    ON emp.deptno = dept.deptno
 WHERE loc = 'NEW YORK';
```

#### 左外连接 LEFT JOIN

```sql
-- 左表为主，左表数据全部显示，右表不匹配填 NULL
SELECT *
  FROM emp
  LEFT JOIN dept
    ON emp.deptno = dept.deptno;
```

#### 右外连接 RIGHT JOIN

```sql
-- 右表为主，右表数据全部显示，左表不匹配填 NULL
SELECT *
  FROM emp
 RIGHT JOIN dept
    ON emp.deptno = dept.deptno;
```

#### 全外连接 FULL JOIN

```sql
-- 两表全部显示，不匹配填 NULL
SELECT *
  FROM emp
  FULL JOIN dept
    ON emp.deptno = dept.deptno;
```

#### 自连接

```sql
-- 查询员工及其领导姓名
SELECT e.ename 员工, m.ename 领导
  FROM emp e
  LEFT JOIN emp m
    ON e.mgr = m.empno;
```

#### 不等值连接

```sql
-- 查询员工薪资所在的工资等级
SELECT e.ename, e.sal, g.grade
  FROM emp e
 INNER JOIN salgrade g
    ON e.sal BETWEEN g.losal AND g.hisal;
```

#### 多表连接

```sql
-- 员工 + 部门 + 工资等级
SELECT e.ename, e.job, e.sal, d.dname, d.loc, g.grade
  FROM emp e
 INNER JOIN dept d ON e.deptno = d.deptno
 INNER JOIN salgrade g ON e.sal BETWEEN g.losal AND g.hisal;
```

---

### 2.8 Oracle 常用函数

#### 转换函数

```sql
-- to_date：字符串转日期
TO_DATE('1982/1/1', 'yyyy/mm/dd')

-- to_char：数值/日期转字符
TO_CHAR(123.45, '000.00')                        -- 123.45
TO_CHAR(123.45, '00000.00')                      -- 00123.45
TO_CHAR(123.45, 'L000.0000元')                   -- ￥123.4500元
TO_CHAR(123.45, '$000.0000')                     -- $123.4500
TO_CHAR(123456789.12, '999,999,999.99')          -- 123,456,789.12
TO_CHAR(SYSDATE, 'yyyy-mm-dd')                   -- 2026-08-29
TO_CHAR(SYSDATE, 'yyyy')                         -- 2026
TO_CHAR(SYSDATE, 'mm')                           -- 08
TO_CHAR(SYSDATE, 'day')                          -- 星期五
```

#### 字符型函数

| 函数 | 作用 | 示例 |
|------|------|------|
| `CONCAT(a, b)` | 连接两个字符（等价于 `\|\|`） | `CONCAT('ab', 'cd')` → `abcd` |
| `UPPER(字符)` | 全部变大写 | `UPPER('abc')` → `ABC` |
| `LOWER(字符)` | 全部变小写 | `LOWER('ABC')` → `abc` |
| `INITCAP(字符)` | 首字母大写 | `INITCAP('abc')` → `Abc` |
| `INSTR(字符, S, n1, n2)` | 从第 n1 位开始，第 n2 次出现 S 的位置 | `INSTR('ABCDE', 'C', 1, 1)` → `3` |
| `LENGTH(字符)` | 获取长度 | `LENGTH('abc')` → `3` |
| `LTRIM(字符, 内容)` | 删除左侧指定内容 | 默认删除左侧空格 |
| `RTRIM(字符, 内容)` | 删除右侧指定内容 | 默认删除右侧空格 |
| `TRIM(字符 FROM 目标)` | 删除左右两侧指定内容 | 默认删除左右空格 |
| `REPLACE(字符, 旧, 新)` | 替换 | `REPLACE('abc', 'b', 'X')` → `aXc` |
| `SUBSTR(字符, 位置, 长度)` | 截取子串 | `SUBSTR('ABCDE', 2, 3)` → `BCD` |
| `LPAD(字符, 长度, 填充)` | 左侧填充 | `LPAD('abc', 6, '*')` → `***abc` |
| `RPAD(字符, 长度, 填充)` | 右侧填充 | `RPAD('abc', 6, '*')` → `abc***` |

#### 数值型函数

| 函数 | 作用 | 示例 |
|------|------|------|
| `ROUND(数值, 精度)` | 四舍五入 | `ROUND(123.456, 2)` → `123.46` |
| `TRUNC(数值, 精度)` | 截断（不四舍五入） | `TRUNC(123.456, 2)` → `123.45` |
| `MOD(数值1, 数值2)` | 取余 | `MOD(10, 3)` → `1` |
| `CEIL(数值)` | 向上取整 | `CEIL(123.01)` → `124` |
| `FLOOR(数值)` | 向下取整 | `FLOOR(123.99)` → `123` |
| `ABS(数值)` | 取绝对值 | `ABS(-10)` → `10` |
| `POWER(底数, 指数)` | 幂运算 | `POWER(2, 3)` → `8` |
| `SQRT(数值)` | 开方 | `SQRT(9)` → `3` |
| `SIGN(数值)` | 判断正负 | `SIGN(5)` → `1`，`SIGN(-5)` → `-1`，`SIGN(0)` → `0` |

#### 日期型函数

| 函数 | 作用 | 示例 |
|------|------|------|
| `SYSDATE` | 获取当前系统日期 | `SELECT SYSDATE FROM dual` |
| `ADD_MONTHS(日期, n)` | 日期加减 n 个月 | `ADD_MONTHS(SYSDATE, 3)` |
| `MONTHS_BETWEEN(日期1, 日期2)` | 两个日期相差的月数 | `MONTHS_BETWEEN(SYSDATE, TO_DATE('2026/1/1','yyyy/mm/dd'))` |
| `LAST_DAY(日期)` | 该月最后一天 | `LAST_DAY(SYSDATE)` |
| `NEXT_DAY(日期, '星期X')` | 下一个星期X | `NEXT_DAY(SYSDATE, '星期一')` |
| `EXTRACT(YEAR/MONTH/DAY FROM 日期)` | 提取年/月/日 | `EXTRACT(YEAR FROM SYSDATE)` → `2026` |

#### 通用函数

| 函数 | 作用 | 示例 |
|------|------|------|
| `NVL(值1, 值2)` | 值1为NULL时返回值2 | `NVL(comm, 0)` |
| `NVL2(值1, 值2, 值3)` | 值1不为NULL返回值2，否则返回值3 | `NVL2(comm, sal+comm, sal)` |
| `DECODE(值, 条件1, 结果1, 条件2, 结果2, 默认值)` | 多条件判断 | `DECODE(deptno, 10, '研发', 20, '销售', '其他')` |
| `CASE WHEN` | 条件表达式 | 见下方 |

#### CASE WHEN 表达式

```sql
-- 简单 CASE
SELECT ename, job, sal,
       CASE job
         WHEN 'MANAGER'  THEN sal * 1.2
         WHEN 'SALESMAN' THEN sal * 1.1
         ELSE sal
       END AS new_sal
  FROM emp;

-- 搜索 CASE
SELECT ename, sal,
       CASE
         WHEN sal >= 3000 THEN '高薪'
         WHEN sal >= 1500 THEN '中薪'
         ELSE '低薪'
       END AS salary_level
  FROM emp;
```

#### 空值（NULL）特性总结

- NULL 不参与比较运算（`=` `<>` `>` `<` 均无法判断）
- NULL 不参与计算（结果仍为 NULL）
- 判断 NULL 使用 `IS NULL` / `IS NOT NULL`
- 空值最大：排序时 NULL 被视为最大值

---

### 2.9 开窗函数（分析函数 / 窗口函数）

> 搭配 `OVER()` 子句使用，能在不改变行数的前提下对数据进行分组分析。

```sql
分析函数名() OVER(PARTITION BY 分组 ORDER BY 排序 ROWS 窗口范围)
```

#### 聚合开窗

```sql
-- 每一行返回该部门的平均工资
SELECT ename, job, sal, deptno,
       AVG(sal) OVER(PARTITION BY deptno) AS avg_dept_sal
  FROM emp;

-- 按部门分组、按工资排序的累计薪资求和
SELECT ename, job, sal, deptno,
       SUM(sal) OVER(PARTITION BY deptno ORDER BY sal)
  FROM emp;
```

#### 排序开窗

| 函数 | 是否考虑并列 | 是否跳过并列 | 示例序列 |
|------|:---:|:---:|------|
| `ROW_NUMBER()` | ❌ | — | 1, 2, 3, 4, 5, 6 |
| `RANK()` | ✅ | ✅ 跳过 | 1, 2, 3, 3, 5, 6 |
| `DENSE_RANK()` | ✅ | ❌ 不跳过 | 1, 2, 3, 3, 4, 5 |

```sql
-- 查询每个部门工资的前两名
SELECT *
  FROM (SELECT emp.*,
               ROW_NUMBER() OVER(PARTITION BY deptno ORDER BY sal DESC) AS rn
          FROM emp)
 WHERE rn <= 2;
```

#### 偏移开窗

```sql
-- LAG：向前偏移（取上一期数据）
-- LAG(字段, 偏移量, 默认返回值) OVER(PARTITION BY ... ORDER BY ...)
SELECT ename, sal, hiredate,
       LAG(sal, 1, 0) OVER(ORDER BY hiredate) AS 上月薪水
  FROM emp;

-- LEAD：向后偏移（取下一期数据）
SELECT ename, sal, hiredate,
       LEAD(sal, 1, 0) OVER(ORDER BY hiredate) AS 下月薪水
  FROM emp;
```

#### 环比增长率计算

```sql
-- 环比 = (本期 - 上期) / 上期 × 100%
SELECT zname, zsal, zdate,
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

---

### 2.10 行列转换

#### 行转列

```sql
-- 方法一：DECODE
SELECT sno,
       SUM(DECODE(cno, 'c001', score, 0)) AS c001,
       SUM(DECODE(cno, 'c002', score, 0)) AS c002,
       SUM(DECODE(cno, 'c003', score, 0)) AS c003
  FROM sc
 GROUP BY sno;

-- 方法二：CASE WHEN
SELECT sno,
       SUM(CASE WHEN cno = 'c001' THEN score ELSE 0 END) AS c001,
       SUM(CASE WHEN cno = 'c002' THEN score ELSE 0 END) AS c002
  FROM sc
 GROUP BY sno;

-- 方法三：PIVOT（专用函数，聚合函数不能用 COUNT）
SELECT *
  FROM sc
  PIVOT(SUM(score) FOR cno IN(
    'c001' AS c001,
    'c002' AS c002,
    'c003' AS c003
  ));
```

#### 列转行

```sql
SELECT *
  FROM 表名
  UNPIVOT(数据列名 FOR 字段名列名 IN(列1, 列2, 列3));
```

---

### 2.11 集合运算

| 运算 | 关键字 | 说明 |
|------|--------|------|
| 并集（去重） | `UNION` | 合并两个结果集，去除重复 |
| 并集（不去重） | `UNION ALL` | 合并两个结果集，保留重复 |
| 交集 | `INTERSECT` | 取两个结果集的公共部分 |
| 差集 | `MINUS` | 取属于 A 但不属于 B 的部分 |

```sql
-- UNION 去重
SELECT * FROM emp WHERE deptno = 10
UNION
SELECT * FROM emp WHERE sal > 2000;

-- UNION ALL 不去重
SELECT * FROM emp WHERE deptno = 10
UNION ALL
SELECT * FROM emp WHERE sal > 2000;

-- 交集
SELECT * FROM emp WHERE deptno = 10
INTERSECT
SELECT * FROM emp WHERE sal > 2000;

-- 差集（有方向！A-B ≠ B-A）
SELECT * FROM emp WHERE deptno = 10   -- A
MINUS
SELECT * FROM emp WHERE sal > 2000;   -- B
```

**注意事项：**
1. 只有差集有上下之分（`A MINUS B` ≠ `B MINUS A`）
2. 执行顺序从上到下，括号内先执行
3. 除 `UNION ALL` 外，其余运算都会按第一列升序排列
4. 各语句的列个数、顺序、数据类型必须一致
5. 列名可以不一致，最终展示第一个语句的字段名

---

## 三、DML — 数据操纵语言

> 针对**数据**进行增、删、改操作的语言。DML 语句需要**提交（COMMIT）才能生效**，提交之后**不能回滚**。

### 3.1 INSERT — 增加数据

```sql
-- 指定列名插入
INSERT INTO emp (empno, ename, job, sal, hiredate, deptno)
       VALUES (2778, '康志存', '男明星', 28000, SYSDATE, 10);

-- 不指定列名，按表结构顺序插入所有字段
INSERT INTO dept VALUES (70, '娱乐部门', '澳门');

-- 没有插入的字段默认是空值
```

#### 快捷插入（子查询插入）

```sql
-- 将查询结果插入到表中
INSERT INTO 表名 [(列名1, 列名2, ...)] SELECT 语句;

-- 示例：将 30 号部门的员工信息插入到奖金表
INSERT INTO bonus SELECT ename, job, sal, comm FROM emp WHERE deptno = 30;
```

**注意事项：**
- 值的个数、顺序、属性要和列一致
- 空字符串 `''` 不等于 NULL

---

### 3.2 DELETE — 删除数据

```sql
-- 删除所有数据（慎用！）
DELETE FROM emp;

-- 按条件删除
DELETE FROM emp WHERE deptno = 10;
DELETE FROM emp WHERE ename = '康志存';
```

---

### 3.3 UPDATE — 修改数据

```sql
-- 修改所有行
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

**注意事项：**
- 值可以是具体值、函数、计算表达式、或子查询（必须单行单列）
- 列名和值不能写反
- 不加 WHERE 条件会修改所有行

---

### 3.4 MERGE INTO — 同时增删改

> 可以同时完成增、删、改操作。

```sql
MERGE INTO 目标表 d
 USING 源表 s
    ON (关联条件)
  WHEN MATCHED THEN
    UPDATE SET d.列 = s.列              -- 匹配到则更新
    [DELETE WHERE 删除条件]              -- 可选：更新后删除
  WHEN NOT MATCHED THEN
    INSERT (列1, 列2) VALUES (值1, 值2); -- 未匹配则插入
```

---

## 四、DDL — 数据定义语言

> 针对**数据库对象**（表、视图、索引、序列等）进行创建、修改、删除操作。DDL 语句执行后**隐式自动提交**。

### 4.1 创建表（CREATE TABLE）

```sql
CREATE TABLE 表名 (
  列名1 字段属性 [约束],
  列名2 字段属性 [约束],
  列名3 字段属性 [约束]
);
```

#### 示例

```sql
CREATE TABLE zy98 (
  zno   NUMBER(7),
  zname VARCHAR2(20),
  zsr   DATE
);
```

---

### 4.2 修改表（ALTER TABLE）

#### 修改表名

```sql
ALTER TABLE 表名 RENAME TO 新表名;
ALTER TABLE zhiyun98 RENAME TO zy98;
```

#### 修改字段名

```sql
ALTER TABLE 表名 RENAME COLUMN 列名 TO 新列名;
ALTER TABLE zy98 RENAME COLUMN zno TO ano;
```

#### 添加字段

```sql
ALTER TABLE 表名 ADD (列名1 字段属性, 列名2 字段属性 ...);
ALTER TABLE zy98 ADD (zno NUMBER, zsr DATE);
```

#### 删除字段

```sql
ALTER TABLE 表名 DROP (列名1, 列名2 ...);
ALTER TABLE zy98 DROP (ano);
```

#### 修改字段属性（MODIFY）

```sql
ALTER TABLE 表名 MODIFY (列名 新字段属性);
ALTER TABLE zy98 MODIFY (zname VARCHAR2(40));   -- 改大 ✓
```

**注意事项：**
- 可以改大，不能小于现有数据的最大长度
- 修改属性类型时，该列必须为空

---

### 4.3 删除表（DROP TABLE）

```sql
DROP TABLE 表名;
DROP TABLE zy98;
```

---

### 4.4 清空表（TRUNCATE）

```sql
TRUNCATE TABLE 表名;
```

> TRUNCATE 与 DELETE 的区别：TRUNCATE 是 DDL（隐式提交，不可回滚），DELETE 是 DML（需显式提交，可回滚）

---

### 4.5 约束（Constraint）

> 约束是强加在表中的规则或条件，使表更符合实际需求。

#### 约束分类

| 约束类型 | 关键字 | 作用 | 能否约束空值 |
|---------|--------|------|:---:|
| 唯一约束 | `UNIQUE` | 该列不能出现重复的非空数据 | **不能** |
| 主键约束 | `PRIMARY KEY` | 不能重复，不能空值（每表仅一个） | **能** |
| 外键约束 | `FOREIGN KEY` | 取值来自另一张表的主键/唯一列 | **不能** |
| 检查约束 | `CHECK` | 自定义约束条件 | **不能** |
| 非空约束 | `NOT NULL` | 该列不能出现空值 | **能** |
| 默认值 | `DEFAULT` | 不插入数据时填入默认值 | — |

#### 建表时添加约束（行级约束）

```sql
CREATE TABLE zy98 (
  zno   NUMBER        PRIMARY KEY,
  zname VARCHAR2(20)  UNIQUE,
  zsr   DATE          CHECK(zsr < TO_DATE('2026/8/27', 'yyyy/mm/dd')),
  zdno  NUMBER        REFERENCES dept(deptno)
);
```

#### 建表时添加约束（表级约束）

```sql
CREATE TABLE zy98 (
  zno   NUMBER,
  zname VARCHAR2(20),
  zsr   DATE,
  zdno  NUMBER,
  CONSTRAINT pk_98 PRIMARY KEY(zno),
  CONSTRAINT un_98 UNIQUE(zname),
  CONSTRAINT ch_98 CHECK(zno > 1 OR zname = '李许铭'),
  CONSTRAINT fk_98 FOREIGN KEY(zdno) REFERENCES dept(deptno)
);
```

#### 建表后添加约束

```sql
-- 唯一约束
ALTER TABLE zy98 ADD CONSTRAINT un_zy98_zname UNIQUE(zname);

-- 主键约束
ALTER TABLE zy98 ADD CONSTRAINT pk_zy98_zno PRIMARY KEY(zno);

-- 外键约束
ALTER TABLE zy98 ADD
  CONSTRAINT fk_zy98_zname
  FOREIGN KEY(zname) REFERENCES emp(ename);

-- 检查约束
ALTER TABLE zy98 ADD
  CONSTRAINT ch_zy98_zno
  CHECK(zno > 1 OR zname = '李许铭');

-- 非空约束
ALTER TABLE zy98 MODIFY zname NOT NULL;

-- 默认值
ALTER TABLE zy98 MODIFY zname DEFAULT '李许铭';
```

#### 删除约束

```sql
-- 删除普通约束
ALTER TABLE 表名 DROP CONSTRAINT 约束名;

-- 删除非空
ALTER TABLE 表名 MODIFY 列名 NULL;

-- 删除默认值
ALTER TABLE 表名 MODIFY 列名 DEFAULT NULL;
```

#### 外键删除策略

| 策略 | 效果 |
|------|------|
| `NO ACTION` | 子表有记录时，父表不能删除（默认） |
| `CASCADE` | 删除父表时，子表相关记录一起删除 |
| `SET NULL` | 删除父表时，子表对应字段设为空 |

---

### 4.6 视图（View）

> 视图是将 SELECT 语句动态保存到数据库中，形成一张虚拟表。本身不包含数据，数据来源于基表。

```sql
-- 创建视图
CREATE [OR REPLACE] VIEW 视图名 AS
  SELECT 语句
  [WITH READ ONLY];       -- 只读视图

-- 删除视图
DROP VIEW 视图名;
```

#### 示例

```sql
-- 单表视图
CREATE OR REPLACE VIEW V_98 AS
  SELECT ename, job, sal, deptno
  FROM emp
  WHERE deptno = 20;

-- 多表视图
CREATE OR REPLACE VIEW V_98 AS
  SELECT ename, job, sal, emp.deptno, dname, loc
  FROM emp
  LEFT JOIN dept ON emp.deptno = dept.deptno;

-- 基于视图创建视图
CREATE OR REPLACE VIEW V_981 AS
  SELECT dname, AVG(sal) AS avg_sal
  FROM V_98
  GROUP BY dname;
```

---

### 4.7 索引（Index）

> 通过建立索引可以提高查询效率，但会降低 DML 语句的速度。

#### 伪列

| 伪列 | 说明 | 示例 |
|------|------|------|
| `ROWID` | 每行数据的物理地址，18位不重复字符串 | `SELECT * FROM emp WHERE ROWID = 'AAAR3sAAEAAAACTAAD'` |
| `ROWNUM` | 对查询结果从1开始的连续序号 | `SELECT emp.*, ROWNUM FROM emp WHERE ROWNUM <= 3` |

#### 扫描方式

| 扫描方式 | 说明 |
|----------|------|
| 全盘扫描 | 从第一行检索到最后一行 |
| 索引扫描 | 从大概位置开始检索到最后一个符合条件的数据 |
| ROWID 扫描 | 直接通过 ROWID 定位 |

#### 索引分类

| 类型 | 说明 | 使用场景 |
|------|------|---------|
| B-tree 索引（默认） | 索引列原始数据 + ROWID | 列基数大（不重复数据多） |
| 位图索引 | 位图 + ROWID | 列基数小，如性别、部门编号 |
| 反向键索引 | 原始数据反向存储 + ROWID | 某节点占比过高 |
| 基于函数的索引 | 函数处理后的数据 + ROWID | 查询常带函数 |
| 单列索引 | 基于一个字段 | — |
| 复合索引 | 基于多个字段 | — |
| 唯一索引 | 索引列数据唯一 | — |
| 非唯一索引 | 索引列数据可重复 | — |

#### 创建索引

```sql
-- B-tree 索引（默认）
CREATE INDEX 索引名 ON 表名(列名1, 列名2 ...);

-- 位图索引
CREATE BITMAP INDEX 索引名 ON 表名(列名1, 列名2 ...);

-- 唯一索引
CREATE UNIQUE INDEX 索引名 ON 表名(列名1, 列名2 ...);

-- 反向键索引
CREATE INDEX 索引名 ON 表名(列名1, 列名2 ...) REVERSE;

-- 基于函数的索引
CREATE INDEX 索引名 ON 表名(函数(列名1));
```

#### 复合索引 — 最左原则

假设索引列为 `(A, B, C)`：

| WHERE 条件 | 是否命中索引 |
|------------|:---:|
| `A = AND B =` | **会** |
| `A = AND C =` | **会**（效率降低） |
| `B = AND C =` | 不会 |
| `C =` | 不会 |

#### 索引失效情况

1. 连接条件使用 `OR` 时，索引失效
2. 通配符出现在搜索词首位时索引失效，如 `ename LIKE '%A%'`
3. 必须将索引列放在条件前面

> **索引不是越多越好**

---

### 4.8 序列（Sequence）

> 序列是 Oracle 提供的一组能够自动增长的序号，主要用于为主键列提供数据。

```sql
-- 创建序列
CREATE SEQUENCE 序列名
  START WITH n            -- 初始值，默认 1
  INCREMENT BY n          -- 步长，默认 1
  MAXVALUE n | NOMAXVALUE -- 最大值，默认 10^27
  MINVALUE n | NOMINVALUE -- 最小值，默认 -10^27
  CYCLE | NOCYCLE         -- 循环/不循环，默认不循环
  CACHE n | NOCACHE       -- 缓存，默认缓存 20 个序号

-- 使用序列
SELECT seq_98.currval FROM dual;   -- 当前值
SELECT seq_98.nextval FROM dual;   -- 下一个值

-- 在插入数据时使用序列
INSERT INTO zy98 VALUES(seq_98.nextval, '李许铭');
```

**注意事项：**
1. 序列刚创建时没有值
2. 第一次使用必须用 `nextval`，返回的是初始值
3. 循环都是从 1 开始
4. 缓存必须 < 最大值 / 步长

---

## 五、DCL — 数据控制语言

> 针对**权限**进行管理的语言。

### 5.1 创建用户

```sql
CREATE USER 用户名 IDENTIFIED BY 密码;
CREATE USER 李许铭 IDENTIFIED BY 123456;
```

### 5.2 赋权（GRANT）

```sql
GRANT 权限 TO 用户;
GRANT CREATE VIEW TO scott;
GRANT CREATE SESSION TO 李许铭;       -- 登录权限
GRANT SELECT ANY TABLE TO 李许铭;    -- 查看任意表
```

### 5.3 收权（REVOKE）

```sql
REVOKE 权限 FROM 用户;
REVOKE CREATE VIEW FROM scott;
```

### 5.4 角色（Role）

> 角色 = 一系列权限的集合

```sql
-- 创建角色
CREATE ROLE 角色名;

-- 给角色赋权
GRANT 权限 TO 角色;

-- 将角色交给用户
GRANT 角色 TO 用户;

-- 删除角色
DROP ROLE 角色名;
```

#### 示例

```sql
CREATE ROLE 保安队长;
GRANT CREATE SESSION TO 保安队长;       -- 登录权限
GRANT SELECT ANY TABLE TO 保安队长;    -- 查看任意表
CREATE USER 李许铭 IDENTIFIED BY 123456;
GRANT 保安队长 TO 李许铭;
SELECT * FROM scott.emp;
DROP ROLE 保安队长;
```

---

## 六、TCL — 事务控制语言

> 针对**事务**进行控制的语言。

### 6.1 基本命令

| 命令 | 作用 |
|------|------|
| `COMMIT` | 提交事务 |
| `ROLLBACK` | 回滚事务 |

```sql
INSERT INTO emp(empno, ename) VALUES(4321, '刘赫');
COMMIT;    -- 提交后不可回滚
```

### 6.2 事务的定义

> 事务：为完成某项业务/任务，由一系列可见的 SQL 和不可见的后台进程组成的逻辑工作单元

### 6.3 事务四大特性（ACID）

| 特性 | 说明 |
|------|------|
| **一致性（Consistency）** | 事务一旦提交，所有数据保持一致 |
| **原子性（Atomicity）** | 事务是一个整体，不能再分割 |
| **隔离性（Isolation）** | 当前事务操作时，其他事务只能查看修改之前的数据 |
| **持久性（Durability）** | 事务一旦提交，数据永久保存 |

### 6.4 提交方式

| 方式 | 说明 | 适用语句 |
|------|------|---------|
| 显式提交 | 需要 `COMMIT` 才能生效 | DML 语句 |
| 隐式提交 | 执行成功后 Oracle 自动提交 | DDL 语句 |

### 6.5 并发问题

| 问题 | 说明 |
|------|------|
| 脏读 | 读到未提交的数据 |
| 幻读 | 两次查询结果行数不一致 |
| 不可重复读 | 两次查询同一行数据不一致 |

### 6.6 锁

| 锁类型 | 说明 |
|--------|------|
| 共享锁 | 多个事务可同时读 |
| 排它锁 | 独占，其他事务不能读写 |

---

## 附录一：常用字段属性

### 数值型

| 属性 | 说明 |
|------|------|
| `NUMBER([p][,s])` | 常用数值型，p = 总长度（含小数位），s = 精度（小数位数） |
| `NUMBER(7, 2)` | 总长 7，小数 2 位，整数最多 5 位 |
| `NUMBER(7)` | 只保留整数 |
| `NUMBER` | 默认长度 38 |
| `INT` | 存储整数，等价于 `NUMBER(38)` |

**`NUMBER(7, 2)` 示例：**

| 输入值 | 存储结果 | 说明 |
|--------|---------|------|
| 12345678 | 报错 | 超过总长度 7 |
| 12345 | 12345.00 | 整数部分 5 位 + 小数 2 位 = 7 |
| 12345.678 | 12345.68 | 四舍五入保留 2 位小数 |

### 字符型

| 属性 | 说明 |
|------|------|
| `CHAR(n)` | 固定长度字符串，不足 n 位则右侧空格填充，最长 2000 |
| `VARCHAR(n)` | 可变长度字符串，按实际长度存储，最长 2000 |
| `VARCHAR2(n)` | Oracle 专用可变长度字符串，最长 4000 |

### 日期型

| 属性 | 说明 |
|------|------|
| `DATE` | 包含世纪、年月日、时分秒 |
| `TIMESTAMP` | 时间戳类型，比 DATE 多出毫秒部分 |

---

## 附录二：执行顺序速记

### DQL 执行顺序

```
FROM → JOIN → WHERE → GROUP BY → HAVING → SELECT → ORDER BY
 数据源   连接    行过滤     分组       分组后过滤   展示      排序
```

### 条件优先级

```
AND（高） → OR（低）
使用 () 可改变优先级
```

### 空值（NULL）速记

- NULL 不参与比较 → 判断用 `IS NULL` / `IS NOT NULL`
- NULL 不参与计算 → 结果仍为 NULL → 用 `NVL` 处理
- NULL 排序最大 → 用 `NULLS FIRST` / `NULLS LAST` 控制
