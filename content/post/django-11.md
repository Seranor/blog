---
title: Forms组件
lastmod: 2021-09-03T16:43:23+08:00
date: 2021-09-02T11:52:03+08:00
tags:
  - Django
categories:
  - Django
url: post/django-11.html
toc: true
---

### forms 组件校验数据功能

#### 校验步骤

```python
定义一个类，继承forms.Form
在类中写要校验的字段，字段属性就是校验规则
实例化得到一个Form对象，要把校验的数据传入
调用register_form.is_valid()校验，校验通过是True
校验通过有register_form.cleaned_data
校验不通过有register_form.errors
```

<!-- more -->

```python
# 定义类
from django import forms
from django.forms import widgets
from django.core.exceptions import ValidationError
class RegisterForm(forms.Form):
    name = forms.CharField(max_length=8, min_length=3, label='用户名')
    password = forms.CharField(max_length=8, min_length=3, label='密码')
    re_password = forms.CharField(max_length=8, min_length=3, label='确认密码')
    email = forms.EmailField(label='邮箱')

# 在视图中使用
register_form = RegisterForm(request.POST)
if register_form.is_valid():
      # 校验通过，存
      # 取出校验通过的数据
      print('校验通过')
      print(register_form.cleaned_data)
else:
     # 校验不通过
     print('校验不通过')
     print(register_form.errors)
```

### forms 渲染标签

#### 方式一

```html
<h1>form自动渲染 一</h1>
<form action="" method="post">
  <p>用户名:{{ register_form.name }}</p>
  <p>密码: {{ register_form.password }}</p>
  <p>确认密码: {{ register_form.re_password }}</p>
  <p>邮箱: {{ register_form.email }}</p>
  <input type="submit" value="提交" />
</form>
```

#### 方式二(常用)

```html
<div class="container-fluid">
  <div class="row">
    <div class="col-md-8 col-md-offset-2">
      <h1>form自动渲染 二</h1>
      <form action="" method="post">
        {% for item in register_form %}
        <p>{{ item.label }}: {{ item }}</p>
        <span style="color: red">{{ item.errors.0 }}</span>
        {% endfor %}
        <input type="submit" value="提交" /><span style="color: red"
          >{{ error }}</span
        >
      </form>
    </div>
  </div>
</div>

''' # 后端的 lable 字段是渲染时给前端页面的提示信息 # 对应这边的 {{ item.label
}} '''
```

#### 方式三

```html
<h1>form自动渲染 三</h1>
<form action="" method="post">
  {{ register_form.as_p }} {# {{ register_form.as_table }}#} {# {{
  register_form.as_ul }}#}
  <input type="submit" value="提交" />
</form>
```

### form 组件渲染错误信息

#### 前端

```html
<div class="container-fluid">
  <div class="row">
    <div class="col-md-8 col-md-offset-2">
      <h1>form自动渲染 二</h1>
      <form action="" method="post">
        {# span 中由form传入错误提示 #} {% for item in register_form %} {#
        item.errors.0取标签内的数据，否则会插入一个 li 标签 #}
        <p>{{ item.label }}: {{ item }}</p>
        <span style="color: red">{{ item.errors.0 }}</span>
        {% endfor %}
        <input type="submit" value="提交" /><span style="color: red"
          >{{ error }}</span
        >
      </form>
    </div>
  </div>
</div>
```

#### 后端

```python
from django import forms
from app01 import models
from django.forms import widgets
from django.core.exceptions import ValidationError


class RegisterForm(forms.Form):
    name = forms.CharField(max_length=16, min_length=3, label='用户名',
                           error_messages={'max_length': '最长位16', 'min_length': '最短为3'},
                           widget=widgets.TextInput(attrs={'class': 'form-control'}))
    password = forms.CharField(max_length=16, min_length=3, label='密码', error_messages={'required': '该字段必填'},
                               widget=widgets.PasswordInput(attrs={'class': 'form-control'}))
    re_password = forms.CharField(max_length=16, min_length=3, label='再次确认密码', error_messages={'required': '该字段必填'},
                                  widget=widgets.PasswordInput(attrs={'class': 'form-control'}))
    email = forms.EmailField(label='邮箱', error_messages={'required': '该字段必填', 'invalid': '必须是邮箱格式'},
                             widget=widgets.TextInput(attrs={'class': 'form-control'}))


# error_messages={} 提示给前端的错误提示
```

### form 组件设置标签参数

```python
from django.forms import widgets

widget=widgets.TextInput(attrs={'class': 'form-control'})
widget=widgets.PasswordInput(attrs={'class': 'form-control'})

# PasswordInput 标签的 input 属性就为 password
# attrs 给标签设置属性  上述设置了一个 class="form-control" 的属性
```

### forms 组件的局部钩子和全局钩子

#### 后端

