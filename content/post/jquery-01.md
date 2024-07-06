---
title: jQuery基础
lastmod: 2021-09-10T16:43:23+08:00
date: 2021-09-10T11:52:03+08:00
tags:
  - jQuery
categories:
  - jQuery
url: post/jquery.html
toc: true
---

## jQuery 介绍

1. jQuery 是一个轻量级的、快速简洁的、兼容多浏览器的 JavaScript 库。

2. jQuery 使用户能够更方便地处理 HTML Document、Events、实现动画效果、方便地进行 Ajax 交互，能够极大地简化 JavaScript 编程。它的宗旨就是：“Write less, do more.“（让你用更少的代码完成更多的事情）

   ![image-20210313141736082](https://gitee.com/gengff/blogimage/raw/master/images/image-20210313141736082.png)
   <!-- more -->

## jQuery 的优势

1. 一款轻量级的 JS 框架。jQuery 核心 js 文件才几十 kb，不会影响页面加载速度。
2. 丰富的 DOM 选择器,jQuery 的选择器用起来很方便，比如要找到某个 DOM 对象的相邻元素，JS 可能要写好几行代码，而 jQuery 一行代码就搞定了，再比如要将一个表格的隔行变色，jQuery 也是一行代码搞定。
3. 链式表达式。jQuery 的链式操作可以把多个操作写在一行代码里，更加简洁。
4. 事件、样式、动画支持。jQuery 还简化了 js 操作 css 的代码，并且代码的可读性也比 js 要强。
5. Ajax 操作支持。jQuery 简化了 AJAX 操作，后端只需返回一个 JSON 格式的字符串就能完成与前端的通信。
6. 跨浏览器兼容。jQuery 基本兼容了现在主流的浏览器，不用再为浏览器的兼容问题而伤透脑筋。
7. 插件扩展开发。jQuery 有着丰富的第三方的插件，例如：树形菜单、日期控件、图片切换插件、弹出窗口等等基本前端页面上的组件都有对应插件，并且用 jQuery 插件做出来的效果很炫，并且可以根据自己需要去改写和封装插件，简单实用。

#### 针对导入问题

下载链接：[jQuery 官网](https://jquery.com/)

前端免费的 CDN 网站：[CDN 加速服务](https://www.bootcdn.cn/)

```
1.文件下载到了本地，如何解决多个文件反复书写引入语句的代码
  可以借助与pycharm自动初始化代码功能完成自动添加

2.直接引入JQuery提供的CDN服务（基于网络直接请求加载）
  CDN:内容分发网络
<script src="https://cdn.bootcdn.net/ajax/libs/jquery/3.4.1/jquery.min.js"></script>
  导入步骤如下：
```

![image-20210313135547190](https://gitee.com/gengff/blogimage/raw/master/images/image-20210313135547190.png)

## jQuery 对象

通过 jQuery 方法得到的标签对象称之为**jQuery 对象**对象。**jQuery 对象**是 jQuery 独有的。如果一个对象是 **jQuery 对象**，那么它就可以使用**jQuery**里的方法：

例如**$(“#d1”)**。

**$("#d1")**的意思是：获取 id 值为 **d1** 的元素的 html 代码。是 jQuery 里的方法。

相当于**JS 对象**：**document.getElementById("d1");**

**jQuery 对象**是包装**原生 JS 对象**后产生的，但是**jQuery 对象**无法使用**JS 对象**的任何方法，同理**JS 对象**也不能使用**jQuery**里的方法。

**jQuery 对象**与**原生 JS 对象**之间的关系：它们两个之间虽然彼此不能调用彼此的方法，但是它们两个之间是有联系的

**jQuery 对象**相当于一个数组，里面是一个个的**原生 JS 对象**

```
$("#d1") //jQuery对象
document.getElementById("d1") //原生JS对象
$("#d1")[0] // 原生JS对象 -- 重点：注意两个对象直接的转换，容易混淆
```

拿上面那个例子举例，如何把原生 JS 对象转成 jQuery 对象：

```
$($("#d1")[0]）//用此类方法把原生JS对象包起来得到的就是jQuery对象
```

## jQuery 基础语法

支持链式操作；

在变量前加"$"符号（var $variable = jQuery 对象）；

注：此规定并不是强制要求。

```
$(selector).action()
```

## 查找标签

### 基本选择器

**id 选择器：**

```
$("#id")
```

**标签选择器：**

```
$("div")
```

**class 选择器：**

```
$(".c1")
```

**配合使用：**

```
$("div.c1")  // 找到class类等于c1的div标签
$("div#d1")  // 找到id等于d1的div标签
```

**所有元素选择器：**

```
$("*")
```

**组合选择器：**

```
$("#id, .className, tagName")
```

### **层级选择器：**

_x 和 y 可以为任意选择器_

```
$("x y");// x的所有后代y（子子孙孙）
$("x > y");// x的所有儿子y（儿子）
$("x + y")// 找到所有紧挨在x后面的y
$("x ~ y")// x之后所有的兄弟y
```

### **基本筛选器：**

```
:first // 第一个
:last // 最后一个
:eq(index)// 索引等于index的那个元素
:even // 匹配所有索引值为偶数的元素，从 0 开始计数
:odd // 匹配所有索引值为奇数的元素，从 0 开始计数
:gt(index)// 匹配所有大于给定索引值的元素
:lt(index)// 匹配所有小于给定索引值的元素
:not(元素选择器)// 移除所有满足not条件的标签
:has(元素选择器)// 选取所有包含一个或多个标签在其内的标签(指的是从后代元素找)

# 例子：
$("div:has(h1)")// 找到所有后代中有h1标签的div标签
$("div:has(.c1)")// 找到所有后代中有c1样式类的div标签
$("li:not(.c1)")// 找到所有不包含c1样式类的li标签
$("li:not(:has(a))")// 找到所有后代中不含a标签的li标签
```

```
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Document</title>
    <style>
        * {
            margin: 0;
            padding: 0;
        }
        html,body {
            height: 100%;
        }

        #bg {
            position: absolute;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            background-color: rgba(0,0,0,0.3);
            display: none;
        }
        #content {
            position: absolute;
            top: 100px;
            left: 50%;
            margin-left: -150px;
            background-color: white;
            width: 300px;
            height: 200px;

        }
        #content p:nth-child(3) {
            position: absolute;
            top: 100px;
        }


    </style>
</head>
<body>
<input type="button" value="弹出" id="btn">
<div id="bg">
    <div id="content">
        <p>
            <label for="inp-username">用户名: </label><input type="text" name="username" id="inp-username">
        </p>
        <p>
            <label for="inp-password">密码: </label><input type="text" name="username" id="inp-password">
        </p>
        <p>
            <input type="button" value="提交" >
            <input type="button" value="取消" id="cancel">
        </p>
    </div>
</div>

<script src="https://cdn.bootcss.com/jquery/3.3.1/jquery.js"></script>
<script>

    $("#btn")[0].onclick=function () {
        $("#bg").css("display","block")
    }

    $("#cancel")[0].onclick=function () {
        $("#inp-username").val("")
        $("#inp-password").val("")
        $("#bg").css("display","none")
    }
</script>

</body>
</html>

JQ版自定义模态框
```

### **属性选择器：**

```
[attribute]
[attribute=value]// 属性等于
[attribute!=value]// 属性不等于
```

**例子：**

```
// 示例
<input type="text">
<input type="password">
<input type="checkbox">
$("input[type='checkbox']");// 取到checkbox类型的input标签
$("input[type!='text']");// 取到类型不是text的input标签
```

### **表单筛选器**：

```
:text
:password:file
:radio
:checkbox
:submit
:reset
:button
```

**例子：**

```
$(":checkbox")  // 找到所有的checkbox
```

表单对象属性:

```
:enabled
:disabled
:checked
:selected
```

**例子：**

找到可用的 input 标签

```
<form>
  <input name="email" disabled="disabled" />
  <input name="id" />
</form>

$("input:enabled")  // 找到可用的input标签
```

找到被选中的 option：

```
<select id="s1">
  <option value="beijing">北京市</option>
  <option value="shanghai">上海市</option>
  <option selected value="guangzhou">广州市</option>
  <option value="shenzhen">深圳市</option>
</select>

$(":selected")  // 找到所有被选中的option
```

## 筛选器方法

下一个元素：

```
$("#id").next()
$("#id").nextAll()
$("#id").nextUntil("#i2")
```

上一个元素：

```
$("#id").prev()
$("#id").prevAll()
$("#id").prevUntil("#i2")
```

父亲元素：

```
$("#id").parent()
$("#id").parents()  // 查找当前元素的所有的父辈元素$("#id").parentsUntil() // 查找当前元素的所有的父辈元素，直到遇到匹配的那个元素为止。
```

儿子和兄弟元素：

```
$("#id").children();// 儿子们
$("#id").siblings();// 兄弟们
```

查找

搜索所有与指定表达式匹配的元素。这个函数是找出正在处理的元素的后代元素的好方法。

```
$("div").find("p")
```

等价于$("div p")

筛选

筛选出与指定表达式匹配的元素集合。这个方法用于缩小匹配的范围。用逗号分隔多个表达式。

```
$("div").filter(".c1")  // 从结果集中过滤出有c1样式类的
```

等价于 $("div.c1")

补充：

```
.first() // 获取匹配的第一个元素
.last() // 获取匹配的最后一个元素
.not() // 从匹配元素的集合中删除与指定表达式匹配的元素
.has() // 保留包含特定后代的元素，去掉那些不含有指定后代的元素。
.eq() // 索引值等于指定值的元素
```

示例：左侧菜单

左侧菜单栏

## 操作标签

### 样式操作

样式类

```
addClass();// 添加指定的CSS类名。
removeClass();// 移除指定的CSS类名。
hasClass();// 判断样式存不存在
toggleClass();// 切换CSS类名，如果有就移除，如果没有就添加。
```

示例：开关灯和模态框

CSS

```
css("color","red")//DOM操作：tag.style.color="red"
```

示例：

```
$("p").css("color", "red"); //将所有p标签的字体设置为红色
```

### 位置操作

```
offset()// 获取匹配元素在当前窗口的相对偏移或设置元素位置
position()// 获取匹配元素相对父元素的偏移
scrollTop()// 获取匹配元素相对滚动条顶部的偏移
scrollLeft()// 获取匹配元素相对滚动条左侧的偏移
```

`.offset()`方法允许我们检索一个元素相对于文档（document）的当前位置。

和 `.position()`的差别在于： `.position()`是相对于相对于父级元素的位移。

示例：

返回顶部示例

### 尺寸：

```
height()
width()
innerHeight()
innerWidth()
outerHeight()
outerWidth()
```

### 文本操作

操作标签内部文本：

| JavaScript | jQuery |
| :--------: | :----: |
| innerText  | text() |
| innerHTML  | html() |

![image-20210313180224291](https://gitee.com/gengff/blogimage/raw/master/images/image-20210313180224291.png)

如何获取 input 框用户输入的内容：

![image-20210313183421953](https://gitee.com/gengff/blogimage/raw/master/images/image-20210313183421953.png)

### 属性操作

用于 ID 等或自定义属性：

```
attr(Name)// 返回第一个匹配元素的属性值
attr(Name, Value)// 为所有匹配元素设置一个属性值
attr({k1: v1, k2:v2})// 为所有匹配元素设置多个属性值
removeAttr()// 从每一个匹配的元素中删除一个属性
在用变量名存储对象的时候，js中推荐使用
	xxxEle		标签对象
在jQuery中推荐使用
	$xxxEle		jQuery对象
```

| JavaScript        | jQuery           |
| ----------------- | ---------------- |
| setAttribute()    | attr(name,value) |
| getAttribute()    | attr(name)       |
| removeAttribute() | removeAttr(name) |

示例：

```js
<body><p id="d1" class="c1" username="jason"></p></body>

$("#d1").attr("id")   // 获取id 注意括号内必须传值，否则报错
$("#d1").attr("class") // 获取class
$("#d1").attr("username") // 获取自定义属性username
$("#d1").attr("xxx")  // 获取没有的则返回undefined

$("#d1").attr("xxx","ooo") // 添加属性
$("#d1").attr({"user":"root","pwd","123"}) // 添加多个属性，需以字典形式
$("#d1").removeAttr("xxx") // 删除属性
```

用于 checkbox 和 radio

```
prop() // 获取属性
removeProp() // 移除属性
```

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="https://cdn.bootcdn.net/ajax/libs/jquery/3.4.1/jquery.min.js"></script>
</head>
<body>
    <p id="d1" class="c1" username="jason"></p>
    <input type="checkbox" checked>111
    <input type="checkbox" >222
    <input type="checkbox" >333

    <select name="" id="">
        <option value="">111</option>
        <option value="" selected>222</option>
        <option value="">333</option>
    </select>
</body>
</html>
```

![image-20210313210501227](https://gitee.com/gengff/blogimage/raw/master/images/image-20210313210501227.png)

**prop 和 attr 的区别：**

attr 全称 attribute(属性)

prop 全称 property(属性)

虽然都是属性，但他们所指的属性并不相同，attr 所指的属性是 HTML 标签属性，而 prop 所指的是 js 对象属性，可以认为 attr 是显式的，而 prop 是隐式的。

上图可以看到 attr 获取一个标签无论有没有被选中的都会得到 checked 或者 undefined，而 prop 获取的是这个 js 对象的属性，因此 checked 为选中返回 true，没选中返回 false。这已经可以证明 attr 的局限性，它的作用范围只限于 HTML 标签内的属性

但要知道的是 prop 不支持获取标签的自定义属性。

**总结一下：**

1. 对于标签上有的能看到的属性和自定义属性都用 attr
2. 对于返回布尔值的比如 checkbox、radio 和 option 的是否被选中都用 prop。

### 文档处理

添加到指定元素**内部**的后面

```
$(A).append(B)// 把B追加到A
$(A).appendTo(B)// 把A追加到B
```

添加到指定元素**内部**的前面

```
$(A).prepend(B)// 把B前置到A
$(A).prependTo(B)// 把A前置到B
```

添加到指定元素**外部**的后面

```
$(A).after(B)// 把B放到A的后面
$(A).insertAfter(B)// 把A放到B的后面
```

添加到指定元素**外部**的前面

```
$(A).before(B)// 把B放到A的前面
$(A).insertBefore(B)// 把A放到B的前面
```

移除和清空元素

```
remove()// 从DOM中删除所有匹配的元素。
empty()// 删除匹配的元素集合中所有的子节点。
```

例子：

点击按钮在表格添加一行数据。

点击每一行的删除按钮删除当前行数据。

替换

```
replaceWith()
replaceAll()
```

克隆

```
clone()// 参数
```

克隆示例：

点击复制按钮

## 事件

### 常用事件

```
click(function(){...})
hover(function(){...})
blur(function(){...})
focus(function(){...})
change(function(){...})
keyup(function(){...})
```

keydown 和 keyup 事件组合示例：

按住 shift 实现批量操作

hover 事件示例：

hover 事件

实时监听 input 输入值变化示例：

input 值变化事件

### 事件绑定

1. `.on( events [, selector ],function(){})`

- events： 事件
- selector: 选择器（可选的）
- function: 事件处理函数

### 移除事件

1. `.off( events [, selector ][,function(){}])`

`off()` 方法移除用 `.on()`绑定的事件处理程序。

- events： 事件
- selector: 选择器（可选的）
- function: 事件处理函数

### 阻止后续事件执行

1. `return false; // 常见阻止表单提交等`
2. e.preventDefault();

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>阻止默认事件</title>
</head>
<body>

<form action="">
    <button id="b1">点我</button>
</form>

<script src="jquery-3.3.1.min.js"></script>
<script>
    $("#b1").click(function (e) {
        alert(123);
        //return false;
        e.preventDefault();
    });
</script>
</body>
</html>
```

注意：

像 click、keydown 等 DOM 中定义的事件，我们都可以使用`.on()`方法来绑定事件，但是`hover`这种 jQuery 中定义的事件就不能用`.on()`方法来绑定了。

想使用事件委托的方式绑定 hover 事件处理函数，可以参照如下代码分两步绑定事件

```
$('ul').on('mouseenter', 'li', function() {//绑定鼠标进入事件
    $(this).addClass('hover');
});
$('ul').on('mouseleave', 'li', function() {//绑定鼠标划出事件
    $(this).removeClass('hover');
});
```

### 阻止事件冒泡

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>阻止事件冒泡</title>
</head>
<body>
<div>
    <p>
        <span>点我</span>
    </p>
</div>
<script src="jquery-3.3.1.min.js"></script>
<script>
    $("span").click(function (e) {
        alert("span");
        e.stopPropagation();
    });

    $("p").click(function () {
        alert("p");
    });
    $("div").click(function () {
        alert("div");
    })
</script>
</body>
</html>
```

### 页面载入

当 DOM 载入就绪可以查询及操纵时绑定一个要执行的函数。这是事件模块中最重要的一个函数，因为它可以极大地提高 web 应用程序的响应速度。

两种写法：

```
$(document).ready(function(){
// 在这里写你的JS代码...
})
```

简写：

```
$(function(){
// 你在这里写你的代码
})
```

文档加载完绑定事件，并且阻止默认事件发生：

登录校验示例

### 与 window.onload 的区别

- window.onload()函数有覆盖现象，必须等待着图片资源加载完成之后才能调用
- jQuery 的这个入口函数没有函数覆盖现象，文档加载完成之后就可以调用（建议使用此函数）

### 事件委托

事件委托是通过事件冒泡的原理，利用父标签去捕获子标签的事件。

示例：

表格中每一行的编辑和删除按钮都能触发相应的事件。

```
$("table").on("click", ".delete", function () {
  // 删除按钮绑定的事件
})
```

## 动画效果

```
// 基本
show([s,[e],[fn]])
hide([s,[e],[fn]])
toggle([s],[e],[fn])
// 滑动
slideDown([s],[e],[fn])
slideUp([s,[e],[fn]])
slideToggle([s],[e],[fn])
// 淡入淡出
fadeIn([s],[e],[fn])
fadeOut([s],[e],[fn])
fadeTo([[s],o,[e],[fn]])
fadeToggle([s,[e],[fn]])
// 自定义（了解即可）
animate(p,[s],[e],[fn])
```

自定义动画示例：

点赞特效简单示例

## 补充

### each

**jQuery.each(collection, callback(indexInArray, valueOfElement))：**

描述：一个通用的迭代函数，它可以用来无缝迭代对象和数组。数组和类似数组的对象通过一个长度属性（如一个函数的参数对象）来迭代数字索引，从 0 到 length - 1。其他对象通过其属性名进行迭代。

```
li =[10,20,30,40]
$.each(li,function(i, v){
  console.log(i, v);//index是索引，ele是每次循环的具体元素。
})
```

输出：

```
010
120
230
340
```

**.each(function(index, Element))：**

描述：遍历一个 jQuery 对象，为每个匹配元素执行一个函数。

`.each()` 方法用来迭代 jQuery 对象中的每一个 DOM 元素。每次回调函数执行时，会传递当前循环次数作为参数(从 0 开始计数)。由于回调函数是在当前 DOM 元素为上下文的语境中触发的，所以关键字 `this` 总是指向这个元素。

```
// 为每一个li标签添加foo
$("li").each(function(){
  $(this).addClass("c1");
});
```

注意: jQuery 的方法返回一个 jQuery 对象，遍历 jQuery 集合中的元素 - 被称为隐式*迭代*的过程。当这种情况发生时，它通常不需要显式地循环的 `.each()`方法：

也就是说，上面的例子没有必要使用 each()方法，直接像下面这样写就可以了：

```
$("li").addClass("c1");  // 对所有标签做统一操作
```

**注意：**

在遍历过程中可以使用 `return false`提前结束 each 循环。

**终止 each 循环**

```
return false；
```

伏笔...

### .data()

在匹配的元素集合中的所有元素上存储任意相关数据或返回匹配的元素集合中的第一个元素的给定名称的数据存储的值。

**.data(key, value):**

描述：在匹配的元素上存储任意相关数据。

```
$("div").data("k",100);//给所有div标签都保存一个名为k，值为100
```

**.data(key):**

描述: 返回匹配的元素集合中的第一个元素的给定名称的数据存储的值—通过 `.data(name, value)`或 `HTML5 data-*`属性设置。

```
$("div").data("k");//返回第一个div标签中保存的"k"的值
```

.removeData(key):

描述：移除存放在元素上的数据，不加 key 参数表示移除所有保存的数据。

```
$("div").removeData("k");  //移除元素上存放k对应的数据
```

示例：

模态框编辑的数据回填表格

### 插件(了解即可)

jQuery.extend(object)

jQuery 的命名空间下添加新的功能。多用于插件开发者向 jQuery 中添加新函数时使用。

示例：

```
<script>
jQuery.extend({
  min:function(a, b){return a < b ? a : b;},
  max:function(a, b){return a > b ? a : b;}
});
jQuery.min(2,3);// => 2
jQuery.max(4,5);// => 5
</script>
```

一个对象的内容合并到 jQuery 的原型，以提供新的 jQuery 实例方法。

```
<script>
  jQuery.fn.extend({
    check:function(){
      return this.each(function(){this.checked =true;});
    },
    uncheck:function(){
      return this.each(function(){this.checked =false;});
    }
  });
// jQuery对象可以使用新添加的check()方法了。
$("input[type='checkbox']").check();
</script>
```

单独写在文件中的扩展：

```
(function(jq){
  jq.extend({
    funcName:function(){
    ...
    },
  });
})(jQuery);
```
