---
date: 2026-08-19
tags:
  - 数据库
  - select
  - SQL
  - 查询语句
---

# Oracle SQL 基础查询与条件查询笔记

---

## 一、SQL 语言分类

| 缩写 | 全称 | 说明 |
|------|------|------|
| **DQL** | Data Query Language | 数据查询语言 |
| **DML** | Data Manipulation Language | 数据操纵语言 |
| **DDL** | Data Definition Language | 数据定义语言 |
| **DCL** | Data Control Language | 数据控制语言 |
| **TCL** | Transaction Control Language | 事务控制语言 |

---

## 二、Oracle 11 g 基础配置

### 2.1 预置用户

| 用户名 | 角色 | 密码 | 权限 |
|--------|------|------|------|
| `system` | 管理员用户 | 自定义 | 高权限 |
| `SCOTT` | 普通用户 | `tiger` | 低权限（学习用） |

### 2.2 SCOTT 用户自带表

| 表名 | 说明 |
|------|------|
| `emp` | 员工信息表 |
| `dept` | 部门信息表 |
| `salgrade` | 工资等级表 |
| `bonus` | 奖金表 |

### 2.3 数据库连接

- **全局数据库服务名（实例名）**：`orcl`
- **连接串格式**：`IP:端口号/实例名` → `ip:1521/orcl`

### 2.4 常用工具

| 工具 | 说明 |
|------|------|
| PL/SQL Developer | 可视化界面工具，仅支持 Oracle |
| DBeaver | 工作常用工具，支持多种数据库 |

---

## 三、简单查询

### 3.1 基本语法

```sql
select-----------------查询
     * ----------------全部字段/列名
    from---------------来自
     表名;--------------表名
```

### 3.2 注意事项

1. `;` 表示语句结束
2. 所有标点符号使用**英文状态**
3. 绿色显示的代表**关键字**
4. 关键字、表名、列名**不区分大小写**
5. **数据区分大小写**
6. 关键字左右需要**空格**分隔

---

### 3.3 常用表字段说明

#### emp 表（员工信息表）

| 字段名 | 说明 |
|--------|------|
| `empno` | 员工编号 |
| `ename` | 员工姓名 |
| `job` | 岗位/职位 |
| `mgr` | 领导编号 |
| `hiredate` | 入职日期 |
| `sal` | 工资 |
| `comm` | 奖金 |
| `deptno` | 部门编号 |

#### dept 表（部门信息表）

| 字段名 | 说明 |
|--------|------|
| `deptno` | 部门编号 |
| `dname` | 部门名称 |
| `loc` | 部门地址 |

**连接串**：IP：端口号/实例名-----ip:1521/orcl

---

### 3.4 查询示例

#### 示例 1：查询特定字段

```sql
-- 查询员工的姓名、岗位、薪资
SELECT ename, job, sal FROM emp;

-- 字段可以重复展示
SELECT ename, job, sal, sal, sal FROM emp;
```

#### 示例 2：查询所有字段（两种方法）

```sql
-- 方法1：使用 *（效率低，不推荐）
SELECT * FROM emp;

-- 方法2：显式列出所有字段（推荐）
SELECT empno, ename, job, mgr, sal, comm, hiredate, deptno FROM emp;
```

---

## 四、连接符 `||`

### 4.1 定义

将符号两侧的字符拼接成**一个字符**

### 4.2 示例

```sql
-- 拼接常量
SELECT 'abc' || '1223' FROM dual;  -- 结果：abc1223

-- 拼接字段
SELECT ename || job, sal FROM emp;  -- 结果：刘亦菲保洁

-- 多字段拼接
SELECT ename || job || sal FROM emp;

-- 所有字段拼接
SELECT empno || ename || job || mgr || sal || comm || hiredate || deptno FROM emp;
```

> **提示**：`dual` 是一张空表，用于补全语法

---

## 五、别名

### 5.1 列别名

#### 语法

```sql
SELECT 列名1 AS "别名",
       列名2    "别名",
       列名3     刨名          -- 常用写法
  FROM 表名;
```

#### 示例

```sql
SELECT ename AS "姓名",
       job    "岗位",
       sal     工资
  FROM emp;
```

