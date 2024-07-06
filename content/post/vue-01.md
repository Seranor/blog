---
title: Vue的基本使用
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - Vue
categories:
  - Vue
url: post/vue-01.html
toc: true
---

### 前端介绍

```python
# 1 HTML(5)、CSS(3)、JavaScript(ES5、ES6)：编写一个个的页面 -> 给后端(PHP、Python、Go、Java) -> 后端嵌入模板语法 -> 后端渲染完数据 -> 返回数据给前端 -> 在浏览器中查看
# javascript=ecmascript+dom+bom  2015年es6 --> 格式化字符串 ``

# 2 Ajax的出现 -> 后台发送异步请求，Render+Ajax混合
# 3 单用Ajax（加载数据，DOM渲染页面）：前后端分离的雏形
# 4.Angular框架的出现（1个JS框架）：出现了“前端工程化”的概念（前端也是1个工程、1个项目）
# 5 React、Vue框架：当下最火的2个前端框架（Vue：国人喜欢用，React：外国人喜欢用）
# 6 移动开发（Android+IOS） + Web（Web+微信小程序+支付宝小程序） + 桌面开发（Windows桌面）：前端 -> 大前端
#  移动端混合开发：原生+页面 ---> 支付宝 ---> 口碑
# 7 一套代码在各个平台运行（大前端）：谷歌Flutter（Dart语言：和Java很像）可以运行在IOS、Android、PC端
# 8 在Vue框架的基础性上 uni-app：一套编码 编到10个平台
		https://uniapp.dcloud.io/case.html
# html css（less，sass） js jq，bootstrap es6 webpack vue react 小程序开发 node git mongodb
```

<!-- more -->

### Vue 特性介绍

```python
# vue介绍
Vue (读音 /vjuː/，类似于 view) 是一套用于构建用户界面的渐进式框架
与其它大型框架不同的是，Vue 被设计为可以自底向上逐层应用
Vue 的核心库只关注视图层，不仅易于上手，还便于与第三方库或既有项目整合
可以一点一点地使用它，只用一部分，也可以整个工程都使用它
js的框架，跟jq是一类东西
bootstrap：ui框架不是js框架（css样式）
vue中使用ui可以引入bootstrap，elementui(饿了么团队出的)，Vant(移动端ui：有赞)，ant-design-vue（ant-design本身是react的ui库）

# 版本
	-主流：2.x
  -最新：3.x

# 官方有教程：https://cn.vuejs.org/v2/guide/
# M-V-VM思想 ----》mvc，mtv，mvp：安卓分层架构
Model ：vue对象的data属性里面的数据，这里的数据要显示到页面中，js中变量
View ：vue中数据要显示的HTML页面，在vue中，也称之为“视图模板” （HTML+CSS）
ViewModel：vue中编写代码时的vm对象，它是vue.js的核心，负责连接 View 和 Model数据的中转，保证视图和数据的一致性，所以前面代码中，data里面的数据被显示中p标签中就是vm对象自动完成的（双向数据绑定：JS中变量变了，HTML中数据也跟着改变）
以后不需要显示的使用dom操作，jquery的作用就不大了
双向数据绑定：JS中变量变了，HTML中数据也跟着改变

# 组件化开发，单页面开发
	-vue项目 ---> index.html页面 ---> 看到的变化，都是组件的切换
  -页面组件 ---> 放了小组件
  -在index.html中替换 组件，就实现页面的变化
```

![img](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/mvvm.jpg)

![image-20220417181446468](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220417181446468.png)

### 快速使用 Vue

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Title</title>
    <!--    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>-->
    <script src="./js/vue.js"></script>
  </head>
  <body>
    <div id="app" class="div_cls">
      <h1>我的名字是：{{name}}</h1>
      <input type="text" v-model="text" />
      <hr />
      {{text}}
    </div>
  </body>
  <script>
    // div(不一定是div) 被vue托管了，在div内部，就可以使用vue的语法：模板，指令系统
    var vm = new Vue({
      // el:"#app",
      el: ".div_cls",
      data: {
        name: "xxx",
        text: "",
      },
    });
  </script>
</html>
```

### 模板语法

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>插值语法</title>
    <script src="./js/vue.js"></script>
  </head>
  <body>
    <div id="app">
      <h1>字符串：{{ name }}</h1>
      <h1>数值：{{ age }}</h1>
      <h2>数组：{{ name_list }}</h2>
      <h3>对象：{{ person_info }}</h3>
      <h4>{{ link }}</h4>
      <h4>对象取值：{{ person_info.age }}</h4>
      <h4>数组取值：{{ name_list[1] }}</h4>
      <h4>运算：{{ 10+20+30 }}</h4>
      <h4>三元运算：{{ 10>20?'是':'否' }}</h4>
    </div>
  </body>
  <script>
    var vm = new Vue({
      el: "#app",
      data: {
        name: "geng", //字符串
        age: 18, //数值
        name_list: ["刘一", "陈二", "张三", "李四", "王五", "赵六"], //对象
        person_info: { name: "geng", age: 18 },
        link: '<a href="http://www.baidu.com">点我</a>',
      },
    });
  </script>
</html>
```

### 指令

#### 文本

```
v-text   标签内容显示js变量对应的值
v-html   让HTML渲染成页面
v-if     放1个布尔值：为真 标签就显示；为假 标签就不显示
v-show   放1个布尔值：为真 标签就显示；为假 标签就不显示

v-show 与 v-if的区别：
v-show：标签还在，只是不显示了（display: none）
v-if：直接操作DOM，删除/插入 标签
```

#### 事件

```
v-on:缩写成@
v-on:click='函数'
@click='函数'
```

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Title</title>
    <script src="js/vue.js"></script>
  </head>
  <body>
    <div id="app">
      <h1 v-text="name"></h1>
      <h1>{{ name_list }}</h1>
      <h1>{{ name_list[1] }}</h1>
      <h1>{{ person_info.name }}</h1>
      <h1 v-html="link"></h1>

      <button @click="handle1">点我消失，显示</button>
      <div v-show="show">
        <span>看得见我，看不见我 </span>
      </div>
      <button @click="handle2('lzj')">点击弹窗</button>
      <br />
      <button @click="handle3">点我消失，显示2</button>
      <div v-if="if_show">哈哈哈</div>
    </div>
  </body>
  <script>
    var vm = new Vue({
      el: "#app",
      data: {
        name: "lzj",
        name_list: ["张三", "李四", "王五"],
        person_info: { name: "lzj", age: 18 },
        link: '<a href="https://www.baidu.com">点击跳转<a>',
        show: true,
        if_show: true,
      },
      methods: {
        handle1() {
          this.show = !this.show;
        },
        handle2(name) {
          alert(name);
        },
        handle3() {
          this.if_show = !this.if_show;
        },
      },
    });
  </script>
</html>
```

#### 属性

```
v-bind: 属性='js的变量'
可以简写成  :属性='js的变量'
```

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Title</title>
    <script src="js/vue.js"></script>
    <style>
      .red {
        background-color: red;
      }
      .green {
        background-color: green;
      }
    </style>
  </head>
  <body>
    <div id="app">
      <button @click="changeColour">点我变色</button>
      <p :class="p_class">{{name}}</p>
      <br />
      <button @click="changePhoto">点我更换图片</button>
      <img :src="img" alt="" /><br />
      <h1><a :href="link">点击跳转去百度</a></h1>

      <button @click="changeBool">点我变色</button>
      <p :class="isActive?'red':'green'">我是p标签</p>
    </div>
  </body>
  <script>
    var vm = new Vue({
      el: "#app",
      data: {
        name: "lzj",
        p_class: "red",
        img: "img/img.png",
        link: "https://www.baidu.com",
        isActive: true,
      },
      methods: {
        changeColour() {
          this.p_class = "green";
        },
        changePhoto() {
          this.img = "img/img_1.png";
        },
        changeBool() {
          this.isActive = !this.isActive;
        },
      },
    });
  </script>
</html>
```

