---
date: 2026-08-19
aliases:
  - SQL查询语句
tags:
  - SQL
  - 查询语句
  - select
---

# 对表格的增删改查的操作
sql-----------结构化查询语言
1.DQL-------数据查询语言
2.DML-------数据操纵语言
3.DDL--------数据定义语言
4.DCL--------数据控制语言
5.TCL---------事务控制语言

## 1.DQL数据查询语言
(1)简单查询
(2)条件查询
(3)排序查询
(4)分组查询
(5)完整的查询语句

配置了 2 个用户
1.system----用户名----管理员用户----权限高
2.SCOTT----用户名-----普通用户------权限低----初始密码 tiger
scott-oracle 给我们提供学习用户----自带了四张表

| emp      | 员工信息表 |
| -------- | ----- |
| dept     | 部门信息表 |
| salgrade | 工资等级表 |
| bonus    | 奖金表   |
orcl----全局数据库服务名（是一个实例名）
plsqldeveloper------可视化界面工具------链接数据库的
### （1）简单查询
语法
```
select-----------------查询
     * ----------------全部字段/列名
    from---------------来自
     表名;--------------表名
```
> [!notice]+
> 1，；表示这句话结束
> 2.所有的标点符号都是英文状态的
> 3.绿色的代表关键字
> 4．关键字表名列名是不区分大小写的
> 5．数据区分大小写
> 6.关键字左右是要有空格的-—-分开
> 
> 

**熟悉字段

| 字段名      |       |
| -------- | ----- |
| empno    | 员工编号  |
| ename    | 员工姓名  |
| job      | 岗位/职位 |
| mgr      | 领导编号  |
| hiredate | 入职日期  |
| sal      | 工资    |
| comm     | 奖金    |
| deptno   | 部门编号  |
**连接串**：IP：端口号/实例名-----ip:1521/orcl

```
select * from dept;
```

| 字段名    |      |
| ------ | ---- |
| deptno | 部门编号 |
| dname  | 部门名称 |
| loc    | 部门地址 |
select 后是展示的内容
from 后是表名--数据源

**连接符**：将符号两侧的字符 拼接到一起，i拼接到一起，变成一个字符。
```
select 'abc'||'qqq' from dual;
--dual是一张空表为了补全语法用的
--常量 可以通过常量创造字段
```
**别名**---外号（简单，方便）
1.给列取别名   列别名
```
select 列名1 as 别名
      ,列名2   "别名"
      ,列名3    别名--------常用的方法  常见
      ................
    from emp;
```
1.给表取别名   表别名
```
select * |列名 from 表名 别名;
select * from emp e;--表别名 体现不出来
```
> [!summary]
> 1.表别名列别名不是改名字，只适用于当前语句
> 2.别名不建议使用中文
> 3.别名不建议使用数字和特殊符号,如果用 " " 不能省略的
> ```
> select ename "1", job "2", sal "@" from emp
> ```
> 4．一旦取了表别名就不能使用原来的名字
> 
```
select *,ename from emp;
*想和其他字段一起展示的时候*的前面必须加表名
select emp.*,ename from emp;
```
###  (2)条件查询
语法
```
select----*|列名|常量|计算|函数
  from-----表名
    where--过滤条件（过滤行）
```
**过滤条件**
1.比较运算
2.null 值判断
3.模糊査询
4.范围查询
5.包含查询
6.any 和 all
7.条件的连接

**1.比较运算**
>  <  >   =    <>  !=
数值之间的比较
例题
查询薪资大于 1000的员工信息-——是指所有字段*
```
select *
  from emp
    where sal > 1000;---薪资大于 1000
(先执行from，再执行where，最后执行select)
```

```
练习 
1. 查询 奖金大于100的员工信息
select * from emp where comm>100; 
2. 查询10号部门的员工的姓名岗位薪资部门编号入职日期
select ename, job, sal, deptno, hiredate from emp where deptno = 10; 
3. 查询30号部门的部门地址和部门名称
select dname, loc from dept where deptno = 30;
```
字符之间的比较
a=a
a<>b
```
例题查询姓名是 SCOTT 的员工信息
select *
  from emp
   where ename = 'SCOTT';
```
注意
1．SCOTT 是大写，因为数据是要区分大小写

常见的数据类型
1.字符型
2.数值型
3.日期型
```
例题查询入职日期是 1981/1/1 之前入职的员工信息
select *
  from emp
    where hiredate < 1981/1/1;
通过一个函数将字符转换成日期
转换函数
to_date(目标字符，日期格式)
作用：是将目标字符转换成日期类型
select *
  from emp
    where hiredate < to_date('1981/1/1' , 'yyyy/mm/dd');
所有的函数格式要加单引号
```
> [!note]+
> -oracle默认的日期格式
> 日-月-年 比如1-1月-1982
> ```
> select *
>   from emp
>     where hiredate > '1-1月-1982'
> ```

**2.null 值判断**
is null --------是空
is not null ----非空
```
emp 表中奖金为空的
select * from emp where comm is null;
```
l 练习
```
查询emp表中最大的领导信息
select * from emp where mgr is null;
```

---
**3.模糊查询**
like '目标格式'---像目标格式即满足条件
not like '目标格式'---不像目标格式即满足条件

