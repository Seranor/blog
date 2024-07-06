---
title: 模板语法与Django部分源码
lastmod: 2021-09-03T16:43:23+08:00
date: 2021-09-02T11:52:03+08:00
tags:
  - Django
categories:
  - Django
url: post/django-05.html
toc: true
---

### 伪静态

```python
将url地址模拟成html结尾，看上去是一个静态文件，目的是为了增加搜索引擎对网站的爬取和SEO
```

<!-- more -->

### 本地虚拟环境

```python
相当于创建一个新的python环境

## Linux使用virtualenv创建虚拟环境
# 安装扩展
pip3 install virtualenv
pip3 install virtualenvwrapper

# pip使用
pip3 list	        # 列出所有的扩展
pip3 freeze       # 查看所有的扩展及版本，便于导出
pip3 show 扩展名	  # 查看扩展详细信息

# 查找路径
which virtualenvwrapper.sh
which python3
which virtualenv

# 编辑全局环境，当前用户的
vi ~/.bashrc

export WORKON_HOME=/home/lzj/.virtualenvs
source /home/lzj/.local/bin/virtualenvwrapper.sh
export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3
export VIRTUALENVWRAPPER_VIRTUALENV=/home/lzj/.local/bin/virtualenv

# 生效
mkdir ~/.virtualenvs
source ~/.bashrc

# 使用
mkvirtualenv test-env	     # 创建test-env 的虚拟环境,默认进入到虚拟环境中
mkvirtualenv --python==	   # 指定python版本路径创建虚拟环境
lsvirtualenv			         # 查看所有的虚拟环境
rmvirtualenv			         # 删除
cdvirtualenv			         # 进入虚拟环境的目录，需要进入到虚拟环境
workon test-env			       # 进入test-env虚拟环境中
deactivate				         # 退出当前虚拟环境
```

### Django 版本差别

```python
django1.x与django2.x 3.x的区别

1.urls.py中的路由匹配方法
  1.x 第一个参数是正则表达式  url()
  2.x和3.x 第一个参数不支持正则表达式，写什么匹配什么  path()

  2.x与3.x 如果要用正则表达式需要导入模块 re_path
  from django.urls import path,re_path
  re_path()  相当于 1.x的url()

2.转换器
	 五种常用转换器
    str,匹配除了路径分隔符（/）之外的非空字符串，这是默认的形式
    int,匹配正整数，包含0。
    slug,匹配字母、数字以及横杠、下划线组成的字符串。
    uuid,匹配格式化的uuid，如 075194d3-6885-417e-a8a8-6c931e272f00。
    path,匹配任何非空字符串，包含了路径分隔符（/）（不能用？）

     	 自定义
    class MonthConverter:
        regex='\d{2}' # 属性名必须为regex

        def to_python(self, value):
            return int(value)

        def to_url(self, value):
            return value # 匹配的regex是两个数字，返回的结果也必须是两个数字

    from django.urls import path,register_converter
    from app01.path_converts import MonthConverter

    register_converter(MonthConverter,'mon')

    from app01 import views

    urlpatterns = [
        path('articles/<int:year>/<mon:month>/<slug:other>/', views.article_detail, name='aaa'),

    ]
```

### 三板斧本质

```python
django视图函数必须要返回一个HttpResponse对象
'''
通过查看 HttpResponse render redirect 等源码可以看出都是返回的HttpResponse
'''
```

### JsonResponse

```python
需求: 给前端返回JSON格式数据

方式1: 自己序列化
  # views.py
 	def index(request):
      import json
      d = {'name': 'jason', 'age': 18, 'pwd': '123','desc':'哈哈哈'}
      res = json.dumps(d,ensure_ascii=False)  # 针对有中文的情况下
      return HttpResponse(res)


方式2: JsonResponse
  from django.http import JsonResponse
  def index(request):
      d = {'name': 'jason', 'age': 18, 'pwd': '123','desc':'哈哈哈'}
      return JsonResponse(d)

  # 额外参数补充 查看JsonResponse源码
  json_dumps_params={'ensure_ascii': False}  # 中文编码
  safe=False                                 # 默认情况只序列化字典类型，这个参数可以将非字典类型序列化
```

### 上传文件

```python
form表单上传文件注意事项
  1.method必须是post
  2.enctype参数修改为 multipart/form-data

前端:
  <form action="" method="post" enctype="multipart/form-data">
    <p><input type="text" name="userinfo"></p>
    <p><input type="file" name="my_file"></p>
    <input type="submit">
  </form>

后端:
  def get_file(request):
      if request.method == 'POST':
      print(request.POST)
      file_obj = request.FILES.get('my_file')
      print(file_obj.name)
      with open(r'%s' % file_obj.name, 'wb') as f:
            for chunks in file_obj.chunks():  # chunks 会一行一行读取 可以提升效率
                f.write(chunks)
    return render(request, 'index.html')

  # urls.py url(...)
```

