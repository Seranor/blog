---
title: JavaScript基础
lastmod: 2021-10-20T16:43:23+08:00
date: 2021-10-20T11:52:03+08:00
tags:
  - JavaScript
categories:
  - JavaScript
url: post/javascript.html
toc: true
---

### JavaScript 简介

```js
JavaScript是Netscape公司创造的，该语言被提交到ECMA国际标准化组织，改名为了ECMAScript，目前使用的最多的版本是ECMAScript5或者ECMAScript5
JavaScript是脚本语言
JavaScript是一种轻量级的编程语言
JavaScript 是可插入 HTML 页面的编程代码
JavaScript 插入 HTML 页面后，可由所有的现代浏览器执行
```

<!-- more -->

### 引入方式及注释

```html
引入方式: 1.标签内编写
<script></script>

2.引入js文件
<script src="script.js"></script>

注释: 1.单行注释 // 2.多行注释 /**/ 结束符是 ;
```

### 变量与常量

```python
声明变量需要使用关键字
老版本(ES5)  var
新版本(ES6)  let
区别: var是声明全局变量，let是声明局部变量
var str1='hello'
let str2='world'

// 全局变量，因此最终结果是9
var a=1;
for(var a=0;a<10;a++){
	console.log(a)
}
a

// 局部变量，此时结果是2
let b=2;
for(let b=0;b<10;b++){
	console.log(b)
}
b


声明常量关键字 const
const pi=3.14
```

### 基本数据类型

```python
基本数据类型有: number string boolean undfined object()

var a=1;
var b='hello';
var c=true;
var d;
var e=[111,222,333];
typeof a; //'number'
typeof b; //'string'
typeof c; //'boolean'
typeof d; //'undefined'
typeof e; //'object'
```

### number 类型

```python
不区分整型与浮点型
var a=10.1;
var b=20;
typeof a;
typeof b;

有一个特殊的数字类型NaN，表示Not a Number

类型转换
parseInt("123");  //123
parseInt("abc");  // NaN
parseFloat("12.34");  //12.34
```

### string 类型

```python
var a="hello";
var b="world";
var c=a+b;
console.log(c);

符号可以是单引号双引号，没有三引号
使用 `` 做多行定义
var d=`
dadsad
dadsad
`
```

常用方法:

| 方法                       | 说明               |
| -------------------------- | ------------------ |
| .length                    | 返回长度           |
| .trim()                    | 移除空白           |
| .trimLeft()                | 移除左边的空白     |
| .trimRight()               | 移除右边的空白     |
| .charAt(n)                 | 返回第 n 个字符    |
| .concat(value, ...)        | 拼接               |
| .indexOf(substring, start) | 子序列位置         |
| .substring(from, to)       | 根据索引获取子序列 |
| .slice(start, end)         | 切片               |
| .toLowerCase()             | 小写               |
| .toUpperCase()             | 大写               |
| .split(delimiter, limit)   | 分割               |

```js
var str1 = "hello world";
var str2 = "  XxxXX  ";

str1.length; //11
str2.trim(); //'XxxXX'
str2.trimLeft(); //'XxxXX  '
str2.trimRight(); //'  XxxXX'
str1.charAt(4); //'o'
str1.concat("Javastript"); //'hello worldJavastript'
str1.indexOf("d"); //10
str1.substring(2, 5); //'llo'
str1.slice(2, 5); //'llo'
str1.toLowerCase(); //'hello world'
str1.toUpperCase(); //'HELLO WORLD'
str1.split(" "); //(2) ['hello', 'world']

// 拼接字符串推荐使用 +

// 模板字符串
var name = "Bob";
var age = 18;
var c = `
name is ${name},age is ${age}
`;
c;
("\nname is Bob,age is 18\n");
```

### boolean 类型

```python
var a=true;
var b=false;

空字符串、0、null、undefined、NaN都是false
```

### null 与 undefined 类型

```python
null 值是空的，一般在需要指定或清空一个变量时才会使用
		var a=null;
undefined 声明了一个变量但是没有被初始化，该变量的默认值是undefined
		var b;
```

### 对象类型

