---
title: HTML基础
lastmod: 2021-10-20T16:43:23+08:00
date: 2021-10-20T11:52:03+08:00
tags:
  - HTML
categories:
  - HTML
url: post/html.html
toc: true
---

### 前端与后端

```python
前端：
如何与操作系统打交道的界面都可以称为'前端'
如：手机界面 电脑软件界面 平板界面

后端：
不直接与用户打交道，而是控制核心逻辑的运行

前端三剑客
HTML  网页的骨架(没有样式)
CSS   网页的样式(给骨架美化)
JS    网页的动态效果(丰富用户体验)
```

<!-- more -->

### B/S 架构

```python
B/S架构的本质也是C/S架构
当编写TC服务端的时候，客户端的选择可以是自己写的客户端代码也可以是浏览器充当客户端

之前编写的TCP服务端浏览器无法识别的原因在于每个人编写的服务端不一样，没有统一的格式，浏览器不能自动识别
 	"""
 	浏览器可以访问很多服务端 如何做到无障碍的与这么多不同程序员开发的软件实现数据的交互
 		1.浏览器自身功能强大 自动识别并切换(太过消耗资源)
 		2.大家统一一个与浏览器交互的数据方式(统一思想)
 	"""
```

### HTTP 协议

规定了浏览器与服务端之间数据交互的方式及其他事项，如果开发的服务端不遵守该协议，那么浏览器无法识别服务端的数据，就需要自己编写一个客户端

#### 四大特性

- 基于请求响应

  ```python
  服务端永远不会主动给客户端发消息，必须是客户端先发请求，如果让服务端主动给客户发送消息可以采用其他网络协议
  ```

- 基于 TCP/IP 作用于应用之上的协议

- 无状态

  ```python
  不保存客户端的状态信息
  ```

- 无连接/短连接

  ```python
  两者请求响应之后立刻断绝关系
  ```

#### 数据格式

- 请求格式

  ```python
  请求首行(网络请求的方法)
  请求头(一堆k:v键值对)
  (换行符 不能省略)
  请求体(并不是所有的请求方法都有)
  ```

- 响应格式

  ```python
  响应首行(响应状态码)
  响应头(一堆K:V键值对)
  (换行符 不能省略)
  响应提(即将交给浏览器的数据)
  ```

- 响应状态码

  ```python
  用数字来表示一串中文意思
    1XX:服务端已经接受到了数据正在处理 你可以继续发送数据也可以等待
    2XX:200 OK请求成功 服务端返回了相应的数据
    3XX:重定向(原本想访问A页面 但是自动跳转到了B页面)
    4XX:403没有权限访问  404请求资源不存在
    5XX:服务器内部错误
    """
   	公司还会自定义状态码 一般以10000开头
    参考:聚合数据
    """
  ```

### HTML 前戏

```python
import socket

server = socket.socket()
server.bind(('127.0.0.1', 8080))
server.listen()

"""
请求首行
b'GET / HTTP/1.1\r\n
请求头
Host: 127.0.0.1:8080\r\n
Connection: keep-alive\r\n
sec-ch-ua: " Not;A Brand";v="99", "Google Chrome";v="97", "Chromium";v="97"\r\n
sec-ch-ua-mobile: ?0\r\n
sec-ch-ua-platform: "macOS"\r\n
Upgrade-Insecure-Requests: 1\r\n
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/97.0.4692.71 Safari/537.36\r\n
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9\r\n
Sec-Fetch-Site: none\r\n
Sec-Fetch-Mode: navigate\r\n
Sec-Fetch-User: ?1\r\n
Sec-Fetch-Dest: document\r\n
Accept-Encoding: gzip, deflate, br\r\n
Accept-Language: zh-CN,zh;q=0.9\r\n
\r\n
请求体(当前为空)
'
"""

while True:
    sock, addr = server.accept()
    while True:
        data = sock.recv(1024)
        if len(data) == 0: break
        print(data)
        # 遵循HTTP响应格式
        sock.send(b'HTTP/1.1 200 OK\r\n\r\n')
        sock.send(b'<h1>hello big baby<h1>')
        sock.send(b'<a href="https://www.jd.com">good see<a>')
        sock.send(
            b'<img src="https://www.baidu.com/img/PCtm_d9c8750bed0b3c7d089fa7d55720d6cf.png"/>')
```

