---
title: CSS基础
lastmod: 2021-06-03T16:43:23+08:00
date: 2021-06-02T11:52:03+08:00
tags:
  - CSS
categories:
  - CSS
url: post/css-01.html
toc: true
---

### CSS 简介

层叠样式表，给 HTML 标签修改样式

<!-- more -->

```
1.语法结构:
选择器 {
    属性名1:属性值1;
    属性名2:属性值2
}

2.注释语法:
/*这是注释*/

3.引入方式:
	1) style内部直接编写代码(练习使用)
	2) link标签引入外部css文件(正式环境中使用)
	3) 标签内直接编写(不推荐使用，会产生冗余)

4.在编写css文件时注意代码的注释，避免找不到功能给谁用的
```

### 基本选择器

为了能够更好的区分相似的标签，使用使用选择器更快的查找指定的标签

#### 标签选择器

```css
通过标签名直接查找
/*查找所有的div标签*/
div {
  color: red;
}
```

#### 类选择器

```css
通过class值查找标签(关键符号为句点符)  	
/*查找所有含有c1样式类的标签*/
.c1 {
  color: red;
}
```

#### `id`选择器

```css
通过id值查找标签(关键符号为井号#)
/*查找id为d1的标签*/
#d1 {
  color: orange;
}
```

#### 通用选择器

```css
/*body内所有的标签*/
* {
  color: darkgray;
}
```

### 组合选择器

```
为了区分嵌套标签之间的关系 我们发明了一种称呼
	<div>
		<p>
			<span></span>
		</p>
	</div>

span是p的儿子，是div的孙子也可以说是div的后代
p是div的儿子，也是div后代，是span的父亲
div是p的父亲，是span的爷爷，也可以说是他们的祖先
```

#### 后代选择器

```css
特征是 空格
/*查找div内部所有的后代span*/
div span {
  color: red;
}
```

#### 儿子选择器

```css
特征是 >
/*查找div内部所有的儿子span*/
div > span {
  color: greenyellow;
}
```

#### 毗邻选择器

```css
特征是 +
/*查找同级别下面紧挨着的第一个span(不能有其他标签间隔)*/
div + span {
  color: pink;
}
```

#### 弟弟选择器

```css
特征是 ~
/*查找同级别下面所有的span(不需要紧挨着)*/
div ~ span {
  color: deeppink;
}
```

### 属性选择器

```css
/*标签可以有默认的属性也可以自定义属性*/
<p id="d1" class="c1" name="jason" pwd="123" > 123</p > [name] {
  /*查找含有name属性名的标签*/
  color: red;
}

[name="jason"] {
  /*查找含有name属性名并且值为jason的*/
  color: red;
}

p[name="jason"] {
  /*查找含有name属性名并且值为jason的p*/
  color: red;
}
```

### 分组与嵌套

```css
# 多个相同选择器并列使用
div,span,p {  /*查找div或者span或者p*/
            color: red;
        }
# 多个不同选择器并列使用
div,#d1,.c1 {  /*标签查找div id查找d1 类查找c1*/
            color: red;
        }
# 不并列同样可以使用组合选择器
.c1 p {   /*查找class为c1的后代p标签*/
            color: red;
        }
# 直接筛选
	div#d1 {  /*查找id为d1的div标签*/
  	color: red;
  }
  div.c1 {  /查找class为c1的div标签/
    color: red;
  }
```

### 伪选择器

```css
/*鼠标悬浮在上面*/
a:hover {   # 重点掌握 很多网址都在用!!!
            color: orange;
        }
"""a标签默认的颜色会变化 第一次是蓝色 后面是紫色"""


input:focus {
            background-color: red;
        }
"""我们将input框被用户点击即将录入数据的过程看成是focus状态(聚焦状态)"""
```

### 伪元素选择器

```python
# 首字调整>>>:也是一种文档布局的方式
p:first-letter {
            font-size: 48px;  /*字体大小*/
            color: red;
        }
# 在文本的前面通过css动态渲染文本>>>:特殊文本无法选中
p:before {
            content: '嘿嘿';
            color: red;
        }
<p>::before言而有信 品行端正 光明磊落 待人以诚</p>
# 在文本的后面通过css动态渲染文本>>>:特殊文本无法选中
p:after {
            content: '呵呵';
            color: greenyellow;
        }
<p>言而有信 品行端正 光明磊落 待人以诚::after</p>
"""
以后我们在编写爬虫程序爬取页面内容的时候如果没有正常文本
那么可能是因为伪元素选择器的问题
"""
```

