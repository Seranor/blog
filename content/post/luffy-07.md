---
title: 课程详情和搜索功能
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - python
categories:
  - python
url: post/luffy-07.html
toc: true
---

### 课程详情接口

```python
# 基于原来的课程列表接口，只需继承 RetrieveModelMixin
from rest_framework.mixins import ListModelMixin, RetrieveModelMixin

class CourseView(GenericViewSet, ListModelMixin, RetrieveModelMixin):
  # ...
```

<!-- more -->

### 课程章节接口

```python
查询所有章节，章节和课程有关联，根据课程 id 号，过滤章节
比如查询课程 id 为1的所有章节
本质就是查询所有章节带过滤功能
```

#### `apps/course/views.py`

```python
from rest_framework.viewsets import GenericViewSet
from rest_framework.mixins import ListModelMixin, RetrieveModelMixin
from rest_framework.filters import OrderingFilter
from django_filters.rest_framework import DjangoFilterBackend
from utils.APIResponse import APIResponse
from .models import CourseCategory, Course, CourseChapter
from .serializer import CourseCategorySerializer, CourseSerializer,CourseChapterSerializer
from .pagination import CommonPageNumberPagination

# 课程章节接口
class CourseChapterView(GenericViewSet, ListModelMixin, RetrieveModelMixin):
    queryset = CourseChapter.objects.filter(is_delete=False, is_show=True).order_by('orders')
    serializer_class = CourseChapterSerializer
    filter_backends = [DjangoFilterBackend, ]
    # 按照课程id过滤
    filter_fields = ['course_id']
```

#### `apps/course/serializer.py`

```python
from rest_framework import serializers
from .models import CourseCategory, Course, Teacher, CourseChapter, CourseSection

class CourseSectionSerializer(serializers.ModelSerializer):
    class Meta:
        model = CourseSection
        fields = ['name', 'orders', 'section_link', 'duration', 'free_trail']


class CourseChapterSerializer(serializers.ModelSerializer):
    coursesections = CourseSectionSerializer(many=True)
    class Meta:
        model = CourseChapter
        fields = ['id', 'chapter', 'name', 'coursesections']  # 返回该章节课时
```

#### `apps/course/urls.py`

```python
from django.urls import path, include
from rest_framework.routers import SimpleRouter
from .views import CourseCategoryView, CourseView, CourseChapterView

router = SimpleRouter()
# 127.0.0.1:8000/api/v1/course/category/  get
router.register('category', CourseCategoryView, 'category')
# 127.0.0.1:8000/api/v1/course/actual/  get
router.register('actual', CourseView, 'actual')
# 127.0.0.1:8000/api/v1/course/chapter/  get
router.register('chapter', CourseChapterView, 'chapter')
urlpatterns = [
    path('', include(router.urls))
]
```

### 课程章节和详情的页面

```javascript
// 取出详情页面课程id号
this.course_id = this.$route.params.id;
// console.log(this.$route)  // 当前路径
// console.log(this.$router)  // 路由对象
```

#### `vue-core-video-player `

```python
# 安装
npm install --save vue-core-video-player

# main.js 中引入
import VueCoreVideoPlayer from 'vue-core-video-player'
Vue.use(VueCoreVideoPlayer)  // 默认是英文的
Vue.use(VueCoreVideoPlayer, {
  lang: 'zh-CN'  // 中文
})

# 在组件html中使用
<div id="app">
  <div class="player-container">
    <vue-core-video-player src="http://img.ksbbs.com/asset/Mon_1703/05cacb4e02f9d9e.mp4"></vue-core-video-player>
  </div>
</div>

# https://www.cnblogs.com/liuqingzheng/p/16204851.html
# https://github.com/core-player/vue-core-video-player-examples
```

#### `router/index.js`

```javascript
// ...
import CourseDetail from "@/views/CourseDetail";

Vue.use(VueRouter);

const routes = [
  // ...
  {
    path: "/actual/detail/:id",
    name: "CourseDetail",
    component: CourseDetail,
  },
];
```

#### `views/CourseDetail.vue`