### HTML 简介

HTML：超文本标记语言，没有逻辑，只有固定的标记功能

#### 基本文档结构

```html
<html>
  <head>
    浏览器上相关信息
  </head>
  <body>
    主体内容
  </body>
</html>
```

#### HTML 注释

```html
<!--单行注释-->

<!--
多行注释
多行注释
-->
```

### HTML 标签

#### 标签分类

```
1.双标签(有闭合)
  <a></a>
2.自闭合标签(单标签)
  <img/>
```

#### head 内常见标签

```
title  定义网页标题
style  内部支持CSS代码
script 内部支持编写JS代码 还可以通过src属性导入外部js文件
link   通过href属性引入外部的css文件
meta   定义网页源信息
       keywords关键字搜索
       description网页描述信息
```

#### body 内常见标签

| 标签  | 功能       |
| ----- | ---------- |
| h1-h6 | 标题标签   |
| p     | 段落标签   |
| b     | 加粗       |
| i     | 斜体       |
| u     | 下划线     |
| s     | 删除线     |
| br    | 换行       |
| hr    | 水平分割线 |

多种标签有些时候可以实现相同的样式

#### 特殊符

| 符号    | 功能     |
| ------- | -------- |
| \&nbsp; | 空格     |
| \&gt;   | 大于号   |
| \&lt;   | 小于号   |
| \&amp;  | &符号    |
| \&yen;  | RMB 符号 |
| \&copy; | 版权符   |
| \&reg;  | 注册     |

#### 常见标签

`div&span`标签

```
div   块级标签
span  行内标签

标签是可以嵌套的，但是需要遵循以下规律:
1.块级标签,可以无限制的嵌套块儿级标签和行内标签
2.p标签虽然是块级标签但是也不能嵌套块级标签
3.行内标签不能嵌套块儿级标签，可以嵌套行内标签

页面布局的技巧:先用div划分区域，再考虑后面填充具体内容
```

`a`标签

```
链接标签
1.可以通过href属性指定URL进行点击跳转
	跳转的两种方式:
	1)在当前页面跳转  target="_self"   默认的参数
	2)新建标签页跳转  target="_blank"
2.锚点功能
	通过href属性指定标签的id值，然后点击即可跳转到对应的位置
	<a href="#10">跳到id为10的标签位置</a>
```

`img`标签

```
图片标签
src属性指定图片地址，可以是本地地址，也可以是网络地址
alt属性编写文本，用于在图片无法加载时的提示信息
tatle属性编写文本，用于鼠标悬浮在图片上时提示的信息
height、width属性写像素(图片大小)，单位是px，调整一个参数时会等比缩放，同时两个参数可能会失真
```

#### 标签属性

`id`属性

```
个体查找，类似于身份证，在同一个html页面上的id不能重复
<h1 id='10'></h1>
```

`class`属性

```
群体查找，可以将多个标签划分为一类，一个标签同时也可以有多个属性，如：
<h1 class='c1 c2'></h1>
<p class='c1 c3'></p>
```

#### 列表标签

- 无序列表

  ```
  <ul>
  	<li>123</li>
    <li>1234</li>
  	<li>12345</li>
  </ul>

  网页上有规则排列的多个横向或者竖向内容几乎都是使用的无序列表
  ```

- 有序列表

  ```
  <ol>
      <li>111</li>
      <li>222</li>
      <li>333</li>
  	</ol>
  ```

- 标题列表

  ```
  <dl>
    <dt>标题1</dt>
      <dd>内容1</dd>
    <dt>标题2</dt>
      <dd>内容1</dd>
      <dd>内容2</dd>
  </dl>
  ```

#### 表格标签

``` 
<table>
	<thead></thead>  表头
	<tbody></tbody>  表单
</table>

<tr>标签表示一行
<th>标签表头内的字段名称
<td>标签普通的单元格数据

例：
<table border="1">
    <thead>
        <tr>
            <th>姓名</th>
            <th>年龄</th>
            <th>密码</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>jason</td>
            <td>18</td>
            <td>123456</td>
        </tr>
            <tr>
            <td>tom</td>
            <td>28</td>
            <td>123</td>
        </tr>
    </tbody>
</table>


border:      表格边框
cellpadding: 内边距
cellspacing: 外边距
width:       像素 百分比(最好通过css来设置长宽)
rowspan:     单元格竖跨多少行
colspan:     单元格横跨多少列(即合并单元格)
```