```python
JavaScript 中的所有事物都是对象：字符串、数值、数组、函数...此外，JavaScript 允许自定义对象
JavaScript 提供多个内建对象，比如 String、Date、Array 等等
对象只是带有属性和方法的特殊数据类型
```

### 对象类型之数组

```python
var a=[123,"abc"];
console.log(a[1]);
```

常用方法:

| 方法               | 说明                                       |
| ------------------ | ------------------------------------------ |
| .length            | 数组的大小                                 |
| .push(ele)         | 尾部追加元素                               |
| .pop()             | 获取尾部的元素                             |
| .unshift(ele)      | 头部插入元素                               |
| .shift()           | 头部移除元素                               |
| .slice(start, end) | 切片                                       |
| .reverse()         | 反转                                       |
| .join(seq)         | 将数组元素连接成字符串                     |
| .concat(val, ...)  | 连接数组                                   |
| .sort()            | 排序                                       |
| .forEach()         | 将数组的每个元素传递给回调函数             |
| .splice()          | 删除元素，并向数组添加新元素。             |
| .map()             | 返回一个数组元素调用函数处理后的值的新数组 |

### 运算符

```python
基本运算符
+ - * / % ++ --

比较运算符
> >= < <= != == === !==

1 == “1”  // true  弱等于
1 === "1"  // false 强等于

逻辑运算符
&& || !

赋值运算符
= += -= *= /=
```

### 流程控制

#### `if判断`

```python
if-else句式

var a=10;
if(a>5){
  console.log('yes');
}else{
  console.log('no');
}

if-else if-else
var a=3;
if(a>5){
  console.log('大于5')
}else if(a<5){
   console.log('小于5')
}else{
  console.log('other')
}
```

#### `switch`

```python
var a=1;
switch (a) {
  case 1:
  console.log('星期一');
  break;
  case 2:
  console.log('星期二');
  break;
  case 3:
  console.log('星期三');
  break;
defaule:
  console.log('其他');
}
```

#### `for`

```python
for(var a=0;a<10;a++){
  console.log(a);
}
```

#### `while`

```python
var a=0;
while(a<10){
  console.log(a);
  a++;
}
```

### 三元运算符

```python
var a=1;
var b=2;
var c=a>b?a:b;
console.log(c)
```

### 函数

```python
// 函数定义
function 函数名(参数1,参数2){
    函数体代码
  	return 返回值;
}

function f2(a, b) {
  console.log(arguments);  // 内置的arguments对象,可以获取传入的所有数据,也支持return和匿名函数
  console.log(arguments.length);
  console.log(a, b);
}
f2(1,3)

// 匿名函数定义
function(参数1,参数2){
  函数体代码
}
var sum = function(a, b){
  return a + b;
}
sum(1, 2);

//立即执行函数在两边都使用()
(function(a, b){return a + b;})(1,3)

//ES6中可以使用 => 定义函数
var f = v => v;
// 等同于
var f = function(v){
  return v;
}
```

### 自定义对象

```python
相当于python中的字典类型
	方式1: var d = {'name':'jason','age':18}
  方式2: var d = Object({'name':'jason','age':18})

var d = {'name':'jason','age':18}
typeof d  //'object'
d['name']  //'jason'
d.name  //'jason'
// 支持for循环
for(let i in d){
  console.log(i,d[i])
}

//增加元素
d.hobby = 'play'



# python 使用 . 拿到value
class MyDict(dict):
    def __getattr__(self, item):
        return self.get(item)
    def __setattr__(self, key, value):
        self[key] = value

res = MyDict(name='jason',age=18)
print(res.name)
print(res.age)
res.xxx = 123
print(res.xxx)
print(res)
```

### 内置对象

#### `Date`对象

```python
创建Date对象
let d = new Date()
d.toLocaleString()

Date对象的方法
var d = new Date();
//getDate()                 获取日
//getDay ()                 获取星期
//getMonth ()               获取月（0-11）
//getFullYear ()            获取完整年份
//getYear ()                获取年
//getHours ()               获取小时
//getMinutes ()             获取分钟
//getSeconds ()             获取秒
//getMilliseconds ()        获取毫秒
//getTime ()                返回累计毫秒数(从1970/1/1午夜)
```

#### `Json`对象