### 选择器的优先级

```python
"""
我们学习了三种css引入方式并且学习了很多选择器
那么如果出现多个选择器修改同一个标签样式 会优先参考谁的
	研究基本选择器即可
		标签选择器 类选择器 id选择器 行内选择器
"""
# 相同选择器不同导入方式
	选择器系统遵循就进原则 从上往下谁离标签更近谁说了算
# 不同选择器不遵循就近原则>>>:优先级
	行内选择器 > id选择器 > 类选择器 > 标签选择器
```

### 字体相关

```python
1.宽和高
	只有块儿级标签可以设置 行内标签无法设置
  	p {
            height: 1000px;
            width: 50px;
        }
2.字体大小
	font-size: 99px;  # 字体大小一般有固定的大小参考(肉眼适应)
3.粗细
	font-weight: bolder;
  font-weight: lighter;
4.文本颜色
	color:red;  # 第一种
  color:#4e4e4e;  # 第二种
  color:rgb(88,88,88)  # 第三种
 				rgba(88,88,88,0.2)  # 最后一个参数调整透明度(0-1)
5.文字对齐
	text-align: center;  # 居中展示
6.文字装饰(很常用!!!)
	text-decoration: none;  # 主要用于去除a标签默认的下划线
7.首行缩进
	text-indent: 32px;  # 默认文字大小是16px
```

### 背景属性

```python
background-color: orange;  # 背景颜色
background-image: url('url');  # 背景图片
background-repeat: no-repeat;  # 是否铺满
background-position:左右 上下;  # 图片位置
"""多个属性名前缀相同 那么可以简写"""
background:orange url('url');  # 一个个编写即可 不写就默认

# 如何实时修改图片位置
	浏览器找到标签的css代码 然后方向键上下按住即可动态调整
```

### 边框属性

```python
				p {
            /*border-left-color: red;*/
            /*border-left-style: solid;*/
            /*border-left-width: 3px;*/
            /*多个属性有相同的前缀  一般都可以简写*/
            /*border-left: 5px red  solid;   !*没有顺序*!*/
            /*border-top:orange 10px dotted;*/
            /*border-right: black dashed 5px;*/
            /*border-bottom: deeppink 8px solid;*/
            /*多个属性有相同的前缀  一般都可以简写*/
            border: 5px red solid;  /*上下左右一致*/
        }
        div {
            height: 500px;
            width: 500px;
            border: 5px solid red;
          	/*画圆*/
            border-radius: 50%;
        }
```

### `display`属性

```python
div {
            display: inline;  /*行内*/
}

span {
            /*display: block;  !*块级*!*/
            display: none;
            /*
            隐藏标签 页面上看不见也不再占用页面位置
            但是通过浏览器查找标签是可以看到的
            到后面学习django会讲跨站请求伪造(钓鱼网站)
            */
        }

p {
            display: inline-block;
            /*
            具备块级标签可以修改长宽的特性
            也具备行内标标签文本多大就占多大的特性
            */
        }
```

### 盒子模型

```python
"""
以快递盒为例
	1.快递盒与快递盒之间的距离			外边距(标签之间的距离)
	2.快递盒的厚度								边框
	3.内部物品到盒子的距离				 内边距(文本内容到边框的距离)
	4.物品本身的大小							 文本大小
"""
# body标签默认自带8px的外边距 在编写的时候应该提前去掉
	 body {
            margin: 0;
        }
1.外边距(标签之间的距离)
	margin简写
  	margin:0px;  # 上下左右都一致
    margin:10px 10px;  # 第一个控制上下 第二个控制左右
    margin:20px 10px 20px;  # 上 左右 下
    margin:10px 2px 3px 5px;  # 上 右 下 左
2.内边距(文本内容到边框的距离)
	padding简写
  	padding:0px;  # 上下左右都一致
    padding:10px 10px;  # 第一个控制上下 第二个控制左右
    padding:20px 10px 20px;  # 上 左右 下
    padding:10px 2px 3px 5px;  # 上 右 下 左
```