```vue
<template>
  <div class="detail">
    <Header />
    <div class="main">
      <div class="course-info">
        <div class="wrap-left">
          <vue-core-video-player
            :src="mp4_url"
            controls="auto"
            autoplay
            :muted="true"
            title="视频"
            @play="playFunc"
            @pause="pauseFunc"
          ></vue-core-video-player>
        </div>
        <div class="wrap-right">
          <h3 class="course-name">{{ course_info.name }}</h3>
          <p class="data">
            {{
              course_info.students
            }}人在学&nbsp;&nbsp;&nbsp;&nbsp;课程总时长：{{
              course_info.sections
            }}课时/{{
              course_info.pub_sections
            }}小时&nbsp;&nbsp;&nbsp;&nbsp;难度：{{ course_info.level_name }}
          </p>
          <div class="sale-time">
            <p class="sale-type">
              价格 <span class="original_price">¥{{ course_info.price }}</span>
            </p>
            <p class="expire"></p>
          </div>
          <div class="buy">
            <div class="buy-btn">
              <button class="buy-now" @click="go_pay(course_info)">
                立即购买
              </button>
              <button class="free">免费试学</button>
            </div>
            <div class="add-cart" @click="add_cart(course_info.id)">
              <img src="@/assets/img/cart-yellow.svg" alt="" />加入购物车
            </div>
          </div>
        </div>
      </div>
      <div class="course-tab">
        <ul class="tab-list">
          <li :class="tabIndex == 1 ? 'active' : ''" @click="tabIndex = 1">
            详情介绍
          </li>
          <li :class="tabIndex == 2 ? 'active' : ''" @click="tabIndex = 2">
            课程章节 <span :class="tabIndex != 2 ? 'free' : ''">(试学)</span>
          </li>
          <li :class="tabIndex == 3 ? 'active' : ''" @click="tabIndex = 3">
            用户评论
          </li>
          <li :class="tabIndex == 4 ? 'active' : ''" @click="tabIndex = 4">
            常见问题
          </li>
        </ul>
      </div>
      <div class="course-content">
        <div class="course-tab-list">
          <div class="tab-item" v-if="tabIndex == 1">
            <div class="course-brief" v-html="course_info.brief"></div>
          </div>
          <div class="tab-item" v-if="tabIndex == 2">
            <div class="tab-item-title">
              <p class="chapter">课程章节</p>
              <p class="chapter-length">
                共{{ course_chapters.length }}章
                {{ course_info.sections }}个课时
              </p>
            </div>
            <div
              class="chapter-item"
              v-for="chapter in course_chapters"
              :key="chapter.name"
            >
              <p class="chapter-title">
                <img src="@/assets/img/enum.svg" alt="" />第{{
                  chapter.chapter
                }}章·{{ chapter.name }}
              </p>
              <ul class="section-list">
                <li
                  class="section-item"
                  v-for="section in chapter.coursesections"
                  :key="section.name"
                >
                  <p class="name">
                    <span class="index"
                      >{{ chapter.chapter }}-{{ section.orders }}</span
                    >
                    {{ section.name
                    }}<span class="free" v-if="section.free_trail">免费</span>
                  </p>
                  <p class="time">
                    {{ section.duration }}
                    <img src="@/assets/img/chapter-player.svg" />
                  </p>
                  <button class="try" v-if="section.free_trail">
                    立即试学
                  </button>
                  <button class="try" v-else>立即购买</button>
                </li>
              </ul>
            </div>
          </div>
          <div class="tab-item" v-if="tabIndex == 3">用户评论</div>
          <div class="tab-item" v-if="tabIndex == 4">常见问题</div>
        </div>
        <div class="course-side">
          <div class="teacher-info">
            <h4 class="side-title"><span>授课老师</span></h4>
            <div class="teacher-content">
              <div class="cont1">
                <img :src="course_info.teacher.image" />
                <div class="name">
                  <p class="teacher-name">
                    {{ course_info.teacher.name }}
                    {{ course_info.teacher.title }}
                  </p>
                  <p class="teacher-title">
                    {{ course_info.teacher.signature }}
                  </p>
                </div>
              </div>
              <p class="narrative">{{ course_info.teacher.brief }}</p>
            </div>
          </div>
        </div>
      </div>
    </div>
    <Footer />
  </div>
</template>

<script>
import Header from "@/components/Header";
import Footer from "@/components/Footer";

export default {
  name: "Detail",
  data() {
    return {
      tabIndex: 2, // 当前选项卡显示的下标
      course_id: 0, // 当前课程信息的ID
      course_info: {
        teacher: {},
      }, // 课程信息
      course_chapters: [], // 课程的章节课时列表
      mp4_url: "http://img.ksbbs.com/asset/Mon_1703/05cacb4e02f9d9e.mp4",
      // 分辨率选择
      // mp4_url: [
      //     {
      //         src: 'http://rb1x3v1fm.sabkt.gdipper.com/%E8%87%B4%E5%91%BD%E8%AF%B1%E6%83%91320p.mp4',
      //         resolution: 360,
      //     },
      //     {
      //         src: 'http://rb1x3v1fm.sabkt.gdipper.com/%E8%87%B4%E5%91%BD%E8%AF%B1%E6%83%91720p.mp4',
      //         resolution: 720,
      //     },
      //     {
      //         src: 'http://rb1x3v1fm.sabkt.gdipper.com/%E8%87%B4%E5%91%BD%E8%AF%B1%E6%83%914k.mp4',
      //         resolution: '4k',
      //
      //     }],
    };
  },
  created() {
    this.get_course_id();
    this.get_course_data();
    this.get_chapter();
  },
  methods: {
    go_pay(course_info) {
      // 1 去cookie中取token，如果没有说明没登陆，不允许购买
      let token = this.$cookies.get("token");
      if (token) {
        this.$axios
          .post(
            this.$settings.base_url + "order/pay/",
            {
              subject: course_info.name,
              total_amount: course_info.price,
              pay_type: 1,
              courses: [course_info.id],
            },
            {
              headers: {
                authorization: "jwt " + token,
              },
            }
          )
          .then((res) => {
            if ((res.data.status = 100)) {
              let pay_url = res.data.pay_url;
              // 跳转,在当前窗口打开这个链接
              open(pay_url, "_self");
            } else {
              this.$message({
                message: "下单失败，请联系统管理员",
              });
            }
          });
      } else {
        this.$message({
          message: "对不起，您没有登录，请登陆后购买！",
        });
      }
    },
    playFunc() {
      console.log("开始了");
    },
    pauseFunc() {
      console.log("暂停了");
    },
    get_course_id() {
      // 获取地址栏上面的课程ID
      this.course_id = this.$route.params.id;
      if (this.course_id < 1) {
        let _this = this;
        _this.$alert("对不起，当前视频不存在！", "警告", {
          callback() {
            _this.$router.go(-1);
          },
        });
      }
    },
    get_course_data() {
      // ajax请求课程信息
      this.$axios
        .get(`${this.$settings.base_url}course/actual/${this.course_id}/`)
        .then((response) => {
          // window.console.log(response.data);
          this.course_info = response.data;
          console.log(this.course_info);
        })
        .catch(() => {
          this.$message({
            message: "对不起，访问页面出错！请联系客服工作人员！",
          });
        });
    },
    get_chapter() {
      // 获取当前课程对应的章节课时信息
      // http://127.0.0.1:8000/course/chapters/?course=(pk)
      this.$axios
        .get(`${this.$settings.base_url}course/chapter/`, {
          params: {
            course_id: this.course_id,
          },
        })
        .then((response) => {
          this.course_chapters = response.data;
        })
        .catch((error) => {
          window.console.log(error.response);
        });
    },
  },
  components: {
    Header,
    Footer,
  },
};
</script>

<style scoped>
.main {
  background: #fff;
  padding-top: 30px;
}

.course-info {
  width: 1200px;
  margin: 0 auto;
  overflow: hidden;
}

.wrap-left {
  float: left;
  width: 690px;
  height: 388px;
  background-color: #000;
}

.wrap-right {
  float: left;
  position: relative;
  height: 388px;
}

.course-name {
  font-size: 20px;
  color: #333;
  padding: 10px 23px;
  letter-spacing: 0.45px;
}

.data {
  padding-left: 23px;
  padding-right: 23px;
  padding-bottom: 16px;
  font-size: 14px;
  color: #9b9b9b;
}

.sale-time {
  width: 464px;
  background: #fa6240;
  font-size: 14px;
  color: #4a4a4a;
  padding: 10px 23px;
  overflow: hidden;
}

.sale-type {
  font-size: 16px;
  color: #fff;
  letter-spacing: 0.36px;
  float: left;
}

.sale-time .expire {
  font-size: 14px;
  color: #fff;
  float: right;
}

.sale-time .expire .second {
  width: 24px;
  display: inline-block;
  background: #fafafa;
  color: #5e5e5e;
  padding: 6px 0;
  text-align: center;
}

.course-price {
  background: #fff;
  font-size: 14px;
  color: #4a4a4a;
  padding: 5px 23px;
}

.discount {
  font-size: 26px;
  color: #fa6240;
  margin-left: 10px;
  display: inline-block;
  margin-bottom: -5px;
}

.original {
  font-size: 14px;
  color: #9b9b9b;
  margin-left: 10px;
  text-decoration: line-through;
}

.buy {
  width: 464px;
  padding: 0px 23px;
  position: absolute;
  left: 0;
  bottom: 20px;
  overflow: hidden;
}

.buy .buy-btn {
  float: left;
}

.buy .buy-now {
  width: 125px;
  height: 40px;
  border: 0;
  background: #ffc210;
  border-radius: 4px;
  color: #fff;
  cursor: pointer;
  margin-right: 15px;
  outline: none;
}

.buy .free {
  width: 125px;
  height: 40px;
  border-radius: 4px;
  cursor: pointer;
  margin-right: 15px;
  background: #fff;
  color: #ffc210;
  border: 1px solid #ffc210;
}

.add-cart {
  float: right;
  font-size: 14px;
  color: #ffc210;
  text-align: center;
  cursor: pointer;
  margin-top: 10px;
}

.add-cart img {
  width: 20px;
  height: 18px;
  margin-right: 7px;
  vertical-align: middle;
}

.course-tab {
  width: 100%;
  background: #fff;
  margin-bottom: 30px;
  box-shadow: 0 2px 4px 0 #f0f0f0;
}

.course-tab .tab-list {
  width: 1200px;
  margin: auto;
  color: #4a4a4a;
  overflow: hidden;
}

.tab-list li {
  float: left;
  margin-right: 15px;
  padding: 26px 20px 16px;
  font-size: 17px;
  cursor: pointer;
}

.tab-list .active {
  color: #ffc210;
  border-bottom: 2px solid #ffc210;
}

.tab-list .free {
  color: #fb7c55;
}

.course-content {
  width: 1200px;
  margin: 0 auto;
  background: #fafafa;
  overflow: hidden;
  padding-bottom: 40px;
}

.course-tab-list {
  width: 880px;
  height: auto;
  padding: 20px;
  background: #fff;
  float: left;
  box-sizing: border-box;
  overflow: hidden;
  position: relative;
  box-shadow: 0 2px 4px 0 #f0f0f0;
}

.tab-item {
  width: 880px;
  background: #fff;
  padding-bottom: 20px;
  box-shadow: 0 2px 4px 0 #f0f0f0;
}

.tab-item-title {
  justify-content: space-between;
  padding: 25px 20px 11px;
  border-radius: 4px;
  margin-bottom: 20px;
  border-bottom: 1px solid #333;
  border-bottom-color: rgba(51, 51, 51, 0.05);
  overflow: hidden;
}

.chapter {
  font-size: 17px;
  color: #4a4a4a;
  float: left;
}

.chapter-length {
  float: right;
  font-size: 14px;
  color: #9b9b9b;
  letter-spacing: 0.19px;
}

.chapter-title {
  font-size: 16px;
  color: #4a4a4a;
  letter-spacing: 0.26px;
  padding: 12px;
  background: #eee;
  border-radius: 2px;
  display: -ms-flexbox;
  display: flex;
  -ms-flex-align: center;
  align-items: center;
}

.chapter-title img {
  width: 18px;
  height: 18px;
  margin-right: 7px;
  vertical-align: middle;
}

.section-list {
  padding: 0 20px;
}

.section-list .section-item {
  padding: 15px 20px 15px 36px;
  cursor: pointer;
  justify-content: space-between;
  position: relative;
  overflow: hidden;
}

.section-item .name {
  font-size: 14px;
  color: #666;
  float: left;
}

.section-item .index {
  margin-right: 5px;
}

.section-item .free {
  font-size: 12px;
  color: #fff;
  letter-spacing: 0.19px;
  background: #ffc210;
  border-radius: 100px;
  padding: 1px 9px;
  margin-left: 10px;
}

.section-item .time {
  font-size: 14px;
  color: #666;
  letter-spacing: 0.23px;
  opacity: 1;
  transition: all 0.15s ease-in-out;
  float: right;
}

.section-item .time img {
  width: 18px;
  height: 18px;
  margin-left: 15px;
  vertical-align: text-bottom;
}

.section-item .try {
  width: 86px;
  height: 28px;
  background: #ffc210;
  border-radius: 4px;
  font-size: 14px;
  color: #fff;
  position: absolute;
  right: 20px;
  top: 10px;
  opacity: 0;
  transition: all 0.2s ease-in-out;
  cursor: pointer;
  outline: none;
  border: none;
}

.section-item:hover {
  background: #fcf7ef;
  box-shadow: 0 0 0 0 #f3f3f3;
}

.section-item:hover .name {
  color: #333;
}

.section-item:hover .try {
  opacity: 1;
}

.course-side {
  width: 300px;
  height: auto;
  margin-left: 20px;
  float: right;
}

.teacher-info {
  background: #fff;
  margin-bottom: 20px;
  box-shadow: 0 2px 4px 0 #f0f0f0;
}

.side-title {
  font-weight: normal;
  font-size: 17px;
  color: #4a4a4a;
  padding: 18px 14px;
  border-bottom: 1px solid #333;
  border-bottom-color: rgba(51, 51, 51, 0.05);
}

.side-title span {
  display: inline-block;
  border-left: 2px solid #ffc210;
  padding-left: 12px;
}

.teacher-content {
  padding: 30px 20px;
  box-sizing: border-box;
}

.teacher-content .cont1 {
  margin-bottom: 12px;
  overflow: hidden;
}

.teacher-content .cont1 img {
  width: 54px;
  height: 54px;
  margin-right: 12px;
  float: left;
}

.teacher-content .cont1 .name {
  float: right;
}

.teacher-content .cont1 .teacher-name {
  width: 188px;
  font-size: 16px;
  color: #4a4a4a;
  padding-bottom: 4px;
}

.teacher-content .cont1 .teacher-title {
  width: 188px;
  font-size: 13px;
  color: #9b9b9b;
  white-space: nowrap;
}

.teacher-content .narrative {
  font-size: 14px;
  color: #666;
  line-height: 24px;
}
</style>
```