```python
Python中
  dumps  序列化
  loads  反序列化
JS中
  JSON.stringify()  序列化
  JSON.parse()  反序列化

let js={'name':'jason','age':18};
res = JSON.stringify(js)  //'{"name":"jason","age":18}'  序列化
JSON.parse(res)  // {name: 'jason', age: 18}  反序列化
```

#### `RegExp`对象

```python
在python中使用正则，需要使用re模块
在JavaScript中需要创建正则对象

//第一种方式
let reg1 = new RegExp("^[a-zA-Z][a-zA-Z0-9]{5,11}");

//第二种方式
let reg2 = /^[a-zA-Z][a-zA-Z0-9]{5,9}/;

//匹配内容
reg1.test('jsdsa123')

let str1 = 'jsdsa123 dsa teqwds'
//获取字符里所有的字母s
str1.match(/s/)  //获取到第一个就停止了
str1.match(/s/g)  // g 全局匹配所有

// 全局匹配模式的问题
let reg3 = /^[a-zA-Z][a-zA-Z0-9]{5,9}/g;
reg3.test('jason123asd')
reg3.test('jason123asd')  // 第二次是false，下一次是true

//全局模式有一个lastIndex属性
reg3.lastIndex
reg3.test('jason123asd')
reg3.lastIndex
reg3.test('jason123asd')


let reg4 = /^[a-zA-Z][a-zA-Z0-9]{5,9}/;
reg4.test()  // 结果是true 什么都不传，默认传的是undefined
```

### BOM 与 DOM 操作

```python
BOM（Browser Object Model）是指浏览器对象模型，它使 JavaScript 有能力与浏览器进行“对话”
DOM （Document Object Model）是指文档对象模型，通过它，可以访问HTML文档的所有元素
```

#### BOM 操作

##### `window`对象

```python
window对象指代的就是浏览器窗口

window.innerHeight  //浏览器窗口当前的高度
window.innerWidth  //浏览器窗口当前的宽度
window.open('https://www.baidu.com','','height=200px,width=200px')  //新的窗口打开网站
window.close()  //关闭当前窗口
```

##### `window`子对象

```python
1.navigator对象，获取浏览器相关信息
navigator.appName　　   // Web浏览器全称
navigator.appVersion　　// Web浏览器厂商和版本的详细字符串
navigator.userAgent　　 // 客户端绝大部分信息
navigator.platform　　　// 浏览器运行所在的操作系统

//如果是window的子对象，window可以省略不写

history对象
history.forward()  // 前进一页
history.back()  // 后退一页

2.location对象
location.href  //获取URL
location.href="URL"  // 跳转到指定页面
location.reload()  //重新加载页面

3.弹出框
  警告框
  alert('警告框')

  确认框
  confirm('确认框')
  //点了确认按钮之后返回true，否则false

  提示框
  prompt("请在下方输入","你的答案")
  可以接收用户在提示框输入的内容

4.计时器相关
过一段时间之后触发
每隔一段时间触发一次(循环触发)

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>

</head>
<body>
<script>
  // function func1(){
  //   alert(123)
  // }
  // let t = setTimeout(func1,3000) // 毫秒为单位 3秒之后自动执行func1函数
  // clearTimeout(t)

  function func2(){
    alert(123)
  }
  function show(){
    let t = setInterval(func2,2000)
    function inner(){
          clearInterval(t)
    }
    setTimeout(inner,8000)
  }
  show()
</script>
</body>
</html>
```

#### DOM 操作

HTML DOM 树

![img](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/867021-20180312215352312-132101897.png)

DOM 标准规定 HTML 文档中的每个成分都是一个节点

```
文档节点(document对象)：代表整个文档
元素节点(element 对象)：代表一个元素（标签）
文本节点(text对象)：代表元素（标签）中的文本
属性节点(attribute对象)：代表一个属性，元素（标签）才有属性
注释是注释节点(comment对象)　
```

##### 标签查找

- 基本查找

  ```python
  document.getELementByID             根据ID获取标签
  document.getELementsByClassName     根据class属性获取
  document.getElementsByTagName       根据标签名获取标签合集

  # 取到合集时用索引取 [0]

  document.getElementById('d1')
  document.getElementsByClassName('c1')
  document.getElementsByTagName('p')[0]
  ```

