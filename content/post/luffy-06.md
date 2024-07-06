---
title: 实战课页面功能
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - python
categories:
  - python
url: post/luffy-06.html
toc: true
---

### 课程页面

<!-- more -->

#### `views/LightCourse.vue`

```vue
<template>
  <div class="course">
    <Header></Header>
    <div class="main">
      <!-- 筛选条件 -->
      <div class="condition">
        <ul class="cate-list">
          <li class="title">课程分类:</li>
          <li class="this">全部</li>
          <li>轻课1</li>
          <li>轻课2</li>
        </ul>

        <div class="ordering">
          <ul>
            <li class="title">
              筛&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;选:
            </li>
            <li class="default this">默认</li>
            <li class="hot">人气</li>
            <li class="price">价格</li>
          </ul>
          <p class="condition-result">共21个课程</p>
        </div>
      </div>
      <!-- 课程列表 -->
      <div class="course-list">
        <div class="course-item">
          <div class="course-image">
            <img src="@/assets/img/course-cover.jpeg" alt="" />
          </div>
          <div class="course-info">
            <h3>
              Python开发21天入门
              <span
                ><img
                  src="@/assets/img/avatar1.svg"
                  alt=""
                />100人已加入学习</span
              >
            </h3>
            <p class="teather-info">
              Alex 金角大王 老男孩Python教学总监 <span>共154课时/更新完成</span>
            </p>
            <ul class="lesson-list">
              <li>
                <span class="lesson-title">01 | 第1节：初识编码</span>
                <span class="free">免费</span>
              </li>
              <li>
                <span class="lesson-title">01 | 第1节：初识编码初识编码</span>
                <span class="free">免费</span>
              </li>
              <li><span class="lesson-title">01 | 第1节：初识编码</span></li>
              <li>
                <span class="lesson-title">01 | 第1节：初识编码初识编码</span>
              </li>
            </ul>
            <div class="pay-box">
              <span class="discount-type">限时免费</span>
              <span class="discount-price">￥0.00元</span>
              <span class="original-price">原价：9.00元</span>
              <span class="buy-now">立即购买</span>
            </div>
          </div>
        </div>
        <div class="course-item">
          <div class="course-image">
            <img src="@/assets/img/course-cover.jpeg" alt="" />
          </div>
          <div class="course-info">
            <h3>
              Python开发21天入门
              <span
                ><img
                  src="@/assets/img/avatar1.svg"
                  alt=""
                />100人已加入学习</span
              >
            </h3>
            <p class="teather-info">
              Alex 金角大王 老男孩Python教学总监 <span>共154课时/更新完成</span>
            </p>
            <ul class="lesson-list">
              <li>
                <span class="lesson-title">01 | 第1节：初识编码</span>
                <span class="free">免费</span>
              </li>
              <li>
                <span class="lesson-title">01 | 第1节：初识编码初识编码</span>
                <span class="free">免费</span>
              </li>
              <li><span class="lesson-title">01 | 第1节：初识编码</span></li>
              <li>
                <span class="lesson-title">01 | 第1节：初识编码初识编码</span>
              </li>
            </ul>
            <div class="pay-box">
              <span class="discount-type">限时免费</span>
              <span class="discount-price">￥0.00元</span>
              <span class="original-price">原价：9.00元</span>
              <span class="buy-now">立即购买</span>
            </div>
          </div>
        </div>
      </div>
    </div>
    <Footer></Footer>
  </div>
</template>

<script>
import Header from "@/components/Header";
import Footer from "@/components/Footer";

export default {
  name: "Course",
  data() {
    return {
      category: 0,
    };
  },
  components: {
    Header,
    Footer,
  },
};
</script>

<style scoped>
.course {
  background: #f6f6f6;
}

.course .main {
  width: 1100px;
  margin: 35px auto 0;
}

.course .condition {
  margin-bottom: 35px;
  padding: 25px 30px 25px 20px;
  background: #fff;
  border-radius: 4px;
  box-shadow: 0 2px 4px 0 #f0f0f0;
}

.course .cate-list {
  border-bottom: 1px solid #333;
  border-bottom-color: rgba(51, 51, 51, 0.05);
  padding-bottom: 18px;
  margin-bottom: 17px;
}

.course .cate-list::after {
  content: "";
  display: block;
  clear: both;
}

.course .cate-list li {
  float: left;
  font-size: 16px;
  padding: 6px 15px;
  line-height: 16px;
  margin-left: 14px;
  position: relative;
  transition: all 0.3s ease;
  cursor: pointer;
  color: #4a4a4a;
  border: 1px solid transparent; /* transparent 透明 */
}

.course .cate-list .title {
  color: #888;
  margin-left: 0;
  letter-spacing: 0.36px;
  padding: 0;
  line-height: 28px;
}

.course .cate-list .this {
  color: #ffc210;
  border: 1px solid #ffc210 !important;
  border-radius: 30px;
}

.course .ordering::after {
  content: "";
  display: block;
  clear: both;
}

.course .ordering ul {
  float: left;
}

.course .ordering ul::after {
  content: "";
  display: block;
  clear: both;
}

.course .ordering .condition-result {
  float: right;
  font-size: 14px;
  color: #9b9b9b;
  line-height: 28px;
}

.course .ordering ul li {
  float: left;
  padding: 6px 15px;
  line-height: 16px;
  margin-left: 14px;
  position: relative;
  transition: all 0.3s ease;
  cursor: pointer;
  color: #4a4a4a;
}

.course .ordering .title {
  font-size: 16px;
  color: #888;
  letter-spacing: 0.36px;
  margin-left: 0;
  padding: 0;
  line-height: 28px;
}

.course .ordering .this {
  color: #ffc210;
}

.course .ordering .price {
  position: relative;
}

.course .ordering .price::before,
.course .ordering .price::after {
  cursor: pointer;
  content: "";
  display: block;
  width: 0px;
  height: 0px;
  border: 5px solid transparent;
  position: absolute;
  right: 0;
}

.course .ordering .price::before {
  border-bottom: 5px solid #aaa;
  margin-bottom: 2px;
  top: 2px;
}

.course .ordering .price::after {
  border-top: 5px solid #aaa;
  bottom: 2px;
}

.course .course-item:hover {
  box-shadow: 4px 6px 16px rgba(0, 0, 0, 0.5);
}

.course .course-item {
  width: 1100px;
  background: #fff;
  padding: 20px 30px 20px 20px;
  margin-bottom: 35px;
  border-radius: 2px;
  cursor: pointer;
  box-shadow: 2px 3px 16px rgba(0, 0, 0, 0.1);
  /* css3.0 过渡动画 hover 事件操作 */
  transition: all 0.2s ease;
}

.course .course-item::after {
  content: "";
  display: block;
  clear: both;
}

/* 顶级元素 父级元素  当前元素{} */
.course .course-item .course-image {
  float: left;
  width: 423px;
  height: 210px;
  margin-right: 30px;
}

.course .course-item .course-image img {
  width: 100%;
}

.course .course-item .course-info {
  float: left;
  width: 596px;
}

.course-item .course-info h3 {
  font-size: 26px;
  color: #333;
  font-weight: normal;
  margin-bottom: 8px;
}

.course-item .course-info h3 span {
  font-size: 14px;
  color: #9b9b9b;
  float: right;
  margin-top: 14px;
}

.course-item .course-info h3 span img {
  width: 11px;
  height: auto;
  margin-right: 7px;
}

.course-item .course-info .teather-info {
  font-size: 14px;
  color: #9b9b9b;
  margin-bottom: 14px;
  padding-bottom: 14px;
  border-bottom: 1px solid #333;
  border-bottom-color: rgba(51, 51, 51, 0.05);
}

.course-item .course-info .teather-info span {
  float: right;
}

.course-item .lesson-list::after {
  content: "";
  display: block;
  clear: both;
}

.course-item .lesson-list li {
  float: left;
  width: 44%;
  font-size: 14px;
  color: #666;
  padding-left: 22px;
  /* background: url("路径") 是否平铺 x轴位置 y轴位置 */
  background: url("/src/assets/img/play-icon-gray.svg") no-repeat left 4px;
  margin-bottom: 15px;
}

.course-item .lesson-list li .lesson-title {
  /* 以下3句，文本内容过多，会自动隐藏，并显示省略符号 */
  text-overflow: ellipsis;
  overflow: hidden;
  white-space: nowrap;
  display: inline-block;
  max-width: 200px;
}

.course-item .lesson-list li:hover {
  background-image: url("/src/assets/img/play-icon-yellow.svg");
  color: #ffc210;
}

.course-item .lesson-list li .free {
  width: 34px;
  height: 20px;
  color: #fd7b4d;
  vertical-align: super;
  margin-left: 10px;
  border: 1px solid #fd7b4d;
  border-radius: 2px;
  text-align: center;
  font-size: 13px;
  white-space: nowrap;
}

.course-item .lesson-list li:hover .free {
  color: #ffc210;
  border-color: #ffc210;
}

.course-item .pay-box::after {
  content: "";
  display: block;
  clear: both;
}

.course-item .pay-box .discount-type {
  padding: 6px 10px;
  font-size: 16px;
  color: #fff;
  text-align: center;
  margin-right: 8px;
  background: #fa6240;
  border: 1px solid #fa6240;
  border-radius: 10px 0 10px 0;
  float: left;
}

.course-item .pay-box .discount-price {
  font-size: 24px;
  color: #fa6240;
  float: left;
}

.course-item .pay-box .original-price {
  text-decoration: line-through;
  font-size: 14px;
  color: #9b9b9b;
  margin-left: 10px;
  float: left;
  margin-top: 10px;
}

.course-item .pay-box .buy-now {
  width: 120px;
  height: 38px;
  background: transparent;
  color: #fa6240;
  font-size: 16px;
  border: 1px solid #fd7b4d;
  border-radius: 3px;
  transition: all 0.2s ease-in-out;
  float: right;
  text-align: center;
  line-height: 38px;
}

.course-item .pay-box .buy-now:hover {
  color: #fff;
  background: #ffc210;
  border: 1px solid #ffc210;
}
</style>
```