### FBV 与 CBV

```python
FBV基于函数的视图

CBV基于类的视图
# views.py
from django.views import View
class MyView(View):
    def get(self, request):
        return HttpResponse('get 请求')

    def post(self, request):
        return HttpResponse('post 请求')
# urls.py
   url(r'^mycbv', views.MyView.as_view())

as_view()是View类的静态方法，返回的是view函数地址，本质还是FBV
```

### CBV 源码

```python
	    def as_view(cls, **initkwargs):
            def view(request, *args, **kwargs):
                self = cls(**initkwargs)  # self = MyView()  生成一个我们自己写的类的对象
                return self.dispatch(request, *args, **kwargs)
            return view
       def dispatch(self, request, *args, **kwargs):
        # 获取当前请求并判断是否属于正常的请求(8个)
        if request.method.lower() in self.http_method_names:
            # get请求  getattr(对象,'get')   handler = 我们自己写的get方法
            handler = getattr(self, request.method.lower(), self.http_method_not_allowed)
        else:
            handler = self.http_method_not_allowed
        return handler(request, *args, **kwargs)  # 执行我们写的get方法并返回该方法的返回值

      # 面向对象的反射
```

### settings 源码

```python
"""
1.django其实有两个配置文件
	一个是暴露给用户可以自定义的配置文件
		项目根目录下的settings.py
	一个是项目默认的配置文件
		当用户不做任何配置的时候自动加载默认配置
        from django.conf import global_settings(所有全局配置global_settings)
2.配置文件变量名必须是大写
"""
疑问:为什么当用户配置了就使用用户配置的 不配置就是要默认的
from django.conf import settings

settings = LazySettings()

os.environ.setdefault("DJANGO_SETTINGS_MODULE", "day05.settings")
ENVIRONMENT_VARIABLE = "DJANGO_SETTINGS_MODULE"
class LazySettings(LazyObject):
    def _setup(self, name=None):
        # os.environ看成是一个全局大字典      'day05.settings'
        settings_module = os.environ.get(ENVIRONMENT_VARIABLE)
        self._wrapped = Settings(settings_module)  # Settings('day05.settings')

class Settings(object):
    def __init__(self, settings_module):  # 'day05.settings'
        for setting in dir(global_settings):  # 获取全局配置文件里面所有的变量名
            if setting.isupper():  # 校验是否是纯大写
                setattr(self, setting, getattr(global_settings, setting))
                # 给Settings对象添加全局配置文件中所有的配置信息

        self.SETTINGS_MODULE = settings_module  # 'day05.settings'
        mod = importlib.import_module(self.SETTINGS_MODULE)
        # from day05 import settings  # 导入暴露给用户的自定义配置文件
        for setting in dir(mod):
            if setting.isupper():
                setting_value = getattr(mod, setting)
                setattr(self, setting, setting_value)
```

### 模板语法

#### 传值

```python
def index(request):
    i = 666
    f = 1.1
    s = 'hello world'
    l = [11, 22, 33, 44]
    d = {'username': 'jason', 'pwd': 123}
    t = (1, 2, 3)
    se = {11, 22, 33}
    b = True
    # 传值方式1
    # return render(request, 'index.html', {'i': i, 's': s, 'd': d})
    # 传值方式2
    # return render(request, 'index.html', locals())


传值方式1: 利用字典单个传
  return render(request, 'index.html', {'i': i, 's': s, 'd': d})

传值方式2: locals() 将当前名称空间所有变量传递给页面
  return render(request, 'index.html', locals())

区别:
  方式1 传值精确 不会造成资源浪费
  方式2 一次性全部传值 可能会造成资源浪费

补充:传递函数名和类名都会自动加括号调用(模板语法不支持额外的传参)
```

#### 获取值

```python
模板语法取值只能采用 句点符(.)
列表的索引值和字典的键值都可以通过 . 一直往下取值
{{  }}

# 后端的视图函数
def index(request):
    d = {'username': 'jason', 'pwd': 123, 'hobby': [111, 222, 333, {'desc': '这个是字典下的列表里面的字典'}]}
    return render(request, 'index.html', locals())

# 前端取值
<p>{{ d.hobby.3.desc }}</p>
```

#### 过滤器