#### `style`和`class`

```
属性之类中比较特殊的style和class
class 可以对应字符串，数组(推荐)，对象
style 可以对应字符串，数组，对象(推荐)
```

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Title</title>
    <script src="js/vue.js"></script>
    <style>
      .red {
        background-color: red;
      }

      .green {
        background-color: green;
      }

      .font {
        font-size: 50px;
      }
    </style>
  </head>
  <body>
    <div id="app">
      <h1 :class="h1_class">我是class</h1>
      <h1 :style="h1_style">我是style</h1>
    </div>
  </body>
  <script>
    var vm = new Vue({
      el: "#app",
      data: {
        // h1_class: 'red font',
        // h1_class: ['green', 'font'], // 可以使用 push 添加
        h1_class: { red: true, font: true },

        // h1_style: 'background-color: pink; font-size: 40px',
        // h1_style: [{'background-color': 'pink'},{'font-size': '40px'}],
        // 对于中间有 - 的属性 可以使用 单引号引起来 或者使用驼峰
        h1_style: { backgroundColor: "yellow", fontSize: "90px" },
      },
    });
  </script>
</html>
```

### 条件渲染

```
v-if
v-else-if
v-else
```

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Title</title>
    <script src="js/vue.js"></script>
  </head>
  <body>
    <div id="app">
      <h2>你的成绩是：</h2>
      <p v-if="score>=90">优秀</p>
      <p v-else-if="score>=80">良好</p>
      <p v-else-if="score>=60">及格</p>
      <p v-else>不及格</p>
    </div>
  </body>
  <script>
    var vm = new Vue({
      el: "#app",
      data: {
        score: 66,
      },
    });
  </script>
</html>
```

### 列表渲染

```
for循环: v-for
可以遍历数组，对象，数字
```

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Title</title>
    <script src="js/vue.js"></script>
  </head>
  <body>
    <div id="app">
      <div v-if="good_list.length>0">
        <table border="1">
          <thead>
            <tr>
              <td>商品</td>
              <td>价格</td>
            </tr>
          </thead>
          <tbody>
            <tr v-for="good in good_list" :key="good.name">
              <td>{{good.name}}</td>
              <td>{{good.price}}</td>
            </tr>
          </tbody>
        </table>
      </div>
      <div v-else>购物车内没有任何东西</div>
      <hr />
      <h1>遍历对象(第一个是value，第二个是key)</h1>
      <p v-for="item in info">{{item}}</p>
      <p v-for="(v,k) in info">value的值是{{v}}，key的值时{{k}}</p>
      <hr />
      <h1>遍历数组</h1>
      <ul>
        <li v-for="food in foods">{{food}}</li>
      </ul>
      <ul>
        <li v-for="(v,i) in foods">序号是{{i}} 商品是{{v}}</li>
      </ul>
      <hr />
      <h1>遍历数字，从1开始</h1>
      <p v-for="i in 5">{{i}}</p>
    </div>
  </body>
  <script>
    var vm = new Vue({
      el: "#app",
      data: {
        good_list: [
          { name: "土豆", price: 19 },
          { name: "青菜", price: 15 },
          { name: "胡萝卜", price: 23 },
        ],
        info: { name: "xxx", age: 18, gender: "man" },
        foods: ["青菜", "土豆", "胡萝卜", "白萝卜", "西蓝花"],
      },
    });
  </script>
</html>
```

### 补充

```python
# 注意！在Vue中：
  数组的index和value是反的
  对象的key和value也是反的

# key值的解释
看到被人代码在循环时，写在标签中  :key='值'
key:一般咱么在循环的时候，都要加 :key='值'，值不要是固定的
vue中使用的是虚拟DOM，会和原生的DOM进行比较，然后进行数据的更新，提高数据的刷新速度（虚拟DOM用了diff算法）
在v-for循环数组、对象时，建议在控件/组件/标签写1个key属性，属性值唯一
页面更新之后，会加速DOM的替换（渲染）
:key="变量"

 # key可以加速页面的替换 ---> key加上，效率高


# 数组更新与检测
# 数组追加一个值，页面里面跟着变
# 可以检测到变动的数组操作
push：最后位置添加
pop：最后位置删除
shift：第一个位置删除
unshift：第一个位置添加
splice：切片
sort：排序
reverse：反转

# 检测不到变动的数组操作： 页面不会变
filter()：过滤
concat()：追加另一个数组
slice()：
map()：

原因：
作者重写了相关方法（只重写了一部分方法，但是还有另一部分没有重写）

数组变了，但页面没变 ---> 解决方案
// 方法1：通过 索引值 更新数组（数据会更新，但是页面不会发生改变）
vm.arrayList[0]
"Alan"
vm.arrayList[0]='Darker'
"Darker"
// 方法2：通过 Vue.set(对象, index/key, value) 更新数组（数据会更新，页面也会发生改变）
Vue.set(vm.arrayList, 0, 'Darker')
```

### 事件处理

```
指的是 input 事件

input    当输入框进行输入的时候 触发的事件
change   当元素的值发生改变时   触发的事件
blur     当输入框失去焦点的时候  触发的事件
```

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Title</title>
    <script src="js/vue.js"></script>
  </head>
  <body>
    <div id="app">
      change: <input type="text" v-model="text1" @change="handleChange" /><br />
      input: <input type="text" v-model="text2" @input="handleInput" /><br />
      blur: <input type="text" v-model="text3" @blur="handleBlur" /><br />
    </div>
  </body>
  <script>
    var vm = new Vue({
      el: "#app",
      data: {
        text1: "",
        text2: "",
        text3: "",
      },
      methods: {
        handleChange() {
          console.log("change触发", this.text1);
        },
        handleInput() {
          console.log("input触发", this.text2);
        },
        handleBlur() {
          console.log("Blur触发", this.text3);
        },
      },
    });
  </script>
</html>
```

#### 过滤案例

##### `filter`

```html
<script>
  // filter的基本使用
  var dataList = ["a", "at", "atom", "be", "beyond", "cs", "csrf"];
  var newList = dataList.filter(function (item) {
    if (item.length > 2) {
      return true;
    } else {
      return false;
    }
  });
  console.log(newList);
</script>
```

```html
<script>
  // 判断一个字符串是否在另一个字符串中
  var text = "at";
  var dataList = ["a", "at", "atom", "be", "beyond", "cs", "csrf"];
  var newList = dataList.filter(function (item) {
    // return item.indexOf(text) >= 0  // 可以简写成这样
    var i = item.indexOf(text);
    if (i >= 0) {
      return true;
    } else {
      return false;
    }
  });
  console.log(newList);
</script>
```

##### 箭头函数

```javascript
var a = function (name) {
  console.log(name);
};
a("xxx1");

// 上面改写成箭头函数，箭头函数没有自己的this
var a = (name) => {
  console.log(name);
};
a("xxx");
```

##### 整体代码

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Title</title>
    <script src="js/vue.js"></script>
  </head>
  <body>
    <div id="app">
      <input type="text" v-model="text" @input="handleInput" />
      <ul>
        <li v-for="item in newDataList">{{item}}</li>
      </ul>
    </div>
  </body>
  <script>
    var vm = new Vue({
      el: "#app",
      data: {
        text: "",
        dataList: ["a", "at", "atom", "be", "beyond", "cs", "csrf"],
        newDataList: ["a", "at", "atom", "be", "beyond", "cs", "csrf"],
      },
      methods: {
        handleInput() {
          this.newDataList = this.dataList.filter((item) => {
            return item.indexOf(this.text) >= 0;
          });
        },
      },
    });
  </script>