### 搜索功能接口

```python
# http://127.0.0.1:8000/api/v1/course/search/?search=

搜索所有类型的课程
该功能先最简单的实现，后期使用ES实现
```

#### `apps/course/views.py`

```python
from rest_framework.viewsets import GenericViewSet
from rest_framework.mixins import ListModelMixin, RetrieveModelMixin
from rest_framework.filters import OrderingFilter, SearchFilter
from django_filters.rest_framework import DjangoFilterBackend
from utils.APIResponse import APIResponse
from .models import CourseCategory, Course, CourseChapter
from .serializer import CourseCategorySerializer, CourseSerializer, CourseChapterSerializer
from .pagination import CommonPageNumberPagination

# 序列化类和 CourseView 的一致
class CourseSearchView(GenericViewSet, ListModelMixin):
    queryset = Course.objects.all().filter(is_delete=False, is_show=True).order_by('orders')
    pagination_class = CommonPageNumberPagination
    serializer_class = CourseSerializer
    filter_backends = [SearchFilter]
    search_fields = ['name', ]

    # def list(self, request, *args, **kwargs):
    #     # 这个查的是实战课
    #     res=super().list(request, *args, **kwargs)
    #     # res2=查询免费课
    #     # res3=查询轻课
    #     # 上面全是取数据库查询
    #     # 后期改成取es中查询，
    #     return APIResponse(result={'free_course':'字典','actual_course':res.data})
```

