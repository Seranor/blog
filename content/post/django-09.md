---
title: Django与Ajax
lastmod: 2021-09-03T16:43:23+08:00
date: 2021-09-02T11:52:03+08:00
tags:
  - Django
categories:
  - Django
url: post/django-09.html
toc: true
---

### Django 与 Ajax

#### 简介

<!-- more -->

```python
概念: Ajax(Asynchronous Javascript And XML) 异步JavaScript和XML，即使用JavaScript语言与服务器进行异步交互，传输的数据为XML(传输数据不止XML，更多的是JSON数据)

特性:
  异步
  局部刷新: js的DOM操作，使页面局部刷新
  使用广泛

方式:
  1.使用原生JavaScript写Ajax请求
  	较为麻烦，因此很少人使用该方式，区分浏览器，需要做浏览器兼容

  2.使用封装的(jQuery，AxioA...)
```

#### 场景

![image-20220301192338140](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220301192338140.png)

#### 执行流程

![img](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/007S8ZIlgy1gj1x4mpne9j30pk08yq39.jpg)

#### 案例

##### 后端

```python
def ajax_test(request):
    return render(request, 'ajax_test.html')

def sum_a(request):
    import time
    time.sleep(2)
    a1 = int(request.GET.get('a1'))
    a2 = int(request.GET.get('a2'))
    return HttpResponse(a1 + a2)
```

##### 前端

```html
<body>
  <input type="text" id="a1" /> + <input type="text" id="a2" /> =
  <input type="text" id="sum_a" />
  <button id="btn_submit">计算</button>
</body>
<script>
  $("#btn_submit").click(function () {
    var a1 = $("#a1").val();
    var a2 = $("#a2").val();
    $.ajax({
      url: "/sum_a/",
      method: "get",
      data: { a1: a1, a2: a2 },
      success: function (data) {
        console.log(data);
        $("#sum_a").val(data);
      },
    });
  });
</script>
```

### Ajax 发送其他请求

```python
注意点:
  1.如果form表单中，写button和input(submit)的标签，会触发form表单的提交，这样会刷新掉Ajax的提交
    解决方式:
    1) 不写在form表单中
    2) 使用input(button)类型标签

  2.后端响应格式是html/text格式，Ajax接收到数据后需要自己转成对象类型
    后端响应格式是json时，Ajax接收到数据后会自动转成对象
    后端返回数据，统一使用JsonResponse

  3.如果使用了Ajax，后端就不要返回rediret render HttpResponse
    直接返回JsonResponse
```

#### 前端

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Title</title>
    <script src="https://cdn.bootcdn.net/ajax/libs/jquery/3.4.1/jquery.min.js"></script>
  </head>
  <body>
    <p>用户名：<input type="text" id="id_name" /></p>
    <p>密码：<input type="password" id="id_password" /></p>
    <button id="id_btn">提交</button>
    <span class="error" style="color: red"></span>
  </body>
  <script>
    $("#id_btn").click(function () {
      $.ajax({
        url: "/login/",
        method: "post",
        data: {
          username: $("#id_name").val(),
          password: $("#id_password").val(),
        },
        success: function (data) {
          //res=JSON.parse(data)  当返回的是json格式化处理的数据需要转为对象类型
          //console.log(data)
          //console.log(res)
          // data 现在是对象类型
          if (data.status == 100) {
            //登录成功，重定向到百度，前端重定向
            location.href = "http://www.baidu.com";
            //location.href='/index/'
          } else {
            //登录失败
            //$('.error').html(data.msg).css({'color':'red'})
            $(".error").html(data.msg);
          }
        },
        error: function (data) {
          console.log(data);
        },
      });
    });
  </script>
</html>
```

#### 后端

```python
from django.shortcuts import render,redirect,HttpResponse
from django.http import JsonResponse
from app01 import models
import json

def login(request):
    if request.method=='GET':
        return render(request,'login.html')
    # elif request.method=='POST':
    elif request.is_ajax():
        response={'status':100,'msg':None}
        name=request.POST.get('username')
        password=request.POST.get('password')
        user=models.User.objects.filter(name=name,password=password).first()
        if user:
            # 用户名和密码都对了
            # return redirect('')  出错
            response['msg']="登录成功"
        else:
            response['status']=101
            response['msg'] = "用户名或密码错误"
        # return  HttpResponse(json.dumps(response))
        # 可以返回该类型，但是在Ajax显示的是字符串，而不是对象类型，Ajax端需要单独处理才行

        return  JsonResponse(response)  # 只能返回JsonResponse类型
        # JsonResponse 对比使用 json.dumps 会将数据单独处理为 application/json 类型

        # return redirect('http://www.baidu.com')
        # return render(request,'login.html')