</html>
```

#### 事件修饰符

```
.stop 只处理自己的事件，父控件冒泡的事件不处理(阻止事件的冒泡)
.self 只处理自己的事件，子控件冒泡的事件不处理
.prevent 阻止a链接的跳转
.once 事件只会触发一次 适合抽奖页面

事件冒泡: 子标签的点击事件传导了父标签上
```

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Title</title>
    <script src="js/vue.js"></script>
  </head>
  <body>
    <div id="app">
      <ul @click.self="handle1">
        <li @click.stop="handle2">事件一</li>
        <li @click="handle3">事件二</li>
        <li>事件三</li>
      </ul>
      <hr />
      <a href="https://www.baidu.com" @click.prevent="handle4">点击跳转百度</a>
      <hr />
      <button @click.once="handle5">秒杀</button>
    </div>
  </body>
  <script>
    var vm = new Vue({
      el: "#app",
      data: {},
      methods: {
        handle1() {
          console.log("最外面被点了");
        },
        handle2() {
          console.log("事件一");
        },
        handle3() {
          console.log("事件二");
        },
        handle4() {
          console.log("a标签被点了，但是没跳");
        },
        handle5() {
          console.log("点了秒杀，只有一次有效");
        },
      },
    });
  </script>
</html>
```

#### 按键修饰符

```
监听所有按键按下去弹起  keyup
监听Enter键          keyup.enter
```

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Title</title>
    <script src="js/vue.js"></script>
  </head>
  <body>
    <div id="app">
      <input
        type="text"
        v-model="text1"
        @keyup="handleKeyUp1($event)"
      />{{text1}}
      <hr />
      <input
        type="text"
        v-model="text2"
        @keyup.enter="handleKeyUp2($event)"
      />{{text2}}
    </div>
  </body>
  <script>
    var vm = new Vue({
      el: "#app",
      data: {
        text1: "",
        text2: "",
      },
      methods: {
        handleKeyUp1(event) {
          console.log(event);
          console.log(event.key, "被按下弹起了");
          if (event.key == "Enter") {
            alert("弹窗");
          }
        },
        handleKeyUp2(event) {
          console.log(event);
          console.log(event.key, "Enter被按下弹起了");
        },
      },
    });
  </script>
</html>
```

### 数据的双向绑定

```
input标签与js变量的绑定
```

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Title</title>
    <script src="js/vue.js"></script>
  </head>
  <body>
    <div id="app">
      <input type="text" v-model="text" /><br />
      输入的内容是：{{text}}
    </div>
  </body>
  <script>
    var vm = new Vue({
      el: "#app",
      data: {
        text: "",
      },
    });
  </script>
</html>
```

### 表单控制

```
input checkbox radio的控制
```

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Title</title>
    <script src="js/vue.js"></script>
  </head>
  <body>
    <div id="app">
      <h1>checkbox单选</h1>
      <p>用户名: <input type="text" v-model="username" /></p>
      <p>密码: <input type="password" v-model="password" /></p>
      <p>记住密码: <input type="checkbox" v-model="remember" /></p>
      <hr />
      <h1>radio单选</h1>
      <input type="radio" v-model="radio" value="1" />男
      <input type="radio" v-model="radio" value="2" />女
      <input type="radio" v-model="radio" value="0" />其他
      <h1>checkbox多选</h1>
      <input type="checkbox" v-model="many" value="篮球" />篮球
      <input type="checkbox" v-model="many" value="足球" />足球
      <input type="checkbox" v-model="many" value="排球" />排球
      <input type="checkbox" v-model="many" value="棒球" />棒球
    </div>
  </body>
  <script>
    var vm = new Vue({
      el: "#app",
      data: {
        username: "",
        password: "",
        remember: true, // checkbox 单选 是布尔
        radio: "", // radio的单选字符串 对应选中的value值
        many: [], // checkbox多选 数组
      },
    });
  </script>
</html>
```

### `v-mode`补充

```
lazy: 等待input框的数据绑定失去焦点之后再变化
number: 数字开头，只保留数字，后面的字母不保留，字母开头，都保留
trim: 去除首尾的空格
```

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Title</title>
    <script src="js/vue.js"></script>
  </head>
  <body>
    <div id="app">
      lazy: <input type="text" v-model.lazy="username" />{{username}}<br />
      number: <input type="text" v-model.number="age" />{{age}}<br />
      trim: <input type="text" v-model.trim="info" />{{info}}<br />
    </div>
  </body>
  <script>
    var vm = new Vue({
      el: "#app",
      data: {
        username: "",
        age: "",
        info: "",
      },
    });
  </script>
</html>
```

### 购物车案例

#### 基本购物车

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
    <script src="js/vue.js"></script>
  </head>
  <body>
    <div id="app">
      <div class="container-fluid">
        <div class="row">
          <div class="col-md-6 col-md-offset-3">
            <h1 class="text-center">购物车</h1>
            <table class="table table-hover table-bordered">
              <tr>
                <td>商品</td>
                <td>价格</td>
                <td>数量</td>
              </tr>
              <tr v-for="data in dataList">
                <td>{{data.name}}</td>
                <td>{{data.price}}</td>
                <td>{{data.number}}</td>
                <td>
                  <input type="checkbox" v-model="checkGroup" :value="data" />
                </td>
              </tr>
            </table>
            <br />
            选中的商品: {{checkGroup}}
            <br />
            总价格: {{getPrice()}}
          </div>
        </div>
      </div>
    </div>
  </body>
  <script>
    var vm = new Vue({
      el: "#app",
      data: {
        dataList: [
          { name: "红楼梦", price: 99, number: 2 },
          { name: "西游记", price: 59, number: 1 },
          { name: "水浒传", price: 89, number: 5 },
        ],
        checkGroup: [],
      },
      methods: {
        getPrice() {
          var total = 0;
          // 方式一 i 是索引 循环选中的商品，基于迭代的循环
          // for (i in this.checkGroup) {
          //     total += this.checkGroup[i].price * this.checkGroup[i].number
          // }

          // 方式二: 基于索引的循环
          // for (var i=0;i<this.checkGroup.length;i++) {
          //     total += this.checkGroup[i].price * this.checkGroup[i].number
          // }

          // 方式三：基于迭代 for of
          // for (v of this.checkGroup) {
          //     total += v.price * v.number
          // }

          // 方式四：forEach 可迭代对象(数组)
          this.checkGroup.forEach((v, i) => {
            total += v.price * v.number;
          });
          return total;
        },
      },
    });
  </script>