#### `apps/course/urls.py`

```python
from django.urls import path, include
from rest_framework.routers import SimpleRouter
from .views import CourseCategoryView, CourseView, CourseChapterView,CourseSearchView

router = SimpleRouter()
# 127.0.0.1:8000/api/v1/course/category/  get
router.register('category', CourseCategoryView, 'category')
# 127.0.0.1:8000/api/v1/course/actual/  get
router.register('actual', CourseView, 'actual')
# 127.0.0.1:8000/api/v1/course/chapter/  get
router.register('chapter', CourseChapterView, 'chapter')
# 127.0.0.1:8000/api/v1/course/search/?search=python  get
router.register('search', CourseSearchView, 'search')
urlpatterns = [
    path('', include(router.urls))
]
```

### 搜索功能页面

```
头部组件中的搜索框
搜索结果展示页面
```

#### `src/components/Header.vue`

```html
<!--该内容在 html 中 Register 组件下-->
<form class="search">
  <div class="tips" v-if="is_search_tip">
    <span @click="search_action('Python')">Python</span>
    <span @click="search_action('Linux')">Linux</span>
  </div>
  <input
    type="text"
    :placeholder="search_placeholder"
    @focus="on_search"
    @blur="off_search"
    v-model="search_word"
  />
  <button
    type="button"
    class="glyphicon glyphicon-search"
    @click="search_action(search_word)"
  ></button>
</form>
```

