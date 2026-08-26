---
date: 2026-08-20
tags:
  - 查询语句
  - select
  - SQL
  - 数据库
---

# Oracle SQL 基础查询语法笔记

---

## 一、基础查询语法

### 1.1 完整查询语句结构

```sql
SELECT 列名 | * | 常量 | 计算 | 函数    -- 展示内容
  FROM 表名                              -- 数据源
 WHERE 过滤条件                           -- 过滤行
 GROUP BY 分组内容                        -- 分组
 HAVING 分组后过滤条件                    -- 分组后过滤
 ORDER BY 排序字段 ASC|DESC;             -- 排序（永远最后执行）
```

**执行顺序：** `FROM` → `WHERE` → `GROUP BY` → `HAVING` → `SELECT` → `ORDER BY`

---

## 二、排序查询 ORDER BY

### 2.1 基本语法

```sql
SELECT 列名
  FROM 表名
 WHERE 过滤条件
 ORDER BY 排序内容 ASC | DESC;
```

| 关键字 | 含义 | 说明 |
|--------|------|------|
| `ASC` | 升序 | **默认排序方式**，可省略 |
| `DESC` | 降序 | 必须显式指定 |

---

### 2.2 注意事项

- **空值最大**：排序时 NULL 值被视为最大值
- `NULLS FIRST`：强制空值排在最前
- `NULLS LAST`：强制空值排在最后
- `ORDER BY` 后可以跟**多个列**，先按第一列排序，再按第二列排序...
- `ORDER BY` 后面**可以使用别名**
- `WHERE` 后面**不能使用别名**

---

### 2.3 基础示例

#### 示例1：按工资升序排列

```sql
-- ASC 可以省略，默认就是升序
SELECT * FROM emp ORDER BY sal ASC;

-- 等价于
SELECT * FROM emp ORDER BY sal;

-- 降序排列
SELECT * FROM emp ORDER BY sal DESC;
```

#### 示例2：处理空值

```sql
-- 降序排列，空值排最后
SELECT * FROM emp ORDER BY sal DESC NULLS LAST;

-- 升序排列，空值排最前
SELECT * FROM emp ORDER BY sal ASC NULLS FIRST;
```

#### 示例3：多列排序

```sql
-- 先按部门升序，再按工资降序
SELECT * FROM emp ORDER BY deptno ASC, sal DESC;
```

#### 示例4：WHERE 与 ORDER BY 配合

```sql
-- 查询工资>1000的员工，按部门升序、工资降序
SELECT *                          -- 3️⃣ 结果展示
  FROM emp                        -- 1️⃣ 数据源
 WHERE sal > 1000                 -- 2️⃣ 行过滤
 ORDER BY deptno ASC, sal DESC;   -- 4️⃣ 排序（永远最后）
```

#### 示例5：别名使用规则

```sql
-- ❌ WHERE 后不能用别名
SELECT ename e, sal s
  FROM emp
 WHERE s > 1000;          -- 错误！

-- ✅ ORDER BY 后可以用别名
SELECT ename e, sal s
  FROM emp
 ORDER BY s DESC;          -- 正确！
```

---

## 三、分组查询 GROUP BY

### 3.1 基本概念

- **定义**：按照一定规则分组，统一分析各组的情况，**每一组返回一个值**
- **分组函数**（聚合函数/聚组函数）：对一组数据进行统计计算

---

### 3.2 聚合函数

| 函数 | 作用 | 说明 |
|------|------|------|
| `SUM(列名)` | 求和 | 忽略 NULL 值 |
| `AVG(列名)` | 求平均 | 忽略 NULL 值 |
| `MIN(列名)` | 最小值 | 忽略 NULL 值 |
| `MAX(列名)` | 最大值 | 忽略 NULL 值 |
| `COUNT(列名)` | 计数 | 统计该列**非空值**的个数 |
| `COUNT(*)` | 计数 | 统计该组的**总行数** |
| `COUNT(1)` | 计数 | 统计该组的**总行数**（效率同 `COUNT(*)`） |

