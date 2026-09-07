---
title: Oracle PL/SQL 判断与循环 + 游标 + 函数 练习题集
date: 2026-09-03
tags:
  - Oracle
  - PL/SQL
  - 练习题
  - 判断与循环
  - 游标
  - 函数
  - SQL笔记
---
 
# Oracle PL/SQL 练习题集

> **日期**：2026-09-03
> **标签**：`Oracle` `PL/SQL` `练习题` `判断与循环` `游标` `函数`

---

# 第一部分：判断与循环 15道 + 难题10道

## 约束

- ✅ 允许：`IF`判断、`FOR i IN x..y`循环、变量、`%TYPE`、赋值、`DBMS_OUTPUT.PUT_LINE`
- ❌ 禁止：游标、异常处理(`EXCEPTION`)、触发器、`WHILE`循环

---

## 简单（7道）

### 1.【简单】输出1到10

使用`FOR i IN 1..10`，输出1到10每一个数字。

```plsql
begin
  for i in 1..10 loop
    dbms_output.put_line(i);
  end loop;
end;
```

---

### 2.【简单】判断奇偶数

定义变量`v_num := 26`，用IF判断数字是奇数还是偶数，输出结果。

```plsql
declare
  v_num number := 26;
begin
  if mod(v_num, 2) = 0 then
    dbms_output.put_line(v_num || ' 是偶数');
  else
    dbms_output.put_line(v_num || ' 是奇数');
  end if;
end;
```

---

### 3.【简单】输出4-13之间所有整数

使用`FOR i IN 4..13`，循环输出4-13之间所有整数。

```plsql
begin
  for i in 4..13 loop
    dbms_output.put_line(i);
  end loop;
end;
```

---

### 4.【简单】求1~15累加总和

利用`FOR i IN 1..15`，求1~15累加总和，循环结束输出总和。

```plsql
declare
  n number := 0;
begin
  for i in 1..15 loop
    n := n + i;
  end loop;
  dbms_output.put_line('1~15累加总和：' || n);
end;
```

---

### 5.【简单】打印两个数中较大的值

声明`v_a:=22, v_b:=47`，只用IF判断，打印两个数中较大的值。

```plsql
declare
  v_a number := 22;
  v_b number := 47;
  n   number;
begin
  if v_a > v_b then
    n := v_a;
  else
    n := v_b;
  end if;
  dbms_output.put_line('较大值：' || n);
end;
```

---

### 6.【简单】分数合格判断

分数变量`v_score:=73`；分数≥60打印`合格`，否则打印`不合格`。

```plsql
declare
  v_score number := 73;
begin
  if v_score >= 60 then
    dbms_output.put_line('合格');
  else
    dbms_output.put_line('不合格');
  end if;
end;
```

---

### 7.【简单】输出1-20中全部偶数

`FOR i IN 1..20`，输出1-20中全部偶数。

```plsql
begin
  for i in 1..20 loop
    if mod(i, 2) = 0 then
      dbms_output.put_line(i);
    end if;
  end loop;
end;
```

---

## 中等（5道）

### 8.【中等】1-60所有4的倍数的总和

循环`FOR i IN 1..60`，计算1-60所有4的倍数的总和，IF做条件过滤，输出总和。

```plsql
declare
  n number := 0;
begin
  for i in 1..60 loop
    if mod(i, 4) = 0 then
      n := n + i;
    end if;
  end loop;
  dbms_output.put_line('1~60中4的倍数总和：' || n);
end;
```

---

### 9.【中等】年龄多分支判断

变量`v_age`，测试取值：15、28、52；多分支IF：小于18输出少年；18-45输出青年；大于45输出中年。

```plsql
declare
  v_age number := &请输入年龄;
begin
  if v_age < 18 then
    dbms_output.put_line('少年');
  elsif v_age between 18 and 45 then
    dbms_output.put_line('青年');
  elsif v_age > 45 then
    dbms_output.put_line('中年');
  end if;
end;
```

---

### 10.【中等】统计区间内满足条件的数字个数

`FOR i IN 12..40`，统计区间里大于20并且小于32的数字总个数，输出计数。