```

![image-20220302091527474](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220302091527474.png)

![image-20220302091649146](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220302091649146.png)

### 上传文件

```python
http的post请求有三种主流的编码格式
  urlencoded  默认的                可以从request.POST取到提交的数据
  form-data   上传文件的             可以从request.FILES中提取文件
  json        Ajax发送json格式数据   无法从request.POST取到提交的数据

使用Ajax和form表单默认都是urlencoded格式
    上传文件时:
    form表单指定格式 enctype="multipart/form-data"
    Ajax需要使用 Formdat a对象

如果编码是urlencoded格式，放入到body体中的数据格式如下
    username=asd&password=123

如果是formdata编码格式，body体中的将会是两部分，数据和文件二进制
如果是json格式body体中的格式是 json格式字符串，这种格式request.POST取不到值
```

#### `form`表单上传文件

##### 前端

```html
<h1>form表单上传文件</h1>
<form action="" method="post" enctype="multipart/form-data">
  <p>用户名：<input type="text" name="name" /></p>
  <p>文件：<input type="file" name="myfile" /></p>
  <input type="submit" value="提交" />
</form>
```

##### 后端

```python
def file_upload(request):
    if request.method == 'GET':
        return render(request, 'file_upload.html')
    else:
        name = request.POST.get('username')
        print(name)
        myfile = request.FILES.get('myfile')
        with open(myfile.name, 'wb') as f:
            for line in myfile:
                f.write(line)
        return HttpResponse('上传成功')
```

#### `Ajax`上传文件

##### 前端

```html
<h1>ajax上传文件</h1>
<p>用户名：<input type="text" id="id_name" /></p>
<p>文件：<input type="file" id="id_myfile" /></p>
<button id="id_btn">提交</button>
<script>
  $("#id_btn").click(function () {
    //如果要上传文件，需要借助于一个js的FormData对象
    var formdata = new FormData(); //实例化得到一个FormData对象
    formdata.append("username", $("#id_name").val()); //追加了一个name对应填入的值
    //能追加文件
    var file = $("#id_myfile")[0].files[0]; //原生js取到文件
    formdata.append("myfile", file);
    $.ajax({
      url: "file_upload",
      method: "post",
      //上传文件，一定要注意如下两行
      processData: false, //不预处理数据，
      contentType: false, //不指定编码格式，使用formdata对象的默认编码就是formdata格式
      data: formdata,
      success: function (data) {
        console.log(data);
      },
    });
  });
</script>
```

##### 后端

```python
def file_upload(request):
    if request.method == 'GET':
        return render(request, 'file_upload.html')
    else:
        name = request.POST.get('username')
        print(name)
        myfile = request.FILES.get('myfile')
        with open(myfile.name, 'wb') as f:
            for line in myfile:
                f.write(line)
        return HttpResponse('上传成功')
```

### Ajax 上传 JSON 格式

#### 前端

```html
<body>
  <h1>Ajax提交json格式</h1>
  <p>用户名: <input type="text" id="id_name" /></p>
  <p>密码: <input type="password" id="id_password" /></p>
  <button id="id_btn">提交</button>
</body>
<script>
  $("#id_btn").click(function () {
    var name = $("#id_name").val();
    var password = $("#id_password").val();
    $.ajax({
      url: "/ajax_json/",
      method: "post",
      contentType: "application/json", //指定编码格式
      data: JSON.stringify({ name: name, password: password }), //JSON格式字符串
      success: function (data) {
        console.log(data);
      },
    });
  });
</script>
```

#### 后端

```python
def ajax_json(request):
    if request.method == 'GET':
        return render(request, 'ajax_json.html')
    else:
        # json格式，从POST中取不出
        # name = request.POST.get('name')
        request.data = json.loads(request.body)
        name = request.data.get('name')  # 在body体中，bytes格式
        password = request.data.get('password')
        print(name)
        print(password)
        return HttpResponse('ok')
```

### Django 内置序列化

```python
把对象转成json格式，django内置的不好用，字段不能控制
如果要做序列化，使用for循环拼列表套字典
```

```python
def test(request):
    user_list = models.User.objects.all()
    # res = serializers.serialize("json", user_list)
    # return HttpResponse(res)

    l=[]
    for user in user_list:
      l.append({'name':user.name,'password':user.password})
    return JsonResponse(l,safe=False)
```