#### `views/FreeCourse.vue`

```vue
<template>
  <div>
    <Header></Header>
    <img
      src="https://tva1.sinaimg.cn/large/e6c9d24egy1h1g112oiclj224l0u0jxl.jpg"
      alt=""
      height="500px"
      width="100%"
    />
    <Footer></Footer>
  </div>
</template>

<script>
import Header from "@/components/Header";
import Footer from "@/components/Footer";

export default {
  name: "ActualCourse",
  components: {
    Header,
    Footer,
  },
};
</script>

<style scoped></style>
```

#### `views/ActualCourse.vue`

```vue
<template>
  <div class="course">
    <Header></Header>
    <div class="main">
      <!-- 筛选条件 -->
      <div class="condition">
        <ul class="cate-list">
          <li class="title">课程分类:</li>
          <li class="this">全部</li>
          <li>Python</li>
          <li>Linux运维</li>
          <li>Python进阶</li>
          <li>开发工具</li>
          <li>Go语言</li>
          <li>机器学习</li>
          <li>技术生涯</li>
        </ul>

        <div class="ordering">
          <ul>
            <li class="title">
              筛&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;选:
            </li>
            <li class="default this">默认</li>
            <li class="hot">人气</li>
            <li class="price">价格</li>
          </ul>
          <p class="condition-result">共21个课程</p>
        </div>
      </div>
      <!-- 课程列表 -->
      <div class="course-list">
        <div class="course-item">
          <div class="course-image">
            <img src="@/assets/img/course-cover.jpeg" alt="" />
          </div>
          <div class="course-info">
            <h3>
              Python开发21天入门
              <span
                ><img
                  src="@/assets/img/avatar1.svg"
                  alt=""
                />100人已加入学习</span
              >
            </h3>
            <p class="teather-info">
              Alex 金角大王 老男孩Python教学总监 <span>共154课时/更新完成</span>
            </p>
            <ul class="lesson-list">
              <li>
                <span class="lesson-title">01 | 第1节：初识编码</span>
                <span class="free">免费</span>
              </li>
              <li>
                <span class="lesson-title">01 | 第1节：初识编码初识编码</span>
                <span class="free">免费</span>
              </li>
              <li><span class="lesson-title">01 | 第1节：初识编码</span></li>
              <li>
                <span class="lesson-title">01 | 第1节：初识编码初识编码</span>
              </li>
            </ul>
            <div class="pay-box">
              <span class="discount-type">限时免费</span>
              <span class="discount-price">￥0.00元</span>
              <span class="original-price">原价：9.00元</span>
              <span class="buy-now">立即购买</span>
            </div>
          </div>
        </div>
        <div class="course-item">
          <div class="course-image">
            <img src="@/assets/img/course-cover.jpeg" alt="" />
          </div>
          <div class="course-info">
            <h3>
              Python开发21天入门
              <span
                ><img
                  src="@/assets/img/avatar1.svg"
                  alt=""
                />100人已加入学习</span
              >
            </h3>
            <p class="teather-info">
              Alex 金角大王 老男孩Python教学总监 <span>共154课时/更新完成</span>
            </p>
            <ul class="lesson-list">
              <li>
                <span class="lesson-title">01 | 第1节：初识编码</span>
                <span class="free">免费</span>
              </li>
              <li>
                <span class="lesson-title">01 | 第1节：初识编码初识编码</span>
                <span class="free">免费</span>
              </li>
              <li><span class="lesson-title">01 | 第1节：初识编码</span></li>
              <li>
                <span class="lesson-title">01 | 第1节：初识编码初识编码</span>
              </li>
            </ul>
            <div class="pay-box">
              <span class="discount-type">限时免费</span>
              <span class="discount-price">￥0.00元</span>
              <span class="original-price">原价：9.00元</span>
              <span class="buy-now">立即购买</span>
            </div>
          </div>
        </div>
        <div class="course-item">
          <div class="course-image">
            <img src="@/assets/img/course-cover.jpeg" alt="" />
          </div>
          <div class="course-info">
            <h3>
              Python开发21天入门
              <span
                ><img
                  src="@/assets/img/avatar1.svg"
                  alt=""
                />100人已加入学习</span
              >
            </h3>
            <p class="teather-info">
              Alex 金角大王 老男孩Python教学总监 <span>共154课时/更新完成</span>
            </p>
            <ul class="lesson-list">
              <li>
                <span class="lesson-title">01 | 第1节：初识编码</span>
                <span class="free">免费</span>
              </li>
              <li>
                <span class="lesson-title">01 | 第1节：初识编码初识编码</span>
                <span class="free">免费</span>
              </li>
              <li><span class="lesson-title">01 | 第1节：初识编码</span></li>
              <li>
                <span class="lesson-title">01 | 第1节：初识编码初识编码</span>
              </li>
            </ul>
            <div class="pay-box">
              <span class="discount-type">限时免费</span>
              <span class="discount-price">￥0.00元</span>
              <span class="original-price">原价：9.00元</span>
              <span class="buy-now">立即购买</span>
            </div>
          </div>
        </div>
        <div class="course-item">
          <div class="course-image">
            <img src="@/assets/img/course-cover.jpeg" alt="" />
          </div>
          <div class="course-info">
            <h3>
              Python开发21天入门
              <span
                ><img
                  src="@/assets/img/avatar1.svg"
                  alt=""
                />100人已加入学习</span
              >
            </h3>
            <p class="teather-info">
              Alex 金角大王 老男孩Python教学总监 <span>共154课时/更新完成</span>
            </p>
            <ul class="lesson-list">
              <li>
                <span class="lesson-title">01 | 第1节：初识编码</span>
                <span class="free">免费</span>
              </li>
              <li>
                <span class="lesson-title">01 | 第1节：初识编码初识编码</span>
                <span class="free">免费</span>
              </li>
              <li><span class="lesson-title">01 | 第1节：初识编码</span></li>
              <li>
                <span class="lesson-title">01 | 第1节：初识编码初识编码</span>
              </li>
            </ul>
            <div class="pay-box">
              <span class="discount-type">限时免费</span>
              <span class="discount-price">￥0.00元</span>
              <span class="original-price">原价：9.00元</span>
              <span class="buy-now">立即购买</span>
            </div>
          </div>
        </div>
      </div>
    </div>
    <!--<Footer></Footer>-->
  </div>
</template>

<script>
import Header from "@/components/Header";
// import Footer from "@/components/Footer"

export default {
  name: "Course",
  data() {
    return {
      category: 0,
    };
  },
  components: {
    Header,
    // Footer,
  },
};
</script>

<style scoped>
.course {
  background: #f6f6f6;
}

.course .main {
  width: 1100px;
  margin: 35px auto 0;
}

.course .condition {
  margin-bottom: 35px;
  padding: 25px 30px 25px 20px;
  background: #fff;
  border-radius: 4px;
  box-shadow: 0 2px 4px 0 #f0f0f0;
}

.course .cate-list {
  border-bottom: 1px solid #333;
  border-bottom-color: rgba(51, 51, 51, 0.05);
  padding-bottom: 18px;
  margin-bottom: 17px;
}

.course .cate-list::after {
  content: "";
  display: block;
  clear: both;
}

.course .cate-list li {
  float: left;
  font-size: 16px;
  padding: 6px 15px;
  line-height: 16px;
  margin-left: 14px;
  position: relative;
  transition: all 0.3s ease;
  cursor: pointer;
  color: #4a4a4a;
  border: 1px solid transparent; /* transparent 透明 */
}

.course .cate-list .title {
  color: #888;
  margin-left: 0;
  letter-spacing: 0.36px;
  padding: 0;
  line-height: 28px;
}

.course .cate-list .this {
  color: #ffc210;
  border: 1px solid #ffc210 !important;
  border-radius: 30px;
}

.course .ordering::after {
  content: "";
  display: block;
  clear: both;
}

.course .ordering ul {
  float: left;
}

.course .ordering ul::after {
  content: "";
  display: block;
  clear: both;
}

.course .ordering .condition-result {
  float: right;
  font-size: 14px;
  color: #9b9b9b;
  line-height: 28px;
}

.course .ordering ul li {
  float: left;
  padding: 6px 15px;
  line-height: 16px;
  margin-left: 14px;
  position: relative;
  transition: all 0.3s ease;
  cursor: pointer;
  color: #4a4a4a;
}

.course .ordering .title {
  font-size: 16px;
  color: #888;
  letter-spacing: 0.36px;
  margin-left: 0;
  padding: 0;
  line-height: 28px;
}

.course .ordering .this {
  color: #ffc210;
}

.course .ordering .price {
  position: relative;
}

.course .ordering .price::before,
.course .ordering .price::after {
  cursor: pointer;
  content: "";
  display: block;
  width: 0px;
  height: 0px;
  border: 5px solid transparent;
  position: absolute;
  right: 0;
}

.course .ordering .price::before {
  border-bottom: 5px solid #aaa;
  margin-bottom: 2px;
  top: 2px;
}

.course .ordering .price::after {
  border-top: 5px solid #aaa;
  bottom: 2px;
}

.course .course-item:hover {
  box-shadow: 4px 6px 16px rgba(0, 0, 0, 0.5);
}

.course .course-item {
  width: 1100px;
  background: #fff;
  padding: 20px 30px 20px 20px;
  margin-bottom: 35px;
  border-radius: 2px;
  cursor: pointer;
  box-shadow: 2px 3px 16px rgba(0, 0, 0, 0.1);
  /* css3.0 过渡动画 hover 事件操作 */
  transition: all 0.2s ease;
}

.course .course-item::after {
  content: "";
  display: block;
  clear: both;
}

/* 顶级元素 父级元素  当前元素{} */
.course .course-item .course-image {
  float: left;
  width: 423px;
  height: 210px;
  margin-right: 30px;
}

.course .course-item .course-image img {
  width: 100%;
}

.course .course-item .course-info {
  float: left;
  width: 596px;
}

.course-item .course-info h3 {
  font-size: 26px;
  color: #333;
  font-weight: normal;
  margin-bottom: 8px;
}

.course-item .course-info h3 span {
  font-size: 14px;
  color: #9b9b9b;
  float: right;
  margin-top: 14px;
}

.course-item .course-info h3 span img {
  width: 11px;
  height: auto;
  margin-right: 7px;
}

.course-item .course-info .teather-info {
  font-size: 14px;
  color: #9b9b9b;
  margin-bottom: 14px;
  padding-bottom: 14px;
  border-bottom: 1px solid #333;
  border-bottom-color: rgba(51, 51, 51, 0.05);
}

.course-item .course-info .teather-info span {
  float: right;
}

.course-item .lesson-list::after {
  content: "";
  display: block;
  clear: both;
}

.course-item .lesson-list li {
  float: left;
  width: 44%;
  font-size: 14px;
  color: #666;
  padding-left: 22px;
  /* background: url("路径") 是否平铺 x轴位置 y轴位置 */
  background: url("/src/assets/img/play-icon-gray.svg") no-repeat left 4px;
  margin-bottom: 15px;
}

.course-item .lesson-list li .lesson-title {
  /* 以下3句，文本内容过多，会自动隐藏，并显示省略符号 */
  text-overflow: ellipsis;
  overflow: hidden;
  white-space: nowrap;
  display: inline-block;
  max-width: 200px;
}

.course-item .lesson-list li:hover {
  background-image: url("/src/assets/img/play-icon-yellow.svg");
  color: #ffc210;
}

.course-item .lesson-list li .free {
  width: 34px;
  height: 20px;
  color: #fd7b4d;
  vertical-align: super;
  margin-left: 10px;
  border: 1px solid #fd7b4d;
  border-radius: 2px;
  text-align: center;
  font-size: 13px;
  white-space: nowrap;
}

.course-item .lesson-list li:hover .free {
  color: #ffc210;
  border-color: #ffc210;
}

.course-item .pay-box::after {
  content: "";
  display: block;
  clear: both;
}

.course-item .pay-box .discount-type {
  padding: 6px 10px;
  font-size: 16px;
  color: #fff;
  text-align: center;
  margin-right: 8px;
  background: #fa6240;
  border: 1px solid #fa6240;
  border-radius: 10px 0 10px 0;
  float: left;
}

.course-item .pay-box .discount-price {
  font-size: 24px;
  color: #fa6240;
  float: left;
}

.course-item .pay-box .original-price {
  text-decoration: line-through;
  font-size: 14px;
  color: #9b9b9b;
  margin-left: 10px;
  float: left;
  margin-top: 10px;
}

.course-item .pay-box .buy-now {
  width: 120px;
  height: 38px;
  background: transparent;
  color: #fa6240;
  font-size: 16px;
  border: 1px solid #fd7b4d;
  border-radius: 3px;
  transition: all 0.2s ease-in-out;
  float: right;
  text-align: center;
  line-height: 38px;
}

.course-item .pay-box .buy-now:hover {
  color: #fff;
  background: #ffc210;
  border: 1px solid #ffc210;
}
</style>
```

