---
title: git管理项目、登录注册功能实现
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - python
categories:
  - python
url: post/luffy-03.html
toc: true
---

### 使用`git`管理代码

#### 提交本地仓库

<!-- more -->

```bash
# 配置个人信息
git config --global user.name "klcc"
git config --global user.email "2564334707@qq.com"

# 后端代码
  # 初始化仓库
  cd luffy_api
  git init

  # 创建过滤文件，内容如下
  vim  .gitignore
  .DS_Store
  *.pyc
  logs/*.log
  .idea/
  __pycache__/

  # 提交到版本库
  git add .
  git commit -m "轮播功能"

# 前端代码 如上操作 但是vue项目在创建的时候是拉取的git 后续只需要提交即可
  git add .
  git commit -m "页面主体，轮播功能"
```

#### 提交到`gitee`

##### 创建仓库

先注册账户: https://gitee.com/signup 然后登录

![image-20220421183157663](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220421183157663.png)

![image-20220421183253531](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220421183253531.png)

![image-20220421183421979](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220421183421979.png)

```bash
cd luffy_api
git remote add origin https://gitee.com/liuzhijin1/luffy_api.git  # 添加远程仓库
git remote -v                 # 仓库远程仓库
git tag -a "v1" -m "轮播功能"  # 打标签
git push orgin --tag v1       # 提交指定标签

git push -u origin "master"   # 提交时需要用户名和密码 可以配置免密
```

##### 配置免密

```bash
# 用户名密码的方式
git remote rm origin          # 删除原本的 origin 远程仓库
# 使用用户名密码
git remote add origin https://用户名:密码@gitee.com/liuzhijin1/luffy_api.git


# SSH 方式
# 生成公钥和私钥(默认放在 ~/.ssh目录下id_rsa.pub公钥、id_rsa私钥）
ssh-keygen

# 拷贝公钥的内容.pub 结尾的文件到gitee中
# 然后配置远程仓库地址
git remote add origin git@gitee.com:liuzhijin1/luffy_api.git

# 提交代码
git push origin --tag v1.1       # 提交指定标签
git push -u origin "master"      # 提交master
```

##### 提交代码

```bash
# 使用SSH方式
## 后端代码
cd luffy_api
git remote rm origin
git remote add origin git@gitee.com:liuzhijin1/luffy_api.git
git add .
git commit -m "轮播功能"
git tag -a "v1" -m "轮播功能"
git push origin "master" --tag v1

## 前端代码
 cd luffycity
 git remote rm origin
 git remote add origin git@gitee.com:liuzhijin1/luffycity.git
 git add .
 git commit -m "轮播功能和主体"
 git tag -a "v1" -m "轮播功能和主体"
 git push -u origin "master" --tag v1

```

### 腾讯云短信申请

```python

# 申请一个公众号：https://mp.weixin.qq.com/cgi-bin/home?t=home/index&lang=zh_CN&token=2082783786
# 个人，身份证

# 访问地址申请：https://console.cloud.tencent.com/smsv2/guide
# 步骤
创建短信签名
  -签名管理---》创建签名--》使用公众号提交申请---》审核
创建短信正文模板
  -正文模板管理---》创建正文模板--》等审核
发送短信
  -API，SDK

# 发送短信，按照文档来:https://cloud.tencent.com/document/product/382/43196
# api和sdk的区别
  -api接口，咱们通过http调用腾讯的发送短信接口，腾讯负责吧短信发送到手机上，http的接口--》基于它来做，比较麻烦，麻烦在请求参数，携带很多，有的时候我们有可能找不到某个参数
  -sdk：使用不同语言封装好了，只需要导入，调用某个函数，传入参数就可以发送，用起来更简单，区分语言，可能官方没有提供sdk


# 发短信sdk的使用
# 3.x的发送短信sdk，tencentcloud 包含的功能更多，不仅仅只能发短信，还能干别的，但是咱们用不到
pip install tencentcloud-sdk-python
# 2.x发送短信sdk：https://cloud.tencent.com/document/product/382/11672
# 只是发短信的sdk，功能少，3.8以后不支持
pip install qcloudsms_py
```

### 登录注册功能

```python
多方式登录接口(手机号，邮箱，用户名+密码)
验证手机号是否存在的接口
发送短信验证码接口  借助第三方发送短信，阿里云 腾讯云 容联云
手机号+验证码登录接口
手机号+验证码+密码注册接口
```

### 验证手机号是否存在

该功能写在 user app 中

#### `views.py`

```python
# luffy_api/apps/user/views.py
from rest_framework.viewsets import ViewSet
from rest_framework.decorators import action
from .models import User
from utils.APIResponse import APIResponse
from rest_framework.exceptions import APIException


class UserView(ViewSet):
    # get 请求携带手机号 校验手机号
    @action(methods=['GET'], detail=False)
    def check_mobile(self, request):
        try:
            mobile = request.query_params.get('mobile')
            User.objects.get(mobile=mobile)
            return APIResponse()  # {code: 100, msg: successfully} 前端判断是否 100 即可
        except Exception as e:
            raise APIException(str(e))  # 处理了全局异常
```

#### `urls.py`

```python
from django.urls import path, include
from rest_framework.routers import SimpleRouter
from .views import UserView

router = SimpleRouter()
# 127.0.0.1:8000/api/v1/user/mobile/check_mobile
router.register('mobile', UserView, 'mobile')
urlpatterns = [
    path('', include(router.urls))
]
```

#### `总路由`

```python
from django.contrib import admin
from django.urls import path, include
from django.conf import settings
from django.views.static import serve

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/v1/home/', include('home.urls')),  # http://127.0.0.1:8000/api/v1/home/banner/
    path('api/v1/user/', include('user.urls')),  # http://127.0.0.1:8000/api/v1/user/
    path('media/<path:path>', serve, {'document_root': settings.MEDIA_ROOT})
]
```

### 注册登录页面

#### 方式一(页面)