</html>
```

#### 全选和全不选

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
    <script src="js/vue.js"></script>
  </head>
  <body>
    <div id="app">
      <div class="container-fluid">
        <div class="row">
          <div class="col-md-6 col-md-offset-3">
            <h1 class="text-center">购物车</h1>
            <table class="table table-hover table-bordered">
              <tr>
                <td>商品</td>
                <td>价格</td>
                <td>数量</td>
                <td>
                  全选/全不选
                  <input
                    type="checkbox"
                    v-model="allCheck"
                    @change="handleAll"
                  />
                </td>
              </tr>
              <tr v-for="data in dataList">
                <td>{{data.name}}</td>
                <td>{{data.price}}</td>
                <td>{{data.number}}</td>
                <td>
                  <input
                    type="checkbox"
                    v-model="checkGroup"
                    :value="data"
                    @change="checkOne"
                  />
                </td>
              </tr>
            </table>
            <br />
            选中的商品: {{checkGroup}}
            <br />
            总价格: {{getPrice()}}
            <br />
            是否全选: {{allCheck}}
          </div>
        </div>
      </div>
    </div>
  </body>
  <script>
    var vm = new Vue({
      el: "#app",
      data: {
        dataList: [
          { name: "红楼梦", price: 99, number: 2 },
          { name: "西游记", price: 59, number: 1 },
          { name: "水浒传", price: 89, number: 5 },
        ],
        checkGroup: [],
        allCheck: false,
      },
      methods: {
        getPrice() {
          var total = 0;
          this.checkGroup.forEach((v, i) => {
            total += v.price * v.number;
          });
          return total;
        },
        handleAll() {
          if (this.allCheck) {
            this.checkGroup = this.dataList;
          } else {
            this.checkGroup = [];
          }
        },
        checkOne() {
          // if (this.checkGroup.length == this.dataList.length) {
          //     this.allCheck = true
          // } else {
          //     this.allCheck = false
          // }
          this.allCheck = this.checkGroup.length === this.dataList.length;
        },
      },
    });
  </script>
</html>
```

#### 带加减

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
    <script src="js/vue.js"></script>
  </head>
  <body>
    <div id="app">
      <div class="container-fluid">
        <div class="row">
          <div class="col-md-6 col-md-offset-3">
            <h1 class="text-center">购物车</h1>
            <table class="table table-hover table-bordered">
              <tr>
                <td>商品</td>
                <td>价格</td>
                <td>数量</td>
                <td>
                  全选/全不选
                  <input
                    type="checkbox"
                    v-model="allCheck"
                    @change="handleAll"
                  />
                </td>
              </tr>
              <tr v-for="data in dataList">
                <td>{{data.name}}</td>
                <td>{{data.price}}</td>
                <td>
                  <button @click="handleCount(data)">-</button>
                  {{data.number}}
                  <button @click="data.number++">+</button>
                </td>
                <td>
                  <input
                    type="checkbox"
                    v-model="checkGroup"
                    :value="data"
                    @change="checkOne"
                  />
                </td>
              </tr>
            </table>
            <br />
            选中的商品: {{checkGroup}}
            <br />
            总价格: {{getPrice()}}
            <br />
            是否全选: {{allCheck}}
          </div>
        </div>
      </div>
    </div>
  </body>
  <script>
    var vm = new Vue({
      el: "#app",
      data: {
        dataList: [
          { name: "红楼梦", price: 99, number: 2 },
          { name: "西游记", price: 59, number: 1 },
          { name: "水浒传", price: 89, number: 5 },
        ],
        checkGroup: [],
        allCheck: false,
      },
      methods: {
        getPrice() {
          var total = 0;
          this.checkGroup.forEach((v, i) => {
            total += v.price * v.number;
          });
          return total;
        },
        handleAll() {
          if (this.allCheck) {
            this.checkGroup = this.dataList;
          } else {
            this.checkGroup = [];
          }
        },
        checkOne() {
          this.allCheck = this.checkGroup.length === this.dataList.length;
        },
        handleCount(item) {
          if (item.number == 1) {
            alert("不能再减了");
          } else {
            item.number--;
          }
        },
      },
    });
  </script>
</html>
```

### 生命周期钩子函数

```python
# new Vue这个对象 ---> 管理一个标签 ---> 把数据，渲染到页面上
# 组件 ---> 对象管理某一个html片段
# 生命周期 ---> 8个声明周期钩子函数 ---> 执行到某个地方，就会执行某个函数
  钩子函数           描述
  beforeCreate	   创建Vue实例之前调用，data空的
  created	         创建Vue实例成功后调用
  beforeMount	     渲染DOM之前调用
  mounted	         渲染DOM之后调用 ---> 看到页面了，插值已经进去了

  beforeUpdate	   重新渲染之前调用（数据更新等操作时，控制DOM重新渲染）
  updated	         重新渲染完成之后调用

  beforeDestroy	销毁之前调用
  destroyed	销毁之后调用

# 有用的：
	created：向后端发请求拿数据，发送ajax请求
  mounted：定时任务，延迟任务  js中
  beforeDestroy：定时任务关闭，销毁一些操作

# 定时器的开启与关闭
 this.t = setInterval(() => {
                console.log('daada')
            }, 3000)

clearInterval(this.t)
this.t = null
```

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Title</title>
    <script src="./js/vue.js"></script>
    <link
      href="https://stackpath.bootstrapcdn.com/bootstrap/3.4.1/css/bootstrap.min.css"
      rel="stylesheet"
    />
  </head>
  <body>
    <div id="app">
      <button @click="handleC">点我显示组件</button>
      <child v-if="is_show"></child>
      <hr />
    </div>
  </body>
  <script>
    // 1 定义个组件 ---> 生命周期
    Vue.component("child", {
      template: `
            <div>
                <h1>{{name}}</h1>
                <button @click="handleC">点我弹窗</button>
            </div>`,
      data() {
        return {
          name: "xxx",
          t: "",
        };
      },
      methods: {
        handleC() {
          this.name = "彭于晏";
          alert(this.name);
        },
      },
      // 生命周期钩子函数8个
      beforeCreate() {
        console.log("当前状态：beforeCreate");
        console.log("当前el状态：", this.$el);
        console.log("当前data状态：", this.$data);
        console.log("当前name状态：", this.name);
      },
      created() {
        // 向后端加载数据
        console.log("当前状态：created");
        console.log("当前el状态：", this.$el);
        console.log("当前data状态：", this.$data);
        console.log("当前name状态：", this.name);
      },

      beforeMount() {
        console.log("当前状态：beforeMount");
        console.log("当前el状态：", this.$el);
        console.log("当前data状态：", this.$data);
        console.log("当前name状态：", this.name);
      },
      mounted() {
        console.log("当前状态：mounted");
        console.log("当前el状态：", this.$el);
        console.log("当前data状态：", this.$data);
        console.log("当前name状 态：", this.name);
        //用的最多，向后端加载数据，创建定时器等
        // setTimeout:延迟执行
        // setInterval：定时执行,每三秒钟打印一下daada
        this.t = setInterval(() => {
          console.log("daada");
        }, 3000);
      },
      beforeUpdate() {
        console.log("当前状态：beforeUpdate");
        console.log("当前el状态：", this.$el);
        console.log("当前data状态：", this.$data);
        console.log("当前name状态：", this.name);
      },
      updated() {
        console.log("当前状态：updated");
        console.log("当前el状态：", this.$el);
        console.log("当前data状态：", this.$data);
        console.log("当前name状态：", this.name);
      },
      beforeDestroy() {
        console.log("当前状态：beforeDestroy");
        console.log("当前el状态：", this.$el);
        console.log("当前data状态：", this.$data);
        console.log("当前name状态：", this.name);
      },
      destroyed() {
        console.log("当前状态：destroyed");
        console.log("当前el状态：", this.$el);
        console.log("当前data状态：", this.$data);
        console.log("当前name状态：", this.name);
        //组件销毁，清理定时器
        clearInterval(this.t);
        this.t = null;
        // console.log('destoryed')
      },
    });
    var vm = new Vue({
      el: "#app",
      data: {
        is_show: false,
      },
      methods: {
        handleC() {
          this.is_show = !this.is_show;
        },
      },
    });
  </script>
</html>
```

### 与后端交互

