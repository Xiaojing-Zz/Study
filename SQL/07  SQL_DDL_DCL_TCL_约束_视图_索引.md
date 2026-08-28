---
date: 2026-08-27
tags:
  - DCL
  - DDL
  - TCL
  - SQL
  - 数据库
  - 约束
  - 索引
  - 视图
---

# Oracle SQL DDL_DCL_TCL _约束_视图_索引
---

## 一、DDL — 数据定义语言

### 1.1 修改表（ALTER）(修改的是数据库对象)
#### 1.1.1 修改表名

```sql
ALTER TABLE 表名 RENAME TO 新表名;
```

```sql
ALTER TABLE zhiyun98 RENAME TO zy98;
```

#### 1.1.2 修改字段名

```sql
ALTER TABLE 表名 RENAME COLUMN 列名 TO 新列名;
```

```sql
ALTER TABLE zy98 RENAME COLUMN zno TO ano;
```

#### 1.1.3 添加 / 删除字段

**添加字段：**

```sql
ALTER TABLE 表名 ADD (列名1 字段属性, 列名2 字段属性 ...);
```

```sql
ALTER TABLE zy98 ADD (zno NUMBER, zsr DATE);
```

**删除字段：**

```sql
ALTER TABLE 表名 DROP (列名1, 列名2 ...);
```

```sql
ALTER TABLE zy98 DROP (ano);
ALTER TABLE zy98 DROP (zsr);
```

#### 1.1.4 修改字段属性（MODIFY）

```sql
ALTER TABLE 表名 MODIFY (列名1 新字段属性, 列名2 新字段属性 ...);
```

**修改属性大小：**

- 可以改大，**不能小于现有数据的最大长度**

```sql
ALTER TABLE zy98 MODIFY (zname VARCHAR2(40));  -- 改大 ✓
ALTER TABLE zy98 MODIFY (zname VARCHAR2(2));   -- 小于已有数据会报错 ✗
```

**修改属性类型：**

- **该列必须为空**，才能改为不同类型的属性

```sql
ALTER TABLE zy98 MODIFY (zname NUMBER(30));  -- 列为空时可改
ALTER TABLE zy98 MODIFY (zname CHAR(30));    -- 列为空时可改
```

---

## 二、DCL — 数据控制语言

### 2.1 赋权 `grant` 与收权 `rewoke`

```sql
-- 赋权
GRANT 权限 TO 用户;
GRANT CREATE VIEW TO scott;

-- 收权
REVOKE 权限 FROM 用户;
```

### 2.2 创建用户

```sql
CREATE USER 用户名 IDENTIFIED BY 密码;
CREATE USER 李许铭 IDENTIFIED BY 123456;
```

### 2.3 角色（Role）

> **角色 = 一系列权限的集合**

#### 系统角色 & 自定义角色

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

**示例：**

```sql
CREATE ROLE 保安队长;
GRANT CREATE SESSION TO 保安队长;      -- 登录权限
GRANT SELECT ANY TABLE TO 保安队长;   -- 查看任意表
CREATE USER 李许铭 IDENTIFIED BY 123456;
GRANT 保安队长 TO 李许铭;
SELECT * FROM scott.emp;
DROP ROLE 保安队长;
```

---

## 三、TCL — 事务控制语言

### 3.1 基本命令

| 命令 | 作用 |
|------|------|
| **COMMIT** | 提交事务 |
| **ROLLBACK** | 回滚事务 |

```sql
INSERT INTO emp(empno, ename) VALUES(4321, '刘赫');
COMMIT;
```

### 3.2 事务的定义

> **事务**：为完成某项业务/任务，由一系列可见的 SQL 和不可见的后台进程组成的**逻辑工作单元**

### 3.3 事务四大特性（ACID）

| 特性 | 说明 |
|------|------|
| **一致性** | 事务一旦提交，所有数据保持一致 |
| **原子性** | 事务是一个整体，不能再分割 |
| **隔离性** | 当前事务操作时，其他事务只能查看修改之前的数据 |
| **持久性** | 事务一旦提交，数据永久保存 |

### 3.4 提交方式

- **显式提交**：需要 `COMMIT` 才能生效，如 **DML** 语句
- **隐式提交**：执行成功后 Oracle 自动提交，如 **DDL** 语句

### 3.5 并发问题与锁

| 问题 | 说明 |
|------|------|
| **脏读** | 读到未提交的数据 |
| **幻读** | 两次查询结果行数不一致 |
| **不可重复读** | 两次查询同一行数据不一致 |

| 锁类型 | 说明 |
|--------|------|
| **共享锁** | 多个事务可同时读 |
| **排它锁** | 独占，其他事务不能读写 |

---

## 四、约束（Constraint）

> **约束**：强加在表中的规则或条件，使表更符合实际需求

### 4.1 约束分类（按作用）