#### `router/index.js`

```js
import Vue from "vue";
import VueRouter from "vue-router";
import HomeView from "../views/HomeView.vue";
import ActualCourse from "@/views/ActualCourse";
import FreeCourse from "@/views/FreeCourse";
import LightCourse from "@/views/LightCourse";

Vue.use(VueRouter);

const routes = [
  {
    path: "/",
    name: "home",
    component: HomeView,
  },
  {
    path: "/home",
    name: "home",
    component: HomeView,
  },
  {
    path: "/actual-course",
    name: "ActualCourse",
    component: ActualCourse,
  },
  {
    path: "/free-course",
    name: "FreeCourse",
    component: FreeCourse,
  },
  {
    path: "/light-course",
    name: "LightCourse",
    component: LightCourse,
  },
];

const router = new VueRouter({
  mode: "history",
  base: process.env.BASE_URL,
  routes,
});

export default router;
```

### 课程表设计

```python
# 三类课程 由于课程内的字段不一样，应该一个课程一张表

# 以实战课为例
课程分类表
实战课表    # 和课程分类是 一对多
课程章节    # 和实战课表是 一对多
课时表      # 和课程章节是 一对多
老师表      # 和课程      一对多

# 评论表
```

#### 创建`app`并配置

```python
# 创建 app
cd luffy_api/apps
python ../../manage.py startapp course

# luffy_api/setting/dev.py 配置文件中注册 app
INSTALLED_APPS = [
    # ...
    'course',
]

# apps/course/urls.py
from django.urls import path, include
from rest_framework.routers import SimpleRouter
router = SimpleRouter()

urlpatterns = [
    path('', include(router.urls))
]

# 总路由配置 luffy_api/urls.py
from django.contrib import admin
from django.urls import path, include
from django.conf import settings
from django.views.static import serve

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/v1/home/', include('home.urls')),  # http://127.0.0.1:8000/api/v1/home/banner/
    path('api/v1/user/', include('user.urls')),  # http://127.0.0.1:8000/api/v1/user/
    path('api/v1/course/', include('course.urls')),  # http://127.0.0.1:8000/api/v1/course/
    path('media/<path:path>', serve, {'document_root': settings.MEDIA_ROOT})
]

# 创建图片目录并放置相关图片
luffy_api/media/courses
luffy_api/media/teacher
```