```
ajax:异步的xml请求，前后端交互就是xml格式，随着json格式发展，目前都是使用json格式
jquery的ajax方法   $.ajax()  方法 ---> 只是方法名正好叫ajax
js原生可以写ajax请求，非常麻烦，考虑兼容性 ---> jquery

方式一：jquery的ajax方法发送请求(基本不用了)
方式二：js官方提供的fetch方法(XMLHttpRequest)（官方的，用的也少）
方式三：axios第三方，做ajax请求
```

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Title</title>
    <script src="js/vue.js"></script>
    <script src="https://unpkg.com/axios/dist/axios.min.js"></script>
    <script src="http://code.jquery.com/jquery-2.1.1.min.js"></script>
  </head>
  <body>
    <div id="app">{{text}}</div>
  </body>
  <script>
    var vm = new Vue({
      el: "#app",
      data: {
        text: "",
      },
      created() {
        // 方式一:
        //向后端发请求，拿数据，拿回来赋值给text
        // $.ajax({
        //     url:'http://127.0.0.1:5000/',
        //     type:'get',
        //     success:(data) =>{
        //         console.log(data)
        //         this.text=data
        //     }
        // })

        // 方式二：js原生的fetch
        // fetch('http://127.0.0.1:5000/').then(res => res.json()).then(res => {
        //     console.log(res)
        //     this.text=res.name
        //
        // })

        // 方式三 axios
        axios.get("http://127.0.0.1:5000").then((data) => {
          console.log(data.data);
          this.text = data.data.name;
        });
      },
    });
  </script>
</html>
```

#### 案例

##### `move.html`

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Title</title>
    <script src="js/vue.js"></script>
    <script src="https://unpkg.com/axios/dist/axios.min.js"></script>
    <script src="http://code.jquery.com/jquery-2.1.1.min.js"></script>
  </head>
  <body>
    <div id="app">
      <ul v-for="film in films_list">
        <li>
          <p>电影名字是: {{film.name}}</p>
          <img :src="film.poster" alt="" width="100px" height="150px" />
          <p>电影介绍: {{film.synopsis}}</p>
        </li>
      </ul>
    </div>
  </body>
  <script>
    var vm = new Vue({
      el: "#app",
      data: {
        films_list: [],
      },
      created() {
        axios.get("http://127.0.0.1:5000/films").then((res) => {
          console.log(res.data);
          this.films_list = res.data.data.films;
        });
      },
    });
  </script>
</html>
```

##### `server.py`

```python
from flask import Flask, make_response, jsonify

app = Flask(__name__)


@app.route('/')
def index():
    obj = make_response(jsonify({'name': 'tom', 'age': 18}))
    obj.headers['Access-Control-Allow-Origin'] = '*'
    return obj


@app.route('/films')
def films():
    import json
    with open('./res.json', 'r', encoding='utf-8') as f:
        res = json.load(f)
    obj = make_response(jsonify(res))
    obj.headers['Access-Control-Allow-Origin'] = '*'
    return obj


if __name__ == '__main__':
    app.run()
```

`res.json`

```json
{
  "status": 0,
  "data": {
    "films": [
      {
        "filmId": 5931,
        "name": "致我的陌生恋人",
        "poster": "https://pic.maizuo.com/usr/movie/923ddd6da070a705d533b48c9eb9996d.jpg",
        "actors": [
          {
            "name": "雨果·热兰",
            "role": "导演",
            "avatarAddress": "https://pic.maizuo.com/usr/movie/e96d642a2c613bb38010394e77770a9d.jpg"
          },
          {
            "name": "弗朗索瓦·西维尔",
            "role": "Raphaël Ramisse / Zoltan",
            "avatarAddress": "https://pic.maizuo.com/usr/movie/9f20b16cbf161f28ad90e21dcd57abd2.jpg"
          },
          {
            "name": "约瑟芬·约比",
            "role": "Olivia Marigny / Shadow",
            "avatarAddress": "https://pic.maizuo.com/usr/movie/b0b7ee851580bce62c00985cf1a1c061.jpg"
          },
          {
            "name": "本杰明·拉维赫尼",
            "role": "Félix / Gumpar",
            "avatarAddress": "https://pic.maizuo.com/usr/movie/21ced090240371f98b1d92a842a2398a.jpg"
          },
          {
            "name": "卡米尔·勒鲁什",
            "role": "Mélanie",
            "avatarAddress": "https://pic.maizuo.com/usr/movie/f0a2cbd274934de44bc02568c751ea19.jpg"
          }
        ],
        "director": "雨果·热兰",
        "category": "喜剧|爱情",
        "synopsis": "穿过千千万万时间线，跨越漫长岁月去寻找曾经的爱人。只是平行时空，重新认识，她还会爱上他吗？一次激烈的争吵，一场意外的时空旅行，拉斐尔（弗朗索瓦·西维尔 饰）从一名成功的畅销书作家，变成平庸的中学语文老师；妻子奥莉薇亚（约瑟芬·约比  饰）从家庭主妇成为了星光熠熠的著名钢琴家。再相遇，身份颠倒，不再是夫妻，平行时空又一次浪漫邂逅，拉斐尔能否守住最初的爱情？",
        "filmType": {
          "name": "2D",
          "value": 1
        },
        "nation": "法国",
        "language": "",
        "videoId": "",
        "premiereAt": 1649894400,
        "timeType": 3,
        "runtime": 118,
        "item": {
          "name": "2D",
          "type": 1
        },
        "isPresale": false,
        "isSale": false
      }
    ],
    "total": 9
  },
  "msg": "ok"
}
```

### 计算属性

```
插值的普通函数，只要页面一刷新，函数就会重新计算，跟函数无关的值的变化，函数也会重新计算
把函数当成属性来用 ---> 只有这个函数使用的属性(变量)变化，函数才重新运算
```

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Title</title>
    <script src="js/vue.js"></script>
  </head>
  <body>
    <div id="app">
      <input type="text" v-model="mytest1" /> >>>>>> {{mytest1}}
      <br />
      <input type="text" v-model="mytest2" /> >>>>>> {{mytest2.substring(0,
      1).toUpperCase() + mytest2.substring(1)}}
      <br />
      函数方式：{{getName1()}}
      <br />
      计算属性: {{getName2}}
    </div>
  </body>
  <script>
    var vm = new Vue({
      el: "#app",
      data: {
        mytest1: "",
        mytest2: "",
      },
      methods: {
        getName1() {
          console.log("执行了");
          return (
            this.mytest2.substring(0, 1).toUpperCase() +
            this.mytest2.substring(1)
          );
        },
      },
      computed: {
        getName2() {
          console.log("计算属性执行了");
          return (
            this.mytest2.substring(0, 1).toUpperCase() +
            this.mytest2.substring(1)
          );
        },
      },
    });
  </script>
</html>
```

#### 重写过滤案例

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Title</title>
    <script src="js/vue.js"></script>
  </head>
  <body>
    <div id="app">
      <p>
        <input type="text" v-model="Mytext" placeholder="请输入要筛选的值" />
      </p>
      <ul>
        <li v-for="data in newList">{{data}}</li>
      </ul>
    </div>
  </body>
  <script>
    var vm = new Vue({
      el: "#app",
      data: {
        Mytext: "",
        dataList: ["a", "at", "atom", "be", "beyond", "cs", "csrf"],
      },
      computed: {
        newList() {
          console.log("执行了");
          var dataList2 = this.dataList.filter((item) => {
            return item.indexOf(this.Mytext) > -1;
          });
          return dataList2;
        },
      },
    });
  </script>
</html>
```

### 侦听属性

```
只要变量发送变化，就会执行监听属性中的方法
```

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Title</title>
    <script src="js/vue.js"></script>
  </head>
  <body>
    <div id="app"><input type="text" v-model="mytext" /> {{mytext}}</div>
  </body>
  <script>
    var vm = new Vue({
      el: "#app",
      data: {
        mytext: "",
      },
      watch: {
        mytext: function () {
          console.log("我变化了，执行");
        },
      },
    });
  </script>