| 约束类型     | 关键字           | 作用               | 能否约束空值 |
| -------- | ------------- | ---------------- | :----: |
| **唯一约束** | `UNIQUE`      | 该列不能出现重复的非空数据    | **不能** |
| **主键约束** | `PRIMARY KEY` | 不能重复，不能空值（每表仅一个） | **能**  |
| **外键约束** | `FOREIGN KEY` | 取值来自另一张表的主键/唯一列  | **不能** |
| **检查约束** | `CHECK`       | 自定义约束条件          | **不能** |
| **非空**   | `NOT NULL`    | 该列不能出现空值         | **能**  |
| **默认值**  | `DEFAULT`     | 不插入数据时填入默认值      |   —    |

### 4.2 约束分类（按位置）

- **行级约束**：建表时跟随在字段和属性之后
- **表级约束**：建表时跟在所有字段之后

---

### 4.3 建表后添加约束

#### 唯一约束

```sql
ALTER TABLE zy98 ADD CONSTRAINT un_zy98_zname UNIQUE(zname);
```

#### 主键约束

语法：

``` sql
alter table  表名  add constraint 约束名 约束类型 (列名1,列名2............)
```

```sql
ALTER TABLE zy98 ADD CONSTRAINT pk_zy98_zno PRIMARY KEY(zno);
```

#### 外键约束

语法：

``` sql
alter table  子表  add
 constraint 约束名
  foreign (列名) references 父表(主键列或唯一列)
```

```sql
ALTER TABLE zy98 ADD
  CONSTRAINT fk_zy98_zname
  FOREIGN KEY(zname) REFERENCES emp(ename);
```

#### 检查约束

语法：

```sql
alter table  表名  add constraint 约束名 check (条件1.....................)
```

```sql
ALTER TABLE zy98 ADD
  CONSTRAINT ch_zy98_zno
  CHECK(zno > 1 OR zname = '李许铭');
```

### 4.4 外键删除策略

| 策略 | 效果 |
|------|------|
| **NO ACTION** | 子表有记录时，父表不能删除（默认） |
| **CASCADE** | 删除父表时，子表相关记录一起删除 |
| **SET NULL** | 删除父表时，子表对应字段设为空 |

---

### 4.5 建表同时添加约束

#### 行级约束

语法：

``` sql
-- 主键约束 唯一约束
create table 表名(列名1 字段属性   约束类型  
                ,列名2 字段属性................)
-- 检查约束
create table 表名(列名1 字段属性   check (条件1.........)  
---------------------------------条件只能是列名1 
                 ,列名2 字段属性................)
-- 外键约束
create table 表名(列名1 字段属性   references 父表(列名)  
                 ,列名2 字段属性................)
```

```sql
-- 系统自动命名
CREATE TABLE zy98 (
  zno   NUMBER        PRIMARY KEY,
  zname VARCHAR2(20)  UNIQUE,
  zsr   DATE          CHECK(zsr < TO_DATE('2026/8/27', 'yyyy/mm/dd')),
  zdno  NUMBER        REFERENCES dept(deptno)
);
```

```sql
-- 手动命名
CREATE TABLE zy98 (
  zno   NUMBER        CONSTRAINT pk_98 PRIMARY KEY,
  zname VARCHAR2(20)  CONSTRAINT un_98 UNIQUE,
  ...
);
```

#### 表级约束

语法：

```sql
-- 主键约束  唯一约束
create  table  表名 (列名1 字段属性
                   ,列名2 字段属性  
                   ,列名3 字段属性
                    ........      
                   ,constraint 约束名 约束类型(列名1,列2....)
                   )
-- 检查约束
create  table  表名 (列名1 字段属性

                    ,列名2 字段属性
                    
                    ,列名3 字段属性
                    ........
                    ,constraint 约束名 check(条件1.....)
                    )
-- 外键约束
create  table  表名 (列名1 字段属性
                    ,列名2 字段属性               
                    ,列名3 字段属性
                    ........   
                    ,constraint 约束名 foreign key(列名1) references 父表(列明) 
                    )
```

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

> **注意**：一个字段可以添加多个约束，但不能添加重复作用的约束

---

### 4.6 非空约束（NOT NULL）

**建表后添加：**

```sql
ALTER TABLE zy98 MODIFY zname NOT NULL;
```

**建表时添加（仅行级）：**

语法：

``` sql
create table 表名(列名1 字段属性   not null 
                ,列名2 字段属性................)
```

```sql
CREATE TABLE zy98 (
  zno   NUMBER       NOT NULL,
  zname VARCHAR2(20) NOT NULL
);
```

> 非空约束 **没有表级写法**

---

### 4.7 默认值（DEFAULT）

**作用**  该列没有插入数据的时候 填入默认值

语法：

```sql
alter  table  表名 modify 列名 default 值 ---默认值
```

**建表后添加：**

```sql
ALTER TABLE zy98 MODIFY zname DEFAULT '李许铭';
ALTER TABLE zy98 MODIFY zno  DEFAULT seq_98.NEXTVAL;
ALTER TABLE zy98 MODIFY zsr  DEFAULT SYSDATE;
```

**建表时添加：**

```sql
CREATE TABLE zy98 (
  zno   NUMBER       DEFAULT 1,
  zname VARCHAR2(20) DEFAULT '李许铭'
);
```