```javascript
// 加在 script 内的内容
// data里面的变量
is_search_tip: true,
search_placeholder: '',
search_word: '',

// methods 里面的方法
search_action(search_word) {
           if (!search_word) {
               this.$message('请输入要搜索的内容');
               return
           }
           // this.$route当前路径，query ?name=python  -->this.$route.query.name 拿到python
           if (search_word !== this.$route.query.word) {
               this.$router.push(`/course/search?word=${search_word}`);
           }
           this.search_word = '';
       },
       on_search() {
           this.search_placeholder = '请输入想搜索的课程';
           this.is_search_tip = false;
       },
       off_search() {
           this.search_placeholder = '';
           this.is_search_tip = true;
},
```

```css
/*搜索的样式*/
.search {
  float: right;
  position: relative;
  margin-top: 22px;
  margin-right: 10px;
}

.search input,
.search button {
  border: none;
  outline: none;
  background-color: white;
}

.search input {
  border-bottom: 1px solid #eeeeee;
}

.search input:focus {
  border-bottom-color: orange;
}

.search input:focus + button {
  color: orange;
}

.search .tips {
  position: absolute;
  bottom: 3px;
  left: 0;
}

.search .tips span {
  border-radius: 11px;
  background-color: #eee;
  line-height: 22px;
  display: inline-block;
  padding: 0 7px;
  margin-right: 3px;
  cursor: pointer;
  color: #aaa;
  font-size: 14px;
}

.search .tips span:hover {
  color: orange;
}
```

