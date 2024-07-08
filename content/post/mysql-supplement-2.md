---
title: MySQL外键与查询-补充
lastmod: 2020-11-21T16:43:23+08:00
date: 2020-11-21T11:52:03+08:00
tags:
  - MySQL
categories:
  - MySQL
url: post/mysql-14.html
toc: true
---

### 外键

<!-- more -->

```python
一张员工信息表内有
id name age dep_name dep_desc 等信息

"""
缺陷
1.表的重点不清晰				可以忽略
	到底是员工表还是部门表
2.表中相关字段一直在重复存储		可以忽略
	浪费存储空间
3.表的扩展性极差,牵一发而动全身   不能忽略
"""
解决方式
  将上述一张表拆分成两张表
  id name age
  id dep_name dep_desc

  emp与dep
  # 上述三个缺陷全部解决

"""
带来了一个小问题 表与表之间的数据没有对应关系了
"""

外键字段>>>:部门编号
  其实就是用来标识表与表之间的数据关系
  # 简单的理解为该字段可以让你去到其他表中查找数据
```

### 表之间的关系

#### 一对多

```python
以员工部门表分析
员工表上一个员工是否可以有多个部门: 不可以
部门表上一个部门是否可以有多个员工: 可以

这种表关系就是一对多，外键字段建在多的一方
'''不存在多对一的关系，就是一对多'''

# 创建
# 可以先创建不含外键的基本表，再添加外键字段
create table emp(
  id int primary key auto_increment,
  name varchar(32),
  age int,
  dep_id int,
  foreign key(dep_id) references dep(id)
);

create table dep(
	id int primary key auto_increment,
  dep_name varchar(32),
  dep_desc varchar (254)
);
```

#### 多对多

```python
书籍表与作者表分析
一个书籍信息是否可以对应多个作者: 可以
一个作者是否可以对应多个书籍信息: 可以

这个关系就是多对多
多对多的表关系，需要单独创建第三张表存储(并且第三张表可以不绑定)

"""
create table book(
	id int primary key auto_increment,
	title varchar(32),
	price float(6,2)
);
create table author(
	id int primary key auto_increment,
	name varchar(32),
	age int
);
create table book2author(
	id int primary key auto_increment,
	author_id int,
	book_id int,
	foreign key(author_id) references author(id)
	on update cascade  # 级联更新
  on delete cascade,  # 级联删除
	foreign key(book_id) references book(id)
	on update cascade  # 级联更新
  on delete cascade  # 级联删除
);
"""
```

#### 一对一

```python
作者表与作者详情表分析
一个作者是否可以对应多个作者详情信息: 不可以
一个作者详情信息是否可以对应多个作者: 不可以

关系可能是一对一或者没关系
外键字段建在任何一方都可以，但是推荐建在查询频率较高的表中

create table author(
	id int primary key auto_increment,
	name varchar(32),
	age int,
	author_id int unique,
  foreign key(author_id) references author_detail(id)
	on update cascade  # 级联更新
  on delete cascade  # 级联删除
);
create table author_detail(
	id int primary key auto_increment,
	phone varchar(32),
	address varchar(32)
);
```

### 外键约束

```python
1.创建表的时候，需要先创建被关联的表(没有外键字段的表)
2.插入新数据的时候，应该先确保被关联表中有数据
3.插入新数据的时候，外键字段只能填写被关联表中已存在的数据
4.在修改和删除被关联表中的数据时，无法直接操作，需要数据之间自动更新需要添加额外的配置
  on update cascade  # 级联更新
  on delete cascade  # 级联删除

由于外键有实质性的诸多约束，当表特别多的时候，外键的增加反而会增加耦合度
因此有些时候并不会使用外键创建表关系，而是通过SQL语句层建立逻辑意义上的表关系
例如操作部门时，先更新部门表，再更新员工表
```

### 表的操作

```python
# 修改表名
alter table 表名 rename  新表名;

# 增加表字段
alter table 表名 add 字段名 数据类型 约束条件;
alter table 表名 add 字段名 数据类型 约束条件 first;
alter table 表名 add 字段名 数据类型 约束条件 after 字段名;

# 删除字段
alter table 表名 drop 字段名;

# 修改字段  modify只能改字段数据类型约束，不能改字段名
alter table 表名 modify 字段名 数据类型 约束条件;
alter table 表名 change 旧字段名 新字段名 旧数据类型 约束条件;
```

### 查询补充

```python
# where
  模糊查询:没有明确的筛选条件
    关键字:like
    关键符号:
      %:匹配任意个数任意字符
      _:匹配单个个数任意字符
  show variables like '%mode%se';

	针对null不能用等号，只能用is

# group by
  分组之后默认只能够直接过去到分组的依据 其他数据都不能直接获取
	针对5.6需要自己设置sql_mode
    	set global sql_mode =   'only_full_group_by,STRICT_TRANS_TABLES,PAD_CHAR_TO_FULL_LENGTH';

# having
  where与having都是筛选功能 但是有区别
    where在分组之前对数据进行筛选
    having在分组之后对数据进行筛选

# 关键字之regexp正则
  select * from emp where name regexp '^j.*(n|y)$';
```

### 练习

```python
# 数据准备
create table emp(
  id int primary key auto_increment,
  name varchar(20) not null,
  sex enum('male','female') not null default 'male', #大部分是男的
  age int(3) unsigned not null default 28,
  hire_date date not null,
  post varchar(50),
  post_comment varchar(100),
  salary double(15,2),
  office int, #一个部门一个屋子
  depart_id int
);

#插入记录
#三个部门：教学，销售，运营
insert into emp(name,sex,age,hire_date,post,salary,office,depart_id) values
('jason','male',18,'20170301','张江第一帅形象代言',7300.33,401,1), #以下是教学部
('tom','male',78,'20150302','teacher',1000000.31,401,1),
('kevin','male',81,'20130305','teacher',8300,401,1),
('tony','male',73,'20140701','teacher',3500,401,1),
('owen','male',28,'20121101','teacher',2100,401,1),
('jack','female',18,'20110211','teacher',9000,401,1),
('jenny','male',18,'19000301','teacher',30000,401,1),
('sank','male',48,'20101111','teacher',10000,401,1),
('哈哈','female',48,'20150311','sale',3000.13,402,2),#以下是销售部门
('呵呵','female',38,'20101101','sale',2000.35,402,2),
('西西','female',18,'20110312','sale',1000.37,402,2),
('乐乐','female',18,'20160513','sale',3000.29,402,2),
('拉拉','female',28,'20170127','sale',4000.33,402,2),
('僧龙','male',28,'20160311','operation',10000.13,403,3), #以下是运营部门
('程咬金','male',18,'19970312','operation',20000,403,3),
('程咬银','female',18,'20130311','operation',19000,403,3),
('程咬铜','male',18,'20150411','operation',18000,403,3),
('程咬铁','female',18,'20140512','operation',17000,403,3);


1. 查询岗位名以及岗位包含的所有员工名字
select post,group_concat(name) from emp group by post;

2. 查询岗位名以及各岗位内包含的员工个数
select post,count(name) from emp group by post;

3. 查询公司内男员工和女员工的个数
select sex,count(*) from emp group by sex;

4. 查询岗位名以及各岗位的平均薪资
select post,avg(salary) from emp group by post;

5. 查询岗位名以及各岗位的最高薪资
select post,max(salary) from emp group by post;

6. 查询岗位名以及各岗位的最低薪资
select post,min(salary) from emp group by post;

7. 查询男员工与男员工的平均薪资，女员工与女员工的平均薪资
select sex,avg(salary) from emp group by sex;
```