```vue
// 创建页面 src/views/Login.vue
<template>
  <div>
    <form action="">
      用户名:
      <p><input type="text" /></p>
      密 码:
      <p><input type="password" /></p>
      <p><input type="submit" value="提交" /></p>
    </form>
  </div>
</template>

<script>
export default {
  name: "Login",
};
</script>

<style scoped></style>

// 需要使用的跳转的页面 // 绑定点击事件
<span @click="go_login">登录</span>

methods: { // 方法 go_login(){ // 跳转到另一个页面 使用 vue-router 跳转到 /login
路由 this.$router.push('/login') }, }, // 路由 src/router/index.js // 先导入
import Login from "@/views/Login"; const routes = [ // 添加该路由
然后就可以访问到 /login { path: '/login', name: 'login', component: Login }, ]
```

#### 方式二(组件、模态框)

```
采用该方式

登录注册是弹出模态框 Login Register 两个组件 放到 components 中
Header 为 Login Register 两个组件的父组件
```

##### `Login.vue`

```vue
<template>
  <div class="login">
    <div class="box">
      <i class="el-icon-close" @click="close_login"></i>
      <div class="content">
        <div class="nav">
          <span
            :class="{ active: login_method === 'is_pwd' }"
            @click="change_login_method('is_pwd')"
            >密码登录</span
          >
          <span
            :class="{ active: login_method === 'is_sms' }"
            @click="change_login_method('is_sms')"
            >短信登录</span
          >
        </div>
        <el-form v-if="login_method === 'is_pwd'">
          <el-input
            placeholder="用户名/手机号/邮箱"
            prefix-icon="el-icon-user"
            v-model="username"
            clearable
          >
          </el-input>
          <el-input
            placeholder="密码"
            prefix-icon="el-icon-key"
            v-model="password"
            clearable
            show-password
          >
          </el-input>
          <el-button type="primary">登录</el-button>
        </el-form>
        <el-form v-if="login_method === 'is_sms'">
          <el-input
            placeholder="手机号"
            prefix-icon="el-icon-phone-outline"
            v-model="mobile"
            clearable
            @blur="check_mobile"
          >
          </el-input>
          <el-input
            placeholder="验证码"
            prefix-icon="el-icon-chat-line-round"
            v-model="sms"
            clearable
          >
            <template slot="append">
              <span class="sms" @click="send_sms">{{ sms_interval }}</span>
            </template>
          </el-input>
          <el-button type="primary">登录</el-button>
        </el-form>
        <div class="foot">
          <span @click="go_register">立即注册</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "Login",
  data() {
    return {
      username: "",
      password: "",
      mobile: "",
      sms: "",
      login_method: "is_pwd",
      sms_interval: "获取验证码",
      is_send: false,
    };
  },
  methods: {
    close_login() {
      this.$emit("close");
    },
    go_register() {
      this.$emit("go");
    },
    change_login_method(method) {
      this.login_method = method;
    },
    check_mobile() {
      if (!this.mobile) return;
      if (!this.mobile.match(/^1[3-9][0-9]{9}$/)) {
        this.$message({
          message: "手机号有误",
          type: "warning",
          duration: 1000,
          onClose: () => {
            this.mobile = "";
          },
        });
        return false;
      }
      this.is_send = true;
    },
    send_sms() {
      if (!this.is_send) return;
      this.is_send = false;
      let sms_interval_time = 60;
      this.sms_interval = "发送中...";
      let timer = setInterval(() => {
        if (sms_interval_time <= 1) {
          clearInterval(timer);
          this.sms_interval = "获取验证码";
          this.is_send = true; // 重新回复点击发送功能的条件
        } else {
          sms_interval_time -= 1;
          this.sms_interval = `${sms_interval_time}秒后再发`;
        }
      }, 1000);
    },
  },
};
</script>

<style scoped>
.login {
  width: 100vw;
  height: 100vh;
  position: fixed;
  top: 0;
  left: 0;
  z-index: 10;
  background-color: rgba(0, 0, 0, 0.3);
}

.box {
  width: 400px;
  height: 420px;
  background-color: white;
  border-radius: 10px;
  position: relative;
  top: calc(50vh - 210px);
  left: calc(50vw - 200px);
}

.el-icon-close {
  position: absolute;
  font-weight: bold;
  font-size: 20px;
  top: 10px;
  right: 10px;
  cursor: pointer;
}

.el-icon-close:hover {
  color: darkred;
}

.content {
  position: absolute;
  top: 40px;
  width: 280px;
  left: 60px;
}

.nav {
  font-size: 20px;
  height: 38px;
  border-bottom: 2px solid darkgrey;
}

.nav > span {
  margin: 0 20px 0 35px;
  color: darkgrey;
  user-select: none;
  cursor: pointer;
  padding-bottom: 10px;
  border-bottom: 2px solid darkgrey;
}

.nav > span.active {
  color: black;
  border-bottom: 3px solid black;
  padding-bottom: 9px;
}

.el-input,
.el-button {
  margin-top: 40px;
}

.el-button {
  width: 100%;
  font-size: 18px;
}

.foot > span {
  float: right;
  margin-top: 20px;
  color: orange;
  cursor: pointer;
}

.sms {
  color: orange;
  cursor: pointer;
  display: inline-block;
  width: 70px;
  text-align: center;
  user-select: none;
}
</style>
```

##### `Register.vue`