#### `src/views/SearchCourse.vue`

```vue
<template>
  <div class="search-course course">
    <Header />

    <!-- 课程列表 -->
    <div class="main">
      <div v-if="course_list.length > 0" class="course-list">
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
              <router-link :to="'/free/detail/' + course.id">{{
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
      <div v-else style="text-align: center; line-height: 60px">
        没有搜索结果
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
  </div>
</template>

<script>
import Header from "../components/Header";

export default {
  name: "SearchCourse",
  components: {
    Header,
  },
  data() {
    return {
      course_list: [],
      course_total: 0,
      filter: {
        page_size: 10,
        page: 1,
        search: "",
      },
    };
  },
  created() {
    this.get_course();
  },
  watch: {
    "$route.query"() {
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
    get_course() {
      // 获取搜索的关键字
      this.filter.search = this.$route.query.word;

      // 获取课程列表信息
      this.$axios
        .get(`${this.$settings.base_url}course/search/`, {
          params: this.filter,
        })
        .then((response) => {
          // 如果后台不分页，数据在response.data中；如果后台分页，数据在response.data.results中
          this.course_list = response.data.results;
          this.course_total = response.data.count;
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

#### `router/index.js`

```javascript
// ...
import SearchCourse from "@/views/SearchCourse";

Vue.use(VueRouter);

const routes = [
  // ...
  {
    path: "/course/search",
    name: "SearchCourse",
    component: SearchCourse,
  },
];
```