#### `course/models.py`

```python
from django.db import models
from utils.model import BaseModel


# 课程分类表
class CourseCategory(BaseModel):
    name = models.CharField(max_length=64, unique=True, verbose_name="分类名称")

    class Meta:
        db_table = "luffy_course_category"
        verbose_name = "分类"
        verbose_name_plural = verbose_name

    def __str__(self):
        return "%s" % self.name


# 实战课表
class Course(BaseModel):
    course_type = (
        (0, '付费'),
        (1, 'VIP专享'),
        (2, '学位课程')
    )
    level_choices = (
        (0, '初级'),
        (1, '中级'),
        (2, '高级'),
    )
    status_choices = (
        (0, '上线'),
        (1, '下线'),
        (2, '预上线'),
    )
    name = models.CharField(max_length=128, verbose_name="课程名称")
    # blank=True 后台录入时可以为空
    course_img = models.ImageField(upload_to="courses", max_length=255, verbose_name="封面图片", blank=True, null=True)
    course_type = models.SmallIntegerField(choices=course_type, default=0, verbose_name="付费类型")
    # 课程详情 详情页有
    brief = models.TextField(max_length=2048, verbose_name="详情介绍", null=True, blank=True)
    level = models.SmallIntegerField(choices=level_choices, default=0, verbose_name="难度等级")
    pub_date = models.DateField(verbose_name="发布日期", auto_now_add=True)
    period = models.IntegerField(verbose_name="建议学习周期(day)", default=7)
    attachment_path = models.FileField(upload_to="attachment", max_length=128, verbose_name="课件路径", blank=True,
                                       null=True)
    status = models.SmallIntegerField(choices=status_choices, default=0, verbose_name="课程状态")
    # 优化字段 和用户相关
    students = models.IntegerField(verbose_name="学习人数", default=0)
    sections = models.IntegerField(verbose_name="总课时数量", default=0)
    pub_sections = models.IntegerField(verbose_name="课时更新数量", default=0)
    price = models.DecimalField(max_digits=6, decimal_places=2, verbose_name="课程原价", default=0)
    # 一个老师可能有多个课程  写在多的一方  on_delete 关联的操作
    teacher = models.ForeignKey("Teacher", on_delete=models.DO_NOTHING, null=True, blank=True, verbose_name="授课老师")
    # 一个分类下有多个课程  写在多的一方
    course_category = models.ForeignKey("CourseCategory", on_delete=models.SET_NULL, db_constraint=False, null=True,
                                        blank=True,
                                        verbose_name="课程分类")

    class Meta:
        db_table = "luffy_course"
        verbose_name = "课程"
        verbose_name_plural = "课程"

    def __str__(self):
        return "%s" % self.name


# 课程章节
class CourseChapter(BaseModel):
    # 和课程是 一对多 一个课程有多个章节
    course = models.ForeignKey("Course", related_name='coursechapters', on_delete=models.CASCADE, verbose_name="课程名称")
    chapter = models.SmallIntegerField(verbose_name="第几章", default=1)
    name = models.CharField(max_length=128, verbose_name="章节标题")
    summary = models.TextField(verbose_name="章节介绍", blank=True, null=True)
    pub_date = models.DateField(verbose_name="发布日期", auto_now_add=True)

    class Meta:
        db_table = "luffy_course_chapter"
        verbose_name = "章节"
        verbose_name_plural = verbose_name

    def __str__(self):
        return "%s:(第%s章)%s" % (self.course, self.chapter, self.name)


# 课时表
class CourseSection(BaseModel):
    """课时"""
    section_type_choices = (
        (0, '文档'),
        (1, '练习'),
        (2, '视频')
    )
    # 和章节 一对多关系 一个章节有多个课时 写在多的一方
    chapter = models.ForeignKey("CourseChapter", related_name='coursesections', on_delete=models.CASCADE,
                                verbose_name="课程章节")
    name = models.CharField(max_length=128, verbose_name="课时标题")
    orders = models.PositiveSmallIntegerField(verbose_name="课时排序")
    section_type = models.SmallIntegerField(default=2, choices=section_type_choices, verbose_name="课时种类")
    section_link = models.CharField(max_length=255, blank=True, null=True, verbose_name="课时链接",
                                    help_text="若是video，填vid,若是文档，填link")
    duration = models.CharField(verbose_name="视频时长", blank=True, null=True, max_length=32)  # 仅在前端展示使用
    pub_date = models.DateTimeField(verbose_name="发布时间", auto_now_add=True)
    free_trail = models.BooleanField(verbose_name="是否可试看", default=False)

    class Meta:
        db_table = "luffy_course_Section"
        verbose_name = "课时"
        verbose_name_plural = verbose_name

    def __str__(self):
        return "%s-%s" % (self.chapter, self.name)


# 老师表
class Teacher(BaseModel):
    """导师"""
    role_choices = (
        (0, '讲师'),
        (1, '导师'),
        (2, '班主任'),
    )
    name = models.CharField(max_length=32, verbose_name="导师名")
    role = models.SmallIntegerField(choices=role_choices, default=0, verbose_name="导师身份")
    title = models.CharField(max_length=64, verbose_name="职位、职称")
    signature = models.CharField(max_length=255, verbose_name="导师签名", help_text="导师签名", blank=True, null=True)
    image = models.ImageField(upload_to="teacher", null=True, verbose_name="导师封面")
    brief = models.TextField(max_length=1024, verbose_name="导师描述")

    class Meta:
        db_table = "luffy_teacher"
        verbose_name = "导师"
        verbose_name_plural = verbose_name

    def __str__(self):
        return "%s" % self.name
```

#### 迁移数据

```python
python manage.py makemigrations
python manage.py migrate
```

#### `ForeignKey`属性补充

```python
# ForeignKey 属性on_delete可以选择如下：
	-CASCADE   级联删除，比如删除老师，老师关联的所有课程都删除---》危险系数太高
  					 作者和作者详情，就可以使用级联删除
  -DO_NOTHING  什么都不做
  -SET_DEFAULT 删了老师，课程这个字段设置成默认，配合default
  -SET_NULL    删了老师，课程中老师这个字段设置为空 null=True
  -SET(值/函数)  删除老师，执行函数，课程中老师这个字段设置为SET的值或函数的执行结果


# ForeignKey 属性 db_constraint
	-ForeignKey是外键---》实际上在数据库会有外键关系
  -实际上外键关系有好处---》做约束---》插入数据时，脏数据插入不进去
  -坏处--》插入速度慢---》插入的时候要校验约束
  -实际编码中，公司里，基本不用外键约束----》这些操作统统由程序员和程序约束--》提高速度
  -db_constraint 不建外键约束---》可以基于对象的跨表查询--》基于双下划线的连表查---》一点不受影响


# ForeignKey 属性 related_name
反向操作时，使用的字段名，用于代替原反向查询时的’表名_set’
反向操作：通过课程查询所有章节：course.表名小写_set.all()
如果写了related_name---》course.coursechapters.all()


# ForeignKey 属性 related_query_name
反向查询操作时，使用的连接前缀，用于替换表名。
原来 __链表查询，使用表名小写，写了它后，直接使用这个字段
```

#### 后台注册表

```python
# apps/course.admin.py
from django.contrib import admin
from .models import *

admin.site.register(CourseCategory)
admin.site.register(Course)
admin.site.register(CourseChapter)
admin.site.register(CourseSection)
admin.site.register(Teacher)
```

#### 录入数据

