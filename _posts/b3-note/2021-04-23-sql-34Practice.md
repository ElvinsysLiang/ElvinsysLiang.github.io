---
layout: post
title:  "SQL 34道作业题"
tags:   [SQL]
date:   2021-04-23 09:19:42
categories: [笔记]
---

# 34道练习题

## 1. 取得每个部门的最高薪水的人员名称。  
1）格式：ename,sal,deptno  
2）期望结果  

+-------+---------+--------+  
| ename | sal     | deptno |  
+-------+---------+--------+  
| KING  | 5000.00 |     10 |  
| SCOTT | 3000.00 |     20 |  
| FORD  | 3000.00 |     20 |  
| BLAKE | 2850.00 |     30 |  
+-------+---------+--------+  

3）实现方法  
第一步，按部门分组，求出每个部门的最高薪水  
`select deptno,max(sal) as MAXSAL from emp group by deptno;`  

第二步，与上表进行连接，查出各部门薪水与上表一致的人名  
```
select e.ename,e.sal,t.deptno
from emp e
join (select deptno,max(sal) as MAXSAL from emp group by deptno) t
on e.deptno=t.deptno and e.sal=t.MaxSal
order by t.deptno,e.sal;
```

## 2. 哪些人的薪水在部门的平均薪水之上
1）格式：ename,sal  
2）期望结果  

+-------+---------+  
| ename | sal     |  
+-------+---------+  
| ALLEN | 1600.00 |  
| BLAKE | 2850.00 |  
| JONES | 2975.00 |  
| SCOTT | 3000.00 |  
| FORD  | 3000.00 |  
| KING  | 5000.00 |  
+-------+---------+  

3）实现方法  
第一步，算出每个部门的平均薪水     
`select deptno,avg(sal) as AVGSAL from emp group by deptno; `  

第二步，与上表进行连接，查出各部门比平均薪水高的员工名     
```
select e.ename,e.sal
from emp e
join (select deptno,avg(sal) as AVGSAL from emp group by deptno) t
on e.deptno = t.deptno and e.sal > AVGSAL
order by e.sal;
```

## 3. 取得部门中（所有人的）平均的薪水等级
1）格式：deptno,avg(grade)  
2）期望结果  

+--------+------------+   
| deptno | avg(grade) |   
+--------+------------+   
|     10 |     3.6667 |   
|     20 |     2.8000 |   
|     30 |     2.5000 |   
+--------+------------+   

3）实现方法  
第一步，先算出各部门每个人的薪水等级   
`select e.deptno,s.grade from emp e join salgrade s on e.sal between losal and hisal; `  

第二步，对上表，按照部门分组查询，算出每个部门薪水等级的平均值   
```
select e.deptno,avg(grade)
from emp e
join salgrade s
on e.sal between losal and hisal
group by deptno;
```

## 4. 取得最高薪水（给出三种解决方案）
1）格式：sal  
2）期望结果  

+---------+   
| sal     |   
+---------+   
| 5000.00 |   
+---------+   

3）实现方法1  
用max()   
`select max(sal) as sal from emp; `  


4）实现方法2  
倒序，limit 1   
`select sal from emp order by sal desc limit 1; `  

5）实现方法3  
第一步，表的自连接，查找出比某个数更小的集合，去重   
`select distinct e.sal from emp e join emp t on e.sal<t.sal; `
第二步，找出不在上表里的值，就是最大值  
```
select sal
from emp
where sal not in(select distinct e.sal from emp e join emp t on e.sal<t.sal);
```

## 5. 取得平均薪水最高的部门的部门编号（至少给出两种解决方案）
1）格式：deptno  
2）期望结果  

+--------+  
| deptno |  
+--------+  
|     10 |  
+--------+  

3）实现方法1  
第一步，分组查询每个部门的平均薪水   
`select deptno,avg(sal) as AVGSAL from emp group by deptno;  `  

第二步，用max()求出上表的最高薪水   
`select max(t.AVGSAL) from (select deptno,avg(sal) as AVGSAL from emp group by deptno) t; `