- 间接查找

  ```python
  paretElement                父节点标签元素
  children                    所有子标签
  firstElement                第一个子标签元素
  lastElementChild            最后一个子标签元素
  nextElementSibling          下一个兄弟标签元素
  previousElementSibling      上一个兄弟标签元素


  document.getElementById('d2').parentElement
  document.getElementById('d1').children
  document.getElementById('d1').firstElementChild
  document.getElementById('d1').lastElementChild
  document.getElementById('d1').nextElementSibling
  document.getElementById('d2').previousElementSibling

  ```

PS：HTML 页面是从上往下解析，因此当 JS 需要等标签元素先加载的情况时，应该将 JS 代码写在 body 内部最下方，或者有在最下方引入 JS 文件

##### 节点操作

```python
# 创建标签
var aEle=document.createElement('a');

# 添加标签的属性，如果是默认的属性可以通过 .属性=属性值 的方式添加，setAttribute可以添加任意属性
aEle.setAttribute('href','https://www.baidu.com');

# 添加文本内容
aEle.innerText='点我去百度';

# 上述并没有添加到HTML文档中，使用如下方式添加
# 先查找到需要添加的位置，以某个标签为参考，然后进行添加
# appendChild() 添加子节点
# insertBefore() 把增加节点放到某个节点的前面
document.getElementsByTagName('p')[0].appendChild(aEle)

# 获取标签属性
aEle.getAttribute('href')

# innerText和innerHTML区别
innerText
    document.getElementsByTagName('a')[0].innerText  # 可以获取a标签内文本内容
    document.getElementById('d1').innerText='hello'  # 设置标签文本内容
    不可以识别HTML标签

innerHTMl
    document.getElementById('d1').innerHTML  # 获取内部标签和文本
    document.getElementById('d1').innerHTML='<h1>hello</h1>'  # 设置标签和文本
    可以识别HTML标签
```

##### 获取值操作

```python
<p>username:<input type="text" class="c1"></p>
<p><input type="file" multiple class="c2"></p>

# 普通文本数据获取
    标签对象.value
    document.getElementsByClassName('c1')[0].value

# 特殊的文件数据获取
    标签对象.value     # 获取的是一个文件地址
    标签对象.files[0]  # 获取单个文件数据
    标签对象.fiiles    # 获取所有文件数据

    document.getElementsByClassName('c2')[0].value
    document.getElementsByClassName('c2')[0].files[0]
		document.getElementsByClassName('c2')[0].files
```

##### `class`操作

```python
<p>username:<input type="text" class="c1 c2"></p>


classList                  查看所有的类
classList.remove(cls)      删除指定类
classList.add(cls)         添加类
classList.contains(cls)    存在返回true，否则返回false
classList.toggle(cls)      存在就删除，否则添加


document.getElementsByClassName('c1')[0].classList
document.getElementsByClassName('c1')[0].classList.remove('c2')
document.getElementsByClassName('c1')[0].classList.add('c3')
document.getElementsByClassName('c1')[0].classList.contains('c1')  # true
document.getElementsByClassName('c1')[0].classList.contains('c4')  # false
document.getElementsByClassName('c1')[0].classList.toggle('c2')
```

##### 样式操作

```python 
标签对象.style.属性名=属性值

# 画圆
<div id="d10"></div>

document.getElementById('d10').style.width="400px";
document.getElementById('d10').style.height="400px";
document.getElementById('d10').style.borderRadius="50%";
document.getElementById('d10').style.border="3px solid red";

# 对于style中属性值有 - 的，将第一个字母换成大写接上即可，小驼峰(borderRadius)
```

##### 事件

常用事件