```sql
-- 老师表数据
INSERT INTO luffy_teacher(id, orders, is_show, is_delete, created_time, updated_time, name, role, title, signature, image, brief) VALUES (1, 1, 1, 0, '2019-07-14 13:44:19.661327', '2019-07-14 13:46:54.246271', 'Alex', 1, '老男孩Python教学总监', '金角大王', 'teacher/alex_icon.png', '老男孩教育CTO & CO-FOUNDER 国内知名PYTHON语言推广者 51CTO学院2016\2017年度最受学员喜爱10大讲师之一 多款开源软件作者 曾任职公安部、飞信、中金公司、NOKIA中国研究院、华尔街英语、ADVENT、汽车之家等公司');

INSERT INTO luffy_teacher(id, orders, is_show, is_delete, created_time, updated_time, name, role, title, signature, image, brief) VALUES (2, 2, 1, 0, '2019-07-14 13:45:25.092902', '2019-07-14 13:45:25.092936', 'Mjj', 0, '前美团前端项目组架构师', NULL, 'teacher/mjj_icon.png', '是马JJ老师, 一个集美貌与才华于一身的男人，搞过几年IOS，又转了前端开发几年，曾就职于美团网任高级前端开发，后来因为不同意王兴(美团老板)的战略布局而出家做老师去了，有丰富的教学经验，开起车来也毫不含糊。一直专注在前端的前沿技术领域。同时，爱好抽烟、喝酒、烫头(锡纸烫)。 我的最爱是前端，因为前端妹子多。');

INSERT INTO luffy_teacher(id, orders, is_show, is_delete, created_time, updated_time, name, role, title, signature, image, brief) VALUES (3, 3, 1, 0, '2019-07-14 13:46:21.997846', '2019-07-14 13:46:21.997880', 'Lyy', 0, '老男孩Linux学科带头人', NULL, 'teacher/lyy_icon.png', 'Linux运维技术专家，老男孩Linux金牌讲师，讲课风趣幽默、深入浅出、声音洪亮到爆炸');

-- 分类表数据
INSERT INTO luffy_course_category(id, orders, is_show, is_delete, created_time, updated_time, name) VALUES (1, 1, 1, 0, '2019-07-14 13:40:58.690413', '2019-07-14 13:40:58.690477', 'Python');

INSERT INTO luffy_course_category(id, orders, is_show, is_delete, created_time, updated_time, name) VALUES (2, 2, 1, 0, '2019-07-14 13:41:08.249735', '2019-07-14 13:41:08.249817', 'Linux');

-- 课程表数据
INSERT INTO luffy_course(id, orders, is_show, is_delete, created_time, updated_time, name, course_img, course_type, brief, level, pub_date, period, attachment_path, status, students, sections, pub_sections, price, course_category_id, teacher_id) VALUES (1, 1, 1, 0, '2019-07-14 13:54:33.095201', '2019-07-14 13:54:33.095238', 'Python开发21天入门', 'courses/alex_python.png', 0, 'Python从入门到入土&&&Python从入门到入土&&&Python从入门到入土&&&Python从入门到入土&&&Python从入门到入土&&&Python从入门到入土&&&Python从入门到入土&&&Python从入门到入土&&&Python从入门到入土&&&Python从入门到入土&&&Python从入门到入土&&&Python从入门到入土', 0, '2019-07-14', 21, '', 0, 231, 120, 120, 0.00, 1, 1);

INSERT INTO luffy_course(id, orders, is_show, is_delete, created_time, updated_time, name, course_img, course_type, brief, level, pub_date, period, attachment_path, status, students, sections, pub_sections, price, course_category_id, teacher_id) VALUES (2, 2, 1, 0, '2019-07-14 13:56:05.051103', '2019-07-14 13:56:05.051142', 'Python项目实战', 'courses/mjj_python.png', 0, '', 1, '2019-07-14', 30, '', 0, 340, 120, 120, 99.00, 1, 2);

INSERT INTO luffy_course(id, orders, is_show, is_delete, created_time, updated_time, name, course_img, course_type, brief, level, pub_date, period, attachment_path, status, students, sections, pub_sections, price, course_category_id, teacher_id) VALUES (3, 3, 1, 0, '2019-07-14 13:57:21.190053', '2019-07-14 13:57:21.190095', 'Linux系统基础5周入门精讲', 'courses/lyy_linux.png', 0, '', 0, '2019-07-14', 25, '', 0, 219, 100, 100, 39.00, 2, 3);

-- 章节表数据
INSERT INTO luffy_course_chapter(id, orders, is_show, is_delete, created_time, updated_time, chapter, name, summary, pub_date, course_id) VALUES (1, 1, 1, 0, '2019-07-14 13:58:34.867005', '2019-07-14 14:00:58.276541', 1, '计算机原理', '', '2019-07-14', 1);

INSERT INTO luffy_course_chapter(id, orders, is_show, is_delete, created_time, updated_time, chapter, name, summary, pub_date, course_id) VALUES (2, 2, 1, 0, '2019-07-14 13:58:48.051543', '2019-07-14 14:01:22.024206', 2, '环境搭建', '', '2019-07-14', 1);

INSERT INTO luffy_course_chapter(id, orders, is_show, is_delete, created_time, updated_time, chapter, name, summary, pub_date, course_id) VALUES (3, 3, 1, 0, '2019-07-14 13:59:09.878183', '2019-07-14 14:01:40.048608', 1, '项目创建', '', '2019-07-14', 2);

INSERT INTO luffy_course_chapter(id, orders, is_show, is_delete, created_time, updated_time, chapter, name, summary, pub_date, course_id) VALUES (4, 4, 1, 0, '2019-07-14 13:59:37.448626', '2019-07-14 14:01:58.709652', 1, 'Linux环境创建', '', '2019-07-14', 3);

-- 课时表数据
INSERT INTO luffy_course_Section(id, is_show, is_delete, created_time, updated_time, name, orders, section_type, section_link, duration, pub_date, free_trail, chapter_id) VALUES (1, 1, 0, '2019-07-14 14:02:33.779098', '2019-07-14 14:02:33.779135', '计算机原理上', 1, 2, NULL, NULL, '2019-07-14 14:02:33.779193', 1, 1);

INSERT INTO luffy_course_Section(id, is_show, is_delete, created_time, updated_time, name, orders, section_type, section_link, duration, pub_date, free_trail, chapter_id) VALUES (2, 1, 0, '2019-07-14 14:02:56.657134', '2019-07-14 14:02:56.657173', '计算机原理下', 2, 2, NULL, NULL, '2019-07-14 14:02:56.657227', 1, 1);

INSERT INTO luffy_course_Section(id, is_show, is_delete, created_time, updated_time, name, orders, section_type, section_link, duration, pub_date, free_trail, chapter_id) VALUES (3, 1, 0, '2019-07-14 14:03:20.493324', '2019-07-14 14:03:52.329394', '环境搭建上', 1, 2, NULL, NULL, '2019-07-14 14:03:20.493420', 0, 2);

INSERT INTO luffy_course_Section(id, is_show, is_delete, created_time, updated_time, name, orders, section_type, section_link, duration, pub_date, free_trail, chapter_id) VALUES (4, 1, 0, '2019-07-14 14:03:36.472742', '2019-07-14 14:03:36.472779', '环境搭建下', 2, 2, NULL, NULL, '2019-07-14 14:03:36.472831', 0, 2);

INSERT INTO luffy_course_Section(id, is_show, is_delete, created_time, updated_time, name, orders, section_type, section_link, duration, pub_date, free_trail, chapter_id) VALUES (5, 1, 0, '2019-07-14 14:04:19.338153', '2019-07-14 14:04:19.338192', 'web项目的创建', 1, 2, NULL, NULL, '2019-07-14 14:04:19.338252', 1, 3);

INSERT INTO luffy_course_Section(id, is_show, is_delete, created_time, updated_time, name, orders, section_type, section_link, duration, pub_date, free_trail, chapter_id) VALUES (6, 1, 0, '2019-07-14 14:04:52.895855', '2019-07-14 14:04:52.895890', 'Linux的环境搭建', 1, 2, NULL, NULL, '2019-07-14 14:04:52.895942', 1, 4);
```

### 课程分类接口

#### `apps/course/urls.py`

```python
from django.urls import path, include
from rest_framework.routers import SimpleRouter
from .views import CourseCategoryView

router = SimpleRouter()
# 127.0.0.1:8000/api/v1/course/category/  get
router.register('category', CourseCategoryView, 'category')
urlpatterns = [
    path('', include(router.urls))
]
```

#### `apps/course/serializer.py`