第三步，按照第一步得出部门平均薪水表，用having拿AVGSAL和max()做比较，取得部门编号  
```
select deptno
from emp
group by deptno
having avg(sal)=
(select max(t.AVGSAL) from (select deptno,avg(sal) as AVGSAL from emp group by deptno) t);
```

4）实现方法2  
分组查询每个部门的平均薪水，逆向排序，limit 1   
```
select deptno
from emp
group by deptno
order by avg(sal) desc limit 1;
```

## 6. 取得平均薪水最高的部门的部门名称
1）格式：ename  
2）期望结果  

+------------+  
| dname      |  
+------------+  
| ACCOUNTING |  
+------------+  

3）实现方法  
在上题的基础上，通过平均薪水最高的部门编号，连接部门信息表，取得部门名   
```
select d.dname
from emp e
join dept d
on e.deptno=d.deptno
group by e.deptno
order by avg(sal) desc limit 1;
```  

## 7. 求平均薪水的等级最低的部门的部门名称
1）格式：dname   
2）期望结果  

+-------+  
| dname |  
+-------+  
| SALES |  
+-------+  

3）实现方法  
第一步，求各部门的平均薪水   
`select deptno,avg(sal) from emp group by deptno;`  

第二步，用上表连接薪水等级表，求出平均薪水的等级，再以等级正向排序，limit 1，  
第三步，再联系部门信息表，输出部门名称     
```
select d.dname
from (select deptno,avg(sal) as AVGSAL from emp group by deptno) t
join salgrade s
on t.AVGSAL between s.losal and s.hisal
join dept d
on t.deptno=d.deptno
order by s.grade asc limit 1;
```

## 8. 取得比普通员工(员工代码没有在 mgr 字段上出现的)的最高薪水还要高的领导人姓名
1）格式：ename,sal   
2）期望结果  

+-------+---------+  
| ename | sal     |  
+-------+---------+  
| JONES | 2975.00 |  
| BLAKE | 2850.00 |  
| CLARK | 2450.00 |  
| SCOTT | 3000.00 |  
| KING  | 5000.00 |  
| FORD  | 3000.00 |  
+-------+---------+  

3）实现方法  
第一步，找出一般员工的最高薪水   
`select max(sal) as MAXSAL
from emp
where empno not in(select distinct mgr from emp where mgr is not null); `  

第二步，找出薪水比这个高的，而且是领导岗位的人。（一定要排除NULL）   
```
select e.ename,e.sal  
from emp e
join (
select max(sal) as MAXSAL
from emp
where empno not in(select distinct mgr from emp where mgr is not null)
) t
on e.sal>t.MAXSAL
where e.empno in(select distinct mgr from emp where mgr is not null);
```

## 9. 取得薪水最高的前五名员工
1）格式：ename,sal  
2）期望结果  

+-------+---------+  
| ename | sal     |  
+-------+---------+  
| KING  | 5000.00 |  
| SCOTT | 3000.00 |  
| FORD  | 3000.00 |  
| JONES | 2975.00 |  
| BLAKE | 2850.00 |  
+-------+---------+  

3）实现方法  
薪水逆向排序，limit 5   
`select ename,sal from emp order by sal desc limit 5; `  

## 10. 取得薪水最高的第六到第十名员工
1）格式：ename,sal   
2）期望结果  

+--------+---------+  
| ename  | sal     |  
+--------+---------+  
| CLARK  | 2450.00 |  
| ALLEN  | 1600.00 |  
| TURNER | 1500.00 |  
| MILLER | 1300.00 |  
| MARTIN | 1250.00 |  
+--------+---------+  

3）实现方法  
薪水逆向排序，limit 5,5   
`select ename,sal from emp order by sal desc limit 5,5; `  

## 11. 取得最后入职的 5 名员工
1）格式：ename,hiredate   
2）期望结果  

+--------+------------+  
| ename  | hiredate   |  
+--------+------------+  
| ADAMS  | 1987-05-23 |  
| SCOTT  | 1987-04-19 |  
| MILLER | 1982-01-23 |  
| FORD   | 1981-12-03 |  
| JAMES  | 1981-12-03 |  
+--------+------------+  