```plsql
declare
  n number := 0;
begin
  for i in 12..40 loop
    if i between 21 and 31 then
      n := n + 1;
    end if;
  end loop;
  dbms_output.put_line('12~40中大于20小于32的数字个数：' || n);
end;
```

---

### 11.【中等】分别统计奇数总和、偶数总和

一轮`FOR i IN 1..25`，分别统计奇数总和、偶数总和，两个变量保存，循环结束打印两个结果。

```plsql
declare
  n1 number := 0;  -- 偶数总和
  n2 number := 0;  -- 奇数总和
begin
  for i in 1..25 loop
    if mod(i, 2) = 0 then
      n1 := n1 + i;
    else
      n2 := n2 + i;
    end if;
  end loop;
  dbms_output.put_line('偶数总和：' || n1);
  dbms_output.put_line('奇数总和：' || n2);
end;
```

---

### 12.【中等】交换两个变量的值

定义`v_x:=11, v_y:=33`，借助临时变量完成两个变量交换，输出交换前、交换后的值。

```plsql
declare
  v_x number := 11;
  v_y number := 33;
  v_t number;
begin
  dbms_output.put_line('交换前：v_x=' || v_x || ', v_y=' || v_y);
  v_t := v_x;
  v_x := v_y;
  v_y := v_t;
  dbms_output.put_line('交换后：v_x=' || v_x || ', v_y=' || v_y);
end;
```

---

## 中等以上（3道）

### 13.【中等以上】7的倍数与含数字7

`FOR i IN 1..100`遍历：
- i是7的倍数 → 输出`七的倍数`
- i的个位等于7 → 输出`含数字7`
- 其他情况直接输出数字i

```plsql
begin
  for i in 1..100 loop
    if mod(i, 7) = 0 then
      dbms_output.put_line(i || ' → 七的倍数');
    elsif mod(i, 10) = 7 then
      dbms_output.put_line(i || ' → 含数字7');
    else
      dbms_output.put_line(i);
    end if;
  end loop;
end;
```

---

### 14.【中等以上】计算阶乘

设定`v_n:=12`，使用`FOR i IN 1..v_n`计算1~v_n的阶乘，输出阶乘结果。

```plsql
declare
  v_n number := 12;
  n   number := 1;
begin
  for i in 1..v_n loop
    n := n * i;
  end loop;
  dbms_output.put_line(v_n || '! = ' || n);
end;
```

---

### 15.【中等以上】找出三个数中的最大值与最小值

定义三个变量`v1:=44, v2:=19, v3:=67`，禁止使用max/min函数，多层IF嵌套，找出最大值与最小值并输出。

```plsql
declare
  v1 number := 44;
  v2 number := 19;
  v3 number := 67;
  v_max number;
  v_min number;
begin
  -- 找最大值
  if v1 >= v2 and v1 >= v3 then
    v_max := v1;
  elsif v2 >= v1 and v2 >= v3 then
    v_max := v2;
  else
    v_max := v3;
  end if;

  -- 找最小值
  if v1 <= v2 and v1 <= v3 then
    v_min := v1;
  elsif v2 <= v1 and v2 <= v3 then
    v_min := v2;
  else
    v_min := v3;
  end if;

  dbms_output.put_line('最大值：' || v_max);
  dbms_output.put_line('最小值：' || v_min);
end;
```

---

## 难题（10道）

### 难题1：既不是3的倍数也不是5的倍数

使用`FOR i IN 1..50`，输出1-50中**既不是3的倍数，也不是5的倍数**的数字；同时统计这类数字一共有多少个，循环结束输出总数量。

```plsql
declare
  n number := 0;
begin
  for i in 1..50 loop
    if mod(i, 3) <> 0 and mod(i, 5) <> 0 then
      dbms_output.put_line(i);
      n := n + 1;
    end if;
  end loop;
  dbms_output.put_line('符合条件的数字共：' || n || ' 个');
end;
```

---

### 难题2：输出2~30范围内全部质数

给定正整数v_n:=30，FOR循环遍历`2..v_n`。判断每一个数字是否质数（只能被1和自身整除），输出全部质数。不能调用任何数学函数，全部用IF嵌套判断。