**重要提示：** 空值不参与聚合函数的统计！

---

### 3.3 分组查询语法

```sql
SELECT 分组字段, 聚合函数
  FROM 表名
 WHERE 分组前的过滤条件        -- 不能使用聚合函数
 GROUP BY 分组内容
 HAVING 分组后的过滤条件       -- 可以使用聚合函数
 ORDER BY 排序字段;
```

---

### 3.4 关键规则

#### 规则1：SELECT 与 GROUP BY 的关系

```sql
-- ❌ 错误：SELECT 后面只能展示 GROUP BY 后面有的字段（或聚合函数）
SELECT ename, deptno, AVG(sal)
  FROM emp
 GROUP BY deptno;

-- ✅ 正确
SELECT deptno, AVG(sal)
  FROM emp
 GROUP BY deptno;
```

#### 规则2：WHERE 与 HAVING 的区别

| 特性 | WHERE | HAVING |
|------|-------|--------|
| **过滤时机** | 分组前 | 分组后 |
| **使用聚合函数** | ❌ 不能 | ✅ 可以 |
| **执行顺序** | 先执行 | 后执行 |

#### 规则3：GROUP BY 多字段

```sql
-- 先按第一列分组，每组再按第二列分...
SELECT deptno, job, AVG(sal)
  FROM emp
 GROUP BY deptno, job;
```

---

### 3.5 分组查询示例

#### 示例1：基本聚合

```sql
-- 所有员工的薪资总和
SELECT SUM(sal) FROM emp;           -- 19688

-- 多个聚合函数
SELECT SUM(sal)/12, AVG(sal), MIN(sal), MAX(sal) FROM emp;
```

#### 示例2：COUNT 函数对比

```sql
SELECT COUNT(comm),    -- 统计该列非空值的个数
       COUNT(*),       -- 统计总行数
       COUNT(1)        -- 统计总行数（同 COUNT(*)）
  FROM emp;
```

#### 示例3：单字段分组

```sql
-- 每个部门的平均工资
SELECT deptno, AVG(sal)
  FROM emp
 GROUP BY deptno;
```

#### 示例4：多字段分组

```sql
-- 每个部门的每个岗位的平均工资
SELECT deptno, job, AVG(sal)
  FROM emp
 GROUP BY deptno, job;
```

#### 示例5：WHERE + GROUP BY + HAVING

```sql
-- 查询工资>1000的员工，每个部门每个岗位的平均工资>2000
SELECT deptno, job, AVG(sal)
  FROM emp
 WHERE sal > 1000                  -- 分组前过滤（不能用聚合函数）
 GROUP BY deptno, job
 HAVING AVG(sal) > 2000;          -- 分组后过滤（可以用聚合函数）
```

#### 示例6：完整分组查询

```sql
-- 查询工资>1000的员工，每个部门每个岗位的平均工资>2000，按平均工资降序
SELECT deptno, job, AVG(sal)
  FROM emp
 WHERE sal > 1000
 GROUP BY deptno, job
 HAVING AVG(sal) > 2000
 ORDER BY AVG(sal) DESC;
```

---

## 四、综合练习题

### 4.1 基础查询练习

#### 题1：查询所有部门情况

```sql
SELECT * FROM dept;
```

#### 题2：查询部门号、部门名称

```sql
SELECT dname, deptno FROM dept;
```

#### 题3：查询10号部门的员工姓名和工资

```sql
SELECT ename, sal FROM emp WHERE deptno = 10;
```

#### 题4：查询CLERK或MANAGER的员工

```sql
-- 方法1：IN
SELECT ename, sal FROM emp WHERE job IN ('CLERK', 'MANAGER');

-- 方法2：OR
SELECT ename, sal FROM emp WHERE job = 'CLERK' OR job = 'MANAGER';
```

#### 题5：查询部门号在10-30的员工

```sql
SELECT ename, deptno, sal, job
  FROM emp
 WHERE deptno BETWEEN 10 AND 30;
```