3）实现方法  
入职时间逆向排序，limit 5   
`select ename,hiredate from emp order by hiredate desc limit 5; `  

## 12. 取得每个薪水等级有多少员工
1）格式：grade,count   
2）期望结果  

+-------+--------------+  
| grade | count(grade) |  
+-------+--------------+  
|     1 |            3 |  
|     2 |            3 |  
|     3 |            2 |  
|     4 |            5 |  
|     5 |            1 |  
+-------+--------------+  

3）实现方法  
连接薪水等级表，再按照等级分组进行count   
```
select s.grade, count(grade)
from emp e
join salgrade s
on e.sal between s.losal and s.hisal
group by s.grade;
```

## 13. 面试题，参看pdf  

## 14. 列出所有员工及领导的姓名
1）格式：ename，mgrname  
2）期望结果  

+--------+-------------+  
| ename  | mgrname     |  
+--------+-------------+  
| SMITH  | FORD        |  
| ALLEN  | BLAKE       |  
| WARD   | BLAKE       |  
| JONES  | KING        |  
| MARTIN | BLAKE       |  
| BLAKE  | KING        |  
| CLARK  | KING        |  
| SCOTT  | JONES       |  
| KING   | has not mgr |  
| TURNER | BLAKE       |  
| ADAMS  | SCOTT       |  
| JAMES  | BLAKE       |  
| FORD   | JONES       |  
| MILLER | CLARK       |  
+--------+-------------+  

3）实现方法  
左外连接      
```
select e.ename,ifnull(m.ename,'has not mgr') as mgrname
from emp e
left join emp m
on e.mgr=m.empno;
```

## 15. 列出受雇日期早于其直接上级的所有员工的编号,姓名,部门名称
1）格式：empno,ename,dname   
2）期望结果  

+-------+-------+------------+  
| empno | ename | dname      |  
+-------+-------+------------+  
|  7782 | CLARK | ACCOUNTING |  
|  7369 | SMITH | RESEARCH   |  
|  7566 | JONES | RESEARCH   |  
|  7499 | ALLEN | SALES      |  
|  7521 | WARD  | SALES      |  
|  7698 | BLAKE | SALES      |  
+-------+-------+------------+  

3）实现方法  
第一步，自连接，通过领导编号找到领导的受雇日期进行比较，条件是要<,并连接部门信息表找到部门名称   
```
select e.empno,e.ename,d.dname
from emp e
join emp m
on e.mgr = m.empno and e.hiredate < m.hiredate
join dept d
on e.deptno = d.deptno;
```

## 16. 列出部门名称和这些部门的员工信息,同时列出那些没有员工的部门
1）格式：dname,empno,ename,job  
2）期望结果  

+------------+-------+--------+-----------+  
| dname      | empno | ename  | job       |  
+------------+-------+--------+-----------+  
| ACCOUNTING |  7782 | CLARK  | MANAGER   |  
| ACCOUNTING |  7839 | KING   | PRESIDENT |  
| ACCOUNTING |  7934 | MILLER | CLERK     |  
| RESEARCH   |  7369 | SMITH  | CLERK     |  
| RESEARCH   |  7566 | JONES  | MANAGER   |  
| RESEARCH   |  7788 | SCOTT  | ANALYST   |  
| RESEARCH   |  7876 | ADAMS  | CLERK     |  
| RESEARCH   |  7902 | FORD   | ANALYST   |  
| SALES      |  7499 | ALLEN  | SALESMAN  |  
| SALES      |  7521 | WARD   | SALESMAN  |  
| SALES      |  7654 | MARTIN | SALESMAN  |  
| SALES      |  7698 | BLAKE  | MANAGER   |  
| SALES      |  7844 | TURNER | SALESMAN  |  
| SALES      |  7900 | JAMES  | CLERK     |  
| OPERATIONS |  NULL | NULL   | NULL      |  
+------------+-------+--------+-----------+  

3）实现方法  
部门信息表 左外连接 员工信息表     
```
select d.dname,e.empno,e.ename,e.job
from dept d
left join emp e
on d.deptno = e.deptno;
```