```plsql
declare
  v_n number := 30;
  n   number;
begin
  for i in 2..v_n loop
    n := 1;  -- 假设i是质数

    for j in 2..i-1 loop
      if mod(i, j) = 0 then
        n := 0;  -- 不是质数
        exit;    -- 跳出内层循环
      end if;
    end loop;

    if n = 1 then
      dbms_output.put_line(i);
    end if;
  end loop;
end;
```

**逻辑说明**：
1. 外层循环：i从2到30，逐个检查每个数字
2. 先假设i是质数（n=1）
3. 内层循环：拿j从2到i-1逐个去除i
   - 如果i能被j整除 → 标记n=0，跳出内层
4. 内层结束后看标记：n=1说明是质数，打印

**输出结果**：2, 3, 5, 7, 11, 13, 17, 19, 23, 29

> 补充：i=2时，内层循环`j in 2..1`一次都不会执行，n保持1，所以2会被正确输出。

---

### 难题3：除以4余2且除以7余3

设定v_total:=100，FOR i IN 1..v_total。找出1-100里面，满足：除以4余2 **并且** 除以7余3的所有整数，打印每一个符合条件数字，统计个数。

```plsql
declare
  v_total number := 100;
  n       number := 0;
begin
  for i in 1..v_total loop
    if mod(i, 4) = 2 and mod(i, 7) = 3 then
      dbms_output.put_line(i);
      n := n + 1;
    end if;
  end loop;
  dbms_output.put_line('符合条件的数字共：' || n || ' 个');
end;
```

---

### 难题4：阶乘累加和 1!+2!+…+n!

给定v_n:=8，计算`1!+2!+3!+…+v_n!`阶乘累加和。FOR循环实现，需要变量保存阶乘中间结果，再累加总和，最终输出阶乘总和。

```plsql
declare
  v_n number := 8;
  n   number := 1;  -- 阶乘中间结果
  m   number := 0;  -- 阶乘累加总和
begin
  for i in 1..v_n loop
    n := n * i;      -- 计算当前i的阶乘
    m := m + n;      -- 累加到总和
  end loop;
  dbms_output.put_line('1!+2!+...+' || v_n || '! = ' || m);
end;
```

**输出结果**：1!+2!+...+8! = 46233

---

### 难题5：逢3输出A，逢5输出B（禁止ELSIF）

遍历`FOR i IN 1..100`，逢3输出A，逢5输出B，既是3又是5倍数输出AB；其余输出数字。要求不使用ELSIF，只能多层IF嵌套实现。

```plsql
begin
  for i in 1..100 loop
    if mod(i, 3) = 0 then
      if mod(i, 5) = 0 then
        dbms_output.put_line(i || ' → AB');
      else
        dbms_output.put_line(i || ' → A');
      end if;
    else
      if mod(i, 5) = 0 then
        dbms_output.put_line(i || ' → B');
      else
        dbms_output.put_line(i);
      end if;
    end if;
  end loop;
end;
```

---

### 难题6：偶数大于25的总和 + 奇数小于40的总和

定义三个变量a:=28, b:=14, c:=45。FOR i IN a..c；统计区间内：偶数并且大于25的数字的总和，以及奇数并且小于40的数字总和；两个结果分开输出。

```plsql
declare
  a number := 28;
  b number := 14;
  c number := 45;
  n1 number := 0;  -- 偶数且大于25的总和
  n2 number := 0;  -- 奇数且小于40的总和
begin
  for i in a..c loop
    if mod(i, 2) = 0 and i > 25 then
      n1 := n1 + i;
    end if;
    if mod(i, 2) = 1 and i < 40 then
      n2 := n2 + i;
    end if;
  end loop;
  dbms_output.put_line('偶数且大于25的总和：' || n1);
  dbms_output.put_line('奇数且小于40的总和：' || n2);
end;
```

---

### 难题7：数字翻转输出

v_n:=15，FOR i IN 1..v_n。把数字翻转输出：输入1-15，打印15,14,13…1。不允许REVERSE关键字，只能通过算术运算实现倒序输出。

```plsql
declare
  v_n number := 15;
begin
  for i in 1..v_n loop
    dbms_output.put_line(1 + v_n - i);
  end loop;
end;
```

**核心公式**：`1 + v_n - i`，当i=1时输出15，i=15时输出1。

---

### 难题8：个位与十位条件筛选

