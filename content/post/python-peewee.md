---
title: Python的其他ORM(peewee)
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - Python
categories:
  - Python
url: post/python-peewee.html
toc: true
---

### 安装

```python
pip3 install peewee
pip3 install pymysql

http://docs.peewee-orm.com/en/latest/peewee/models.html
```

<!-- more -->

### 快速使用

```python
import datetime
from peewee import *

# db = SqliteDatabase('my_app.db')
db = MySQLDatabase('peewee', host='127.0.0.1', user='root', password='asd123...')


class User(Model):
    username = CharField(unique=True)

    class Meta:
        database = db


class Tweet(Model):
    user = ForeignKeyField(User, backref='tweets')
    message = TextField()
    created_date = DateTimeField(default=datetime.datetime.now)
    is_published = BooleanField(default=True)

    class Meta:
        database = db


if __name__ == '__main__':
    # 生成表结构
    db.connect()
    db.create_tables([User, Tweet])
    # 直接运行生成表 修改表结构就需要到数据库中修改
```

### 打印 SQL 语句

```python
import logging

logger = logging.getLogger("peewee")
logger.setLevel(logging.DEBUG)
logger.addHandler(logging.StreamHandler())
```

### 新增数据

```python
# ...

if __name__ == '__main__':
    # ...

    # 添加
    charlie = User(username="charlie")
    charlie.save()                    # 既可以新建  也可以更新  有返回值 row 影响的行
    charlie.save(force_insert=True)   # force_insert=True 强制为更新操作
    # 对象的主键是否设置

    huey = User.create(username="huey")
```

### 查询数据

```python
# ...

if __name__ == '__main__':
    # ...

    # 查询
    # get 方法 返回的直接的 user 对象 如果查询不到会抛出异常
    try:
         charlie = User.get(User.username == "charlie")
         charlie_id = User.get_by_id("2")
         print(charlie.username)
         print(charlie_id.username)
    except User.DoesNotExist as e:
         print("查询不到")

    # 查询所有
    users = User.select()  # ModelSelect 对象 使用时才会发起查询  用于组装 sql
    print(users.sql())
    print(type(users))

    user = users[0]
    print(type(user))  # User 对象

    usernames = ["charlie", "huey"]
    users = User.select().where(User.username.in_(usernames))
    for user in users:
        print(user.username)


    for user in User.select():
        print(user.username)
```

### 更新数据

```python
# 方式一
charlie = User(username="charlie")  # update set xx=x where username = "charlie"
print(charlie.save())

# 方式二
print(User.update(age=20).where(User.username == "charlie").execute())
```

### 删除数据

```python
# 方式一
user = User.get(User.username == "charlie")
user.delete_instance()

# 方式二
query = User.delete().where(User.username == "huey").execute()
print(query)  # 影响的行数
```

### 定义表

```python
import datetime
from peewee import *
import logging

logger = logging.getLogger("peewee")
logger.setLevel(logging.DEBUG)
logger.addHandler(logging.StreamHandler())

db = MySQLDatabase('peewee', host='127.0.0.1', user='root', password='asd123...')


class BaseModel(Model):
    add_time = DateTimeField(default=datetime.datetime.now, verbose_name="添加时间")

    class Meta:
        database = db  # 数据库连接


class Person(BaseModel):
    name = CharField(max_length=10, null=False, index=True, verbose_name="姓名")
    passwd = CharField(max_length=20, null=False, default='123456', verbose_name="密码")
    email = CharField(max_length=50, null=True, unique=True, verbose_name="邮箱")
    gender = IntegerField(null=True, default=1, verbose_name="性别")
    birthday = DateField(null=True, default=None, verbose_name="生日")
    is_admin = BooleanField(default=True, verbose_name="是否是管理员")

    class Meta:
        table_name = 'persons'  # 自定义表名


if __name__ == '__main__':
    db.connect()
    db.create_tables([Person, ])
```

### 主键约束

```python
import datetime

from peewee import *
import logging

logger = logging.getLogger("peewee")
logger.setLevel(logging.DEBUG)
logger.addHandler(logging.StreamHandler())

db = MySQLDatabase('peewee', host='127.0.0.1', user='root', password='asd123...')


class BaseModel(Model):
    add_time = DateTimeField(default=datetime.datetime.now, verbose_name="添加时间")

    class Meta:
        database = db  # 数据库连接


class Person(BaseModel):
    first = CharField()
    last = CharField()

    class Meta:
        primary_key = CompositeKey('first', 'last')


class Pet(BaseModel):
    owner_first = CharField()
    owner_last = CharField()
    pet_name = CharField()

    class Meta:
        constraints = [SQL('FOREIGN KEY(owner_first,owner_last) REFERENCES person(first,last)')]


class Blog(BaseModel):
    pass


class Tag(BaseModel):
    pass


# 复合主键
class BlogToTag(BaseModel):
    blog = ForeignKeyField(Blog)
    tag = ForeignKeyField(Tag)

    class Meta:
        primary_key = CompositeKey('blog', 'tag')


if __name__ == '__main__':
    db.connect()
    db.create_tables([Person, Pet, Tag, Blog, BlogToTag])
```

### 多数据插入

```python
p_id = Person.insert({
    'first': 'liu',
    'last': 'jin'
}).execute()
print(p_id)

id = Blog.insert({}).execute()  # add_time 字段此时
print(id)

blog = Blog()
print(blog.id)
blog.save()
print(blog.id)   # add_time 就有当前时间了 自动设置


for data_dict in data_source:
    Model.create(**data_dict)

# 性能高的方法
blogs = [
    {"add_time": datetime.datetime.now()},
    {"add_time": datetime.datetime.now()},
]
query = Blog.insert_many(blogs).execute()
print(query)
```

### 复合条件查询

```python
query1 = Person.select().where((Person.name == "fff") | (Person.name == "xxx"))
query1 = Person.select().where((Person.name == "fff") & (Person.is_relative == True))


person = Person.select().where((Person.first == "liu") | (Person.first == "liu1"))
print(type(person))  # ModelSelect 对象
for p in person:
    print(p.last)
```

### 模糊查询

```python
query = Person.select().where(Person.first.contains("liu"))

# 更多的方法 http://docs.peewee-orm.com/en/latest/peewee/query_operators.html

# query = Person.select().where(Person.first.startswith("liu"))
# print(query)

for q in query:
    print(q.first)
```

### limit、排序、去重

```python
# limit
query = Person.select().limit(2)
for row in query:
    print(row)

# 排序
query = Person.select().order_by(Person.add_time.desc() )  # desc 降序 默认是升序
# query = Person.select().order_by(-Person.add_time)   # 降序
for r in query:
    print(r.add_time)

# 去重
query = Person.select(Person.first).distinct()
for q in query:
    print(q.first)

# 去重后计数
query = Person.select(Person.first).distinct().count()
print(query)
```

### 聚合函数

```python

```