```vue
<template>
  <div class="register">
    <div class="box">
      <i class="el-icon-close" @click="close_register"></i>
      <div class="content">
        <div class="nav">
          <span class="active">新用户注册</span>
        </div>
        <el-form>
          <el-input
            placeholder="手机号"
            prefix-icon="el-icon-phone-outline"
            v-model="mobile"
            clearable
            @blur="check_mobile"
          >
          </el-input>
          <el-input
            placeholder="密码"
            prefix-icon="el-icon-key"
            v-model="password"
            clearable
            show-password
          >
          </el-input>
          <el-input
            placeholder="验证码"
            prefix-icon="el-icon-chat-line-round"
            v-model="sms"
            clearable
          >
            <template slot="append">
              <span class="sms" @click="send_sms">{{ sms_interval }}</span>
            </template>
          </el-input>
          <el-button type="primary">注册</el-button>
        </el-form>
        <div class="foot">
          <span @click="go_login">立即登录</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "Register",
  data() {
    return {
      mobile: "",
      password: "",
      sms: "",
      sms_interval: "获取验证码",
      is_send: false,
    };
  },
  methods: {
    close_register() {
      this.$emit("close", false);
    },
    go_login() {
      this.$emit("go");
    },
    check_mobile() {
      if (!this.mobile) return;
      if (!this.mobile.match(/^1[3-9][0-9]{9}$/)) {
        this.$message({
          message: "手机号有误",
          type: "warning",
          duration: 1000,
          onClose: () => {
            this.mobile = "";
          },
        });
        return false;
      }
      this.is_send = true;
    },
    send_sms() {
      if (!this.is_send) return;
      this.is_send = false;
      let sms_interval_time = 60;
      this.sms_interval = "发送中...";
      let timer = setInterval(() => {
        if (sms_interval_time <= 1) {
          clearInterval(timer);
          this.sms_interval = "获取验证码";
          this.is_send = true; // 重新回复点击发送功能的条件
        } else {
          sms_interval_time -= 1;
          this.sms_interval = `${sms_interval_time}秒后再发`;
        }
      }, 1000);
    },
  },
};
</script>

<style scoped>
.register {
  width: 100vw;
  height: 100vh;
  position: fixed;
  top: 0;
  left: 0;
  z-index: 10;
  background-color: rgba(0, 0, 0, 0.3);
}

.box {
  width: 400px;
  height: 480px;
  background-color: white;
  border-radius: 10px;
  position: relative;
  top: calc(50vh - 240px);
  left: calc(50vw - 200px);
}

.el-icon-close {
  position: absolute;
  font-weight: bold;
  font-size: 20px;
  top: 10px;
  right: 10px;
  cursor: pointer;
}

.el-icon-close:hover {
  color: darkred;
}

.content {
  position: absolute;
  top: 40px;
  width: 280px;
  left: 60px;
}

.nav {
  font-size: 20px;
  height: 38px;
  border-bottom: 2px solid darkgrey;
}

.nav > span {
  margin-left: 90px;
  color: darkgrey;
  user-select: none;
  cursor: pointer;
  padding-bottom: 10px;
  border-bottom: 2px solid darkgrey;
}

.nav > span.active {
  color: black;
  border-bottom: 3px solid black;
  padding-bottom: 9px;
}

.el-input,
.el-button {
  margin-top: 40px;
}

.el-button {
  width: 100%;
  font-size: 18px;
}

.foot > span {
  float: right;
  margin-top: 20px;
  color: orange;
  cursor: pointer;
}

.sms {
  color: orange;
  cursor: pointer;
  display: inline-block;
  width: 70px;
  text-align: center;
  user-select: none;
}
</style>
```

##### `Header.vue`

```vue
<template>
  <div class="header">
    <div class="slogan">
      <p>老男孩IT教育 | 帮助有志向的年轻人通过努力学习获得体面的工作和生活</p>
    </div>
    <div class="nav">
      <ul class="left-part">
        <li class="logo">
          <router-link to="/">
            <img src="../assets/img/head-logo.svg" alt="" />
          </router-link>
        </li>
        <li class="ele">
          <span
            @click="goPage('/free-course')"
            :class="{ active: url_path === '/free-course' }"
            >免费课</span
          >
        </li>
        <li class="ele">
          <span
            @click="goPage('/actual-course')"
            :class="{ active: url_path === '/actual-course' }"
            >实战课</span
          >
        </li>
        <li class="ele">
          <span
            @click="goPage('/light-course')"
            :class="{ active: url_path === '/light-course' }"
            >轻课</span
          >
        </li>
      </ul>

      <div class="right-part">
        <div>
          <span @click="put_login">登录</span>
          <span class="line">|</span>
          <span @click="put_register">注册</span>
        </div>
      </div>

      <Login v-if="is_login" @close="close_login" @go="put_register" />
      <Register v-if="is_register" @close="close_register" @go="put_login" />
    </div>
  </div>
</template>

<script>
import Login from "@/components/Login";
import Register from "@/components/Register";
export default {
  name: "Header",
  data() {
    return {
      url_path: sessionStorage.url_path || "/",
      is_login: false,
      is_register: false,
    };
  },
  methods: {
    goPage(url_path) {
      // 已经是当前路由就没有必要重新跳转
      if (this.url_path !== url_path) {
        this.$router.push(url_path);
      }
      sessionStorage.url_path = url_path;
    },
    close_login() {
      this.is_login = false;
    },
    close_register() {
      this.is_register = false;
    },
    put_register() {
      this.is_register = true;
      this.is_login = false;
    },
    put_login() {
      this.is_register = false;
      this.is_login = true;
    },
  },
  created() {
    sessionStorage.url_path = this.$route.path;
    this.url_path = this.$route.path;
  },
  components: {
    Login,
    Register,
  },
};
</script>

<style scoped>
.header {
  background-color: white;
  box-shadow: 0 0 5px 0 #aaa;
}

.header:after {
  content: "";
  display: block;
  clear: both;
}

.slogan {
  background-color: #eee;
  height: 40px;
}

.slogan p {
  width: 1200px;
  margin: 0 auto;
  color: #aaa;
  font-size: 13px;
  line-height: 40px;
}

.nav {
  background-color: white;
  user-select: none;
  width: 1200px;
  margin: 0 auto;
}

.nav ul {
  padding: 15px 0;
  float: left;
}

.nav ul:after {
  clear: both;
  content: "";
  display: block;
}

.nav ul li {
  float: left;
}

.logo {
  margin-right: 20px;
}

.ele {
  margin: 0 20px;
}

.ele span {
  display: block;
  font: 15px/36px "微软雅黑";
  border-bottom: 2px solid transparent;
  cursor: pointer;
}

.ele span:hover {
  border-bottom-color: orange;
}

.ele span.active {
  color: orange;
  border-bottom-color: orange;
}

.right-part {
  float: right;
}

.right-part .line {
  margin: 0 10px;
}

.right-part span {
  line-height: 68px;
  cursor: pointer;
}
</style>
```

#### vue 中跳转方式

```python
# 第一种
this.$router.push('/login')

# 第二种
<router-link to="/login"><span>登录</span></router-link>
```

#### `Banner.vue`