```python
onclick        当用户点击某个对象时调用的事件句柄
ondblclick     当用户双击某个对象时调用的事件句柄

onfocus        元素获得焦点
onblur         元素失去焦点                应用场景：用于表单验证,用户离开某个输入框时,代表已经输入完了,我们可以对它进行验证
onchange       域的内容被改变              应用场景：通常用于表单元素,当元素内容被改变时触发.（select联动）

onkeydown      某个键盘按键被按下           应用场景: 当用户在最后一个输入框按下回车按键时,表单提交
onkeypress     某个键盘按键被按下并松开
onkeyup        某个键盘按键被松开
onload         一张页面或一幅图像完成加载
onmousedown    鼠标按钮被按下
onmousemove    鼠标被移动
onmouseout     鼠标从某元素移开
onmouseover    鼠标移到某元素之上

onselect      在文本框中的文本被选中时发生
onsubmit      确认按钮被点击，使用的对象是form
```

绑定事件的方式

```python
# 方式一
<input type="button" onclick="func1()" value="点我">
<script>
    function func1(){
        alert(123)
    }
</script>

# 推荐使用方式二
<input type="button" id="d1" value="点点">
<script>
    var btEle = document.getElementById('d1');
    btEle.onclick=function (){
        alert(456)
    }
</script>

# 如果某个标签已经有事件了 那么绑定会冲突
```

##### 内置参数`this`

```python
# this指代的就是当前被操作对象本身
在事件的函数体代码内部使用
btnEle.onclick = function () {
		alert(456)
		console.log(this)
}
```

##### 事件案列

点击按钮变色

```html
<head>
    <style>
        .c1 {
            width: 400px;
            height: 400px;
            border-radius: 50%;
        }

        .bg_red {
            background-color: red;
        }

        .bg_yellow {
            background-color: greenyellow;
        }</head>
<body>
<div class="c1 bg_red bg_yellow"></div>
<button id="d2">变色</button>
<script>
    var btEle = document.getElementById('d2')
    btEle.onclick=function (){
        document.getElementsByClassName('c1')[0].classList.toggle('bg_yellow')
    }
  </script>
</body>
```

input 框获取焦点失去焦点

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Title</title>
  </head>
  <body>
    <input type="text" value="默认的内容" id="d1" />
    <script>
      let iEle = document.getElementById("d1");
      iEle.onfocus = function () {
        iEle.value = "";
      };
      iEle.onblur = function () {
        iEle.value = "哈哈哈";
      };
    </script>
  </body>
</html>
```

定时器

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Title</title>
  </head>
  <body>
    <input type="text" id="d1" />
    <button id="d2">开始</button>
    <button id="d3">结束</button>
    <script>
      let t = null;
      var inpEle = document.getElementById("d1");

      //两个按钮点击实现开始和结束
      var startBEle = document.getElementById("d2");
      var endBEle = document.getElementById("d3");

      function showTime() {
        let currentTime = new Date();
        inpEle.value = currentTime.toLocaleString();
      }

      startBEle.onclick = function () {
        //限定只能开一个定时器
        if (!t) {
          t = setInterval(showTime, 1000);
        }
      };
      endBEle.onclick = function () {
        clearInterval(t); //清空定时器
        //清空t，重置为空
        t = null;
      };
    </script>
  </body>
</html>
```

select 联动

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Title</title>
  </head>
  <body>
    <select name="" id="d1">
      <option value="" selected disabled>--请选择--</option>
    </select>
    <select name="" id="d2"></select>
    <script>
      let proEle = document.getElementById("d1");
      let cityEle = document.getElementById("d2");
      data = {
        河北省: ["廊坊", "邯郸"],
        北京: ["朝阳区", "海淀区"],
        山东: ["威海市", "烟台市"],
      };
      for (let i in data) {
        let opEle = document.createElement("option");
        opEle.innerText = i;
        opEle.value = i;
        proEle.appendChild(opEle);
      }
      //文本域变化
      proEle.onchange = function () {
        let currentPro = proEle.value; //选中谁取到谁，拿到省
        let currentCityList = data[currentPro]; //得到省下的市
        //清空一下select中所有的option
        cityEle.innerHTML = "";
        //自己加个 请选择
        let ss = "<option disabled selected>--请选择--</option>";
        cityEle.innerHTML = ss;
        for (let j = 0; j < currentCityList.length; j++) {
          let currentCity = currentCityList[j];
          let opEle = document.createElement("option");
          opEle.innerText = currentCity;
          opEle.value = currentCity;
          cityEle.appendChild(opEle);
        }
      };
    </script>
  </body>
</html>
```