```python
from django import forms
from app01 import models
from django.forms import widgets
from django.core.exceptions import ValidationError

class RegisterForm(forms.Form):
    name = forms.CharField(max_length=16, min_length=3, label='用户名',
                           error_messages={'max_length': '最长位16', 'min_length': '最短为3'},
                           widget=widgets.TextInput(attrs={'class': 'form-control'}))
    password = forms.CharField(max_length=16, min_length=3, label='密码', error_messages={'required': '该字段必填'},
                               widget=widgets.PasswordInput(attrs={'class': 'form-control'}))
    re_password = forms.CharField(max_length=16, min_length=3, label='再次确认密码', error_messages={'required': '该字段必填'},
                                  widget=widgets.PasswordInput(attrs={'class': 'form-control'}))
    email = forms.EmailField(label='邮箱', error_messages={'required': '该字段必填', 'invalid': '必须是邮箱格式'},
                             widget=widgets.TextInput(attrs={'class': 'form-control'}))

    def clean_name(self):  # name   字段的局部钩子
        # 校验名字不能以sb开头
        name = self.cleaned_data.get('name')
        if name.startswith('sb'):
            # 校验不通过，必须抛出异常
            raise ValidationError('不能以sb开头')
        else:  # 校验通过返回 name 值
            return name

    def clean(self):  # 全局钩子
        password = self.cleaned_data.get('password')
        re_password = self.cleaned_data.get('re_password')
        if re_password == password:
            return self.cleaned_data  # 校验通过返回检验正确的值
        else:
            raise ValidationError('两次密码不一致')  # 不通过抛出异常


def register(request):
    if request.method == 'GET':
        # 生成一个空form对象
        register_form = RegisterForm()
        return render(request, 'register.html', locals())  # 传入空 register_form 给前端做渲染
    else:
        # 实例化得到对象 传入要校验的值
        register_form = RegisterForm(request.POST)
        if register_form.is_valid():
            print('校验通过')
            print(register_form.cleaned_data)
            # 校验通过 弹出多的 re_password
            register_form.cleaned_data.pop('re_password')
            # 存入数据库 ** 打散存
            models.User.objects.create(**register_form.cleaned_data)
            return HttpResponse('ok')
        else:
            print('校验不通过')
            print(register_form.errors)
            error = register_form.errors.get('__all__')[0]  # 出的错误可能是很多个 取一个给前端提示
            return render(request, 'register.html', locals())
```

#### 前端

```html
<div class="container-fluid">
  <div class="row">
    <div class="col-md-8 col-md-offset-2">
      <h1>form自动渲染 二</h1>
      <form action="" method="post">
        {% for item in register_form %}
        <p>{{ item.label }}: {{ item }}</p>
        <span style="color: red">{{ item.errors.0 }}</span>
        {% endfor %}
        <input type="submit" value="提交" /><span style="color: red"
          >{{ error }}</span
        >
      </form>
    </div>
  </div>
</div>
```

### form 组件大概总结

```python
forms组件
	-数据校验
    -渲染页面
    -错误信息
    -局部全局钩子
  -使用步骤：
    	-写一个类，继承Form类
      -写字段，字段参数（限制该字段的长短）
      -错误信息中文：字段参数
      -widget：控制生成标签的属性
      -视图函数中：
        	-实例化得到form对象时，把要校验的数据传入
          -is_valid():clean_data和errors就有值了
          -如果校验通过就存，不通过就给页面提示
      -渲染页面
    		  -for循环的方式渲染页面（在标签前后可以再加标签）
      -局部钩子
	        -def clean_字段名(self):
        	  -校验规则
            -如果通过，return 值
            -如果不通过，抛异常
      -全局钩子（多个字段校验）
		     -def clean(self):
        	  -如果通过，return clean_data
            -如果不通过，抛异常
```

### forms 组件源码分析

```python
1 为什么局部钩子要写成 clean_字段名，为什么要抛异常
2 入口在 is_valid()
3 校验流程
    -先校验字段自己的规则（最大，最小，是否必填，是不是合法）
    -校验局部钩子函数
    -全局钩子校验


4 流程
    -is_valid()---》return self.is_bound and not self.errors
    -self.errors：方法包装成了数据数据
    	-一旦self._errors有值，就不进行校验了（之前调用过了）
    -self.full_clean()：核心
    	self._errors = ErrorDict()
        if not self.is_bound:
            return
        self.cleaned_data = {}
        self._clean_fields()
        self._clean_form()
        self._post_clean()


    -self._clean_fields()：核心代码,局部钩子执行位置

     value = field.clean(value)# 字段自己的校验规则
     self.cleaned_data[name] = value #把校验后数据放到cleaned_data
     if hasattr(self, 'clean_%s' % name): # 判断有没有局部钩子
        value = getattr(self, 'clean_%s' % name)() #执行局部钩子
        self.cleaned_data[name] = value #校验通过，把数据替换一下
   	# 如果 校验不通过，会抛异常，会被捕获，捕获后执行
    self.add_error(name, e)

  - def _clean_form(self):#全局钩子执行位置
	def _clean_form(self):
        try:
            #如果自己定义的form类中写了clean，他就会执行
            cleaned_data = self.clean()
        except ValidationError as e:
            self.add_error(None, e)
```