---

### 4.8 删除约束

```sql
-- 删除普通约束
ALTER TABLE 表名 DROP CONSTRAINT 约束名;

-- 删除非空
ALTER TABLE 表名 MODIFY 列名 NULL;

-- 删除默认值
ALTER TABLE 表名 MODIFY 列名 DEFAULT NULL;
```

---

## 五、视图（View）

> **视图**：将 SELECT 语句动态保存到数据库中，形成一张 **虚拟表**
> - 可基于一张表、多张表、或视图创建（基表）
> - 本身不包含数据，数据来源于基表
> - 可创建为 **只读视图**

### 5.1 创建视图

```sql
CREATE [OR REPLACE] VIEW 视图名 AS
  SELECT 语句
  [WITH READ ONLY];
```

**示例 — 单表视图：**

```sql
CREATE OR REPLACE VIEW V_98 AS
  SELECT ename, job, sal, deptno
  FROM emp
  WHERE deptno = 20;

SELECT * FROM V_98;
```

**示例 — 多表视图：**

```sql
CREATE OR REPLACE VIEW V_98 AS
  SELECT ename, job, sal, emp.deptno, dname, loc, LOWER(ename) AS lname
  FROM emp
  LEFT JOIN dept ON emp.deptno = dept.deptno;
```

**示例 — 基于视图创建视图：**

```sql
CREATE OR REPLACE VIEW V_981 AS
  SELECT dname, AVG(sal) AS avg_sal
  FROM V_98
  GROUP BY dname;
```

### 5.2 删除视图

```sql
DROP VIEW 视图名;
```

---

## 六、索引（Index）

> **通过建立索引可以提高查询效率，但会降低 DML 语句的速度**
> Oracle 会自动使用和维护索引

### 6.1 伪列

#### ROWID

> 类似身份证，每行数据存入数据库时 Oracle 自动分配 **18 位不重复字符串**，记录物理位置

```sql
SELECT emp.*, ROWID FROM emp;
SELECT * FROM emp WHERE ROWID = 'AAAR3sAAEAAAACTAAD';
```

#### ROWNUM

> 对查询结果从 **1** 开始的连续自然数序号

```sql
SELECT emp.*, ROWNUM FROM emp;
```

**查询前 N 条（必须从 1 开始）：**

```sql
SELECT emp.*, ROWNUM FROM emp WHERE ROWNUM <= 3;
```

**查询工资前两名（子查询 + ROWNUM）：**

```sql
SELECT * FROM (
  SELECT emp.* FROM emp ORDER BY sal DESC NULLS LAST
)
WHERE ROWNUM <= 2;
```

> **注意**：ROWNUM 必须从 1 开始连续，`WHERE ROWNUM >= 3` 或 `WHERE ROWNUM = 3` 查不到数据

---

### 6.2 扫描方式

| 扫描方式 | 说明 |
|----------|------|
| **全盘扫描** | 从第一行检索到最后一行，取出 ROWID，再取整行数据 |
| **索引扫描** | 从大概位置开始检索到最后一个符合条件的数据 |
| **ROWID 扫描** | 直接通过 ROWID 定位 `SELECT * FROM 表 WHERE ROWID = ''` |

---

### 6.3 索引分类

#### 按存储内容

| 类型                | 说明               | 使用场景                     |
| ----------------- | ---------------- | ------------------------ |
| **B-tree 索引**（默认） | 索引列原始数据 + ROWID  | 列基数大（不重复数据多），如姓名、empno   |
| **位图索引**          | 位图 + ROWID       | 列基数小，如性别、部门编号            |
| **反向键索引**         | 原始数据反向存储 + ROWID | 某节点占比过高，如身高              |
| **基于函数的索引**       | 函数处理后的数据 + ROWID | 查询常带函数，如 `LENGTH(ename)` |

#### 按列个数

- **单列索引**：基于一个字段
- **复合索引**：基于多个字段

#### 按唯一性

- **唯一索引**：索引列数据唯一
- **非唯一索引**：索引列数据可重复

> **注意**：创建唯一约束或主键约束时，会自动创建同名唯一索引；位图索引不能创建唯一索引

---

### 6.4 创建索引

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

### 6.5 复合索引 — 最左原则

假设索引列为 `(A, B, C)`：

| WHERE 条件 | 是否命中索引 |
|------------|:------------:|
| `A = AND B =` | **会** |
| `A = AND C =` | **会**（效率降低） |
| `B = AND C =` | 不会 |
| `C =` | 不会 |

---

### 6.6 索引失效情况

1. 连接条件使用 **OR** 时，索引失效
2. **通配符出现在搜索词首位**时索引失效，如 `ename LIKE '%A%'`
3. 必须将 **索引列放在条件前面**（如 `deptno = 10 AND ename = '李许铭'`）

> **索引不是越多越好**

---

### 6.7 索引管理

```sql
-- 删除索引
DROP INDEX 索引名;

-- 禁用索引
ALTER INDEX 索引名 UNUSABLE;

-- 解禁/重建索引
ALTER INDEX 索引名 REBUILD;
```
