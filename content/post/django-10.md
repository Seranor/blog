---
title: 分页器
lastmod: 2021-09-03T16:43:23+08:00
date: 2021-09-02T11:52:03+08:00
tags:
  - Django
categories:
  - Django
url: post/django-10.html
toc: true
---

### 批量插入数据

<!-- more -->

```python
# models.py
class Books(models.Model):
    name = models.CharField(max_length=32)
    price = models.DecimalField(max_digits=5, decimal_places=2)
    publish = models.CharField(max_length=32)

# views.py
def book_page(request):
    # 第一种方案，每循环依次，操作一下数据库，性能低
    # for i in range(1000):
    #     book = models.Books.objects.create(name='图书%s' % i, price=i + 10, publish='北京出版社')

    # 分批插入
    book_list = []
    for i in range(1000):
        book = models.Books(name='图书%s' % i, price=i + 10, publish='北京出版社')
        book_list.append(book)
    models.Books.objects.bulk_create(book_list, batch_size=10)  # batch_size 分几批插入
    return HttpResponse('ok')

# urls.py
url(r'^book_page/', views.book_page),
```

### 分页器的方法

```python
def books_page(request):
    from django.core.paginator import Paginator
    book_list = models.Books.objects.all()
    paginator = Paginator(book_list, 10)
    # Paginator对象属性
    print(paginator.count)
    print(paginator.num_pages)
    print(paginator.per_page)
    print(paginator.page_range)
    print(paginator.page(1))

    # Page对象的属性和方法
    page = paginator.page(2)
    print(page.has_next())
    print(page.next_page_number())
    print(page.has_previous())
    print(page.previous_page_number())
    print(page.object_list)
    print(page.number)
    return render(request, 'book_page.html', locals())


'''
# Paginator对象属性
    paginator.count            # 数据总条数
    paginator.num_pages        # 总页数
    paginator.per_page         # 每页显示条数
    paginator.page_range       # range(1, 101)
    paginator.page(1)
# Page对象的属性和方法
    page = paginator.page(2)
    page.has_next              # 是否有下一页
    page.next_page_number      # 下一页页码
    page.has_previous          # 是否有上一页
    page.previous_page_number  # 上一页页码
    page.object_list           # 分页之后的数据列表
    page.number                # 当前页
'''
```

### 分页器使用

#### 后端

```python
from django.shortcuts import render, HttpResponse, redirect
from app01 import models
from django.core.paginator import Paginator


def book_page(request):
    current_num = int(request.GET.get('page_num', 1))
    book_list = models.Books.objects.all()

    paginator = Paginator(book_list, 20)
    # 如果页码不在范围内直接赋值为1，相当于调到第一页
    try:
        page = paginator.page(current_num)
    except Exception as e:
        current_num = 1
        page = paginator.page(current_num)

    # 将页码分为前五后五的情况，总数为11页
    if paginator.num_pages > 11:
        if current_num - 5 < 1:
            page_range = range(1, 12)
        elif current_num + 5 > paginator.num_pages:
            page_range = range(paginator.num_pages - 10, paginator.num_pages + 1)
        else:
            page_range = range(current_num - 5, current_num + 6)
    else:
        page_range = paginator.page_range
    return render(request, 'book_page.html', locals())
```

