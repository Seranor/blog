---
title: MySQL数据类型-补充
lastmod: 2020-11-21T16:43:23+08:00
date: 2020-11-21T11:52:03+08:00
tags:
  - MySQL
categories:
  - MySQL
url: post/mysql-13.html
toc: true
---

### 存储引擎

<!-- more -->

```python
MyISAM
  mysql5.5之前默认支持存储引擎，不支持事务、行级锁

InnoDB
  mysql5.5之后默认的存储引擎。支持事务，行级锁，更安全

memory
  基于内存存取数据，断电之后数据会丢失

blackhole
  写入其中的数据都会立刻消失 /dev/null

# 创建不同存储引擎的表
create table t1(id int) engine=myisam;
create table t2(id int) engine=innodb;
create table t3(id int) engine=memory;
create table t4(id int) engine=blackhole;

'''
myisam引擎 创建的表有三个文件
.frm  表结构文件
.myd  表数据文件
.myi  表索引文件

innodb引擎 创建的表有两个文件
.frm 表结构文件
.ibd 表数据和表索引

memory引擎
  .frm 表结构文件

blackhole引擎
  .frm 表结构文件
'''
```

### MySQL 基本数据类型

#### 整型

```python
tinyint smallint int bigint

正负号会占一个比特位
所有的int类型默认都需要正负号

create table t(id int(3));
insert into t values(4444444);

"""
在整型中括号内的数字并不是用来限制存储的长度 而是用来控制展示的长度
我们以后在定义整型字段的时候 不需要自己添加数字 使用默认的就可以
"""

create table t14(id int(3) zerofill);
insert into t13 values(4);
# 整型比较的特殊 是唯一个不是用来限制存储长度的类型
```

#### 浮点型

```python
float double decimal

float(255,30)   # 总共255位 小数占30位
double(255,30)  # 总共255位 小数占30位
decimal(65,30)  # 总共65位 小数占30位

create table t5(id float(255,30));
create table t6(id double(255,30));
create table t7(id decimal(65,30));
insert into t5 values(1.1111111111111111111111);
insert into t6 values(1.1111111111111111111111);
insert into t7 values(1.1111111111111111111111);
select * from t5;
select * from t6;
select * from t7;

精确度: float < double < decimal
```

#### 字符类型

```python
char(4)     # 定长类型 最多存4个字符   多了报错少了空格会填充至四个
varchar(3)  # 变长类型 有几个存几个字符 最多存4个字符

create table t6(name char(4));
create table t7(name varchar(4));

# mysql5.6 不会报错 5.7会报错
# 5.7在sql_mode上做了SQL92严格模式
# show variables like '%sql_mode%'
# set global sql_mode = 'strict_trans_tables,pad_char_to_full_length'
insert into t6 values('xxxxx');
insert into t7 values('xxxxx');

对比:
char:
  优势: 整存整取，速度快
  劣势: 浪费存储空间

varchar:
  优势: 节省存储空间
  劣势: 存取数据的速度较char慢

varchar类型会把字段信息也存入，取时就根据该信息取
```

#### 枚举和集合类型

```python
enum()  # 多选一

create tables user(
  id int,
  name varchar(32),
  gender enum('male','female','others')
);
insert into user values(1,'tom','male')
# gender字段的取值只有'male','female','others'里面的一个

set()  # 多选多(包含多选一)

create table userinfo(
	id int,
  hobby set('basketball','football','doublecolorball')
);
```

#### 日期类型

```python
create table client(
	id int,
    name varchar(32),
    reg_time date,
    birth datetime,
    study_time time,
    join_time year
);
insert into client values(1,'jason','2000-11-11','2000-1-21 11:11:11','11:11:11',1995);
```

### 约束条件

```python
"""
约束条件相当于是在字段类型的基础之上添加的额外约束
	eg: id int unsigned
"""
unsigned		让数字没有正负号
zerofill		多余的使用数字0填充
not null		非空
	"""
	新增表数据的方式
		方式1:  按照字段顺序一一传值
			 insert into t1 values(1,'jason');
		方式2:  自定义传值顺序 甚至不传
			insert into t1(name,id) values('jason',1);
			insert into t1(id) values(1);
	在MySQL中不传数据 会使用关键字NULL填充意思就是空 类似于python的None
	"""
    create table t2(
    	id int,
        name varchar(32) not null
    );
default			默认值
	"""
	所有的字段都可以设置默认值
		用户不给该字段传值则使用默认的 否则使用传了的
	create table t3(
		id int default 911,
		name varchar(16) default 'jason'
	);
	"""
unique			唯一值
	"""
	单列唯一
		create table t4(
			id int,
			name varchar(32) unique
		);
	联合唯一
		create table t5(
			id int,
			host varchar(32),
			port int,
			unique(host,port)
		);
	"""
primary key		主键
	"""
	但从约束层面上来说 相当于是 not null + unique(非空且唯一)
	在此基础之上还可以加快数据的查询

	InnoDB存储引擎规定了一张表必须有且只有一个主键
		因为InnoDB是通过主键的方式来构造表的
		如果没有设置主键
			情况1:没有主键和其他约束条件
				InnoDB会采用隐藏的字段作为主键 不能加快数据的查询
			情况2:没有主键但是有非空且唯一的字段
				自动将该字段升级为主键
				create table t6(
					id int,
					age int not null unique,
					pwd int not null unique
				);
	结论:
		以后我们在创建表的时候一定要设置主键
		并且主键字段一般都是表的id字段(uid sid pid cid)
		create table user(
			id int primary key,
			name varchar(32)
		);
	"""
auto_increment		自增
	"""
	由于主键类似于数据的唯一标识 并且主键一般都是数字类型
	我们在添加数据的时候不可能记住接下来的序号是多少 太麻烦
	create table user1(
			id int primary key auto_increment,
			name varchar(32)
		);
	"""
  自增不会因为删除操作而回退
	  delete from无法影响自增

  如果想要重置需需要使用truncate关键字
	  truncate 表名  # 清空表数据并且重置主键值
```

```mysql
create database db1 charset='utf8';
use db1;
create table student(
  id int primary key auto_increment comment '学号',
  name varchar(64) not null comment '学生姓名',
  age int not null comment '学生年龄',
  gender enum('male','female','others')
);
```