### `fload`浮动

```python
CSS中，任何元素都可以浮动，浮动元素会生成一个块级框，而不论它本身是何种元素，主要用于页面布局

特点：
  1·浮动的框可以向左或向右移动，直到它的外边缘碰到包含框或另一个浮动框的边框为止
  2.由于浮动框不在文档的普通流中，所以文档的普通流中的块框表现得就像浮动框不存在一样

取值:
  left: 向左浮动
  right: 向右浮动
  none: 默认值，不浮动

浮动会造成父标签塌陷问题,如下例
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        body {
            margin: 0;
        }
        #d1 {
            border: 3px solid black;
        }
        #d2 {
            background-color: red;
            height: 80px;
            width: 80px;
            float: left;
        }
        #d3 {
            background-color: greenyellow;
            height: 80px;
            width: 80px;
            float: left;
        }
    </style>
</head>
<body>
<div id="d1">
    <div id="d2"></div>
    <div id="d3"></div>
</div>
</body>
</html>

解决办法:
  1.可以再写一个div撑出
  2.关键字clear
  3.通用解决策略,只要父标签塌陷就使用
      .clearfix:after {
            content: '';
            clear: both;
            display: block;
        }
    上述中在id="d1"后加class="clearfix"属性即可

浏览器默认文本优先展示(出现塌陷时)
```

### 定位

```python
1.静态定位	static
	所有的标签默认都是静态定位即不能改变位置
2.相对定位	relative
	相对标签原来的位置做定位
3.绝对定位  absolute
	相对已经定位过的父标签做定位(没有则参考body标签)
    	eg:小米官网导航条内购物车
4.固定定位  fixed
	相对浏览器窗口做定位
    	eg:小米官网右边回到顶部

如何使用css完成定位
	定位关键字position
    位置关键字left、right、top、bottom
```

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>相对定位</title>
    <style>
      body {
        margin: 0;
      }
      #d1 {
        height: 100px;
        width: 100px;
        background-color: red;
        left: 50px; /*从左往右 负数方向相反*/
        top: 50px; /*从上往下 负数方向相反*/
        position: relative; /* 默认是static 不能移动 */
        /* relative 将不是定位的标签改成定位的标签 */
      }
    </style>
  </head>
  <body>
    <div id="d1"></div>
  </body>
</html>
```

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>绝对定位</title>
    <style>
      body {
        margin: 0;
      }
      #d2 {
        height: 100px;
        width: 200px;
        background-color: red;
        position: relative;
      }
      #d3 {
        height: 200px;
        width: 400px;
        background-color: green;
        position: absolute;
        left: 200px;
        top: 100px;
      }
    </style>
  </head>
  <body>
    <div id="d2">
      <div id="d3"></div>
    </div>
  </body>
</html>
```

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>绝对定位</title>
    <style>
      body {
        margin: 0;
      }
      #d4 {
        position: fixed;
        bottom: 10px;
        right: 20px;
        height: 50px;
        width: 100px;
        background-color: white;
        border: 3px solid black;
      }
    </style>
  </head>
  <body>
    <div style="height: 500px;background-color: red"></div>
    <div style="height: 500px;background-color: greenyellow"></div>
    <div style="height: 500px;background-color: blue"></div>
    <div id="d4">回到顶部</div>
  </body>
</html>
```

### 是否脱离文档流

```python
# 标签位置改变之后 原来的位置是否会空出来
	如果空出来了被其他标签自动占有 那么表示脱离否则不脱离

脱离文档流
   浮动、绝对定位、固定定位
不脱离文档流
   相对定位
```

```html
<!--浮动-->
<div
  style="height: 100px;width: 100px;background-color: red;float: right"