```vue
// router-link 的使用
<template>
  <div class="banner">
    <el-carousel :interval="5000" arrow="always" height="400px">
      <el-carousel-item v-for="item in banner_list" :key="item.image">
        <!--可以根据请求的地址判断 进行跳转-->
        <div v-if="!(item.link.indexOf('http') > -1)">
          <router-link :to="item.link">
            <img :src="item.image" alt="" />
          </router-link>
        </div>
        <div v-else>
          <a :href="item.link">
            <img :src="item.image" alt="" />
          </a>
        </div>
      </el-carousel-item>
    </el-carousel>
  </div>
</template>

<script>
export default {
  name: "Banner",
  data() {
    return {
      banner_list: [],
    };
  },
  created() {
    this.$axios.get(this.$settings.base_url + "home/banner/").then((res) => {
      if (res.data.status == 100) {
        this.banner_list = res.data.data;
      }
    });
  },
};
</script>

<style scoped>
el-carousel-item {
  height: 400px;
  min-width: 1200px;
}

.el-carousel__item img {
  height: 400px;
  margin-left: calc(50% - 1920px / 2);
}
</style>
```

### 多方式登录功能

```
需求: 输入用户名(手机号，邮箱)，密码，都能登陆成功，签发token
例
{
  username:name/130348883775/1233@qq.com,
  password:lqz123
}
到后端去数据库查用户，如果用户名密码正确，签发token，如果不正确，返回错误


pip install djangorestframework-jwt
```

#### 路由

```python
# luffy_api/apps/user/urls.py

from django.urls import path, include
from rest_framework.routers import SimpleRouter
from .views import MobileView, LoginView

router = SimpleRouter()
# 127.0.0.1:8000/api/v1/user/mobile/check_mobile/  get
router.register('mobile', MobileView, 'mobile')
# 127.0.0.1:8000/api/v1/user/login/mul_login/  post
router.register('login', LoginView, 'login')
urlpatterns = [
    path('', include(router.urls))
]
```

#### 序列化类

```python
# luffy_api/apps/user/serializer.py
import re
from rest_framework import serializers
from rest_framework.exceptions import ValidationError
from rest_framework_jwt.serializers import jwt_payload_handler, jwt_encode_handler
from .models import User

# 这个序列化类 只做反序列化 数据库校验 不保存 不用序列化
class MulLoginSerializer(serializers.ModelSerializer):
    # 一定要重写 username 字段 校验规则是从User中来 unique
    # 如果存在的用户 再传入该用户 自己的校验规则就会校验失败
    username = serializers.CharField(max_length=18, min_length=3)  # 一定要重写 否则校验不过去
    class Meta:
        model = User
        fields = ['username', 'password']

    def validate(self, attrs):
        # 在这完成校验 校验失败抛出异常
        # 1 多方式得到user
        user = self._get_user(attrs)
        # 2  user签发token
        token = self._get_token(user)
        # 3  把token,username,icon放到context中
        self.context['token'] = token
        self.context['username'] = user.username
        request = self.context['request']
        # request.META['HTTP_HOST']取出服务端的ip地址
        icon = 'http://%s/media/%s' % (request.META['HTTP_HOST'], str(user.icon))
        self.context['icon'] = icon
        return attrs

    def _get_user(self, attrs):
        username = attrs.get('username')
        if re.match(r'^1[3-9][0-9]{9}$', username):
            user = User.objects.filter(mobile=username).first()
        elif re.match(r'^.+@.+$', username):
            user = User.objects.filter(email=username).first()
        else:
            user = User.objects.filter(username=username).first()
        if not user:
            raise ValidationError("用户名或密码错误")
        # 取出前端传入的密码
        password = attrs.get("password")
        if not user.check_password(password):
            raise ValidationError("用户名或密码错误")
        return user

    def _get_token(self, user):
        payload = jwt_payload_handler(user)
        token = jwt_encode_handler(payload)
        return token
```

#### 视图类

```python
# luffy_api/apps/user/views.py
# luffy_api/media/icon/default.png 该图片要存在
from rest_framework.viewsets import ViewSet, GenericViewSet
from rest_framework.decorators import action
from rest_framework.exceptions import APIException
from utils.APIResponse import APIResponse
from .models import User
from .serializer import MulLoginSerializer

class MobileView(ViewSet):
    # get 请求携带手机号 校验手机号
    @action(methods=['GET'], detail=False)
    def check_mobile(self, request):
        try:
            mobile = request.query_params.get('mobile')
            User.objects.get(mobile=mobile)
            return APIResponse()  # {code: 100, msg: successfully} 前端判断是否 100 即可
        except Exception as e:
            raise APIException(str(e))  # 处理了全局异常

class LoginView(GenericViewSet):
    serializer_class = MulLoginSerializer
    queryset = User
    # 多方式登录
    # login 不是保存 但是用 post 把验证逻辑写到序列化类中
    @action(methods=['POST'], detail=False)
    def mul_login(self, request):
        try:
            ser = MulLoginSerializer(data=request.data, context={'request': request})
            ser.is_valid(raise_exception=True)  # 如果校验失败 直接抛出异常 不用if判断
            token = ser.context.get('token')
            username = ser.context.get('username')
            icon = ser.context.get('icon')
            return APIResponse(data={'token': token, 'username': username, 'icon': icon})
        except Exception as e:
            raise APIException(str(e))
```

### 二次封装腾讯云短信发送