### 5.2 表别名

#### 语法

```sql
SELECT * | 列名 FROM 表名 表别名;
```

#### 示例

```sql
-- 表别名在简单查询中体现不明显
SELECT * FROM emp e;

-- 使用表别名限定字段
SELECT e.ename, e.job, e.sal FROM emp e;
```

### 5.3 别名注意事项

1. 别名**不是改名**，只在当前语句有效
2. 别名**不建议使用中文**
3. 别名**不建议使用数字和特殊符号**，如使用则 `""` 不能省略
4. 一旦取了表别名，**不能再使用原表名**

### 5.4 `*` 与其他字段混用

```sql
-- ❌ 错误：* 与其他字段混用时会报错
SELECT *, ename FROM emp;

-- ✅ 正确：* 前必须加表名归属
SELECT emp.*, ename FROM emp;

-- 使用表别名
SELECT e.*, ename FROM emp e;
```

---

## 六、数值计算

### 6.1 运算符

| 运算符 | 说明 |
|--------|------|
| `+` | 加 |
| `-` | 减 |
| `*` | 乘 |
| `/` | 除 |
| `()` | 括号（优先级最高） |

### 6.2 示例

```sql
-- 计算日薪资（每月30天）
SELECT ename, job, sal, comm, hiredate, sal/30 FROM emp;

-- 计算年薪（工资+奖金）*12
SELECT ename, job, sal, comm, hiredate, (sal + comm) * 12 FROM emp;
```

### 6.3 空值（NULL）注意

**空值不参与计算**，任何数值与空值进行加减乘除，结果仍为空值！

```sql
-- 如果 comm 为 NULL，则 sal + comm 结果为 NULL
SELECT ename, sal, comm, sal + comm FROM emp;
```

---

## 七、条件查询

### 7.1 基本语法

```sql
SELECT * | 列名 | 常量 | 计算 | 函数
  FROM 表名
 WHERE 过滤条件;    -- 过滤的是行
```

---

### 7.2 过滤条件类型

---

#### 类型 1：比较运算

| 运算符 | 说明 |
|--------|------|
| `=` | 等于 |
| `<>` 或 `!=` | 不等于 |
| `<` | 小于 |
| `>` | 大于 |
| `<=` | 小于等于 |
| `>=` | 大于等于 |

##### 数值比较示例

```sql
-- 查询薪资大于1000的员工
SELECT * FROM emp WHERE sal > 1000;

-- 查询薪资小于等于1000的员工
SELECT * FROM emp WHERE sal <= 1000;
```

> **注意**：NULL 值不参与比较

##### 字符比较示例

```sql
-- 查询姓名是 SCOTT 的员工（数据区分大小写）
SELECT * FROM emp WHERE ename = 'SCOTT';

-- ❌ 错误：小写查不到
SELECT * FROM emp WHERE ename = 'scott';

-- ❌ 错误：字符必须加单引号
SELECT * FROM emp WHERE ename = SCOTT;
```

##### 字符比较规则

- **数据区分大小写**：`SCOTT` ≠ `scott`
- **字符值必须加单引号**：`'SCOTT'`

##### 日期比较示例

```sql
-- 查询1982年1月1日之前入职的员工
SELECT * FROM emp WHERE hiredate < TO_DATE('1982/1/1', 'yyyy/mm/dd');

-- Oracle 默认日期格式：日-月-年
SELECT * FROM emp WHERE hiredate > '1-1月-1982';
```

---

#### 类型 2：NULL 值判断

| 语法 | 说明 |
|------|------|
| `IS NULL` | 是空值 |
| `IS NOT NULL` | 非空值 |

##### 示例

```sql
-- 查询有奖金的员工
SELECT * FROM emp WHERE comm IS NOT NULL;

-- 查询没有奖金的员工
SELECT * FROM emp WHERE comm IS NULL;

-- 查询最大的领导（没有上级的员工）
SELECT * FROM emp WHERE mgr IS NULL;
```

---

#### 类型 3：模糊查询 LIKE

| 通配符 | 说明 |
|--------|------|
| `%` | 占 0 位或多位 |
| `_` | 占 1 位 |