></div>
<div style="height: 100px;width: 100px;background-color: greenyellow;"></div>
```

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>相对定位</title>
  </head>
  <body>
    <div
      style="height: 100px;width: 100px;background-color: red;position: relative;left: 300px"
    ></div>
    <div style="height: 100px;width: 100px;background-color: greenyellow"></div>
  </body>
</html>
```

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>绝对定位</title>
  </head>
  <body>
    <div style="height: 100px;width: 100px;background-color: red;"></div>
    <div
      style="height: 100px;width: 100px;background-color: yellow;position: absolute;left: 300px"
    ></div>
    <!--当没有父标签做定位，此时参考 body -->
    <div style="height: 100px;width: 100px;background-color: blue;"></div>
  </body>
</html>
```

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>固定定位</title>
  </head>
  <body>
    <div style="height: 100px;width: 100px;background-color: red;"></div>
    <div
      style="height: 100px;width: 100px;background-color: yellow;position: fixed;bottom: 30px;right: 300px"
    ></div>
    <div style="height: 100px;width: 100px;background-color: blue;"></div>
  </body>
</html>
```

### 溢出属性

```python
# 圆形头像

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        #d1 {
            height: 100px;
            width: 100px;
            border: 3px solid blue;
            border-radius: 50%;
            overflow: hidden;
        }
        #d1 img {
            width: 100%;
        }
    </style>
</head>
<body>
<div id="d1">
    <img src="111.jpeg" alt="">
</div>
</body>
</html>
```

### `z-index`属性

```python
# 浏览器平面不是一个二维坐标系而是一个三维坐标系
eg:百度登录或者退出界面>>>:三明治结构(模态框)
    1.最底部是正常内容(z=0)  最远
    2.黑色的透明区(z=99)    中间
    3.白色的注册区域(z=100)  离用户最近
```

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Title</title>
    <style>
      body {
        margin: 0;
      }
      .cover {
        position: fixed;
        left: 0;
        top: 0;
        right: 0;
        bottom: 0;
        background-color: rgba(0, 0, 0, 0.5);
        z-index: 99;
      }
      .modal {
        background-color: white;
        width: 400px;
        height: 200px;
        position: fixed;
        left: 50%;
        top: 50%;
        z-index: 100;
        margin-left: -200px;
        margin-top: -100px;
      }
    </style>
  </head>
  <body>
    <div>这是底层内容</div>
    <div class="cover"></div>
    <div class="modal">
      <h1>登录页面</h1>
      <p>username: <input type="text" /></p>
      <p>password: <input type="text" /></p>
      <button>点点点</button>
    </div>
  </body>
</html>
```

### 透明度

```python
rgba(124,124,124,0.5)
	只影响颜色
opacity:0.5
    影响颜色和字体
```

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Title</title>
    <style>
      #d1 {
        background-color: rgba(0, 0, 0, 0.5);
      }
      #d2 {
        opacity: 0.5;
        color: blue;
      }
    </style>
  </head>
  <body>
    <p id="d1">111</p>
    <p id="d2">222</p>
  </body>
</html>
```