```python
# 将腾讯云提供的SDK二次封装为一个包
# 创建 luffy_api/libs/tencent_sms_v3 文件夹

# settings.py
SECRETID = ""
SECRETKEY = ""
APPID = ""
SIGNNAME = ""
TEMPLATEID = ""

# sms.py
import random
from tencentcloud.common import credential
from tencentcloud.common.exception.tencent_cloud_sdk_exception import TencentCloudSDKException
from tencentcloud.sms.v20210111 import sms_client, models
from tencentcloud.common.profile.client_profile import ClientProfile
from tencentcloud.common.profile.http_profile import HttpProfile
from utils.luffy_logging import logger
from . import settings


def get_code(count=4):
    code_str = ''
    for i in range(count):
        num = random.randint(0, 9)
        code_str += str(num)

    return code_str


def send_sms(phone, code):
    try:
        cred = credential.Credential(settings.SECRETID, settings.SECRETKEY)
        httpProfile = HttpProfile()

        httpProfile.reqMethod = "POST"  # post请求(默认为post请求)
        httpProfile.reqTimeout = 30  # 请求超时时间，单位为秒(默认60秒)
        httpProfile.endpoint = "sms.tencentcloudapi.com"  # 指定接入地域域名(默认就近接入)
        clientProfile = ClientProfile()
        clientProfile.signMethod = "TC3-HMAC-SHA256"  # 指定签名算法
        clientProfile.language = "en-US"
        clientProfile.httpProfile = httpProfile
        client = sms_client.SmsClient(cred, "ap-guangzhou", clientProfile)
        req = models.SendSmsRequest()
        req.SmsSdkAppId = settings.APPID
        req.SignName = settings.SIGNNAME
        req.TemplateId = settings.TEMPLATEID
        req.TemplateParamSet = [code, "5"]
        req.PhoneNumberSet = ["+86%s" % phone, ]
        req.SessionContext = ""
        req.ExtendCode = ""
        req.SenderId = ""
        client.SendSms(req)
        # print(resp.to_json_string(indent=2))
        return True
    except TencentCloudSDKException as err:
        # 如果短信发送失败 记录日志 如果给别人使用 需要修改
        logger.error("手机号为: %s 发送短信失败，失败原因: %s" %(phone,str(err)))


if __name__ == '__main__':
    print(get_code())

# __init__.py
from .sms import get_code, send_sms
```

### 发送短信接口

```python
get 请求 携带手机号就发送短信  ?phnoe=phone
验证码保存在缓存中
缓存：是一个存数据的地方，可以存，可以取（缓存可以缓存到的位置有很多，内存，文件，redis，mysql）
django内置了一个缓存功能 导入使用即可
from django.core.cache import cache
cache.set('sms_cache_%s'%phone,code) # 设置值，key value形式，key应该唯一，使用手机号
cache.get() # 取值，根据key取
```

#### 路由

```python
# luffy_api/apps/user/urls.py
from django.urls import path, include
from rest_framework.routers import SimpleRouter
from .views import MobileView, LoginView,SendSmsView

router = SimpleRouter()
# 127.0.0.1:8000/api/v1/user/mobile/check_mobile/  get
router.register('mobile', MobileView, 'mobile')
# 127.0.0.1:8000/api/v1/user/login/mul_login/  post
router.register('login', LoginView, 'login')
# 127.0.0.1:8000/api/v1/user/send/send_message/  get
router.register('send', SendSmsView, 'send')
urlpatterns = [
    path('', include(router.urls))
]
```

#### 视图

```python
# luffy_api/apps/user/views.py 添加代码
from libs import tencent_sms_v3
from django.core.cache import cache

class SendSmsView(ViewSet):
    @action(methods=["GET"],detail=False)
    def send_message(self, request):
        try:
            phone = request.query_params.get('phone')
            code = tencent_sms_v3.get_code()
            # 保存验证码到缓存中 cache 第三个参数超时时间
            cache.set("sms_cache_%s" % phone, code, 300)
            res = tencent_sms_v3.send_sms(phone, code)
            if res:
                return APIResponse(data="短信发送成功")
        except Exception as e:
            raise APIException(str(e))
```

### 短信登录接口

``` 
根据原型图(页面) ---> 该接口需要两个参数
{mobile:130xxxx,code:8888}

post:
127.0.0.1:8000/api/v1/user/login/sms_login/
```

#### 视图类

```python
# 多方式登录接口
class LoginView(GenericViewSet):
    serializer_class = MulLoginSerializer
    queryset = User

    # 多方式登录
    # login 不是保存 但是用 post 把验证逻辑写到序列化类中
    @action(methods=['POST'], detail=False)
    def mul_login(self, request):
        return self._common_login(request)

		# 验证码登录
    @action(methods=['POST'], detail=False)
    def sms_login(self, request):
        return self._common_login(request)

    def _common_login(self, request):
        try:
            ser = self.get_serializer(data=request.data, context={'request': request})
            ser.is_valid(raise_exception=True)  # 如果校验失败 直接抛出异常 不用if判断
            token = ser.context.get('token')
            username = ser.context.get('username')
            icon = ser.context.get('icon')
            return APIResponse(data={'token': token, 'username': username, 'icon': icon})
        except Exception as e:
            raise APIException(str(e))

    # 根据请求判断取出不同的序列化类
    def get_serializer_class(self):
        if self.action == 'mul_login':
            return self.serializer_class
        else:
            return SmsLoginSerializer
```

#### 序列化类

```python
from django.core.cache import cache
class SmsLoginSerializer(serializers.ModelSerializer):
    code = serializers.CharField(max_length=4, min_length=4)
    mobile = serializers.CharField(max_length=11, min_length=11)

    class Meta:
        model = User
        fields = ['mobile', 'code']

    def validate(self, attrs):
        self._check_code(attrs)
        user = self._get_user(attrs)
        token = self._get_token(user)
        self.context['token'] = token
        self.context['username'] = user.username
        request = self.context['request']
        icon = 'http://%s/media/%s' % (request.META['HTTP_HOST'], str(user.icon))
        self.context['icon'] = icon
        return attrs

    def _check_code(self, attrs):
        mobile = attrs.get('mobile')
        new_code = attrs.get('code')
        if mobile:
            old_code = cache.get("sms_cache_%s" % mobile)
            if old_code != new_code:
                raise ValidationError('验证码错误')
        else:
            raise ValidationError('没带手机号')
        return attrs

    def _get_user(self, attrs):
        mobile = attrs.get('mobile')
        return User.objects.get(mobile=mobile)  # 捕获了全局异常 get出错了也能捕获

    def _get_token(self, user):
        payload = jwt_payload_handler(user)
        token = jwt_encode_handler(payload)
        return token
```

### 短信注册接口

```
{mobile:1234444,code:8888,password:123456} ---> 创建一个新用户
```

#### 路由