#### 题6：查询姓名以J开头的员工

```sql
SELECT ename, sal, job FROM emp WHERE ename LIKE 'J%';
```

#### 题7：工资<2000按工资降序

```sql
SELECT ename, job, sal
  FROM emp
 WHERE sal < 2000
 ORDER BY sal DESC;
```

#### 题8：查询CLERK的所有信息

```sql
SELECT ename, sal, deptno FROM emp WHERE job = 'CLERK';
```

#### 题12：工资1000-3000的员工所在部门的全部信息

```sql
SELECT * FROM emp WHERE sal BETWEEN 1000 AND 3000;
```

#### 题15：选择部门30的员工

```sql
SELECT * FROM emp WHERE deptno = 30;
```

#### 题16：所有CLERK的姓名、编号、部门

```sql
SELECT ename, empno, deptno FROM emp WHERE job = 'CLERK';
```

#### 题17：佣金高于薪金的员工

```sql
SELECT * FROM emp WHERE comm > sal;
```

#### 题18：佣金高于薪金60%的员工

```sql
SELECT * FROM emp WHERE comm > sal * 0.6;
```

#### 题19：部门10的经理和部门20的办事员

```sql
SELECT * FROM emp
 WHERE (deptno = 10 AND job = 'MANAGER')
    OR (deptno = 20 AND job = 'CLERK');
```

#### 题20：收取佣金的员工的工作

```sql
SELECT * FROM emp WHERE comm IS NOT NULL;
```

#### 题21：不收取佣金或佣金<100的员工

```sql
SELECT * FROM emp WHERE comm IS NULL OR comm < 100;
```

#### 题22：不带'R'的员工姓名

```sql
SELECT * FROM emp WHERE ename NOT LIKE '%R%';
```

#### 题23：按姓名排序

```sql
SELECT * FROM emp ORDER BY ename;
```

#### 题24：按服务年限排序（老员工在前）

```sql
SELECT * FROM emp ORDER BY hiredate ASC;
```

#### 题25：按工作降序、工资升序

```sql
SELECT ename, job, sal
  FROM emp
 ORDER BY job DESC, sal ASC;
```

#### 题26：30天制日薪

```sql
SELECT ename, sal, sal/30 FROM emp;
```

#### 题27：年薪排序

```sql
SELECT ename, deptno, (sal + comm) * 12 年薪
  FROM emp
 ORDER BY 年薪 ASC;
```

#### 题32：各部门的详细信息和部门人数

```sql
SELECT deptno, COUNT(empno)
  FROM emp
 GROUP BY deptno;
```

---

### 4.2 排序查询练习

#### 题2：按部门升序、薪资降序

```sql
SELECT empno, ename, sal, deptno
  FROM emp
 ORDER BY deptno ASC, sal DESC;
```

#### 题3：除20号部门外，多字段排序

```sql
-- 方法1：NOT IN
SELECT empno, ename, sal, deptno
  FROM emp
 WHERE deptno NOT IN (20)
 ORDER BY deptno ASC, sal ASC, empno DESC;

-- 方法2：NOT LIKE
SELECT empno, ename, sal, deptno
  FROM emp
 WHERE deptno NOT LIKE '20'
 ORDER BY deptno ASC, sal ASC, empno DESC;
```

#### 题4：薪资佣金合计排序

```sql
-- 方法1：使用表达式排序
SELECT ename, sal, comm, sal + comm 总计
  FROM emp
 ORDER BY sal + comm ASC;

-- 方法2：使用别名排序
SELECT ename, sal, comm, sal + comm 总计
  FROM emp
 ORDER BY 总计 ASC;
```

#### 题8：按资历排序（老员工在前）

```sql
SELECT ename, hiredate
  FROM emp
 ORDER BY hiredate ASC;
```

---

### 4.3 分组查询练习

#### 题1：各部门平均工资

```sql
SELECT AVG(sal) FROM emp GROUP BY job;
```

#### 题2：各工种最大工资

```sql
SELECT MAX(sal) FROM emp GROUP BY job;
```