```目标格式
%-----占 0 位或者多位
_------占 1 位
```
例题
> [!note]+
> 查询 姓名首字母是s的员工信息
> ```
> select * from emp where ename like'S%';
> ```
> 1.查找姓名以S开头的员工信息 
> ```
> select * from emp where ename like'S%';
> ```
> 2.查找姓名前边是SM、后边是TH、中间有一位不确定的员工信息 
> ```
> select * from emp where ename like'SM_TH';
> ```
> 3.查找姓名总共有五位的员工信息
> ```
> select * from emp where ename like'_____';
> ```
>  4.查找姓名前边是S、后边是H、中间有三位不确定的员工信息 
>  ```
>  select * from emp where ename like'S___H';
>  ```
>  5.查找姓名前边是SMIT、最后一位不确定的员工信息
>  ```
>  select * from emp where ename like'SMIT_';
>  ```
>  6.查找姓名不以S开头的员工信息 
>  ```
>  select * from emp where ename not like'S%';
>  ```
>  7.查找名字中带有A字母的员工信息 
>  ```
> select * from emp where ename like'%A%';
>  ```
>  8.查找姓名总共有5位且首字母是A的员工信息 
>  ```
> select * from emp where ename like'A____';
>  ```
>  9.查找姓名是以A开头且倒数第二位是M的员工信息
>  ```
> select * from emp where ename like'A%M_';
>  ```

**4.范围查询**
between 值 1 and 值 2---------在值 1 和值 2 之间
> [!note]+
> 1.between  and 是包含边界的
> 2.将小值写在前面
```
例题
1.查询薪资在1000以上的员工 
select ename from emp where sal > 1000;
2.查询薪资在3000以下的员工 
select ename from emp where sal < 3000;
3.查询薪资在1000到3000之间的员工 
select ename from emp where sal between 1000 and 3000;
4.查询入职时间在1980年至1981年5月之间的员工信息
select * from emp where hiredate between to_date('1980', 'yyyy') and to_date('1981/5', 'yyyy/mm');

```
5.包含查询
in(集合)------------在  集合   里面即满足条件
not in(集合)--------不在  集合   里面即满足条件
集合：必须是同一种属性
```
in(1,2,3)  in(100,200) in('a','b')

in(to_date('1981/1/1','yyyy/mm/dd') , to_date('1982/1/1','yyyy/mm/dd'))

1.查询部门编号是10号或20号的员工信息
select  *  
 from  emp 
  where deptno  in(10,20)
2.查询薪资是3000或5000的员工信息
select  * 
 from emp 
  where sal in(3000,5000)
3.查询岗位是SALESMAN或者MANAGER的员工信息
select  * 
 from emp 
  where job in('SALESMAN' ,  'MANAGER')
4.查询岗位既不是SALESMAN也不是MANAGER的员工信息=
select  * 
 from emp 
  where job not in('SALESMAN' ,  'MANAGER')
5.查询入职时间是1980年12月17号或者1981年2月20号的员工信息
select *
  from emp
 where hiredate in (to_date('1980/12/17', 'yyyy/mm/dd'),
                    to_date('1981/2/20', 'yyyy/mm/dd'))
```
**6.any 和 all**
any(集合)-----满足集合中任何一个值即满足条件
all(集合)-------满足集合中所有值即满足条件
```
>any
<all

例题 查询 工资 大于 1000 或者大于3000 的员工信息
select  * 
 from  emp 
  where sal >any(1000,3000)
例题 查询 工资 大于 1000 并且大于3000 的员工信息
select  * 
 from  emp 
  where sal >all(1000,3000)
```
**7.条件的连接**
and  ---并且
or   ----或者 
-----所有的条件就只有 这两种连接方式
例题 查询 10 号部门的经理
```
select  * 
 from emp 
  where deptno=10  and  job='MANAGER'
```
例题 查询 10 号部门的人或者 20 号部门的人
```
select  * 
 from emp 
  where deptno=10 or   deptno=20 
  
select  * 
 from emp 
  where deptno in(10,20)
```
例题 查询 10 号部门的经理 或者 30 号部门的销售
```
select  * 
 from emp 
  where deptno=10  and job='MANAGER'
           or
        deptno=30  and job='SALESMAN' 
```
  
  注意 ： and  优先于 or ,如果加() 先执行()
  
> [!note]+
>  
> 1.查询薪资超过 1000 并且小于 3000 的员工信息（2 种）
> ```
> select  * 
>  from  emp 
>   where  sal between  1000 and 3000 
>   
> select  * 
>  from emp 
>   where deptno>=1000  and  sal <=3000
> ```
> 2.查询部门编号是 10 号或 20 号的员工信息（2 种）
> ```
> select  * 
>  from emp 
>   where deptno in(10,20)
>  
> select  * 
>  from emp 
>   where deptno =10 or  deptno=20
>   
> select  * 
>  from emp 
>   where deptno =any(10,20)
> ```
> 3.查询岗位是销售 SALESMAN，并且奖金超过 400 的员工信息
>```
> select * 
>  from emp 
>   where job='SALESMAN' and  comm>400
>```
> 4.查询 20 号部门的经理
> ```
> select  * from emp where deptno=20 and  job='MANAGER'
> ```
> 5.查询所有 20 号部门的员工或岗位是 MANAGER 的员工信息
> ```
> select  * 
>  from emp 
>   where deptno=20  or  job='MANAGER'
> 
> select  * 
>  from emp 
>   where deptno=20 and job='CLERK'   or  job='MANAGER'
> ```
> 6.查询 10 号部门的部门经理或 20 号部门的分析师 ANALYST
>```
> select * 
>  from emp 
>   where (deptno =10 and   job='MANAGER')
>        or
>         (deptno=20    and job='ANALYST
>```
> 7.查询 10 号部门的员工、30 号部门的经理及所有的分析师 ANALYST
>```
> select * 
>  from emp 
>   where deptno =10  
>         or
>         deptno =30 and job='MANAGER'
>         or
>         job='ANALYST'
```