</html>
```

### 组件

```
扩展HTML元素，封装可重用的代码，目的是复用
  例如：有一个轮播，可以在很多页面中使用，一个轮播有js，css，html
  组件把js，css，html放到一起，有逻辑，有样式，有html

注意事项
  自定义组件需要一个root element,一般包裹在一个div中
  父子组件的data是无法共享的
  组件可以有data，methods，computed... 但是data 必须是一个函数
```

#### 局部组件

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Title</title>
    <script src="js/vue.js"></script>
  </head>
  <body>
    <div id="app">
      <Top></Top>
      <br />
      <Bottom></Bottom>
    </div>
  </body>
  <script>
    var vm = new Vue({
      el: "#app",
      data: {},
      // 定义再这里面的叫局部组件，只能再局部使用，只能再id为app的标签内使用
      components: {
        Top: {
          template: `
                  <div>
                  <h1 style="background-color: red;font-size: 40px;text-align: center">{{ name }}</h1>
                  <hr>
                  <button @click="handle1">点击弹窗</button>
                  </div>
                `,
          data() {
            return {
              name: "我是头部",
            };
          },
          methods: {
            handle1() {
              alert("弹窗");
            },
          },
        },
        Bottom: {
          template: `
                  <div>
                  <h1 style="background-color: yellow;font-size: 50px;text-align: center">{{ name }}</h1>
                  </div>>
                `,
          data() {
            return {
              name: "我是尾部",
            };
          },
        },
      },
    });
  </script>
</html>
```

#### 全局组件

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Title</title>
    <script src="js/vue.js"></script>
  </head>
  <body>
    <div id="app">
      <Top></Top>
    </div>
  </body>
  <script>
    Vue.component("Top", {
      template: `
                      <div>
                      <h1 style="background-color: greenyellow;font-size: 50px;text-align: center">{{ name }}</h1>
                      <hr>
                      <button @click="handle1">点击弹窗</button>
                      </div>
                    `,
      data() {
        return {
          name: "我是头部",
        };
      },
      methods: {
        handle1() {
          alert("这是弹窗");
        },
      },
    });
    var vm = new Vue({
      el: "#app",
      data: {},
    });
  </script>
</html>
```

### 组件通信

```
组件之间 data 数据不共享，数据传递
  从父组件到子组件
  	自定义属性

  从子组件到父组件
  	自定义事件
```

#### 父传子

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Title</title>
    <script src="js/vue.js"></script>
  </head>
  <body>
    <div id="app">
      <Top :myheader="headerName"></Top>
      {{headerName}}
      <br />
      <input type="text" v-model="headerName" />
    </div>
  </body>
  <script>
    Vue.component("Top", {
      template: `
                      <div>
                      <h1 style="background: pink;font-size: 60px;text-align: center">{{ myheader }}</h1>
                      </div>>
                    `,
      data() {
        return {
          name: "我是头部",
        };
      },
      props: {
        // 必须叫props，数组内放自定义属性的名字
        // props:['myheader',],
        // 属性验证
        myheader: String, // key是自定义属性名，value是类型名，如果是别的类型就报错
      },
    });
    var vm = new Vue({
      el: "#app",
      data: {
        headerName: "",
      },
    });
  </script>
</html>
```

#### 子传父

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Title</title>
    <script src="js/vue.js"></script>
  </head>
  <body>
    <div id="app">
      <Top @myevent="handelRecv"></Top>
      <hr />
      接收到子组件的数据: {{childText}}
    </div>
  </body>
  <script>
    Vue.component("Top", {
      template: `
                      <div>
                      <h1 style="background: pink;font-size: 60px;text-align: center">{{ myheader }}</h1>
                      <input type="text" v-model="mytext">
                      <button @click="handleSend">点我传出去</button>
                      </div>
                    `,
      data() {
        return {
          myheader: "我是头部",
          mytext: "",
        };
      },
      methods: {
        handleSend() {
          // 触发绑定在该组件上的事件，myevent ---> 父组件中会执行事件对应的函数 handelRecv
          this.$emit("myevent", this.mytext);
        },
      },
    });
    var vm = new Vue({
      el: "#app",
      data: {
        childText: "",
      },
      methods: {
        handelRecv(text) {
          // 接收一个参数，赋值给父组件的childText
          this.childText = text;
        },
      },
    });
  </script>
</html>
```

#### `ref`进行父子通信

```
ref属性，如果放在普通标签上，就是普通标签的原生html，操作，设置
ref属性，如果放在组件上，就是当前组件对象
```

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Title</title>
    <script src="js/vue.js"></script>
  </head>
  <body>
    <div id="app">
      <Top ref="Top"></Top>
      <input type="text" v-model="text" ref="myinput" />
      <hr />
      <img src="" alt="" ref="myimg" height="80px" />
      <button @click="handle2">点我</button>
    </div>
  </body>
  <script>
    Vue.component("Top", {
      template: `
                      <div>
                      <h1>{{ myheader }}</h1>
                      <button @click="handle1">点我</button>
                      <hr>
                      </div>
                    `,
      data() {
        return {
          myheader: "我是头部",
        };
      },
      methods: {
        handle1() {
          alert("弹窗出来了");
        },
      },
    });
    var vm = new Vue({
      el: "#app",
      data: {
        text: "",
      },
      methods: {
        handle2() {
          // console.log('被点了一下')
          // 所有有ref属性的标签 弄到一个对象中
          // console.log(this.$refs)
          // 1. ref放到 普通标签上
          // 取到input的value值
          // console.log(this.$refs.myinput.value)
          // 给ref属性的标签重新设定值
          // this.$refs.myinput.value = 'xxx DSB'
          // this.$refs.myimg.src = 'https://tva1.sinaimg.cn/large/00831rSTly1gd1u0jw182j30u00u043b.jpg'
          // 2. ref放到组件上
          // 拿到的是组件对象，因此组件内的data中的值也可以拿到，组件中的方法也可以调用
          // console.log(this.$refs.Top)
          // 拿到组件中data的值
          // console.log(this.$refs.Top.myheader)
          // 此时就不区分父子关系的传值了 可以双向
          // 子传父
          // this.text = this.$refs.Top.myheader
          // 父传子
          // this.$refs.Top.myheader = 'xxx DSB'
          // this.$refs.Top.myheader = this.text
          // 调用子组件中的方法
          // this.$refs.Top.handle1()
        },
      },
    });
  </script>
</html>
```

### 动态组件和`keep-alive`

```html
keep-alive 组件不销毁 componet 有个is 属性 指定显示的组件是哪个
```

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Title</title>
    <script src="js/vue.js"></script>
  </head>
  <body>
    <div id="app">
      <ul>
        <li @click="changeC('index')">首页</li>
        <li @click="changeC('order')">订单</li>
        <li @click="changeC('good')">商品</li>
      </ul>

      <!--    <index v-if="index_show"></index>-->
      <!--    <order v-if="order_show"></order>-->
      <!--    <good v-if="good_show"></good>-->

      <keep-alive>
        <component :is="who"></component>
      </keep-alive>
    </div>
  </body>
  <script>
    Vue.component("index", {
      template: `
            <div>
              <h1>我是首页</h1>
            </div>
            `,
    });
    Vue.component("order", {
      template: `
            <div>
              <h1>我是订单</h1>
              输入订单： <input type="text">
            </div>
            `,
    });
    Vue.component("good", {
      template: `
            <div>
              <h1>我是商品</h1>
            </div>
            `,
    });
    var vm = new Vue({
      el: "#app",
      data: {
        // index_show: true,
        // order_show: false,
        // good_show: false,
        who: "index",
      },
      methods: {
        changeC(i) {
          this.who = i;
        },
      },
    });
  </script>