```python
类似于python的内置方法
前端:
<p>统计长度：{{ s|length }}</p>
<p>加法运算：{{ i|add:1000 }}</p>
<p>字符拼接：{{ s|add:'python' }}</p>
<p>日期格式：{{ ctime|date:'Y年-m月-d日' }}</p>
<p>默认值：{{ b|default:'aaaa' }}</p>
<p>默认值{{ b1|default:'aaaa' }}</p>
<p>文件大小{{ file_size|filesizeformat }}</p>
<p>截取文本(三个点也算){{ s|truncatechars:6 }}</p>
<p>截取文本(三个点不算){{ s|truncatewords:2 }}</p>
<p>safe可以解析字符串内的标签{{ h|safe }}</p>
<p>{{ sss }}</p>
<p>{{ sss1 }}</p>

后端views.py
def index(request):
    i = 666
    f = 1.1
    s = 'hello world'
    l = [11, 22, 33, 44]
    d = {'username': 'jason', 'pwd': 123, 'hobby': [111, 222, 333, {'desc': '这个是字典下的列表里面的字典'}]}
    t = (1, 2, 3)
    se = {11, 22, 33}
    b = True
    b1 = False
    ctime = datetime.today()
    file_size = 5012341
    h = '<h1>哈哈哈</h1>'
    sss = '<h2>老子要挣大钱</h2>'
    from django.utils.safestring import mark_safe
    sss1 = mark_safe('<h2>老子要挣大钱</h2>')
    return render(request, 'index.html', locals())

# 后端写好的标签字符串也可以在前端解析
  方式1: 后端字符串变量|safe
  方式2:
    from django.utils.safestring import mark_safe
    sss1 = mark_safe('<h2>老子要挣大钱</h2>')  # 前端直接使用该变量值就可以解析
```

#### 标签

```python
类似于python的逻辑控制
# if语句
{% if file_size %}
    <p>有值</p>
{% else %}
    <p>无值</p>
{% endif %}

# forloop是内置对象 有first  last等 可以提供循环的判断
{% for foo in l %}
    <p>{{ forloop }}</p>
{% endfor %}

# for和if判断结合
{% for foo in s %}
    {% if forloop.first %}
        <p>这是第一次</p>
    {% elif forloop.last %}
        <p>这是最后一次</p>
    {% else %}
        <p>{{ foo }}</p>
    {% endif %}
     {% empty %}
        <p>传入的数据是空的</p>
{% endfor %}

# empty 当循环的值时空的时候就会走 这条后的逻辑

{{}}    # 变量相关
{% %}   # 逻辑相关

# 相当于取别名，而且该别名只在 with 块内生效
{% with d.hobby.3.desc as desc %}
    {{ desc }}
{% endwith %}
```

#### 自定义

```python
类似于python里面的自定义函数
1.应用下创建一个名字必须为 templatetags 文件夹
2.上述文件夹内创建一个任意名称的py文件
3.在该文件内固定先书写以下两行代码
	from django import template
  register = template.Library()

# templatetags/mytag.py
# 自定义过滤器
@register.filter(name='myfilter')
def index(a, b):  # 只能接收两个参数，需要传入多个参数可以让这两个参数按照一定字符分割'a|b|c'
    return a + b

# 自定义标签
@register.simple_tag(name='mysimple')
def func1(a, b, c, d):
    return '%s ** %s | %s > %s' % (a, b, c, d)

# 自定义inclusion_tag
@register.inclusion_tag('login.html', name='my_inclusion_tag')
def func2(n):
    l = []
    for i in range(1, n + 1):
        l.append('第%s页' % i)
    return locals()

# login.html
<ul>
    {% for foo in l %}
        <li>{{ foo }}</li>
    {% endfor %}
</ul>

# 前端使用
{% load mytag %}                   # 先加载mytag.py的名称
{{ i|myfilter:100 }}               # i 参数来自后端 100 是第二个参数 然后相加
{% mysimple 2 'xxx' 8 'jjj' %}     # 传入四个参数
{% my_inclusion_tag 10 %}          # 将 my_inclusion_tag 生成的 login.html 传给当前html
{% my_inclusion_tag 8 %}
```

### 模板导入

```python
类似于python的模块，可以将局部页面直接导入即可

# 模板页面内容
<h1>这是一个form表单</h1>

# 想要引用模板文件，用如下方式
{% include 'myform.html' %}

# 这种导入和上述的 inclusion_tag 有点像，但是这种方式是导入的静态写死的页面
```

### 模板继承

```python
在html页面内使用 block 划定指定区域
母版
    {% block 区域名称 %}
    {% endblock %}

子版
	{% extends 'home.html' %}
  {% block 区域名称 %}
  		可以替换成自己页面
      同时还可以继承使用母版   {{ block.super }}
	{% endblock %}

母版在划定区域的时候一般划成三个区域
	css
  html
  js

  {% block css %}
  {% endblock %}

  {% block content %}
  {% endblock %}

  {% block js %}
  {% endblock %}
```

![image-20220224195506829](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220224195506829.png)

![image-20220224195530524](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220224195530524.png)

![image-20220224195619269](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220224195619269.png)

![image-20220224195632402](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220224195632402.png)

![image-20220224195650917](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220224195650917.png)

![image-20220224195723636](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220224195723636.png)