### 博客园首页案例

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Title</title>
    <link rel="stylesheet" href="blog.css" />
  </head>
  <body>
    <div class="blog-left">
      <div class="left-head">
        <img src="111.jpeg" alt="" />
      </div>
      <div class="left-name">
        <span>Xxx的博客</span>
      </div>
      <div class="left-autograph">
        <span>这个家伙很懒，什么都没留下</span>
      </div>
      <div class="left-info">
        <ul>
          <li><a href="#">关于我</a></li>
          <li><a href="#">微博</a></li>
          <li><a href="#">微信公众号</a></li>
        </ul>
      </div>
      <div class="left-label">
        <ul>
          <li><a href="#">#Python</a></li>
          <li><a href="#">#Golang</a></li>
          <li><a href="#">#Linux</a></li>
        </ul>
      </div>
    </div>
    <div class="blog-right">
      <div class="right-article">
        <div class="article-date">
          <span>2022/2/7</span>
        </div>
        <div class="article-title">
          <span>这是一个标题</span>
        </div>
        <div class="article-abstract">
          <span>这是一片很重要的文章，详细看看....</span>
        </div>
        <div class="article-label">
          <span>#Python&nbsp&nbsp#JavaScript</span>
        </div>
      </div>
      <div class="right-article">
        <div class="article-date">
          <span>2022/2/7</span>
        </div>
        <div class="article-title">
          <span>这是一个标题</span>
        </div>
        <div class="article-abstract">
          <span>这是一片很重要的文章，详细看看....</span>
        </div>
        <div class="article-label">
          <span>#Python&nbsp&nbsp#JavaScript</span>
        </div>
      </div>
      <div class="right-article">
        <div class="article-date">
          <span>2022/2/7</span>
        </div>
        <div class="article-title">
          <span>这是一个标题</span>
        </div>
        <div class="article-abstract">
          <span>这是一片很重要的文章，详细看看....</span>
        </div>
        <div class="article-label">
          <span>#Python&nbsp&nbsp#JavaScript</span>
        </div>
      </div>
      <div class="right-article">
        <div class="article-date">
          <span>2022/2/7</span>
        </div>
        <div class="article-title">
          <span>这是一个标题</span>
        </div>
        <div class="article-abstract">
          <span>这是一片很重要的文章，详细看看....</span>
        </div>
        <div class="article-label">
          <span>#Python&nbsp&nbsp#JavaScript</span>
        </div>
      </div>
      <div class="right-article">
        <div class="article-date">
          <span>2022/2/7</span>
        </div>
        <div class="article-title">
          <span>这是一个标题</span>
        </div>
        <div class="article-abstract">
          <span>这是一片很重要的文章，详细看看....</span>
        </div>
        <div class="article-label">
          <span>#Python&nbsp&nbsp#JavaScript</span>
        </div>
      </div>
      <div class="right-article">
        <div class="article-date">
          <span>2022/2/7</span>
        </div>
        <div class="article-title">
          <span>这是一个标题</span>
        </div>
        <div class="article-abstract">
          <span>这是一片很重要的文章，详细看看....</span>
        </div>
        <div class="article-label">
          <span>#Python&nbsp&nbsp#JavaScript</span>
        </div>
      </div>
      <div class="right-article">
        <div class="article-date">
          <span>2022/2/7</span>
        </div>
        <div class="article-title">
          <span>这是一个标题</span>
        </div>
        <div class="article-abstract">
          <span>这是一片很重要的文章，详细看看....</span>
        </div>
        <div class="article-label">
          <span>#Python&nbsp&nbsp#JavaScript</span>
        </div>
      </div>
      <div class="right-article">
        <div class="article-date">
          <span>2022/2/7</span>
        </div>
        <div class="article-title">
          <span>这是一个标题</span>
        </div>
        <div class="article-abstract">
          <span>这是一片很重要的文章，详细看看....</span>
        </div>
        <div class="article-label">
          <span>#Python&nbsp&nbsp#JavaScript</span>
        </div>
      </div>
    </div>
  </body>
</html>
```

```css
/*这是博客园的css样式文件*/
/*通用样式*/
body {
  margin: 0;
  background-color: beige;
}

a {
  text-decoration: none;
  color: #b0b0b0;
}

a:hover {
  color: white;
}

ul {
  list-style-type: none;
  padding-left: 0;
}

/*左侧样式*/
.blog-left {
  background-color: rgb(78, 78, 78);
  width: 20%;
  height: 1000px;
  position: fixed;
  float: left;
}

/*头像设置*/
.left-head {
  height: 150px;
  width: 150px;
  border: 3px solid white;
  border-radius: 50%;
  overflow: hidden;
  margin: 10px auto;
}

.left-head img {
  width: 100%;
}

.left-name,
.left-autograph {
  text-align: center;
  font-size: 10px;
  color: darkgray;
  margin: 10px;
}

.left-info,
.left-label {
  text-align: center;
  font-size: 16px;
  margin: 60px;
}

/*右侧样式*/
.blog-right {
  float: right;
  width: 80%;
}

.right-article {
  margin: 20px 40px 20px 10px;
  background-color: white;
  box-shadow: 5px 5px 5px rgba(0, 0, 0, 0.5);
  border-radius: 8px;
}

.article-title {
  font-size: 24px;
  border-left: 5px solid red;
  padding-left: 14px;
}

.article-date {
  float: right;
  font-size: 14px;
  margin: 10px;
}

.article-abstract {
  margin: 10px;
  border-bottom: 1px solid black;
  padding-bottom: 6px;
}

.article-label {
  font-size: 14px;
  padding-left: 10px;
}
```