```python
from rest_framework import serializers
from .models import CourseCategory


class CourseCategorySerializer(serializers.ModelSerializer):
    class Meta:
        model = CourseCategory
        fields = ['id', 'name']
```

#### `apps/course/views.py`

```python
from rest_framework.viewsets import GenericViewSet
from rest_framework.mixins import ListModelMixin
from utils.APIResponse import APIResponse
from .models import CourseCategory
from .serializer import CourseCategorySerializer


class CourseCategoryView(GenericViewSet, ListModelMixin):
    queryset = CourseCategory.objects.filter(is_delete=False, is_show=True).order_by('orders')
    serializer_class = CourseCategorySerializer

    def list(self, request, *args, **kwargs):
        res = super().list(request, *args, **kwargs)
        return APIResponse(data=res.data)
```

### 课程列表接口

#### `apps/course/urls.py`

```python
from django.urls import path, include
from rest_framework.routers import SimpleRouter
from .views import CourseCategoryView,CourseView

router = SimpleRouter()
# 127.0.0.1:8000/api/v1/course/category/  get
router.register('category', CourseCategoryView, 'category')
# 127.0.0.1:8000/api/v1/course/actual/  get
router.register('actual', CourseView, 'actual')
urlpatterns = [
    path('', include(router.urls))
]
```

#### `apps/course/views.py`

```python
from rest_framework.viewsets import GenericViewSet
from rest_framework.mixins import ListModelMixin
from rest_framework.filters import OrderingFilter
from django_filters.rest_framework import DjangoFilterBackend
from utils.APIResponse import APIResponse
from .models import CourseCategory, Course
from .serializer import CourseCategorySerializer, CourseSerializer
from .pagination import CommonPageNumberPagination


# 课程列表接口
class CourseView(GenericViewSet, ListModelMixin):
    queryset = Course.objects.filter(is_delete=False, is_show=True).order_by('orders')
    serializer_class = CourseSerializer
    # 加入分页
    pagination_class = CommonPageNumberPagination
    # 加入排序  过滤 按课程分类过滤 django-filter
    filter_backends = [OrderingFilter, DjangoFilterBackend, ]
    ordering_fields = ['price', 'students']
    filter_fields = ['course_category']

    def list(self, request, *args, **kwargs):
        res = super().list(request, *args, **kwargs)
        return APIResponse(data=res.data)

'''
# 安装
  pip3 install django-filter

# 配置文件中注册
  INSTALLED_APPS = [
      'django_filters',
  ]

# 导入使用
from django_filters.rest_framework import DjangoFilterBackend
'''
```

#### `apps/course/serializer.py`

```python
from rest_framework import serializers
from .models import CourseCategory, Course, Teacher

class TeacherSerializer(serializers.ModelSerializer):
    class Meta:
        model = Teacher
        fields = [
            'id',
            'name',
            'role_name',
            'title',
            'signature',
            'image',
            'brief',
        ]

class CourseSerializer(serializers.ModelSerializer):
    teacher = TeacherSerializer()  # 子序列化

    class Meta:
        model = Course
        fields = [
            'id',
            'name',
            'course_img',
            'brief',  # 课程介绍--->后面课程详情使用同一个序列化类
            'attachment_path',  # 课件
            'pub_sections',  # 发布的课时数
            'price',  # 价格
            'students',  # 学习人数
            'period',  # 学习周期
            'sections',  # 总课时数

            'course_type_name',  # choice字段---》表模型中写
            'level_name',  # choice字段---》表模型中写
            'status_name',  # choice字段---》表模型中写

            'teacher',  # 表模型中写，序列化类中写，子序列化
            'section_list',  # 表模型中写 -章节--->Course表中没有---》重写：序列类写，表模型中写
        ]
```

#### `apps/course/models.py`

```python
from django.db import models
from utils.model import BaseModel


# 课程分类表
class CourseCategory(BaseModel):
    """分类"""
    name = models.CharField(max_length=64, unique=True, verbose_name="分类名称")

    class Meta:
        db_table = "luffy_course_category"
        verbose_name = "分类"
        verbose_name_plural = verbose_name

    def __str__(self):
        return "%s" % self.name


# 实战课表
class Course(BaseModel):
    # choice
    course_type = (
        (0, '付费'),
        (1, '超级VIP专享'),

    )
    level_choices = (
        (0, '初级'),
        (1, '中级'),
        (2, '高级'),
        (3, '特高级'),
        (4, '超神'),
    )
    status_choices = (
        (0, '上线'),
        (1, '下线'),
        (2, '预上线'),
    )
    # 课程名
    name = models.CharField(max_length=128, verbose_name="课程名称")
    # 课程图片  null：数据库可以为空，blank:后台管理录入的时候可以不填，
    course_img = models.ImageField(upload_to="courses", max_length=255, verbose_name="封面图片", blank=True, null=True)
    # 付费类型
    course_type = models.SmallIntegerField(choices=course_type, default=0, verbose_name="付费类型")
    # 详情介绍--》课程详情页面---》TextField---》bbs项目的文章详情，html内容
    brief = models.TextField(max_length=2048, verbose_name="详情介绍", null=True, blank=True)
    # 难度等级
    level = models.SmallIntegerField(choices=level_choices, default=0, verbose_name="难度等级")
    # 发布日期  课程录入一个时间---》没有发布---》发布是在网站上可以看到了
    pub_date = models.DateField(verbose_name="发布日期", auto_now_add=True)
    # 建议学习周期
    period = models.IntegerField(verbose_name="建议学习周期(day)", default=7)
    # 课件路径--》课程有课件  ppt，png，md---》压缩成zip
    attachment_path = models.FileField(upload_to="attachment", max_length=128, verbose_name="课件路径", blank=True,
                                       null=True)
    # 课程状态
    status = models.SmallIntegerField(choices=status_choices, default=0, verbose_name="课程状态")
    # 学习人数 ---》优化字段，正常课程跟用户是有关系的，不需要关联查询统计用户个数了
    students = models.IntegerField(verbose_name="学习人数", default=0)
    # 总课时数量---3个章节20课时的内容
    sections = models.IntegerField(verbose_name="总课时数量", default=0)
    # 课时更新数量  ---》3个章节20课时的内容现在只更新了10个
    pub_sections = models.IntegerField(verbose_name="课时更新数量", default=0)
    # 课程原价
    price = models.DecimalField(max_digits=6, decimal_places=2, verbose_name="课程原价", default=0)

    ### 关联字段---》老师---》一个老师有多门课程，关联字段写在多的一方，写在课程中
    teacher = models.ForeignKey("Teacher", on_delete=models.DO_NOTHING, null=True, blank=True, verbose_name="授课老师")
    ###  关联字段---》课程分类--->一个分类下有多个课程，关联字段写在多的一方
    course_category = models.ForeignKey("CourseCategory", on_delete=models.SET_NULL, db_constraint=False, null=True,
                                        blank=True,
                                        verbose_name="课程分类")

    class Meta:
        db_table = "luffy_course"
        verbose_name = "课程"
        verbose_name_plural = "课程"

    def __str__(self):
        return "%s" % self.name

    @property  # 返回课程类型的中文，不这么写，它是一个数字
    def course_type_name(self):
        return self.get_course_type_display()

    def level_name(self):
        return self.get_level_display()

    def status_name(self):
        return self.get_status_display()

    @property
    def section_list(self):
        sections = []
        # 如果课时小于等于四条，返回总课时，如果大于4条，最多返回4条
        # 第一步：通过课程拿到所有章节
        # course_chapter_list=self.coursechapter_set.all() # 不需要
        course_chapter_list = self.coursechapters.all()
        # 第二步：循环所有章节
        for course_chapter in course_chapter_list:
            # 第三步：通过章节，拿到该章节的所有课时
            course_section_list = course_chapter.coursesections.all()
            # 第四步：循环取出所有章节，追加到一个列表中，准备返回
            for course_section in course_section_list:
                sections.append({
                    'name': course_section.name,
                    'section_link': course_section.section_link,
                    'duration': course_section.duration,
                    'free_trail': course_section.free_trail,
                })
                if len(sections) >= 4:  # 当课程大于等于4时直接 return 出去
                    return sections
        # 在for循环外层
        return sections


# 课程章节
class CourseChapter(BaseModel):
    # 一对多，写在多的一方
    course = models.ForeignKey("Course", related_name='coursechapters', on_delete=models.CASCADE, verbose_name="课程名称")
    # 章节数字--->第几章
    chapter = models.SmallIntegerField(verbose_name="第几章", default=1)
    # 章节标题
    name = models.CharField(max_length=128, verbose_name="章节标题")
    # 章节介绍
    summary = models.TextField(verbose_name="章节介绍", blank=True, null=True)
    # 发布日期
    pub_date = models.DateField(verbose_name="发布日期", auto_now_add=True)

    class Meta:
        db_table = "luffy_course_chapter"
        verbose_name = "章节"
        verbose_name_plural = verbose_name

    def __str__(self):
        return "%s:(第%s章)%s" % (self.course, self.chapter, self.name)


# 课时表
class CourseSection(BaseModel):
    """课时"""
    section_type_choices = (
        (0, '文档'),
        (1, '练习'),
        (2, '视频')
    )
    # 跟章节一对多，关联字段写在多的一方
    chapter = models.ForeignKey("CourseChapter", related_name='coursesections', on_delete=models.CASCADE,
                                verbose_name="课程章节")

    # 课时名
    name = models.CharField(max_length=128, verbose_name="课时标题")
    # 重写字段
    orders = models.PositiveSmallIntegerField(verbose_name="课时排序")
    # 课时种类：视频，文档，练习
    section_type = models.SmallIntegerField(default=2, choices=section_type_choices, verbose_name="课时种类")
    # 课时链接：视频地址，文档地址
    section_link = models.CharField(max_length=255, blank=True, null=True, verbose_name="课时链接",
                                    help_text="若是video，填vid,若是文档，填link")
    # 视频时长 ，仅在前端展示使用
    duration = models.CharField(verbose_name="视频时长", blank=True, null=True, max_length=32)
    # 发布时间
    pub_date = models.DateTimeField(verbose_name="发布时间", auto_now_add=True)
    # 是否可试看  允许免费看几个视频
    free_trail = models.BooleanField(verbose_name="是否可试看", default=False)

    class Meta:
        db_table = "luffy_course_section"
        verbose_name = "课时"
        verbose_name_plural = verbose_name

    def __str__(self):
        return "%s-%s" % (self.chapter, self.name)


# 老师
class Teacher(BaseModel):
    """导师"""
    role_choices = (
        (0, '讲师'),
        (1, '导师'),
        (2, '班主任'),
    )
    # 老师名
    name = models.CharField(max_length=32, verbose_name="导师名")
    # 老师身份---》讲师，导师，班主任
    role = models.SmallIntegerField(choices=role_choices, default=0, verbose_name="导师身份")
    # 职位、职称
    title = models.CharField(max_length=64, verbose_name="职位、职称")
    # 导师签名
    signature = models.CharField(max_length=255, verbose_name="导师签名", help_text="导师签名", blank=True, null=True)
    # 老师图片
    image = models.ImageField(upload_to="teacher", null=True, verbose_name="导师封面")
    # 导师描述-->很详细-->html
    brief = models.TextField(max_length=1024, verbose_name="导师描述")

    class Meta:
        db_table = "luffy_teacher"
        verbose_name = "导师"
        verbose_name_plural = verbose_name

    def __str__(self):
        return "%s" % self.name

    def role_name(self):
        return self.get_role_display()
```