## 17. 列出至少有 5 个员工的所有部门
1）格式：dname,count()  
2）期望结果  

+----------+-------+  
| dname    | COUNT |  
+----------+-------+  
| RESEARCH |     5 |  
| SALES    |     6 |  
+----------+-------+  

3）实现方法
第一步，以部门编号分组，统计部门人数  
`select deptno,count(ename) as COUNT from emp group by deptno; `

第二步，一次表连接部门信息表，输出大于等于5人的部门名称   
```
select d.dname,t.COUNT
from (select deptno,count(ename) as COUNT from emp group by deptno) t
join dept d
on t.deptno = d.deptno
where t.COUNT >= 5;
```

## 18. 列出薪金比"SMITH"多的所有员工信息
1）格式：empno,ename,job   
2）期望结果  

+-------+--------+-----------+  
| empno | ename  | job       |  
+-------+--------+-----------+  
|  7499 | ALLEN  | SALESMAN  |  
|  7521 | WARD   | SALESMAN  |  
|  7566 | JONES  | MANAGER   |  
|  7654 | MARTIN | SALESMAN  |  
|  7698 | BLAKE  | MANAGER   |  
|  7782 | CLARK  | MANAGER   |  
|  7788 | SCOTT  | ANALYST   |  
|  7839 | KING   | PRESIDENT |  
|  7844 | TURNER | SALESMAN  |  
|  7876 | ADAMS  | CLERK     |  
|  7900 | JAMES  | CLERK     |  
|  7902 | FORD   | ANALYST   |  
|  7934 | MILLER | CLERK     |  
+-------+--------+-----------+  

3）实现方法1  
把SMITH的薪水先查找出来作为一张表，在用员工信息表跟它做表连接     
```
select e.empno,e.ename,e.job
from emp e
join (select sal from emp where ename='SMITH') s
on e.sal>s.sal;
```

4）实现方法2  
自连接，把select语句嵌套进where语句中  
```
select empno,ename,job
from emp
where sal>(select sal from emp where ename='SMITH');
```

## 19. 列出所有"CLERK"(办事员)的姓名及其部门名称,部门的人数
1）格式：ename,dname,COUNT  
2）期望结果  

+--------+------------+-------+  
| ename  | dname      | COUNT |  
+--------+------------+-------+  
| SMITH  | RESEARCH   |     5 |  
| ADAMS  | RESEARCH   |     5 |  
| JAMES  | SALES      |     6 |  
| MILLER | ACCOUNTING |     3 |  
+--------+------------+-------+  

3）实现方法  
第一步，先求部门人数
`select deptno,count(empno) from emp group by deptno; `  

第二步，查询职位为“CLERK”的人名，连接部门信息表，连接上表   
```
select e.ename,d.dname,COUNT
from emp e
join dept d
on e.deptno = d.deptno
join (select deptno,count(empno) as COUNT from emp group by deptno) t
on e.deptno = t.deptno
where job = 'CLERK';
```

## 20. 列出最低薪金大于 1500 的各种工作及从事此工作的全部雇员人数
1）格式：job,COUNT   
2）期望结果  

+-----------+-------+  
| job       | COUNT |  
+-----------+-------+  
| ANALYST   |     2 |  
| MANAGER   |     3 |  
| PRESIDENT |     1 |  
+-----------+-------+  

3）实现方法  
第一步，找出各个岗位的人数和最低薪水   
`select job,count(empno) as COUNT,min(sal) as MINSAL from emp group by job; `  

第二步，过滤出最低薪水大于1500的岗位，并显示岗位名称和人数     
```
select job,COUNT
from(select job,count(empno) as COUNT,min(sal) as MINSAL from emp group by job) t
where t.MINSAL>1500;
```

## 21. 列出在部门"SALES"<销售部>工作的员工的姓名,假定不知道销售部的部门编号
1）格式：ename  
2）期望结果  

+--------+  
| ename  |  
+--------+  
| ALLEN  |  
| WARD   |  
| MARTIN |  
| BLAKE  |  
| TURNER |  
| JAMES  |  
+--------+  

