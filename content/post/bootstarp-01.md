---
title: Bootstrap框架
lastmod: 2021-06-03T16:43:23+08:00
date: 2021-06-02T11:52:03+08:00
tags:
  - Bootstrap
categories:
  - Bootstrap
url: post/bootstrap.html
toc: true
---

主要查看官方文档: https://v3.bootcss.com/

### 简介

<!-- more -->

```python
Bootstrap 是最受欢迎的 HTML、CSS 和 JS 框架，用于开发响应式布局、移动设备优先的 WEB 项目
Bootstrap框架版本有 2.x 3.x 4.x # 推荐使用3.x
3.x版本文档: https://v3.bootcss.com/

使用框架调整页面样式一般都是操作标签的class属性
bootstarp需要依赖jQuery才能正常执行

引入方式:
  1.本地引入
    Bootstrap下载地址: https://github.com/twbs/bootstrap/releases/download/v3.4.1/bootstrap-3.4.1-dist.zip
    先导入jQuery文件
    导入Bootstrap的css文件
    导入Bootstrap的js文件

    <script src="jQuery-3.6.0.js"></script>
    <link rel="stylesheet" href="bootstrap-3.4.1-dist/css/bootstrap.css">
    <script src="bootstrap-3.4.1-dist/js/bootstrap.min.js"></script>

  2.CDN引入
    引入jQuery CND
    引入Bootstrap的css CDN
    引入Bootstrap的js  CDN

    <script src="https://cdn.bootcdn.net/ajax/libs/jquery/3.4.1/jquery.min.js"></script>
    <link href="https://cdn.bootcdn.net/ajax/libs/twitter-bootstrap/3.4.1/css/bootstrap.min.css" rel="stylesheet">
    <script src="https://cdn.bootcdn.net/ajax/libs/twitter-bootstrap/3.4.1/js/bootstrap.min.js"></script>

PS: pycharm中第一次最好使用本地导入，可以有代码提示

Normalize.css:增强跨浏览器渲染的一致性
  https://necolas.github.io/normalize.css/
```

### 布局容器

```python
container 左右留白
container-fluid 左右不留白


<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="jQuery-3.6.0.js"></script>
    <link rel="stylesheet" href="bootstrap-3.4.1-dist/css/bootstrap.css">
    <script src="bootstrap-3.4.1-dist/js/bootstrap.min.js"></script>
    <style>
        .c1 {
            background-color: red;
            height: 100px;
        }
    </style>
</head>
<body>
    <div class="container c1"></div>
</body>
</html>
```

### 栅格系统

```python
Bootstrap 提供了一套响应式、移动设备优先的流式栅格系统，随着屏幕或视口（viewport）尺寸的增加，系统会自动分为最多12列。它包含了易于使用的预定义类，还有强大的mixin 用于生成更具语义的布局

row 行

一行中占几份，添加下列所有自适应显示
col-xs-1
col-sm-1
col-md-1
col-lg-1
```

以下代码可以看到被分成了 12 份

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Title</title>
    <script src="jQuery-3.6.0.js"></script>
    <link rel="stylesheet" href="bootstrap-3.4.1-dist/css/bootstrap.css" />
    <script src="bootstrap-3.4.1-dist/js/bootstrap.min.js"></script>
    <style>
      .c1 {
        background-color: red;
        height: 100px;
        border: 3px solid black;
      }
    </style>
  </head>
  <body>
    <div class="container">
      <div class="row">
        <div class="col-md-1 c1"></div>
        <div class="col-md-1 c1"></div>
        <div class="col-md-1 c1"></div>
        <div class="col-md-1 c1"></div>
        <div class="col-md-1 c1"></div>
        <div class="col-md-1 c1"></div>
        <div class="col-md-1 c1"></div>
        <div class="col-md-1 c1"></div>
        <div class="col-md-1 c1"></div>
        <div class="col-md-1 c1"></div>
        <div class="col-md-1 c1"></div>
        <div class="col-md-1 c1"></div>
      </div>
    </div>
  </body>
</html>
```

### 表格

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
    <div class="container table-responsive">
      <table
        class="table table-striped table-bordered table-hover table table-condensed"
      >
        <tr>
          <th>ID</th>
          <th>UserName</th>
          <th>PassWord</th>
        </tr>
        <tr class="danger">
          <td>1</td>
          <td>tony</td>
          <td>123</td>
        </tr>
        <tr class="success">
          <td>2</td>
          <td>jason</td>
          <td>313</td>
        </tr>
        <tr class="warning">
          <td>3</td>
          <td>xxx</td>
          <td>007</td>
        </tr>
        <tr class="info">
          <td>4</td>
          <td>zzz</td>
          <td>800</td>
        </tr>
      </table>
    </div>
  </body>
</html>
```

### 表单

`class=form-control`

checkbox 和 radio 最好加，否则样式会更难看

```html
<div class="container">
  <div class="col-md-8 col-md-offset-2">
    <h2>登录页面</h2>
    <form action="">
      <p>
        username:
        <input type="text" class="form-control" />
      </p>
      <p>
        password:
        <input type="text" class="form-control" />
      </p>
      <p>
        <select name="" id="" class="form-control">
          <option value="">111</option>
          <option value="">222</option>
          <option value="">333</option>
        </select>
      </p>
      <textarea
        name=""
        id=""
        cols="30"
        rows="10"
        class="form-control"
      ></textarea>
      <input type="submit" />
    </form>
  </div>
</div>
```

### 按钮组

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
    <a href="https://www.baidu.com" class="btn btn-primary btn-block">点点点</a>
    <button class="btn btn-danger btn-lg">这是按钮</button>
    <button class="btn btn-default btn-sm">这是按钮</button>
    <button class="btn btn-success btn-xs">这是按钮</button>
    <button class="btn btn-info">这是按钮</button>
    <button class="btn btn-warning">这是按钮</button>
  </body>
</html>
```

### 图片

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
    <div class="container">
      <div class="row">
        <div class="col-md-8 col-md-offset-2">
          <img
            src="https://img0.baidu.com/it/u=2777952672,1831955507&fm=253&fmt=auto&app=138&f=JPEG?w=500&h=313"
            alt=""
            class="img-rounded"
          />
          <img
            src="https://img0.baidu.com/it/u=2777952672,1831955507&fm=253&fmt=auto&app=138&f=JPEG?w=500&h=313"
            alt=""
            class="img-circle"
          />
          <img
            src="https://img0.baidu.com/it/u=2777952672,1831955507&fm=253&fmt=auto&app=138&f=JPEG?w=500&h=313"
            alt=""
            class="img-thumbnail"
          />
        </div>
      </div>
    </div>
  </body>
</html>
```

### 图标

```python
1.bootstrap自带的Glyphicons 字体图标
通过span标签修改class属性值
<span class="glyphicon glyphicon-user" style="color: red;"></span>

2.fontawesome网站

  官网: http://www.fontawesome.com.cn/

  文件引入方式: <link rel="stylesheet" href="font-awesome-4.7.0/css/font-awesome.min.css">

  使用: <i class="fa fa-university" aria-hidden="true"></i>

3. 两者之间完全兼容
```