```python
from django.urls import path, include
from rest_framework.routers import SimpleRouter
from .views import MobileView, LoginView, SendSmsView, RegisterView

router = SimpleRouter()
# 127.0.0.1:8000/api/v1/user/mobile/check_mobile/  get
router.register('mobile', MobileView, 'mobile')
# 127.0.0.1:8000/api/v1/user/login/mul_login/  post
# 127.0.0.1:8000/api/v1/user/login/sms_login/  post
router.register('login', LoginView, 'login')
# 127.0.0.1:8000/api/v1/user/send/send_message/  get
router.register('send', SendSmsView, 'send')
# 127.0.0.1:8000/api/v1/user/register/  post
router.register('register', RegisterView, 'register')
urlpatterns = [
    path('', include(router.urls))
]
```

#### 视图类

```python
from django.core.cache import cache
from rest_framework.viewsets import ViewSet, GenericViewSet
from rest_framework.mixins import CreateModelMixin
from rest_framework.decorators import action
from rest_framework.exceptions import APIException
from utils.APIResponse import APIResponse
from libs import tencent_sms_v3
from .models import User
from .serializer import MulLoginSerializer, SmsLoginSerializer,RegisterSerializer

class RegisterView(GenericViewSet, CreateModelMixin):
    serializer_class = RegisterSerializer
    queryset = User.objects.all()

    def create(self, request, *args, **kwargs):
        super().create(request, *args, **kwargs)
        return APIResponse(data="注册成功")
```

#### 序列化类

```python
class RegisterSerializer(serializers.ModelSerializer):
    # code 不在数据库中 需要重写 只写 读不了数据库
    code = serializers.CharField(max_length=4, min_length=4, write_only=True)

    class Meta:
        model = User
        fields = ['mobile', 'code', 'password']
        extra_kwargs = {
            'password': {'write_only': True},  # 不需要给前端
        }

    # 局部钩子 校验手机号是否合法
    def validate_mobile(self, value):
        if not re.match(r'^1[3-9][0-9]{9}$', value):
            raise ValidationError('手机号不合法')
        return value

    # 全局钩子
    def validate(self, attrs):
        # 校验验证码
        self._check_code(attrs)
        # 数据清理
        self._per_save(attrs)
        return attrs

    def _check_code(self, attrs):
        # 校验code
        new_code = attrs.get('code')
        mobile = attrs.get('mobile')
        old_code = cache.get("sms_cache_%s" % mobile)
        if new_code != old_code:
            raise ValidationError('验证码错误')
        return attrs

    # 存入数据之前将code剔除 并设置用户名
    def _per_save(self, attrs):
        attrs.pop('code')
        attrs['username'] = attrs.get('mobile')
        return attrs

    # 创建用户
    def create(self, validated_data):
        # auth 组件 创建用户
        user = User.objects.create_user(**validated_data)
        return user
```

### 登录注册前端页面

#### 前端存入数据

```python
1. 存到 cookie 中，js 操作，在 vue 中可以借助 vue-cookies 第三方插件
   npm install vue-cookies -S
   导入: src/main.js
      import cookies from 'vue-cookies'
      Vue.prototype.$cookies = cookies;
   使用:
      this.$cookies.set()
      this.$cookies.get()

2. localStorage 永久存储
     localStorage.setItem('key', 'value');
     localStorage.key = "value"
     localStorage["key"] = "value"

3. sessionStorage 临时存储，关闭浏览器丢失
     sessionStorage.setItem("age",'19')
```

![image-20220425214754903](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220425214754903.png)

#### `Header.vue`

```vue
<template>
  <div class="header">
    <div class="slogan">
      <p>老男孩IT教育 | 帮助有志向的年轻人通过努力学习获得体面的工作和生活</p>
    </div>
    <div class="nav">
      <ul class="left-part">
        <li class="logo">
          <router-link to="/">
            <img src="../assets/img/head-logo.svg" alt="" />
          </router-link>
        </li>
        <li class="ele">
          <span
            @click="goPage('/free-course')"
            :class="{ active: url_path === '/free-course' }"
            >免费课</span
          >
        </li>
        <li class="ele">
          <span
            @click="goPage('/actual-course')"
            :class="{ active: url_path === '/actual-course' }"
            >实战课</span
          >
        </li>
        <li class="ele">
          <span
            @click="goPage('/light-course')"
            :class="{ active: url_path === '/light-course' }"
            >轻课</span
          >
        </li>
      </ul>

      <div class="right-part">
        <div v-if="username">
          <span style="margin-right: 10px"
            ><img :src="icon" alt="" width="25px"
          /></span>
          <span>{{ username }}</span>
          <span class="line">|</span>
          <span @click="handleLogout">退出</span>
        </div>
        <div v-else>
          <span @click="put_login">登录</span>
          <span class="line">|</span>
          <span @click="put_register">注册</span>
        </div>
      </div>

      <Login v-if="is_login" @close="close_login" @go="put_register" />
      <Register v-if="is_register" @close="close_register" @go="put_login" />
    </div>
  </div>
</template>

<script>
import Login from "@/components/Login";
import Register from "@/components/Register";

export default {
  name: "Header",
  data() {
    return {
      url_path: sessionStorage.url_path || "/",
      is_login: false,
      is_register: false,
      username: "",
      icon: "",
    };
  },
  methods: {
    goPage(url_path) {
      // 已经是当前路由就没有必要重新跳转
      if (this.url_path !== url_path) {
        this.$router.push(url_path);
      }
      sessionStorage.url_path = url_path;
    },
    close_login() {
      this.is_login = false;
      // 登录了，从 cookie 中取出 username icon
      this.username = this.$cookies.get("username");
      this.icon = this.$cookies.get("icon");
    },
    close_register() {
      this.is_register = false;
    },
    put_register() {
      this.is_register = true;
      this.is_login = false;
    },
    put_login() {
      this.is_register = false;
      this.is_login = true;
    },
    handleLogout() {
      // 删除cookie 退出
      this.$cookies.set("username", "");
      this.$cookies.set("token", "");
      this.$cookies.set("icon", "");
      this.username = "";
      this.icon = "";
    },
  },
  created() {
    sessionStorage.url_path = this.$route.path;
    this.url_path = this.$route.path;
    this.username = this.$cookies.get("username");
    this.icon = this.$cookies.get("icon");
  },
  components: {
    Login,
    Register,
  },
};
</script>

<style scoped>
.header {
  background-color: white;
  box-shadow: 0 0 5px 0 #aaa;
}

.header:after {
  content: "";
  display: block;
  clear: both;
}

.slogan {
  background-color: #eee;
  height: 40px;
}

.slogan p {
  width: 1200px;
  margin: 0 auto;
  color: #aaa;
  font-size: 13px;
  line-height: 40px;
}

.nav {
  background-color: white;
  user-select: none;
  width: 1200px;
  margin: 0 auto;
}

.nav ul {
  padding: 15px 0;
  float: left;
}

.nav ul:after {
  clear: both;
  content: "";
  display: block;
}

.nav ul li {
  float: left;
}

.logo {
  margin-right: 20px;
}

.ele {
  margin: 0 20px;
}

.ele span {
  display: block;
  font: 15px/36px "微软雅黑";
  border-bottom: 2px solid transparent;
  cursor: pointer;
}

.ele span:hover {
  border-bottom-color: orange;
}

.ele span.active {
  color: orange;
  border-bottom-color: orange;
}

.right-part {
  float: right;
}

.right-part .line {
  margin: 0 10px;
}

.right-part span {
  line-height: 68px;
  cursor: pointer;
}
</style>
```