</html>
```

### 插槽

#### 普通插槽

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Title</title>
    <script src="js/vue.js"></script>
  </head>
  <body>
    <div id="app">
      <index>
        <div>用户名：<input type="text" /> 密码：<input type="text" /></div>
      </index>
    </div>
  </body>
  <script>
    Vue.component("index", {
      template: `
            <div>
              <h1>我是首页</h1>
              <slot></slot>
            </div>
            `,
    });
    var vm = new Vue({
      el: "#app",
      data: {},
      methods: {},
    });
  </script>
</html>
```

#### 具名插槽

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Title</title>
    <script src="js/vue.js"></script>
  </head>
  <body>
    <div id="app">
      <index>
        <p slot="a">用户登录</p>
        <div slot="b">
          用户名：<input type="text" /> 密码：<input type="text" />
        </div>
      </index>
    </div>
  </body>
  <script>
    Vue.component("index", {
      template: `
            <div>
              <h1>我是首页</h1>
              <slot name="a"></slot>
              <slot name="b"></slot>
            </div>
            `,
    });
    var vm = new Vue({
      el: "#app",
      data: {},
      methods: {},
    });
  </script>
</html>
```

### `vue-cli`创建项目

#### 简介

```
单文件组件 ---> 一个文件，以 .vue 结尾，就是一个组件
vue-cli 创建项目，webpack构建，需要nodejs环境
	nodejs 是一门后端语言，JavaScript的解释型语言，只能运行在解释器中，浏览器中集成了js的解释器
	JavaScript只能运行在浏览器中，谷歌浏览器的v8引擎，运行在操作系统之上
	nodejs解释器上就可以运行JavaScript
```

#### `nodejs`安装

```
nodejs安装: http://nodejs.cn/
  node(python解释器)
  npm(pip)
  以往版本: https://registry.npmmirror.com/binary.html

Windows版本: 一路安装即可

Mac版本:
  # 解决普通用户的权限问题
  sudo chown -R $(whoami) $(sudo npm config get prefix)/{lib/node_modules,bin,share}

Linux版本:
  wget https://npmmirror.com/mirrors/node/v14.19.1/node-v14.19.1-linux-x64.tar.xz
  tar xf node-v14.19.1-linux-x64.tar.xz
  mv node-v14.19.1-linux-x64 /usr/local/node
  echo 'export PATH=$PATH:/usr/local/node/bin/' >> /etc/profile
  source /etc/profile

更换源和下载cnpm:
  npm config set registry https://registry.npmmirror.com
  npm config get registry
  npm install -g cnpm --registry=https://registry.npmmirror.com
```

#### `Vue`脚手架

```
npm install -g @vue/cli   # -g 全局
cnpm install -g @vue/cli  # cnpm替换了npm 解决了源的问题，和npm使用了淘宝源一样

创建项目:
  方式一: 命令
  vue create myfirstvue

  方式二: 图形化界面
  vue ui  # 启动一个服务，进入该服务可以页面创建vue项目
```

##### 命令行创建

![image-20220415184820538](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220415184820538.png)
![image-20220415185134280](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220415185134280.png)
![image-20220415185205010](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220415185205010.png)
![image-20220415185315378](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220415185315378.png)
![image-20220415185342118](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220415185342118.png)
![image-20220415185442622](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220415185442622.png)
![image-20220415185516063](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220415185516063.png)

##### UI 创建

![image-20220415185622038](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220415185622038.png)
![image-20220415185713104](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220415185713104.png)
![image-20220415185730486](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220415185730486.png)
![image-20220415185818860](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220415185818860.png)
![image-20220415185845458](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220415185845458.png)
![image-20220415185918726](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220415185918726.png)
![image-20220415185939014](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220415185939014.png)
![image-20220415190014835](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220415190014835.png)
![image-20220415190027373](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220415190027373.png)

### `Vue`项目目录介绍

```shell
myvue1                      # 项目名称
├── README.md
├── babel.config.js         # babel配置
├── jsconfig.json
├── node_modules            # 存放当前项目所有的依赖，删除之后项目无法运行，可以 npm install 重新装上，项目发送给别人和提交git时，该文件夹要删掉
├── package-lock.json
├── package.json            # 项目的所有依赖，类似 requirements.txt，npm install 根据这个文件下载的依赖
├── public                  # 文件夹
│   ├── favicon.ico         # 小图标
│   └── index.html          # 单页面开发，整个项目就这一个页面，不能动
├── src                     # 在该目录下写代码
│   ├── App.vue             # 根组件
│   ├── assets              # 存放静态资源的目录，img js css
│   ├── components          # 组件 xxx.vue 组件，小组件，给页面组件用
│   │   └── HelloWorld.vue  # 提供的默认组件，示例
│   ├── main.js             # 项目的入口
│   ├── router              # vue-router 就会有这个文件夹
│   │   └── index.js        # vue-router的js代码
│   ├── store               # Vuex 就会有这个文件夹
│   │   └── index.js        # Vuex 的js代码
│   └── views               # 组件，页面组件
│       ├── AboutView.vuV   # 默认提供了示例组件
│       └── HomeView.vue    # 默认提供了示例组件
├── .gitignore              # git的忽略文件配置
└── vue.config.js           # vue的配置
```

### `Vue`项目运行

```shell
# 在终端中 当前vue项目目录中执行以下命令
npm run serve

# pycharm管理
```

![image-20220415190549754](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220415190549754.png)

![image-20220415190637391](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220415190637391.png)

![image-20220415190707664](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220415190707664.png)

### `es6`语法导入导出

```
js模块化开发 ---> 模块，包的概念
```

#### 导出

```javascript
export default 对象

// 导出
// src/assets/js/settings.js
  let name = 'lxx'
  function printName() {
      console.log(name)
  }

  // 第一种导出方式
  export default printName

  // 第二种导出方式
  // export default {name: name,printName:printName}
  // export default {name, printName}
```

#### 导入

```javascript
import Vue from "vue"; // vue 在 node_modules 文件夹中，直接写名字就行
import 别名 from "路径"; // 自己写的包就要写路径

// 导入
// src/views/HomeView.vue  运行项目，在 console 上可以看到有打印 lxx
// 第一种导入方式的使用
import printName from "../assets/js/settings";
console.log(printName());

// 第二种导入方式的使用 拿到的就是导出的对象
import setting from "../assets/js/settings";
console.log(setting.printName());
console.log(setting.name);
```

#### 包

```javascript
// 创建 src/test/index.js 文件
let name = "Tom";
let age = 19;
export default { name, age };

// 使用 src/views/HomeView.vue
import test from "../test";
console.log(test.name);
console.log(test.age);

// @ 代表 /src
// import HelloWorld from '@/components/HelloWorld.vue'
```

### 定义并使用组件

#### 格式

```vue
新建一个 xx.vue ，包含三块

<template></template>
写原来模板字符串 `` html内容

<script>
export default {
  data() {
    return {
      name: "lxx",
    };
  },
};
</script>

<style scoped>
scoped 代表样式只在当前组件中生效
</style>
```

#### 示例

```vue
// src/components/MyAssembly.vue
<template>
  <div>
    <h1>{{ name }}</h1>
    <button @click="handle1">点我弹窗</button>
  </div>
</template>

<script>
export default {
  name: "MyAssembly",
  data() {
    return {
      name: "这个是标题",
    };
  },
  methods: {
    handle1() {
      alert("我出来啦");
    },
  },
};
</script>