#### `form`标签

form 表单的功能是获取用户的数据并发送给后端，例如登录、注册等信息

##### `input`标签

```
input标签是最常用的获取数据标签，该标签为行内标签，可以通过type参数的不同变换不同的表现形式

# type的属性
1) text  普通文本

2) password  密码隐藏

3) date  日期选择

4) radio 单选
         当有多个选项标签需要相同的name属性
         默认选中需要 checked='checked'
         当属性名与属性值相等可以简写checked

5) checkbox  多选，默认选中需要 checked='checked'

6) email  邮箱格式，不满足邮箱格式提交时会提示

7) file 上传文件，默认只支持单个文件，上传多个文件需要 multiple 参数

8) submit 提交按钮，点击时会将表单内的信息进行提交

9) button 普通按钮，本身无任何功能，一般结合JS定制

10) rest 重置按钮

按钮的提示信息可以通过value属性自定义，不自定义的时候不同的浏览器会显示不同的信息
```

##### `select`标签

```
下拉框选项，选项的内容用 option 标签包裹
默认单选，多选可以使用 multiple 参数
```

##### `textarea`标签

```
获取大段文本内容
```

##### 其他注意事项

```
2.直接编写input会出现黄色阴影，原因在于input需要结合lable一起使用
	方式1:lable包裹input并绑定id
    	<label for='input标签id值'>input标签</label>
  方式2:label与input单独出现并绑定id
    	<label for="d1">username:</label>
      <input type="text" id="d1">

3.form表单提交数据
	数据的提交地址由form表单的action参数来控制
  	action="URL"
    	# 不写默认朝当前页面所在的地址提交
    method="数据的提交方式"
    	# 数据的提交方式有很多种 这里先忽略(后续讲解)
      	get post put delete patch...
"""
form表单在提交数据的时候 如果含文件则需要指定两个固定参数
	method='post'
	enctype="multipart/form-data"
"""
```

### 前后端简易交互

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Title</title>
  </head>
  <body>
    <form
      action="http://127.0.0.1:5000/index/"
      method="post"
      enctype="multipart/form-data"
    >
      <p>用户名: <input type="text" name="name" /></p>
      <p>密码: <input type="password" name="passwd" /></p>
      <p>邮箱: <input type="email" name="email" /></p>
      <p>出生日期: <input type="date" name="birthday" /></p>
      <p>
        性别: <input type="radio" name="gender" value="male" />男
        <input type="radio" name="gender" value="female" />女
        <input type="radio" name="gender" value="secrecy" />保密
      </p>
      <p>
        爱好: <input type="checkbox" name="hobby" value="Basketball" /> 篮球
        <input type="checkbox" name="hobby" value="Football" /> 足球
        <input type="checkbox" name="hobby" value="Volleyball" /> 排球
      </p>
      城市:
      <select name="pro">
        <option value="shanghai">上海</option>
        <option value="beijing">北京</option>
        <option value="guangzhou">广州</option>
        <option value="shenzhen">深圳</option>
      </select>
      <p>
        自我评价: <textarea name="info" id="" cols="30" rows="5"></textarea>
      </p>
      <p>
        file:
        <input type="file" name="file" />
      </p>
      <p>
        file:
        <input type="file" name="files" multiple />
      </p>
      <input type="submit" value="注册" />
      <input type="button" value="按钮" />
      <input type="reset" value="重置" />
    </form>
  </body>
</html>
```

![image-20220120145019978](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220120145019978.png)

```python
from flask import Flask,request

app = Flask(__name__)

@app.route('/index/',methods=['GET','POST'])
def index():
    print(request.form)  # 获取普通数据
    print(request.files)  # 获取文件数据
    print(request.form.get('name'))  #  单独获取表单内的值
    file_obj = request.files.get('file')  # 获取文件对象
    file_obj.save('xxx.md')  # 保存文件
    return 'flask框架'


app.run()
```