#### `Login.vue`

```vue
<template>
  <div class="login">
    <div class="box">
      <i class="el-icon-close" @click="close_login"></i>
      <div class="content">
        <div class="nav">
          <span
            :class="{ active: login_method === 'is_pwd' }"
            @click="change_login_method('is_pwd')"
            >密码登录</span
          >
          <span
            :class="{ active: login_method === 'is_sms' }"
            @click="change_login_method('is_sms')"
            >短信登录</span
          >
        </div>
        <el-form v-if="login_method === 'is_pwd'">
          <el-input
            placeholder="用户名/手机号/邮箱"
            prefix-icon="el-icon-user"
            v-model="username"
            clearable
          >
          </el-input>
          <el-input
            placeholder="密码"
            prefix-icon="el-icon-key"
            v-model="password"
            clearable
            show-password
          >
          </el-input>
          <el-button type="primary" @click="handlePasswordLogin"
            >登录</el-button
          >
        </el-form>
        <el-form v-if="login_method === 'is_sms'">
          <el-input
            placeholder="手机号"
            prefix-icon="el-icon-phone-outline"
            v-model="mobile"
            clearable
            @blur="check_mobile"
          >
          </el-input>
          <el-input
            placeholder="验证码"
            prefix-icon="el-icon-chat-line-round"
            v-model="sms"
            clearable
          >
            <template slot="append">
              <span class="sms" @click="send_sms">{{ sms_interval }}</span>
            </template>
          </el-input>
          <el-button type="primary" @click="handleMobileLogin">登录</el-button>
        </el-form>
        <div class="foot">
          <span @click="go_register">立即注册</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "Login",
  data() {
    return {
      username: "",
      password: "",
      mobile: "",
      sms: "",
      login_method: "is_pwd",
      sms_interval: "获取验证码",
      is_send: false,
    };
  },
  methods: {
    close_login() {
      this.$emit("close");
    },
    go_register() {
      this.$emit("go");
    },
    change_login_method(method) {
      this.login_method = method;
    },
    check_mobile() {
      if (!this.mobile) return;
      if (!this.mobile.match(/^1[3-9][0-9]{9}$/)) {
        this.$message({
          message: "手机号有误",
          type: "warning",
          duration: 1000,
          onClose: () => {
            this.mobile = "";
          },
        });
        return false;
      }
      this.is_send = true;
    },
    send_sms() {
      if (!this.is_send) return;
      this.is_send = false;
      let sms_interval_time = 60;
      this.sms_interval = "发送中...";
      let timer = setInterval(() => {
        if (sms_interval_time <= 1) {
          clearInterval(timer);
          this.sms_interval = "获取验证码";
          this.is_send = true; // 重新回复点击发送功能的条件
        } else {
          sms_interval_time -= 1;
          this.sms_interval = `${sms_interval_time}秒后再发`;
        }
      }, 1000);
      // 发送短信验证码
      this.$axios
        .get(
          this.$settings.base_url +
            "user/send/send_message/?phone=" +
            this.mobile
        )
        .then((res) => {
          if (res.data.status == 100) {
            this.$message({
              message: "验证码发送成功",
              type: "success",
            });
          } else {
            this.$message({
              message: "验证码发送失败，请稍后再试",
              type: "warning",
            });
          }
        });
    },
    handlePasswordLogin() {
      // 用户名和密码是否填入
      if (this.username && this.password) {
        // 向后端请求
        this.$axios
          .post(this.$settings.base_url + "user/login/mul_login/", {
            username: this.username,
            password: this.password,
          })
          .then((res) => {
            if (res.data.status == 100) {
              console.log(res.data);

              // 1. 把 token 和 username 存到 cookie 中
              this.$cookies.set("username", res.data.data.username, "7d");
              this.$cookies.set("token", res.data.data.token, "7d");
              this.$cookies.set("icon", res.data.data.icon, "7d");
              this.$message({
                message: "恭喜你，登录成功",
                type: "success",
              });
              // 2. 关闭登录框
              this.close_login();
            } else {
              this.$message.error(res.data.msg);
            }
          });
      } else {
        this.$message.error("用户名和密码必填");
      }
    },
    handleMobileLogin() {
      if (this.mobile && this.sms) {
        // 向后端请求
        this.$axios
          .post(this.$settings.base_url + "user/login/sms_login/", {
            mobile: this.mobile,
            code: this.sms,
          })
          .then((res) => {
            if (res.data.status == 100) {
              console.log(res.data);

              // 1. 把 token 和 username 存到 cookie 中
              this.$cookies.set("username", res.data.data.username);
              this.$cookies.set("token", res.data.data.token);
              this.$cookies.set("icon", res.data.data.icon);
              this.$message({
                message: "恭喜你，登录成功",
                type: "success",
              });
              // 2. 关闭登录框
              this.close_login();
            } else {
              this.$message.error(res.data.msg);
            }
          });
      } else {
        this.$message.error("用户名和密码必填");
      }
    },
  },
};
</script>

<style scoped>
.login {
  width: 100vw;
  height: 100vh;
  position: fixed;
  top: 0;
  left: 0;
  z-index: 10;
  background-color: rgba(0, 0, 0, 0.3);
}

.box {
  width: 400px;
  height: 420px;
  background-color: white;
  border-radius: 10px;
  position: relative;
  top: calc(50vh - 210px);
  left: calc(50vw - 200px);
}

.el-icon-close {
  position: absolute;
  font-weight: bold;
  font-size: 20px;
  top: 10px;
  right: 10px;
  cursor: pointer;
}

.el-icon-close:hover {
  color: darkred;
}

.content {
  position: absolute;
  top: 40px;
  width: 280px;
  left: 60px;
}

.nav {
  font-size: 20px;
  height: 38px;
  border-bottom: 2px solid darkgrey;
}

.nav > span {
  margin: 0 20px 0 35px;
  color: darkgrey;
  user-select: none;
  cursor: pointer;
  padding-bottom: 10px;
  border-bottom: 2px solid darkgrey;
}

.nav > span.active {
  color: black;
  border-bottom: 3px solid black;
  padding-bottom: 9px;
}

.el-input,
.el-button {
  margin-top: 40px;
}

.el-button {
  width: 100%;
  font-size: 18px;
}

.foot > span {
  float: right;
  margin-top: 20px;
  color: orange;
  cursor: pointer;
}

.sms {
  color: orange;
  cursor: pointer;
  display: inline-block;
  width: 70px;
  text-align: center;
  user-select: none;
}
</style>
```