| 语法 | 说明 |
|------|------|
| `LIKE '目标格式'` | 匹配目标格式 |
| `NOT LIKE '目标格式'` | 不匹配目标格式 |

##### 示例

```sql
-- 姓名以S开头
SELECT * FROM emp WHERE ename LIKE 'S%';

-- 姓名不以S开头
SELECT * FROM emp WHERE ename NOT LIKE 'S%';

-- 姓名倒数第二位是T
SELECT * FROM emp WHERE ename LIKE '%T_';

-- 姓名以SM开头、TH结尾，中间一位不确定
SELECT * FROM emp WHERE ename LIKE 'SM_TH';

-- 姓名总共5位
SELECT * FROM emp WHERE ename LIKE '_____';

-- 姓名以S开头、H结尾，中间3位不确定
SELECT * FROM emp WHERE ename LIKE 'S___H';

-- 姓名以SMIT开头，最后一位不确定
SELECT * FROM emp WHERE ename LIKE 'SMIT_';

-- 姓名中包含A
SELECT * FROM emp WHERE ename LIKE '%A%';

-- 姓名总共5位且首字母是A
SELECT * FROM emp WHERE ename LIKE 'A____';

-- 姓名以A开头且倒数第二位是M
SELECT * FROM emp WHERE ename LIKE 'A%M_';
```

---

#### 类型 4：范围查询 BETWEEN AND

| 语法 | 说明 |
|------|------|
| `BETWEEN 值1 AND 值2` | 在值 1 和值 2 之间（**包含边界**） |

##### 注意事项

1. **包含边界值**
2. **小值写在前面**

##### 示例

```sql
-- 薪资在1000到3000之间
SELECT * FROM emp WHERE sal BETWEEN 1000 AND 3000;

-- ❌ 错误：大值在前
SELECT * FROM emp WHERE sal BETWEEN 3000 AND 1000;

-- 入职时间在1980年至1981年5月之间
SELECT * FROM emp
 WHERE hiredate BETWEEN TO_DATE('1980/1/1', 'yyyy/mm/dd')
                    AND TO_DATE('1981/5/31', 'yyyy/mm/dd');
```

> **注意**：`TO_DATE('1980', 'yyyy')` 默认填入系统的月和日，建议显式指定完整日期

---

#### 类型 5：包含查询 IN

| 语法 | 说明 |
|------|------|
| `IN(集合)` | 在集合中即满足条件 |
| `NOT IN(集合)` | 不在集合中即满足条件 |

##### 集合要求

- 必须是**同一数据类型**
- 示例：`IN(1,2,3)`、`IN('a','b')`、`IN(TO_DATE(...), TO_DATE(...))`

##### 示例

```sql
-- 查询10号或30号部门的员工
SELECT * FROM emp WHERE deptno IN (10, 30);

-- 查询不是10号或30号部门的员工
SELECT * FROM emp WHERE deptno NOT IN (10, 30);

-- 查询薪资是3000或5000的员工
SELECT * FROM emp WHERE sal IN (3000, 5000);

-- 查询岗位是SALESMAN或MANAGER的员工
SELECT * FROM emp WHERE job IN ('SALESMAN', 'MANAGER');

-- 查询岗位既不是SALESMAN也不是MANAGER的员工
SELECT * FROM emp WHERE job NOT IN ('SALESMAN', 'MANAGER');

-- 查询特定入职日期的员工
SELECT * FROM emp
 WHERE hiredate IN (TO_DATE('1980/12/17', 'yyyy/mm/dd'),
                    TO_DATE('1981/2/20', 'yyyy/mm/dd'));
```

---

#### 类型 6：ANY 和 ALL

| 语法 | 说明 |
|------|------|
| `> ANY(集合)` | 大于集合中**任意一个**值即满足 |
| `< ALL(集合)` | 小于集合中**所有**值才满足 |

##### 示例

```sql
-- 工资大于1000或大于3000（即大于最小值1000）
SELECT * FROM emp WHERE sal > ANY(1000, 3000);

-- 工资大于1000且大于3000（即大于最大值3000）
SELECT * FROM emp WHERE sal > ALL(1000, 3000);
```

