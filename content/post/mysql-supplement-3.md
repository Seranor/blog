---
title: 多表查询和pymysql
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - MySQL
categories:
  - MySQL
url: post/mysql-15.html
toc: true
---

### 多表查询

<!-- more -->

```python
先将需要使用到的表拼接成一张大表 之后基于单表查询完成
    inner join   内连接
    left join    左连接
    right join   右连接
    union        全连接


"""
# 涉及到多表查询的时候 字段名称容易冲突 需要使用表名点字段的方式区分
# inner join:只拼接两张表中共有的部分
select * from emp inner join dep on emp.dep_id = dep.id;
# left join:以左表为基准展示所有的内容 没有的NULL填充
select * from emp left join dep on emp.dep_id = dep.id;
# right join:以右表为基准展示所有的内容 没有的NULL填充
select * from emp right join dep on emp.dep_id = dep.id;
# union:左右表所有的数据都在 没有的NULL填充
select * from emp left join dep on emp.dep_id = dep.id
union
select * from emp right join dep on emp.dep_id = dep.id;
```

### 多表查询练习

#### 数据准备

```
wget https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/sql/lx.sql
```

#### 题目

```mysql
1、查询所有的课程的名称以及对应的任课老师姓名
select course.cname,teacher.tname
from teacher
left join course on course.teacher_id=teacher.tid;

2、查询学生表中男女生各有多少人
select gender,count(gender)
from student
group by gender;

3、查询物理成绩等于100的学生的姓名
select student.sname
from  course
join score  on course.cid=score.course_id
join student on student.sid=score.student_id
where course.cname='物理' and score.num=100;

4、查询平均成绩大于八十分的同学的姓名和平均成绩
select student.sname,avg(score.num)
from student
join score on student.sid=score.student_id
group by student.sname
having avg(score.num)>80;


5、查询所有学生的学号，姓名，选课数，总成绩
select
  student.sid,
  student.sname,
  count(course.cid),
  sum(score.num)
from  course
join score  on course.cid=score.course_id
right join student on student.sid=score.student_id
group by student.sname
order by student.sid;

# ONLY_FULL_GROUP_BY 有限制 只选择出现在group by后面的列，或者给列增加聚合函数
# set sql_mode='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION';

6、 查询姓李老师的个数
select count(tid) from teacher where tname like '李%';


7、 查询没有报李平老师课的学生姓名
select student.sname from student where student.sname not in
(select
  distinct(student.sname)
from  course
join score  on course.cid=score.course_id
right join student on student.sid=score.student_id
left join teacher on teacher.tid=course.teacher_id
where teacher.tname='李平老师');

8、 查询物理课程比生物课程高的学生的学号
select t1.student_id
from
(select
student_id,
num
from student
left join score on score.student_id=student.sid
left join course on course.cid=score.course_id
where course.cname='物理') as t1
join
(select
student_id,
num
from student
left join score on score.student_id=student.sid
left join course on course.cid=score.course_id
where  course.cname='生物') as t2
on t1.student_id=t2.student_id
where t1.num > t2.num;


9、 查询没有同时选修物理课程和体育课程的学生姓名
select
  *
from student
left join score on score.student_id=student.sid
left join course on course.cid=score.course_id
where course.cid=2 or course.cid=3;


10、查询挂科超过两门(包括两门)的学生姓名和班级
select student.sname ,count(num)
from score
join student on student.sid=score.student_id
join class on class.cid=student.class_id
where score.num<60
group by student.sname
having count(num)>=2;

11、查询选修了所有课程的学生姓名
select
 student.sname
from student
join score on score.student_id=student.sid
join course on course.cid=score.course_id
group by student.sname
having count(course.cname)=4;


12、查询李平老师教的课程的所有成绩记录
select
  course.cname,
  score.num,
  student.sname
from  course
join score  on course.cid=score.course_id
right join student on student.sid=score.student_id
left join teacher on teacher.tid=course.teacher_id
where teacher.tname='李平老师';


13、查询全部学生都选修了的课程号和课程名

14、查询每门课程被选修的次数

15、查询之选修了一门课程的学生姓名和学号

16、查询所有学生考出的成绩并按从高到低排序（成绩去重）

17、查询平均成绩大于85的学生姓名和平均成绩

18、查询生物成绩不及格的学生姓名和对应生物分数

19、查询在所有选修了李平老师课程的学生中，这些课程(李平老师的课程，不是所有课程)平均成绩最高的学生姓名

20、查询每门课程成绩最好的前两名学生姓名

21、查询不同课程但成绩相同的学号，课程号，成绩

22、查询没学过“叶平”老师课程的学生姓名以及选修的课程名称；

23、查询所有选修了学号为1的同学选修过的一门或者多门课程的同学学号和姓名；

24、任课最多的老师中学生单科成绩最高的学生姓名
```

### Python 操作 MySQL

```python
pip3 install pymsql

import pymsql

# 连接MySQL服务端
conn = pymysql.connect(
    host='127.0.0.1',
    port=3306,
    user='root',
    password='123',
    database='db8_2',
    charset='utf8'
)
# 产生一个游标对象
cursor = conn.cursor(cursor=pymysql.cursors.DictCursor)
# 编写SQL语句
sql = 'select * from teacher'
affect_rows = cursor.execute(sql)
print(affect_rows)
# 获取执行结果
print(cursor.fetchall())
```

### SQL 注入问题

```python
import pymysql


# 连接MySQL服务端
conn = pymysql.connect(
    host='127.0.0.1',
    port=3306,
    user='root',
    password='123',
    database='db8_3',
    charset='utf8',
    autocommit=True  # 针对增 改 删自动二次确认
)
# 产生一个游标对象
cursor = conn.cursor(cursor=pymysql.cursors.DictCursor)
# 编写SQL语句
username = input('username>>>:').strip()
password = input('password>>>:').strip()
sql = "select * from userinfo where name=%s and pwd=%s"
cursor.execute(sql,(username,password))
data = cursor.fetchall()
if data:
    print(data)
    print('登录成功')
else:
    print('用户名或密码错误')


# 1.只需要用户名也可以登录
# 2.不需要用户名和密码也可以登录

"""
SQL注入的原因 是由于特殊符号的组合会产生特殊的效果
    实际生活中 尤其是在注册用户名的时候 会非常明显的提示你很多特殊符号不能用
        原因也是一样的
结论:设计到敏感数据部分 不要自己拼接 交给现成的方法拼接即可
"""

# sql = 'insert into userinfo(name,pwd) values("jason","123"),("kevin","321")'
# res = cursor.execute(sql)

# print(res)

"""
在使用代码进行数据操作的时候 不同操作的级别是不一样的
    针对查无所谓
    针对增 改 删都需要二次确认
        conn.commit()
"""
```