<style scoped>
h1 {
  background: yellowgreen;
  font-size: 50px;
  text-align: center;
}
</style>
```

```Vue
// src/views/HomeView.vue
<template>
  <div class="home">
    <MyAssembly></MyAssembly>  // 使用
    <img alt="Vue logo" src="../assets/logo.png">
    <HelloWorld msg="Welcome to Your Vue.js App"/>
  </div>
</template>

<script>
// @ is an alias to /src
import HelloWorld from '@/components/HelloWorld.vue'
import MyAssembly from '@/components/MyAssembly.vue'  // 导入


export default {
  name: 'HomeView',
  components: {
    HelloWorld,
    MyAssembly,  // 注册进来
  }
}
</script>
```

### `Vue`使用插件

#### 使用`Bootstrap`和 `jQuery`

```
1.安装
  npm install bootstrap@3 -S  #  -S 表示把当前模块加入到package.json文件中
  npm install jquery -S

2.在main.js中配置
  import 'bootstrap'
  import 'bootstrap/dist/css/bootstrap.min.css'

3.vue.congig.js配置
    const {defineConfig} = require('@vue/cli-service')
    const webpack = require("webpack");
    module.exports = defineConfig({
        transpileDependencies: true,
        configureWebpack: {
            plugins: [
                new webpack.ProvidePlugin({
                    $: "jquery",
                    jQuery: "jquery",
                    "window.jQuery": "jquery",
                    "window.$": "jquery",
                    Popper: ["popper.js", "default"]
                })
            ]
        },
    })

4.使用
	<button @click="handle1" class="btn btn-success">点我弹窗</button>
```

#### 使用`Element UI`

```
1.安装
  npm install element-ui -S

2.main.js配置
  import ElementUI from 'element-ui';
  import 'element-ui/lib/theme-chalk/index.css';
  Vue.use(ElementUI);

3.使用参考官网 https://element.eleme.cn
  <el-button type="primary" @click="handle1">主要按钮</el-button>
```

### `Vue`项目与后端交互

```
1.安装
  npm install axios -S

2.main.js配置
  import axios from 'axios'      // 导入axios
  Vue.prototype.$axios = axios;  // 类的原型中放入一个变量 例如Python Person.$name = 'xxx' P.$name

3.使用 在任意组件中 this.$axios 就是 axios 对象
  this.$axios.get().then(res=>{})

4.其他用法 在任意组件中
  import axios from 'axios'
  axios.get('').then(res=>{})
```

#### 案例

##### Vue 相关

```vue
// main.js 使用了 element-ui 和 axios import ElementUI from 'element-ui'; import
'element-ui/lib/theme-chalk/index.css'; Vue.use(ElementUI); import axios from
'axios' Vue.prototype.$axios = axios;
```

```vue
// src/components/MyAssembly.vue
<template>
  <div>
    <el-row>
      <el-col
        :span="8"
        v-for="(o, index) in films_list"
        :key="o.id"
        :offset="index > 0 ? 2 : 0"
      >
        <el-card :body-style="{ padding: '0px' }">
          <img :src="o.poster" class="image" />
          <div style="padding: 14px;">
            <span>电影名称: {{ o.name }}</span>
            <br />
            <span>电影简介: {{ o.synopsis }}</span>
            <div class="bottom clearfix">
              <time class="time">{{ currentDate }}</time>
              <el-button type="text" class="button">操作按钮</el-button>
            </div>
          </div>
        </el-card>
      </el-col>
    </el-row>
  </div>
</template>

<script>
export default {
  name: "MyAssembly",
  data() {
    return {
      films_list: [],
      currentDate: new Date(),
    };
  },
  created() {
    this.$axios.get("http://127.0.0.1:5000/films").then((res) => {
      console.log(res.data);
      this.films_list = res.data.data.films;
    });
  },
};
</script>

<style scoped>
.time {
  font-size: 13px;
  color: #999;
}

.bottom {
  margin-top: 13px;
  line-height: 12px;
}

.button {
  padding: 0;
  float: right;
}

.image {
  width: 100%;
  display: block;
}

.clearfix:before,
.clearfix:after {
  display: table;
  content: "";
}

.clearfix:after {
  clear: both;
}
</style>
```

```vue
// src/views/HomeView.vue
<template>
  <div class="home">
    <img alt="Vue logo" src="../assets/logo.png" />
    <HelloWorld msg="Welcome to Your Vue.js App" />
    <MyAssembly></MyAssembly>
  </div>
</template>

<script>
// @ is an alias to /src
import HelloWorld from "@/components/HelloWorld.vue";
import MyAssembly from "@/components/MyAssembly.vue";

export default {
  name: "HomeView",
  components: {
    HelloWorld,
    MyAssembly,
  },
};
</script>
```

##### 后端代码

`server.py`

```python
from flask import Flask, make_response, jsonify

app = Flask(__name__)


@app.route('/')
def index():
    obj = make_response(jsonify({'name': 'tom', 'age': 18}))
    obj.headers['Access-Control-Allow-Origin'] = '*'
    return obj


@app.route('/films')
def films():
    import json
    with open('./res.json', 'r', encoding='utf-8') as f:
        res = json.load(f)
    obj = make_response(jsonify(res))
    obj.headers['Access-Control-Allow-Origin'] = '*'
    return obj


if __name__ == '__main__':
    app.run()
```

`res.json`

```json
{
  "status": 0,
  "data": {
    "films": [
      {
        "filmId": 5931,
        "name": "致我的陌生恋人",
        "poster": "https://pic.maizuo.com/usr/movie/923ddd6da070a705d533b48c9eb9996d.jpg",
        "actors": [
          {
            "name": "雨果·热兰",
            "role": "导演",
            "avatarAddress": "https://pic.maizuo.com/usr/movie/e96d642a2c613bb38010394e77770a9d.jpg"
          },
          {
            "name": "弗朗索瓦·西维尔",
            "role": "Raphaël Ramisse / Zoltan",
            "avatarAddress": "https://pic.maizuo.com/usr/movie/9f20b16cbf161f28ad90e21dcd57abd2.jpg"
          },
          {
            "name": "约瑟芬·约比",
            "role": "Olivia Marigny / Shadow",
            "avatarAddress": "https://pic.maizuo.com/usr/movie/b0b7ee851580bce62c00985cf1a1c061.jpg"
          },
          {
            "name": "本杰明·拉维赫尼",
            "role": "Félix / Gumpar",
            "avatarAddress": "https://pic.maizuo.com/usr/movie/21ced090240371f98b1d92a842a2398a.jpg"
          },
          {
            "name": "卡米尔·勒鲁什",
            "role": "Mélanie",
            "avatarAddress": "https://pic.maizuo.com/usr/movie/f0a2cbd274934de44bc02568c751ea19.jpg"
          }
        ],
        "director": "雨果·热兰",
        "category": "喜剧|爱情",
        "synopsis": "穿过千千万万时间线，跨越漫长岁月去寻找曾经的爱人。只是平行时空，重新认识，她还会爱上他吗？一次激烈的争吵，一场意外的时空旅行，拉斐尔（弗朗索瓦·西维尔 饰）从一名成功的畅销书作家，变成平庸的中学语文老师；妻子奥莉薇亚（约瑟芬·约比  饰）从家庭主妇成为了星光熠熠的著名钢琴家。再相遇，身份颠倒，不再是夫妻，平行时空又一次浪漫邂逅，拉斐尔能否守住最初的爱情？",
        "filmType": {
          "name": "2D",
          "value": 1
        },
        "nation": "法国",
        "language": "",
        "videoId": "",
        "premiereAt": 1649894400,
        "timeType": 3,
        "runtime": 118,
        "item": {
          "name": "2D",
          "type": 1
        },
        "isPresale": false,
        "isSale": false
      }
    ],
    "total": 9
  },
  "msg": "ok"
}
```