---

#### 类型 7：条件连接 AND / OR

| 运算符 | 说明 | 优先级 |
|--------|------|--------|
| `AND` | 并且（同时满足） | **高** |
| `OR` | 或者（满足其一） | 低 |

> **优先级**：`AND` 优先于 `OR`，加 `()` 可改变优先级

##### 示例

```sql
-- 10号部门的经理
SELECT * FROM emp WHERE deptno = 10 AND job = 'MANAGER';

-- 10号或20号部门的员工
SELECT * FROM emp WHERE deptno = 10 OR deptno = 20;

-- 等价写法
SELECT * FROM emp WHERE deptno IN (10, 20);

-- 10号部门的经理 或 30号部门的销售
SELECT * FROM emp
 WHERE deptno = 10 AND job = 'MANAGER'
    OR deptno = 30 AND job = 'SALESMAN';

-- 加括号明确优先级
SELECT * FROM emp
 WHERE (deptno = 10 AND job = 'MANAGER')
    OR (deptno = 20 AND job = 'ANALYST');

-- 10号部门的员工、30号部门的经理、所有ANALYST
SELECT * FROM emp
 WHERE deptno = 10
    OR deptno = 30 AND job = 'MANAGER'
    OR job = 'ANALYST';
```

---

## 八、综合练习

### 8.1 简单查询

#### 题 1：查询员工姓名、岗位、薪资

```sql
SELECT ename, job, sal FROM emp;
```

#### 题 2：查询部门名称和部门地址

```sql
SELECT dname, loc FROM dept;
```

#### 题 3：查询 10 号部门的员工姓名、岗位、薪资、入职日期

```sql
SELECT ename, job, sal, hiredate FROM emp WHERE deptno = 10;
```

#### 题 4：查询 30 号部门的部门地址和部门名称

```sql
SELECT loc, dname FROM dept WHERE deptno = 30;
```

---

### 8.2 比较运算

#### 题 1：查询奖金大于 100 的员工

```sql
SELECT * FROM emp WHERE comm > 100;
```

#### 题 2：查询姓名是 SMITH 的员工

```sql
SELECT * FROM emp WHERE ename = 'SMITH';
```

#### 题 3：查询岗位是销售的员工姓名、岗位、薪资、入职日期

```sql
SELECT ename, job, sal, hiredate FROM emp WHERE job = 'SALESMAN';
```

#### 题 4：查询地址是纽约的部门信息

```sql
SELECT * FROM dept WHERE loc = 'NEW YORK';

-- 查询不是纽约的部门
SELECT * FROM dept WHERE loc <> 'NEW YORK';
```

---

### 8.3 NULL 值判断

#### 题 1：查询有奖金的员工

```sql
SELECT * FROM emp WHERE comm IS NOT NULL;
```

#### 题 2：查询没有奖金的员工

```sql
SELECT * FROM emp WHERE comm IS NULL;
```

#### 题 3：查询最大的领导

```sql
SELECT * FROM emp WHERE mgr IS NULL;
```

---

### 8.4 模糊查询

| 题号 | 需求 | SQL |
|------|------|-----|
| 1 | 姓名以 S 开头 | `SELECT * FROM emp WHERE ename LIKE 'S%'` |
| 2 | SM_TH 模式 | `SELECT * FROM emp WHERE ename LIKE 'SM_TH'` |
| 3 | 姓名总共 5 位 | `SELECT * FROM emp WHERE ename LIKE '_____'` |
| 4 | S 开头 H 结尾中间 3 位 | `SELECT * FROM emp WHERE ename LIKE 'S___H'` |
| 5 | SMIT 开头最后一位不确定 | `SELECT * FROM emp WHERE ename LIKE 'SMIT_'` |
| 6 | 姓名不以 S 开头 | `SELECT * FROM emp WHERE ename NOT LIKE 'S%'` |
| 7 | 姓名包含 A | `SELECT * FROM emp WHERE ename LIKE '%A%'` |
| 8 | 5 位且首字母 A | `SELECT * FROM emp WHERE ename LIKE 'A____'` |
| 9 | A 开头倒数第二位 M | `SELECT * FROM emp WHERE ename LIKE 'A%M_'` |