#### 前端

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Title</title>
    <script src="https://cdn.bootcdn.net/ajax/libs/jquery/3.4.1/jquery.min.js"></script>
    <link
      href="https://cdn.bootcdn.net/ajax/libs/twitter-bootstrap/3.4.1/css/bootstrap.min.css"
      rel="stylesheet"
    />
    <script src="https://cdn.bootcdn.net/ajax/libs/twitter-bootstrap/3.4.1/js/bootstrap.min.js"></script>
  </head>
  <body>
    <div class="row">
      <div class="col-md-8 col-md-offset-2">
        <table class="table table-striped table-hover">
          <thead>
            <tr>
              <td>id</td>
              <td>书名</td>
              <td>价格</td>
              <td>出版社</td>
            </tr>
          </thead>
          <tbody>
            {# 只显示当前页码的数据 #} {% for book in page.object_list %}
            <tr>
              <td>{{ book.id }}</td>
              <td>{{ book.name }}</td>
              <td>{{ book.price }}</td>
              <td>{{ book.publish }}</td>
            </tr>
            {% endfor %}
          </tbody>
        </table>
      </div>
      <div class="col-md-6 col-md-offset-4">
        <nav aria-label="Page navigation">
          <ul class="pagination">
            {# 判断当在第一页时，就没有上一页了，将上一页按钮设置为不可选中 #}
            {% if page.has_previous %}
            <li>
              <a
                href="/book_page/?page_num={{ page.previous_page_number }}"
                aria-label="Previous"
              >
                <span aria-hidden="true">&laquo;</span>
              </a>
            </li>
            {% else %}
            <li class="disabled">
              <a href="#" aria-label="Previous">
                <span aria-hidden="true">&laquo;</span>
              </a>
            </li>
            {% endif %} {#
            使用GET方法提交当前页数给后端，后端处理后获取显示多少页码数进行循环
            #} {% for foo in page_range %} {% if current_num == foo %}
            <li class="active">
              <a href="/book_page/?page_num={{ foo }}">{{ foo }}</a>
            </li>
            {% else %}
            <li><a href="/book_page/?page_num={{ foo }}">{{ foo }}</a></li>
            {% endif %} {% endfor %} {#
            判断当在最后一页时，就没有最后一页了，将下一页按钮设置为不可选中 #}
            {% if page.has_next %}
            <li>
              <a
                href="/book_page/?page_num={{ page.next_page_number }}"
                aria-label="Next"
              >
                <span aria-hidden="true">&raquo;</span>
              </a>
            </li>
            {% else %}
            <li class="disabled">
              <a href="#" aria-label="Next">
                <span aria-hidden="true">&raquo;</span>
              </a>
            </li>
            {% endif %}
          </ul>
        </nav>
      </div>
    </div>
  </body>
</html>
```

### 自定义分页器

当我们需要使用到非 django 内置的第三方功能或者组件代码的时候，我们一般情况下会创建一个名为 utils 的文件夹，在该文件夹内对模块进行功能性划分。eg：mypage.py

我们到了后期封装代码的时候，不再局限于函数，而是尽量朝面向对象去封装

将下面封装好的模板拷贝到 utils 文件夹下的 mypage.py（自定义的名字随意取）

```python
class Pagination(object):
    def __init__(self, current_page, all_count, per_page_num=2, pager_count=5):
        """
        封装分页相关数据
        :param current_page: 当前页
        :param all_count:    数据库中的数据总条数
        :param per_page_num: 每页显示的数据条数
        :param pager_count:  最多显示的页码个数
        """
        try:
            current_page = int(current_page)
        except Exception as e:
            current_page = 1

        if current_page < 1:
            current_page = 1

        self.current_page = current_page

        self.all_count = all_count
        self.per_page_num = per_page_num

        # 总页码
        all_pager, tmp = divmod(all_count, per_page_num)
        if tmp:
            all_pager += 1
        self.all_pager = all_pager

        self.pager_count = pager_count
        self.pager_count_half = int((pager_count - 1) / 2)

    @property
    def start(self):
        return (self.current_page - 1) * self.per_page_num

    @property
    def end(self):
        return self.current_page * self.per_page_num

    def page_html(self):
        # 如果总页码 < 11个：
        if self.all_pager <= self.pager_count:
            pager_start = 1
            pager_end = self.all_pager + 1
        # 总页码  > 11
        else:
            # 当前页如果<=页面上最多显示11/2个页码
            if self.current_page <= self.pager_count_half:
                pager_start = 1
                pager_end = self.pager_count + 1

            # 当前页大于5
            else:
                # 页码翻到最后
                if (self.current_page + self.pager_count_half) > self.all_pager:
                    pager_end = self.all_pager + 1
                    pager_start = self.all_pager - self.pager_count + 1
                else:
                    pager_start = self.current_page - self.pager_count_half
                    pager_end = self.current_page + self.pager_count_half + 1

        page_html_list = []
        # 添加前面的nav和ul标签
        page_html_list.append('''
                    <nav aria-label='Page navigation>'
                    <ul class='pagination'>
                ''')
        first_page = '<li><a href="?page=%s">首页</a></li>' % (1)
        page_html_list.append(first_page)

        if self.current_page <= 1:
            prev_page = '<li class="disabled"><a href="#">上一页</a></li>'
        else:
            prev_page = '<li><a href="?page=%s">上一页</a></li>' % (self.current_page - 1,)

        page_html_list.append(prev_page)

        for i in range(pager_start, pager_end):
            if i == self.current_page:
                temp = '<li class="active"><a href="?page=%s">%s</a></li>' % (i, i,)
            else:
                temp = '<li><a href="?page=%s">%s</a></li>' % (i, i,)
            page_html_list.append(temp)

        if self.current_page >= self.all_pager:
            next_page = '<li class="disabled"><a href="#">下一页</a></li>'
        else:
            next_page = '<li><a href="?page=%s">下一页</a></li>' % (self.current_page + 1,)
        page_html_list.append(next_page)

        last_page = '<li><a href="?page=%s">尾页</a></li>' % (self.all_pager,)
        page_html_list.append(last_page)
        # 尾部添加标签
        page_html_list.append('''
                                           </nav>
                                           </ul>
                                       ''')
        return ''.join(page_html_list)
```

#### 后端模板

```python
from utils.mypage import Pagination  # 导入模板
# 书籍的展示
def books(request):
    # 先查询出所有要展示的数据信息，
    book_queryset = models.Book.objects.all()
    current_page = request.GET.get('page',1)
    all_count = book_queryset.count()
    # 传值生成对象
    page_obj = Pagination(current_page=current_page,all_count=all_count)
    # 直接对总数据进行切片操作
    page_queryset = book_queryset[page_obj.start:page_obj.end]
    # 将page_queryset传递到页面，替换之前的book_queryset
    return render(request,'books.html',locals())
```

#### 前端

```html
<div class="text-center">{{ page_obj.page_html|safe }}</div>
```

#### 实例

```html
{% extends 'home.html' %} {% block main %}
<script>
  $(".index").removeClass("active");
  $(".books").addClass("active");
  $(".publish").removeClass("active");
  $(".author").removeClass("active");
</script>
<div>
  <div class="panel panel-primary">
    <div class="panel-heading">
      <h3 class="panel-title">图书管理</h3>
    </div>
    <div class="panel-body">
      <div>
        <a href="/book_add/" class="btn btn-success pull-right">添加书籍</a>
      </div>
      <br />
      <br />

      <table class="table table-striped table-hover table-bordered">
        <thead>
          <tr>
            <th>ID</th>
            <th>书名</th>
            <th>价格</th>
            <th>出版日期</th>
            <th>出版社</th>
            <th>作者</th>
            <th>操作</th>
          </tr>
        </thead>
        <tbody>
          {% for book in page_queryset %}
          <tr>
            <td>{{ book.id }}</td>
            <td class="warning">{{ book.title }}</td>
            <td class="info">{{ book.price }}</td>
            <td class="dark">{{ book.publish_date|date:'Y-m-d' }}</td>
            <td class="success">{{ book.publish.name }}</td>
            <td>
              {% for author in book.authors.all %} {% if forloop.last %} {{
              author.name }} {% else %} {{ author.name }}、 {% endif %} {%
              endfor %}
            </td>
            <td>
              <a
                href="{% url 'book_edit' book.pk %}"
                class="btn btn-primary btn-xs"
                >编辑</a
              >
              <a
                href="{% url 'book_delete' book.pk %}"
                class="btn btn-danger btn-xs"
                >删除</a
              >
              <button
                type="button"
                class="btn btn-primary btn-lg"
                data-toggle="modal"
                data-target="#myModal"
                id="bbb"
                onclick="AA('{{ html }}')"
              >
                Launch
              </button>
            </td>
          </tr>
          {% endfor %}
        </tbody>
      </table>
      {# 只需要这一行代码就可以实现分页 #}
      <div class="text-center">{{ page_obj.page_html|safe }}</div>
    </div>
  </div>
</div>

{% endblock %}
```