3）实现方法  
第一步，通过部门名称'SALES'查询部门信息表中的部门编号   
`select deptno from dept where dname='SALES'; `  

第二步，再通过部门编号来获取员工姓名   
```
select ename
from emp
where deptno = (select deptno from dept where dname='SALES');
```

## 22. 列出薪金高于公司平均薪金的所有员工,所在部门,上级领导,雇员的工资等级
1）格式：ename,dname,mgr,grade   
2）期望结果  

+-------+------------+-------+-------+  
| ename | dname      | mgr   | grade |  
+-------+------------+-------+-------+  
| JONES | RESEARCH   | KING  |     4 |  
| BLAKE | SALES      | KING  |     4 |  
| CLARK | ACCOUNTING | KING  |     4 |  
| SCOTT | RESEARCH   | JONES |     4 |  
| KING  | ACCOUNTING | NULL  |     5 |  
| FORD  | RESEARCH   | JONES |     4 |  
+-------+------------+-------+-------+  

3）实现方法  
第一步，算出公司平均薪水   
`select avg(sal) from emp; `  

第二步，连接部门信息表，连接员工信息表（左外连接），连接薪水等级信息表   
```
select e.ename,d.dname,m.ename as mgr,s.grade
from emp e
join dept d
on e.deptno = d.deptno
left join emp m
on e.mgr = m.empno
join salgrade s
on e.sal between s.losal and s.hisal
where e.sal > (select avg(sal) from emp);
```

## 23. 列出与"SCOTT"从事相同工作的所有员工及部门名称
1）格式：ename,dname   
2）期望结果  

+-------+----------+  
| ename | dname    |  
+-------+----------+  
| FORD  | RESEARCH |  
+-------+----------+  

3）实现方法  
第一步，找出“SCOTT”的岗位   
`select job from emp where ename='SCOTT'; `  

第二步，   
```
select e.ename,d.dname
from emp e
join dept d
on e.deptno = d.deptno
where job = (select job from emp where ename='SCOTT') and ename <> 'SCOTT';
```

## 24. 列出薪金等于 部门30中员工的薪金 的其他员工的姓名和薪金
1）格式：ename,sal   
2）期望结果  

Empty set (0.00 sec)  

3）实现方法  
第一步，查询部门30中员工的薪水，做去重处理   
`select distinct sal from emp where deptno=30; `  

第二步，查询不是部门30，薪水属上表结果一样的员工姓名和薪水   
```
select ename,sal
from emp
where sal in(select distinct sal from emp where deptno=30) and deptno <> 30;
```

## 25. 列出薪金高于在部门 30 工作的所有员工的薪金的员工姓名和薪金.部门名称
1）格式：ename,sal,dname   
2）期望结果  

+-------+---------+------------+  
| ename | sal     | dname      |  
+-------+---------+------------+  
| JONES | 2975.00 | RESEARCH   |  
| SCOTT | 3000.00 | RESEARCH   |  
| KING  | 5000.00 | ACCOUNTING |  
| FORD  | 3000.00 | RESEARCH   |  
+-------+---------+------------+  

3）实现方法  
第一步，求出部门30中最高薪水   
`select max(sal) from emp where deptno = 30; `  

第二步，求出所有非部门30的，薪水高于上面结果的员工姓名，薪水和部门名称   
```
select e.ename,e.sal,d.dname
from emp e
join dept d
on e.deptno = d.deptno
where e.sal > (select max(sal) from emp where deptno = 30) and e.deptno <> 30;
```

## 26. 列出在每个部门工作的员工数量,平均工资和平均服务期限
1）格式：dname,COUNT,AVGSAL,AVGSEVTIME   
2）期望结果  

+------------+-------+-------------+------------+  
| dname      | COUNT | AVGSAL      | AVGSEVTIME |  
+------------+-------+-------------+------------+  
| ACCOUNTING |     3 | 2916.666667 |    39.0000 |  
| RESEARCH   |     5 | 2175.000000 |    37.2000 |  
| SALES      |     6 | 1566.666667 |    39.3333 |  
| OPERATIONS |     0 |    0.000000 |     0.0000 |  
+------------+-------+-------------+------------+  