#### `apps/course/pagination.py`

```python
from rest_framework.pagination import PageNumberPagination

class CommonPageNumberPagination(PageNumberPagination):
    page_size = 2
    page_query_param = 'page'
    page_size_query_param = 'page_size'
    max_page_size = 10
```

### 前后端课程页面调通

#### `views/ActualCourse.vue`

```vue
<template>
  <div class="course">
    <Header></Header>
    <div class="main">
      <!-- 筛选条件 -->
      <div class="condition">
        <ul class="cate-list">
          <li class="title">课程分类:</li>
          <li
            :class="filter.course_category == 0 ? 'this' : ''"
            @click="filter.course_category = 0"
          >
            全部
          </li>
          <li
            :class="filter.course_category == category.id ? 'this' : ''"
            v-for="category in category_list"
            @click="filter.course_category = category.id"
            :key="category.name"
          >
            {{ category.name }}
          </li>
        </ul>

        <div class="ordering">
          <ul>
            <li class="title">
              筛&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;选:
            </li>
            <li
              class="default"
              :class="
                filter.ordering == 'id' || filter.ordering == '-id'
                  ? 'this'
                  : ''
              "
              @click="filter.ordering = '-id'"
            >
              默认
            </li>
            <li
              class="hot"
              :class="
                filter.ordering == 'students' || filter.ordering == '-students'
                  ? 'this'
                  : ''
              "
              @click="
                filter.ordering =
                  filter.ordering == '-students' ? 'students' : '-students'
              "
            >
              人气
            </li>
            <li
              class="price"
              :class="
                filter.ordering == 'price'
                  ? 'price_up this'
                  : filter.ordering == '-price'
                  ? 'price_down this'
                  : ''
              "
              @click="
                filter.ordering =
                  filter.ordering == '-price' ? 'price' : '-price'
              "
            >
              价格
            </li>
          </ul>
          <p class="condition-result">共{{ course_total }}个课程</p>
        </div>
      </div>
      <!-- 课程列表 -->
      <div class="course-list">
        <div
          class="course-item"
          v-for="course in course_list"
          :key="course.name"
        >
          <div class="course-image">
            <img :src="course.course_img" alt="" />
          </div>
          <div class="course-info">
            <h3>
              <router-link :to="'/actual/detail/' + course.id">{{
                course.name
              }}</router-link>
              <span
                ><img src="@/assets/img/avatar1.svg" alt="" />{{
                  course.students
                }}人已加入学习</span
              >
            </h3>
            <p class="teather-info">
              {{ course.teacher.name }} {{ course.teacher.title }}
              {{ course.teacher.signature }}
              <span v-if="course.sections > course.pub_sections"
                >共{{ course.sections }}课时/已更新{{
                  course.pub_sections
                }}课时</span
              >
              <span v-else>共{{ course.sections }}课时/更新完成</span>
            </p>
            <ul class="section-list">
              <li
                v-for="(section, key) in course.section_list"
                :key="section.name"
              >
                <span class="section-title"
                  >0{{ key + 1 }} | {{ section.name }}</span
                >
                <span class="free" v-if="section.free_trail">免费</span>
              </li>
            </ul>
            <div class="pay-box">
              <div v-if="course.discount_type">
                <span class="discount-type">{{ course.discount_type }}</span>
                <span class="discount-price">￥{{ course.real_price }}元</span>
                <span class="original-price">原价：{{ course.price }}元</span>
              </div>
              <span v-else class="discount-price">￥{{ course.price }}元</span>
              <span class="buy-now">立即购买</span>
            </div>
          </div>
        </div>
      </div>
      <div class="course_pagination block">
        <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page.sync="filter.page"
          :page-sizes="[2, 3, 5, 10]"
          :page-size="filter.page_size"
          layout="sizes, prev, pager, next"
          :total="course_total"
        >
        </el-pagination>
      </div>
    </div>
    <Footer></Footer>
  </div>
</template>

<script>
import Header from "@/components/Header";
import Footer from "@/components/Footer";

export default {
  name: "Course",
  data() {
    return {
      category_list: [], // 课程分类列表
      course_list: [], // 课程列表
      course_total: 0, // 当前课程的总数量
      filter: {
        course_category: 0, // 当前用户选择的课程分类，刚进入页面默认为全部，值为0
        ordering: "-id", // 数据的排序方式，默认值是-id，表示对于id进行降序排列
        page_size: 2, // 单页数据量
        page: 1,
      },
    };
  },
  created() {
    this.get_category(); // 加载课程分类
    this.get_course(); // 加载科创城
  },
  components: {
    Header,
    Footer,
  },
  watch: {
    "filter.course_category": function () {
      this.filter.page = 1;
      this.get_course();
    },
    "filter.ordering": function () {
      this.get_course();
    },
    "filter.page_size": function () {
      this.get_course();
    },
    "filter.page": function () {
      this.get_course();
    },
  },
  methods: {
    handleSizeChange(val) {
      // 每页数据量发生变化时执行的方法
      this.filter.page = 1;
      this.filter.page_size = val;
    },
    handleCurrentChange(val) {
      // 页码发生变化时执行的方法
      this.filter.page = val;
    },
    get_category() {
      // 获取课程分类信息
      this.$axios
        .get(`${this.$settings.base_url}course/category/`)
        .then((response) => {
          this.category_list = response.data.data;
        })
        .catch(() => {
          this.$message({
            message: "获取课程分类信息有误，请联系客服工作人员",
          });
        });
    },
    get_course() {
      // 排序
      let filters = {
        ordering: this.filter.ordering, // 排序
      };
      // 判决是否进行分类课程的展示
      if (this.filter.course_category > 0) {
        filters.course_category = this.filter.course_category;
      }

      // 设置单页数据量
      if (this.filter.page_size > 0) {
        filters.page_size = this.filter.page_size;
      } else {
        filters.page_size = 5;
      }

      // 设置当前页码
      if (this.filter.page > 1) {
        filters.page = this.filter.page;
      } else {
        filters.page = 1;
      }

      // 获取课程列表信息
      this.$axios
        .get(`${this.$settings.base_url}course/actual/`, {
          params: filters,
        })
        .then((response) => {
          this.course_list = response.data.data.results;
          this.course_total = response.data.data.count;
        })
        .catch(() => {
          this.$message({
            message: "获取课程信息有误，请联系客服工作人员",
          });
        });
    },
  },
};
</script>

<style scoped>
.course {
  background: #f6f6f6;
}

.course .main {
  width: 1100px;
  margin: 35px auto 0;
}

.course .condition {
  margin-bottom: 35px;
  padding: 25px 30px 25px 20px;
  background: #fff;
  border-radius: 4px;
  box-shadow: 0 2px 4px 0 #f0f0f0;
}

.course .cate-list {
  border-bottom: 1px solid #333;
  border-bottom-color: rgba(51, 51, 51, 0.05);
  padding-bottom: 18px;
  margin-bottom: 17px;
}

.course .cate-list::after {
  content: "";
  display: block;
  clear: both;
}

.course .cate-list li {
  float: left;
  font-size: 16px;
  padding: 6px 15px;
  line-height: 16px;
  margin-left: 14px;
  position: relative;
  transition: all 0.3s ease;
  cursor: pointer;
  color: #4a4a4a;
  border: 1px solid transparent; /* transparent 透明 */
}

.course .cate-list .title {
  color: #888;
  margin-left: 0;
  letter-spacing: 0.36px;
  padding: 0;
  line-height: 28px;
}

.course .cate-list .this {
  color: #ffc210;
  border: 1px solid #ffc210 !important;
  border-radius: 30px;
}

.course .ordering::after {
  content: "";
  display: block;
  clear: both;
}

.course .ordering ul {
  float: left;
}

.course .ordering ul::after {
  content: "";
  display: block;
  clear: both;
}

.course .ordering .condition-result {
  float: right;
  font-size: 14px;
  color: #9b9b9b;
  line-height: 28px;
}

.course .ordering ul li {
  float: left;
  padding: 6px 15px;
  line-height: 16px;
  margin-left: 14px;
  position: relative;
  transition: all 0.3s ease;
  cursor: pointer;
  color: #4a4a4a;
}

.course .ordering .title {
  font-size: 16px;
  color: #888;
  letter-spacing: 0.36px;
  margin-left: 0;
  padding: 0;
  line-height: 28px;
}

.course .ordering .this {
  color: #ffc210;
}

.course .ordering .price {
  position: relative;
}

.course .ordering .price::before,
.course .ordering .price::after {
  cursor: pointer;
  content: "";
  display: block;
  width: 0px;
  height: 0px;
  border: 5px solid transparent;
  position: absolute;
  right: 0;
}

.course .ordering .price::before {
  border-bottom: 5px solid #aaa;
  margin-bottom: 2px;
  top: 2px;
}

.course .ordering .price::after {
  border-top: 5px solid #aaa;
  bottom: 2px;
}

.course .ordering .price_up::before {
  border-bottom-color: #ffc210;
}

.course .ordering .price_down::after {
  border-top-color: #ffc210;
}

.course .course-item:hover {
  box-shadow: 4px 6px 16px rgba(0, 0, 0, 0.5);
}

.course .course-item {
  width: 1100px;
  background: #fff;
  padding: 20px 30px 20px 20px;
  margin-bottom: 35px;
  border-radius: 2px;
  cursor: pointer;
  box-shadow: 2px 3px 16px rgba(0, 0, 0, 0.1);
  /* css3.0 过渡动画 hover 事件操作 */
  transition: all 0.2s ease;
}

.course .course-item::after {
  content: "";
  display: block;
  clear: both;
}

/* 顶级元素 父级元素  当前元素{} */
.course .course-item .course-image {
  float: left;
  width: 423px;
  height: 210px;
  margin-right: 30px;
}

.course .course-item .course-image img {
  max-width: 100%;
  max-height: 210px;
}

.course .course-item .course-info {
  float: left;
  width: 596px;
}

.course-item .course-info h3 a {
  font-size: 26px;
  color: #333;
  font-weight: normal;
  margin-bottom: 8px;
}

.course-item .course-info h3 span {
  font-size: 14px;
  color: #9b9b9b;
  float: right;
  margin-top: 14px;
}

.course-item .course-info h3 span img {
  width: 11px;
  height: auto;
  margin-right: 7px;
}

.course-item .course-info .teather-info {
  font-size: 14px;
  color: #9b9b9b;
  margin-bottom: 14px;
  padding-bottom: 14px;
  border-bottom: 1px solid #333;
  border-bottom-color: rgba(51, 51, 51, 0.05);
}

.course-item .course-info .teather-info span {
  float: right;
}

.course-item .section-list::after {
  content: "";
  display: block;
  clear: both;
}

.course-item .section-list li {
  float: left;
  width: 44%;
  font-size: 14px;
  color: #666;
  padding-left: 22px;
  /* background: url("路径") 是否平铺 x轴位置 y轴位置 */
  background: url("/src/assets/img/play-icon-gray.svg") no-repeat left 4px;
  margin-bottom: 15px;
}

.course-item .section-list li .section-title {
  /* 以下3句，文本内容过多，会自动隐藏，并显示省略符号 */
  text-overflow: ellipsis;
  overflow: hidden;
  white-space: nowrap;
  display: inline-block;
  max-width: 200px;
}

.course-item .section-list li:hover {
  background-image: url("/src/assets/img/play-icon-yellow.svg");
  color: #ffc210;
}

.course-item .section-list li .free {
  width: 34px;
  height: 20px;
  color: #fd7b4d;
  vertical-align: super;
  margin-left: 10px;
  border: 1px solid #fd7b4d;
  border-radius: 2px;
  text-align: center;
  font-size: 13px;
  white-space: nowrap;
}

.course-item .section-list li:hover .free {
  color: #ffc210;
  border-color: #ffc210;
}

.course-item {
  position: relative;
}

.course-item .pay-box {
  position: absolute;
  bottom: 20px;
  width: 600px;
}

.course-item .pay-box::after {
  content: "";
  display: block;
  clear: both;
}

.course-item .pay-box .discount-type {
  padding: 6px 10px;
  font-size: 16px;
  color: #fff;
  text-align: center;
  margin-right: 8px;
  background: #fa6240;
  border: 1px solid #fa6240;
  border-radius: 10px 0 10px 0;
  float: left;
}

.course-item .pay-box .discount-price {
  font-size: 24px;
  color: #fa6240;
  float: left;
}

.course-item .pay-box .original-price {
  text-decoration: line-through;
  font-size: 14px;
  color: #9b9b9b;
  margin-left: 10px;
  float: left;
  margin-top: 10px;
}

.course-item .pay-box .buy-now {
  width: 120px;
  height: 38px;
  background: transparent;
  color: #fa6240;
  font-size: 16px;
  border: 1px solid #fd7b4d;
  border-radius: 3px;
  transition: all 0.2s ease-in-out;
  float: right;
  text-align: center;
  line-height: 38px;
  position: absolute;
  right: 0;
  bottom: 5px;
}

.course-item .pay-box .buy-now:hover {
  color: #fff;
  background: #ffc210;
  border: 1px solid #ffc210;
}

.course .course_pagination {
  margin-bottom: 60px;
  text-align: center;
}
</style>
```