---

### 8.5 范围查询

#### 题 1：薪资 1000 以上的员工

```sql
SELECT * FROM emp WHERE sal > 1000;
```

#### 题 2：薪资 3000 以下的员工

```sql
SELECT * FROM emp WHERE sal < 3000;
```

#### 题 3：薪资 1000 到 3000 之间

```sql
SELECT * FROM emp WHERE sal BETWEEN 1000 AND 3000;
```

#### 题 4：入职时间 1980 年至 1981 年 5 月

```sql
SELECT * FROM emp
 WHERE hiredate BETWEEN TO_DATE('1980/1/1', 'yyyy/mm/dd')
                    AND TO_DATE('1981/5/31', 'yyyy/mm/dd');
```

---

### 8.6 包含查询

| 题号 | 需求 | SQL |
|------|------|-----|
| 1 | 10 号或 20 号部门 | `SELECT * FROM emp WHERE deptno IN (10, 20)` |
| 2 | 薪资 3000 或 5000 | `SELECT * FROM emp WHERE sal IN (3000, 5000)` |
| 3 | SALESMAN 或 MANAGER | `SELECT * FROM emp WHERE job IN ('SALESMAN', 'MANAGER')` |
| 4 | 既不是 SALESMAN 也不是 MANAGER | `SELECT * FROM emp WHERE job NOT IN ('SALESMAN', 'MANAGER')` |
| 5 | 特定入职日期 | `SELECT * FROM emp WHERE hiredate IN (TO_DATE('1980/12/17', 'yyyy/mm/dd'), TO_DATE('1981/2/20', 'yyyy/mm/dd'))` |

---

### 8.7 条件连接

#### 题 1：薪资超过 1000 且小于 3000（两种写法）

```sql
-- 写法1：BETWEEN AND
SELECT * FROM emp WHERE sal BETWEEN 1000 AND 3000;

-- 写法2：AND
SELECT * FROM emp WHERE sal >= 1000 AND sal <= 3000;
```

#### 题 2：10 号或 20 号部门（三种写法）

```sql
-- 写法1：IN
SELECT * FROM emp WHERE deptno IN (10, 20);

-- 写法2：OR
SELECT * FROM emp WHERE deptno = 10 OR deptno = 20;

-- 写法3：ANY
SELECT * FROM emp WHERE deptno = ANY(10, 20);
```

#### 题 3：销售且奖金超过 400

```sql
SELECT * FROM emp WHERE job = 'SALESMAN' AND comm > 400;
```

#### 题 4：20 号部门的经理

```sql
SELECT * FROM emp WHERE deptno = 20 AND job = 'MANAGER';
```

#### 题 5：20 号部门的员工或 MANAGER

```sql
SELECT * FROM emp WHERE deptno = 20 OR job = 'MANAGER';
```

#### 题 6：10 号部门经理或 20 号部门分析师

```sql
SELECT * FROM emp
 WHERE (deptno = 10 AND job = 'MANAGER')
    OR (deptno = 20 AND job = 'ANALYST');
```

#### 题 7：10 号部门员工、30 号部门经理、所有 ANALYST

```sql
SELECT * FROM emp
 WHERE deptno = 10
    OR deptno = 30 AND job = 'MANAGER'
    OR job = 'ANALYST';
```

---

## 九、核心知识点总结

### 9.1 常见数据类型

| 类型 | 说明 | 示例 |
|------|------|------|
| 字符型 | 字符串 | `'SCOTT'` |
| 数值型 | 数字 | `1000`、`3.14` |
| 日期型 | 日期 | `'1-1 月-1982'` |

### 9.2 查询执行顺序

```
FROM → WHERE → SELECT
数据源   过滤     展示
```

### 9.3 空值（NULL）特性

- NULL 不参与比较运算
- NULL 不参与计算（结果仍为 NULL）
- 判断 NULL 使用 `IS NULL` / `IS NOT NULL`

### 9.4 条件优先级

```
AND（高） → OR（低）
使用 () 可改变优先级
```