3）实现方法  
按照部门分组，连接部门信息表，算出人数，平均工资和平均服务期限,员工可能没工资，部门可能没人   
```
select
    d.dname,
    count(e.empno) as COUNT,
    ifnull(avg(e.sal),0) as AVGSAL,
    ifnull(avg(timestampdiff(YEAR, hiredate, now())), 0) as AVGSEVTIME
from emp e
right join dept d
on e.deptno = d.deptno
group by d.deptno;

```

## 27. 列出所有员工的姓名、部门名称和工资
1）格式：ename,dname,sal  
2）期望结果  

+--------+------------+---------+  
| ename  | dname      | sal     |  
+--------+------------+---------+  
| CLARK  | ACCOUNTING | 2450.00 |  
| KING   | ACCOUNTING | 5000.00 |  
| MILLER | ACCOUNTING | 1300.00 |  
| SMITH  | RESEARCH   |  800.00 |  
| JONES  | RESEARCH   | 2975.00 |  
| SCOTT  | RESEARCH   | 3000.00 |  
| ADAMS  | RESEARCH   | 1100.00 |  
| FORD   | RESEARCH   | 3000.00 |  
| ALLEN  | SALES      | 1600.00 |  
| WARD   | SALES      | 1250.00 |  
| MARTIN | SALES      | 1250.00 |  
| BLAKE  | SALES      | 2850.00 |  
| TURNER | SALES      | 1500.00 |  
| JAMES  | SALES      |  950.00 |  
+--------+------------+---------+  

3）实现方法     
```
select e.ename,d.dname,e.sal
from emp e
join dept d
on e.deptno = d.deptno;
```

28. 列出所有部门的详细信息和人数
1）格式：d.*,COUNT  
2）期望结果  

+--------+------------+----------+-------+
| DEPTNO | DNAME      | LOC      | COUNT |
+--------+------------+----------+-------+
|     10 | ACCOUNTING | NEW YORK |     3 |
|     20 | RESEARCH   | DALLAS   |     5 |
|     30 | SALES      | CHICAGO  |     6 |
|     40 | OPERATIONS | BOSTON   |     0 |
+--------+------------+----------+-------+

3）实现方法  
```
select d.*,count(e.empno) as COUNT
from dept d
left join emp e
on d.deptno = e.deptno
group by d.deptno;
```

## 29. 列出各种工作的最低工资的雇员姓名既工资
1）格式：empno,ename,job,sal  
2）期望结果  

+-------+--------+-----------+---------+  
| empno | ename  | job       | sal     |  
+-------+--------+-----------+---------+  
|  7369 | SMITH  | CLERK     |  800.00 |  
|  7521 | WARD   | SALESMAN  | 1250.00 |  
|  7654 | MARTIN | SALESMAN  | 1250.00 |  
|  7782 | CLARK  | MANAGER   | 2450.00 |  
|  7788 | SCOTT  | ANALYST   | 3000.00 |  
|  7839 | KING   | PRESIDENT | 5000.00 |  
|  7902 | FORD   | ANALYST   | 3000.00 |  
+-------+--------+-----------+---------+  

3）实现方法  
第一步，找出每个岗位的最低工资   
`select job,min(sal) as MINSAL from emp group by job; `  

第二步，在找到每个岗位中领最低工资的员工   
```
select e.empno,e.ename,e.job,e.sal
from emp e
join (select job,min(sal) as MINSAL from emp group by job) t
on e.job = t.job and e.sal = t.MINSAL;
```

## 30. 列出各个部门的mgr的最低薪金
1）格式：deptno,MINSAL   
2）期望结果  

+--------+---------+  
| deptno | MINSAL  |  
+--------+---------+  
|     10 | 2450.00 |  
|     20 | 2975.00 |  
|     30 | 2850.00 |  
+--------+---------+  

3）实现方法  
```
select deptno,min(sal) as MINSAL
from emp
where empno in(select distinct mgr from emp )
group by deptno;
```