#### 题3：各工种总工资

```sql
SELECT SUM(sal) FROM emp GROUP BY job;
```

#### 题4：每个部门人数

```sql
SELECT deptno, COUNT(empno) FROM emp GROUP BY deptno;
```

#### 题5：各部门最小工资（>=1500）

```sql
SELECT deptno, MIN(sal)
  FROM emp
 GROUP BY deptno
HAVING MIN(sal) >= 1500;
```

#### 题6：各工种统计信息并排序

```sql
SELECT job, MIN(sal), MAX(sal), SUM(sal), AVG(sal), COUNT(empno)
  FROM emp
 GROUP BY job
 ORDER BY SUM(sal) ASC, AVG(sal) ASC;
```

#### 题7：工资>=2000，平均工资>=2500

```sql
SELECT job, AVG(sal)
  FROM emp
 WHERE sal >= 2000
 GROUP BY job
HAVING AVG(sal) >= 2500
 ORDER BY AVG(sal) DESC;
```

#### 题8：无奖金员工，平均工资1000-2000，非10号部门

```sql
SELECT job, AVG(sal)
  FROM emp
 WHERE comm IS NULL AND deptno <> 10
 GROUP BY job
HAVING AVG(sal) BETWEEN 1000 AND 2000;
```

#### 综合题1：部门平均工资>=2000，按平均工资排名

```sql
SELECT deptno, AVG(sal)
  FROM emp
 WHERE sal >= 2000
 GROUP BY deptno
 ORDER BY AVG(sal) ASC;
```

#### 综合题2：各部门工资总和排序

```sql
SELECT deptno, SUM(sal)
  FROM emp
 GROUP BY deptno
 ORDER BY SUM(sal);
```

#### 综合题4：各部门最大最小工资

```sql
SELECT deptno, MIN(sal), MAX(sal)
  FROM emp
 GROUP BY deptno;
```

#### 综合题5：各部门员工数量

```sql
SELECT deptno, COUNT(empno)
  FROM emp
 GROUP BY deptno;
```

#### 综合题6：员工数量>5的部门

```sql
SELECT deptno, COUNT(empno)
  FROM emp
 GROUP BY deptno
HAVING COUNT(empno) > 5;
```

#### 综合题7：1981-4-13后入职，平均工资<2000

```sql
SELECT deptno, AVG(sal)
  FROM emp
 WHERE hiredate > TO_DATE('1981/4/14', 'yyyy/mm/dd')
 GROUP BY deptno
HAVING AVG(sal) < 2000;
```

#### 特殊题：统计有佣金和无佣金的员工数

```sql
SELECT COUNT(1),                    -- 总人数
       COUNT(comm),                 -- 有佣金的人数
       COUNT(1) - COUNT(comm)       -- 无佣金的人数
  FROM emp
 WHERE sal BETWEEN 1000 AND 3000;
```

---

## 五、重点总结

### 5.1 执行顺序记忆口诀

```
FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY
 数据源   行过滤    分组      组过滤    展示      排序
```

### 5.2 WHERE 与 HAVING 区别

| 特性 | WHERE | HAVING |
|------|-------|--------|
| 时机 | 分组前过滤 | 分组后过滤 |
| 聚合函数 | ❌ 不能使用 | ✅ 可以使用 |
| 执行顺序 | 先 | 后 |

### 5.3 COUNT 函数对比

| 写法 | 统计内容 | NULL 处理 |
|------|----------|-----------|
| `COUNT(列名)` | 该列非空值个数 | 忽略 NULL |
| `COUNT(*)` | 总行数 | 不忽略 NULL |
| `COUNT(1)` | 总行数 | 不忽略 NULL |

### 5.4 ORDER BY 注意事项

- `ASC` 可省略（默认升序），`DESC` 必须显式写
- 空值最大，用 `NULLS FIRST` / `NULLS LAST` 控制
- 可跟多个列，先按第一列再按第二列
- **可以使用别名**
- `WHERE` 后**不能使用别名**