遍历`FOR i IN 1..100`，统计两类：
1. 个位数字等于2或者6
2. 十位数字等于3或者7

同时满足①②条件的数字打印出来，统计总共有多少个。

```plsql
declare
  n       number := 0;
  v_ge    number;  -- 个位
  v_shi   number;  -- 十位
begin
  for i in 1..100 loop
    v_ge  := mod(i, 10);              -- 个位：取模10
    v_shi := trunc(i / 10);           -- 十位：除以10取整

    if (v_ge = 2 or v_ge = 6) and (v_shi = 3 or v_shi = 7) then
      dbms_output.put_line(i);
      n := n + 1;
    end if;
  end loop;
  dbms_output.put_line('符合条件的数字共：' || n || ' 个');
end;
```

---

### 难题9：求最大公约数

给定v_m:=7, v_n:=28，FOR循环，求v_m与v_n两个数字的最大公约数。不能调用gcd函数，完全依靠IF判断取模结果实现。

```plsql
declare
  v_m number := 7;
  v_n number := 28;
begin
  for i in reverse 1..v_m loop
    if mod(v_n, i) = 0 and mod(v_m, i) = 0 then
      dbms_output.put_line(v_m || ' 和 ' || v_n || ' 的最大公约数是：' || i);
      exit;  -- 找到最大的就退出
    end if;
  end loop;
end;
```

**思路**：从较小数往1倒序遍历，第一个同时能整除两个数的就是最大公约数。

---

### 难题10：九九乘法表上三角

输出九九乘法表上三角部分。只用FOR嵌套循环，IF判断控制打印换行。

```plsql
begin
  for i in 1..9 loop
    for j in 1..9 loop
      if i <= j then
        dbms_output.put(i || '*' || j || '=' || i * j || '   ');
      else
        dbms_output.put('         ');
      end if;
    end loop;
    dbms_output.put_line('');
  end loop;
end;
```

---

# 第二部分：FOR循环接收SELECT语句

## 语法

```plsql
declare
begin
  for 变量 in (select语句) loop
    -- 变量是复合变量，通过 变量.列名 访问
  end loop;
end;
```

**作用**：可以处理多行数据。

---

## 例题：打印emp表中所有员工的姓名和岗位

```plsql
begin
  for i in (select ename, job from emp) loop
    dbms_output.put_line(i.ename || ' - ' || i.job);
  end loop;
end;
```

---

## 练习：打印员工的姓名、岗位、薪资、部门编号、部门名称及部门平均工资

```plsql
begin
  for i in (select ename, job, sal, a.deptno, dname,
                   (select avg(sal) from emp b where b.deptno = a.deptno) v
              from emp a
              left join dept on a.deptno = dept.deptno) loop
    dbms_output.put_line(i.ename || ' ' || i.job || ' ' || i.sal ||
                         ' ' || i.deptno || ' ' || i.dname || ' ' || i.v);
  end loop;
end;
```

---

## 练习：按部门打印员工姓名和岗位

输出样式：
```
10号部门：
张三 经理
李四 董事长
20号部门：
王五 保洁
赵六 保安
```

```plsql
begin
  for i in (select distinct deptno from dept) loop  -- 外层：部门
    dbms_output.put_line(i.deptno || '号部门：');

    for j in (select ename, job from emp where deptno = i.deptno) loop  -- 内层：员工
      dbms_output.put_line('  ' || j.ename || ' ' || j.job);
    end loop;

  end loop;
end;
```

---

# 第三部分：游标（Cursor）

## 概念

**游标**：游动的标记，类似于指针，指向结果集。初始时指向第一行数据。

## 游标的四个步骤

1. **声明游标**：`cursor 游标名 is select语句;`
2. **打开游标**：`open 游标名;`
3. **提取数据**：`fetch 游标名 into 变量...;`（提取数据 + 赋值给变量 + 指针下移）
4. **关闭游标**：`close 游标名;`

## 游标的四个属性

| 属性 | 说明 |
|------|------|
| `游标名%found` | 当游标有值时返回true |
| `游标名%notfound` | 当游标没有值时返回true |
| `游标名%isopen` | 判断游标是否打开 |
| `游标名%rowcount` | 记录游标处理的行数（返回数字） |

---