## 31. 列出所有员工的年工资,按年薪从低到高排序
1）格式：ename,income  
2）期望结果  

+--------+----------+  
| ename  | income   |  
+--------+----------+  
| SMITH  |  9600.00 |  
| JAMES  | 11400.00 |  
| ADAMS  | 13200.00 |  
| MILLER | 15600.00 |  
| TURNER | 18000.00 |  
| WARD   | 21000.00 |  
| ALLEN  | 22800.00 |  
| CLARK  | 29400.00 |  
| MARTIN | 31800.00 |  
| BLAKE  | 34200.00 |  
| JONES  | 35700.00 |  
| FORD   | 36000.00 |  
| SCOTT  | 36000.00 |  
| KING   | 60000.00 |  
+--------+----------+  

3）实现方法  
第一步，...   
`... `  

第二步，...   
```
select ename,(sal+ifnull(comm,0))*12 as income
from emp
order by income;
```

## 32. 求出员工领导的薪水超过 3000 的员工名称与领导名称
1）格式：ename,mgrname   
2）期望结果  

+-------+---------+  
| ename | mgrname |  
+-------+---------+  
| JONES | KING    |  
| BLAKE | KING    |  
| CLARK | KING    |  
+-------+---------+  

3）实现方法  
第一步，找出，是领导，薪水又超过3000的人   
`select empno from emp where empno in(select distinct mgr from emp) and sal > 3000; `  

第二步，再找出这位领导的手下   
```
select e.ename,m.ename as mgrname
from emp e
join emp m
on e.mgr = m.empno
where e.mgr =
(select empno from emp where empno in(select distinct mgr from emp) and sal > 3000);
```

## 33. 求出部门名称中,带'S'字符的部门,员工的工资合计、部门人数
1）格式：dname,SUMSAL,COUNTEMP  
2）期望结果  

+------------+----------+----------+  
| dname      | SUMSAL   | COUNTEMP |  
+------------+----------+----------+  
| RESEARCH   | 10875.00 |        5 |  
| SALES      |  9400.00 |        6 |  
| OPERATIONS |     0.00 |        0 |  
+------------+----------+----------+  

3）实现方法  
```
select d.dname,ifnull(sum(e.sal),0) as SUMSAL, count(e.empno) as COUNTEMP
from emp e
right join dept d
on e.deptno = d.deptno
where d.dname like '%S%'
group by d.deptno;
```

## 34. 给任职日期超过 30 年的员工加薪 10%
1）格式：ename,hiredate,sal,NEWSAL  
2）期望结果  

+--------+------------+---------+---------+  
| ename  | hiredate   | sal     | NEWSAL  |  
+--------+------------+---------+---------+  
| SMITH  | 1980-12-17 |  800.00 |  880.00 |  
| ALLEN  | 1981-02-20 | 1600.00 | 1760.00 |  
| WARD   | 1981-02-22 | 1250.00 | 1375.00 |  
| JONES  | 1981-04-02 | 2975.00 | 3272.50 |  
| MARTIN | 1981-09-28 | 1250.00 | 1375.00 |  
| BLAKE  | 1981-05-01 | 2850.00 | 3135.00 |  
| CLARK  | 1981-06-09 | 2450.00 | 2695.00 |  
| SCOTT  | 1987-04-19 | 3000.00 | 3300.00 |  
| KING   | 1981-11-17 | 5000.00 | 5500.00 |  
| TURNER | 1981-09-08 | 1500.00 | 1650.00 |  
| ADAMS  | 1987-05-23 | 1100.00 | 1210.00 |  
| JAMES  | 1981-12-03 |  950.00 | 1045.00 |  
| FORD   | 1981-12-03 | 3000.00 | 3300.00 |  
| MILLER | 1982-01-23 | 1300.00 | 1430.00 |  
+--------+------------+---------+---------+  

3）实现方法  
```
select ename,hiredate,sal,sal*1.1 as NEWSAL
from emp
where timestampdiff(YEAR, hiredate, now()) > 30;
```