#### `Register.vue`

```vue
<template>
  <div class="register">
    <div class="box">
      <i class="el-icon-close" @click="close_register"></i>
      <div class="content">
        <div class="nav">
          <span class="active">新用户注册</span>
        </div>
        <el-form>
          <el-input
            placeholder="手机号"
            prefix-icon="el-icon-phone-outline"
            v-model="mobile"
            clearable
            @blur="check_mobile"
          >
          </el-input>
          <el-input
            placeholder="密码"
            prefix-icon="el-icon-key"
            v-model="password"
            clearable
            show-password
          >
          </el-input>
          <el-input
            placeholder="验证码"
            prefix-icon="el-icon-chat-line-round"
            v-model="sms"
            clearable
          >
            <template slot="append">
              <span class="sms" @click="send_sms">{{ sms_interval }}</span>
            </template>
          </el-input>
          <el-button type="primary" @click="handleRegister">注册</el-button>
        </el-form>
        <div
          class=" foot
          "
        >
          <span @click="go_login">立即登录</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "Register",
  data() {
    return {
      mobile: "",
      password: "",
      sms: "",
      sms_interval: "获取验证码",
      is_send: false,
    };
  },
  methods: {
    close_register() {
      this.$emit("close", false);
    },
    go_login() {
      this.$emit("go");
    },
    check_mobile() {
      if (!this.mobile) return;
      if (!this.mobile.match(/^1[3-9][0-9]{9}$/)) {
        this.$message({
          message: "手机号有误",
          type: "warning",
          duration: 1000,
          onClose: () => {
            this.mobile = "";
          },
        });
        return false;
      }
      this.is_send = true;
    },
    send_sms() {
      if (!this.is_send) return;
      this.is_send = false;
      let sms_interval_time = 60;
      this.sms_interval = "发送中...";
      let timer = setInterval(() => {
        if (sms_interval_time <= 1) {
          clearInterval(timer);
          this.sms_interval = "获取验证码";
          this.is_send = true; // 重新回复点击发送功能的条件
        } else {
          sms_interval_time -= 1;
          this.sms_interval = `${sms_interval_time}秒后再发`;
        }
      }, 1000);
      // 发送短信
      this.$axios
        .get(
          this.$settings.base_url +
            "user/send/send_message/?phone=" +
            this.mobile
        )
        .then((res) => {
          if (res.data.status == 100) {
            this.$message({
              message: "验证码发送成功",
              type: "success",
            });
          } else {
            this.$message({
              message: "验证码发送失败，请稍后再试",
              type: "warning",
            });
          }
        });
    },
    handleRegister() {
      if (this.mobile && this.sms && this.password) {
        // 向后端请求
        this.$axios
          .post(this.$settings.base_url + "user/register/", {
            mobile: this.mobile,
            code: this.sms,
            password: this.password,
          })
          .then((res) => {
            if (res.data.status == 100) {
              console.log(res.data);
              this.$message({
                message: "恭喜你，注册成功",
                type: "success",
              });
              // 关闭注册框
              this.close_register();
            } else {
              this.$message.error(res.data.msg);
            }
          });
      } else {
        this.$message.error("用户名和密码必填");
      }
    },
  },
};
</script>

<style scoped>
.register {
  width: 100vw;
  height: 100vh;
  position: fixed;
  top: 0;
  left: 0;
  z-index: 10;
  background-color: rgba(0, 0, 0, 0.3);
}

.box {
  width: 400px;
  height: 480px;
  background-color: white;
  border-radius: 10px;
  position: relative;
  top: calc(50vh - 240px);
  left: calc(50vw - 200px);
}

.el-icon-close {
  position: absolute;
  font-weight: bold;
  font-size: 20px;
  top: 10px;
  right: 10px;
  cursor: pointer;
}

.el-icon-close:hover {
  color: darkred;
}

.content {
  position: absolute;
  top: 40px;
  width: 280px;
  left: 60px;
}

.nav {
  font-size: 20px;
  height: 38px;
  border-bottom: 2px solid darkgrey;
}

.nav > span {
  margin-left: 90px;
  color: darkgrey;
  user-select: none;
  cursor: pointer;
  padding-bottom: 10px;
  border-bottom: 2px solid darkgrey;
}

.nav > span.active {
  color: black;
  border-bottom: 3px solid black;
  padding-bottom: 9px;
}

.el-input,
.el-button {
  margin-top: 40px;
}

.el-button {
  width: 100%;
  font-size: 18px;
}

.foot > span {
  float: right;
  margin-top: 20px;
  color: orange;
  cursor: pointer;
}

.sms {
  color: orange;
  cursor: pointer;
  display: inline-block;
  width: 70px;
  text-align: center;
  user-select: none;
}
</style>
```