## 例题：LOOP + 游标

打印输出emp表中每个员工的姓名和岗位。

```plsql
declare
  cursor c1 is select ename, job from emp;  -- 声明游标
  v_ename varchar2(20);
  v_job   emp.job%type;
begin
  open c1;  -- 打开游标

  loop
    fetch c1 into v_ename, v_job;  -- 提取数据
    exit when c1%notfound;         -- 没有数据时退出
    dbms_output.put_line(v_ename || ' - ' || v_job);
  end loop;

  close c1;  -- 关闭游标
end;
```

---

## 例题：WHILE + 游标

```plsql
declare
  cursor c1 is select ename, job from emp;
  v_ename varchar2(20);
  v_job   emp.job%type;
begin
  open c1;
  fetch c1 into v_ename, v_job;  -- 先提取第一行

  while c1%found loop
    dbms_output.put_line(v_ename || ' - ' || v_job);
    fetch c1 into v_ename, v_job;  -- 提取下一行
  end loop;

  close c1;
end;
```

---

## FOR循环配合游标

**特点**：
1. FOR循环会**自动打开和关闭**游标
2. FOR循环会**自动fetch**游标

```plsql
declare
  cursor c1 is select ename, job from emp;
begin
  for i in c1 loop
    dbms_output.put_line(i.ename || ' - ' || i.job);
  end loop;
end;
```

**等价写法**（直接写SELECT子查询）：

```plsql
begin
  for i in (select ename, job from emp) loop
    dbms_output.put_line(i.ename || ' - ' || i.job);
  end loop;
end;
```

---

# 第四部分：自定义函数（Function）

## 语法

```plsql
create [or replace] function 函数名(形参1 类型, 形参2 类型...)
return 返回值类型
is|as
  -- 声明部分
begin
  -- 执行部分
  return 结果;
end;
```

> **注意**：红线上面的类型不能写长度。

---

## 例题：没有参数的函数，返回上个月最后一天

```plsql
create or replace function fu_98
return date
is
  v_d date;
begin
  select add_months(last_day(sysdate), -1)
    into v_d
    from dual;
  return v_d;
end;
```

```sql
select fu_98, sysdate from dual;
```

---

## 例题：传入日期，返回该日期上个月的最后一天

```plsql
create or replace function fu_98(v_date date)
return date
is
  v_d date;
begin
  select add_months(last_day(v_date), -1)
    into v_d
    from dual;
  return v_d;
end;
```

```sql
select fu_98(to_date('1981/3/1', 'yyyy/mm/dd')) from dual;
select fu_98(to_date('2000/3/1', 'yyyy/mm/dd')) from dual;
```

---

## 练习：传入员工编号，返回该员工部门的平均工资

```plsql
create or replace function fu_98(v_empno number)
return number
is
  v_deptno number;
  v_avg    number;
begin
  select deptno into v_deptno from emp where empno = v_empno;
  select avg(sal) into v_avg from emp where deptno = v_deptno;
  return v_avg;
end;
```

```sql
select fu_98(7566) from dual;
```

---

# 知识点速查表

| 知识点 | 语法 | 说明 |
|--------|------|------|
| FOR循环 | `for i in x..y loop ... end loop;` | 自动递增，x>y时不执行 |
| FOR倒序 | `for i in reverse x..y loop` | 从y递减到x |
| IF判断 | `if ... then ... elsif ... else ... end if;` | 多分支判断 |
| MOD函数 | `mod(a, b)` | 求a除以b的余数 |
| FOR+SELECT | `for i in (select ...) loop` | 自动处理多行数据 |
| 游标声明 | `cursor c is select ...;` | 声明游标 |
| 游标属性 | `%found` `%notfound` `%isopen` `%rowcount` | 游标状态判断 |
| 自定义函数 | `create or replace function ... return ... is begin return; end;` | 有且只有一个返回值 |
| 字符串拼接 | `\|\|` | Oracle字符串连接符 |
| 取整 | `trunc(n)` | 截断取整（向下取整） |
| 截取字符 | `substr(str, pos, len)` | 从pos位置截取len个字符 |
| 字符转ASCII | `ascii('A')` | 返回字符的ASCII码 |
| 换行符 | `chr(10)` | 换行符 |
