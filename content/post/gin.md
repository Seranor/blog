---
title: Gin框架
lastmod: 2021-10-20T16:43:23+08:00
date: 2021-10-20T11:52:03+08:00
tags:
  - Gin
  - Golang
categories:
  - Gin
  - Golang
url: post/gin.html
toc: true
---

## Gin 框架

### 官网

```go
https://github.com/gin-gonic/gin
https://gin-gonic.com/
```

<!-- more -->

### 安装

```go
go get -u github.com/gin-gonic/gin
```

### 快速使用

```go
package main

import (
	"github.com/gin-gonic/gin"
	"net/http"
)

func pong(c *gin.Context) {
	c.JSON(http.StatusOK, gin.H{  // type H map[string]interface{}
		"message": "pong",
	})
}

func main() {
	r := gin.Default()
	r.GET("/ping", pong)
	r.Run(":8081")
}
```

### 配置方法

```go
package main

import "github.com/gin-gonic/gin"

func main() {
	// router := gin.New()
	// Default 会默认 开启 logger h和 recovery (crach-free) 中间件
	router := gin.Default()
	router.GET("/someGet", getting)
	router.POST("/somePost", posting)
	router.PUT("/somePut", putting)
	router.DELETE("/someDelete", deleteing)
	router.PATCH("/somePatch", patching)
	router.HEAD("/someHead", head)
	router.OPTIONS("/someOptions", options)
	router.Run()
}
// 方法的函数没有写  只是格式
```

### 路由初始化

```go
package main

import (
	"github.com/gin-gonic/gin"
	"net/http"
)

func pong(c *gin.Context) {
	// panic("异常了")
	c.JSON(http.StatusOK, gin.H{ // type H map[string]interface{}
		"message": "pong",
	})
}

func main() {
	// r := gin.New()   // 如果使用该方式 遇到异常会出现 ERR_EMPTY_RESPONSE
	r := gin.Default()  // Default 会默认 开启 logger 和 recovery (crach-free) 中间件
	r.GET("/ping", pong)
	r.Run(":8081")
}
```

### 路由分组

```go
package main

import (
	"github.com/gin-gonic/gin"
	"net/http"
)

func main() {
	router := gin.Default()

	//router.GET("/goods/list", goodsList)
	//router.GET("/goods/1", goodsDetail)
	//router.POST("/goods/add", createGoods)

	goodsGroup := router.Group("/goods")
	{
		goodsGroup.GET("/list", goodsList)
		goodsGroup.GET("/1", goodsDetail)
		goodsGroup.POST("/add", createGoods)
	}
	router.Run(":8081")
}

func goodsList(context *gin.Context) {
	context.JSON(http.StatusOK, gin.H{
		"name": "goodsList",
	})
}
func goodsDetail(context *gin.Context) {}
func createGoods(context *gin.Context) {}

// 访问 127.0.0.1:8081/goods/list
```

### url 参数

```go
package main

import (
	"github.com/gin-gonic/gin"
	"net/http"
)

func main() {
	router := gin.Default()

	//router.GET("/goods/list", goodsList)
	//router.GET("/goods/1", goodsDetail)
	//router.POST("/goods/add", createGoods)

	goodsGroup := router.Group("/goods")
	{
		goodsGroup.GET("/list", goodsList)
		goodsGroup.GET("/:id/:action", goodsDetail) // 获取 id  的详细信息
		//goodsGroup.GET("/:id/:action/add", goodsDetail) // 需要强制加上 add 才可以
		//goodsGroup.GET("/:id/*action", goodsDetail) // id 后面所有的字符串都会给 action
		goodsGroup.POST("/add", createGoods)
	}

	router.Run(":8081")
}

func goodsList(context *gin.Context) {
	context.JSON(http.StatusOK, gin.H{
		"name": "goodsList",
	})
}
func goodsDetail(c *gin.Context) {
	action := c.Param("action")
	id := c.Param("id")
	c.JSON(http.StatusOK, gin.H{
		"id":     id,
		"action": action,
	})
}
func createGoods(context *gin.Context) {}

//
/*
访问 127.0.0.1:8081/goods/1/add
返回的是
{
  "id":1,
  "action":add
}

约束性不强
*/
```

### url 参数约束性

```go
package main

import (
	"github.com/gin-gonic/gin"
	"net/http"
)

type Person struct {
	//ID   string `uri:"id" binding:"required,uuid"`  // id 必须是 uuid 否则 404
	ID   int    `uri:"id" binding:"required"`         // 这时 id 就必须是 int 类型
	Name string `uri:"name" binding:"required"`
}

func main() {
	router := gin.Default()
	router.GET("/:name/:id", func(context *gin.Context) {
		var person Person
		if err := context.ShouldBindUri(&person); err != nil {
			context.Status(404)
			return
		}
		context.JSON(http.StatusOK, gin.H{
			"name": person.Name,
			"id":   person.ID,
		})
	})
	router.Run(":8081")
}

/*
127.0.0.1:8081/lzj/1

{
    "id": 1,
    "name": "lzj"
}

*/
```

### 获取 get 和 post 的参数

#### 获取 get 参数

```go
package main

import (
	"github.com/gin-gonic/gin"
	"net/http"
)

func main() {
	router := gin.Default()
	router.GET("/welcome", welcome)
	router.Run(":8081")
}

func welcome(context *gin.Context) {
	firstName := context.DefaultQuery("firstname", "liu")
	lastName := context.DefaultQuery("lastname", "zhijin")
	//lastName := context.Query("lastname")  // 这种事不带默认值的
	context.JSON(http.StatusOK, gin.H{
		"firstname": firstName,
		"lastname":  lastName,
	})

}


/*
127.0.0.1:8081/welcome  返回默认的
{
    "firstname": "liu",
    "lastname": "zhijin"
}


127.0.0.1:8081/welcome?firstname=lll&lastname=jjj
{
    "firstname": "lll",
    "lastname": "jjj"
}
*/
```

#### 获取 post 参数

```go
package main

import (
	"github.com/gin-gonic/gin"
	"net/http"
)

func main() {
	router := gin.Default()
	router.POST("/from", formPost)
	router.Run(":8081")
}

func formPost(c *gin.Context) {
	message := c.PostForm("message")
	nick := c.DefaultPostForm("nick", "anonymous")
	c.JSON(http.StatusOK, gin.H{
		"message": message,
		"nick":    nick,
	})
}

/*
127.0.0.1:8081/from
postman 根据 使用 post 请求 body form-data
message info
nick    admin   不带该参数就会返回默认的  anonymous

结果
{
    "message": "info",
    "nick": "admin"
}

*/
```

#### 获取混合参数

```go
package main

import (
	"github.com/gin-gonic/gin"
	"net/http"
)

func main() {
	router := gin.Default()
	router.POST("/post", getPost)
	router.Run(":8081")
}

func getPost(c *gin.Context) {
	id := c.Query("id")
	page := c.DefaultQuery("page", "0")
	name := c.PostForm("name")
	message := c.DefaultPostForm("message", "info")
	c.JSON(http.StatusOK, gin.H{
		"id":      id,
		"page":    page,
		"name":    name,
		"message": message,
	})
}

/*
请求url  127.0.0.1:8081/post?id=1&page=2
使用postman post 请求
message  isok
name     liu

返回结果
{
    "id": "1",
    "message": "is ok",
    "name": "liu",
    "page": "2"
}
*/
```

### `JSON`渲染

```go
package main

import (
	"github.com/gin-gonic/gin"
	"net/http"
)

func main() {
	router := gin.Default()
	router.GET("/moreJson", moreJson)
	router.Run(":8080")
}

func moreJson(c *gin.Context) {
	var msg struct {
		Name    string `json:"user"`
		Message string
		Number  int
	}
	msg.Name = "klcc"
	msg.Message = "这是一个测试json"
	msg.Number = 18

	c.JSON(http.StatusOK, msg)
}

/*

{
    "user": "klcc",
    "Message": "这是一个测试json",
    "Number": 18
}

*/
```

### `ProtoBuf`渲染

```protobuf
syntax = "proto3";

option go_package = "../proto";

message Teacher {
  string name = 1;
  repeated string course = 2;
}
```

#### server

```go
//  protoc -I . user.proto --go_out=plugins=grpc:.
package main

import (
	"PackageTest/gin_start/ch06/proto"
	"github.com/gin-gonic/gin"
	"net/http"
)

func main() {
	router := gin.Default()
	router.GET("/moreJson", moreJson)
	router.GET("/someProtoBuf", returnProto)
	router.Run(":8080")
}

func returnProto(c *gin.Context) {
	course := []string{"python", "go"}
	user := &proto.Teacher{
		Name:   "klcc",
		Course: course,
	}
	c.ProtoBuf(http.StatusOK, user)
}

func moreJson(c *gin.Context) {
	var msg struct {
		Name    string `json:"user"`
		Message string
		Number  int
	}
	msg.Name = "klcc"
	msg.Message = "这是一个测试json"
	msg.Number = 18

	c.JSON(http.StatusOK, msg)
}
```

#### client

```python
import requests
from request_test.proto import user_pb2

user = user_pb2.Teacher()
res = requests.get("http://127.0.0.1:8080/someProtoBuf")
user.ParseFromString(res.content)
print(user.name, user.course)  # klcc ['python', 'go']

#  python3 -m grpc_tools.protoc --python_out=. --grpc_python_out=. -I. user.proto
```

### 表单验证

```go
// https://github.com/go-playground/validator
package main

import (
	"fmt"
	"github.com/gin-gonic/gin"
	"github.com/gin-gonic/gin/binding"
	"github.com/go-playground/locales/en"
	"github.com/go-playground/locales/zh"
	ut "github.com/go-playground/universal-translator"
	"github.com/go-playground/validator/v10"
	en_translations "github.com/go-playground/validator/v10/translations/en"
	zh_translations "github.com/go-playground/validator/v10/translations/zh"
	"net/http"
	"reflect"
	"strings"
)

var trans ut.Translator

type LoginForm struct {
	//User     string `form:"user" json:"user" xml:"user" binding:"required"`  // 可以有多种格式
	User     string `json:"user" binding:"required,min=3,max=10"`  // 字符最短为3 最长为10 且必填
	Password string `json:"password" binding:"required"`
}

type SinUpForm struct {
	Age        uint8  `json:"age" binding:"gte=18,lte=50"`
	Name       string `json:"name" binding:"required,min=3"`
	Email      string `json:"email" binding:"required,email"`
	Password   string `json:"password" binding:"required"`
	RePassword string `json:"repassword" binding:"required,eqfield=Password"` // 跨字段可以使用 eqfield 等于
}

// 处理错误信息 将 "LoginForm.user" --> "user"
func removeTopStruct(fileds map[string]string) map[string]string {
	rsp := map[string]string{}
	for field, err := range fileds {
		rsp[field[strings.Index(field, ".")+1:]] = err
		fmt.Println(err)
		fmt.Println(field)
		fmt.Println(rsp)
	}
	return rsp
}

// 语言国际化处理
func InitTrans(locale string) (err error) {
	// 修改 gin 框架中环南的 validator 引擎 实现定制
	if v, ok := binding.Validator.Engine().(*validator.Validate); ok {
		// 注册一个获取 json 的 tag 的自定义方法
		v.RegisterTagNameFunc(func(fld reflect.StructField) string {
			name := strings.SplitN(fld.Tag.Get("json"), ",", 2)[0]
			if name == "-" {
				return ""
			}
			return name
		})

		zhT := zh.New() // 中文翻译器
		enT := en.New() // 英文翻译器
		// 第一个是备用的语言环境 后面是应该支持的语言环境
		uni := ut.New(enT, zhT, enT)
		trans, ok = uni.GetTranslator(locale)
		if !ok {
			return fmt.Errorf("uni.GetTranslator(%s)", locale)
		}
		switch locale {
		case "en":
			en_translations.RegisterDefaultTranslations(v, trans)
		case "zh":
			zh_translations.RegisterDefaultTranslations(v, trans)
		default:
			en_translations.RegisterDefaultTranslations(v, trans)
		}
		return
	}
	return
}

func main() {
	if err := InitTrans("zh"); err != nil {
		fmt.Println("初始化翻译器错误")
		return
	}
	router := gin.Default()
	router.POST("/loginJson", func(c *gin.Context) {
		var loginForm LoginForm
		if err := c.ShouldBind(&loginForm); err != nil {
			errs, ok := err.(validator.ValidationErrors)
			if !ok {  // 如果使用国际化失败 使用原本的错误信息格式
				c.JSON(http.StatusOK, gin.H{
					"msg": err.Error(),
				})
			}
			c.JSON(http.StatusBadRequest, gin.H{
        // 使用 removeTopStruct 处理后 "LoginForm.user"  --> "user"
				"error": removeTopStruct(errs.Translate(trans)),
			})
			return
		}

		c.JSON(http.StatusOK, gin.H{
			"msg": "登录成功",
		})
	})
	router.POST("/siginup", func(c *gin.Context) {
		var signForm SinUpForm
		if err := c.ShouldBind(&signForm); err != nil {
			fmt.Println(err.Error())
			c.JSON(http.StatusBadRequest, gin.H{
				"error": err.Error(),
			})
			return
		}
		c.JSON(http.StatusOK, gin.H{
			"msg": "注册成功",
		})
	})
	router.Run(":8080")
}
```

### 自定义中间件

```go
// 例统计运行时间 不建议写在函数内 代码侵入性强

package main

import (
	"fmt"
	"github.com/gin-gonic/gin"
	"net/http"
	"time"
)

func MyLogger() gin.HandlerFunc {
	return func(c *gin.Context) {
		t := time.Now()
		c.Set("test", "123456")
		// 让原本该执行的逻辑继续执行
		c.Next()
		end := time.Since(t) // 给一个开始时间 返回与当前相差的时间
		fmt.Printf("耗时：%v\n", end)
    fmt.Println(c.Writer.Status()) // 打印状态码
	}
}

func main() {
	//router := gin.New()
	// 使用 logger 中间件和 recovery 中间件 全局使用
	//router.Use(gin.Logger(), gin.Recovery())

	//authrized := router.Group("/goods")
	// /goods 组内的 url 添加中间件
	//authrized.Use(AuthRequired)

	router := gin.Default()
	router.Use(MyLogger())
	router.GET("/ping", func(c *gin.Context) {
		c.JSON(http.StatusOK, gin.H{
			"message": "pong",
		})
	})
	router.Run(":8080")
}
```

#### 中间件的终止

```go
package main

import (
	"fmt"
	"github.com/gin-gonic/gin"
	"net/http"
	"time"
)

func TokenRequired() gin.HandlerFunc {
	return func(c *gin.Context) {
		var token string
		for k, v := range c.Request.Header {
			if k == "X-Token" { // 做了转换 X 和 T 是大写
				token = v[0]
			} else {
				fmt.Println(k, v)
			}
		}
		if token != "bobby" {
			c.JSON(http.StatusUnauthorized, gin.H{
				"message": "未登录",
			})
			//return // 没有起到作用
			c.Abort() // 终止
		}
		c.Next()
	}
}

func main() {
	router := gin.Default()
	router.Use(TokenRequired())
	router.GET("/ping", func(c *gin.Context) {
		c.JSON(http.StatusOK, gin.H{
			"message": "pong",
		})
	})
	router.Run(":8080")
}

/*
gin框架中  中间件和逻辑函数 在一个队列中的
index 控制执行顺序
初始化一个index
Next() 会往下继续执行
func (c *Context) Next() {
	c.index++
	for c.index < int8(len(c.handlers)) {
		c.handlers[c.index](c)
		c.index++
	}
}

如果遇到 return 只是断开的当前函数，并不会影响到 gin 的总体
如果需要停止 需要将 index 指向到不存在的位置即可
Abort()

func (c *Context) Abort() {
	c.index = abortIndex
}

const abortIndex int8 = math.MaxInt8 / 2

*/
```

### `html`模板

#### 返回单个模板文件

```go
// 模板  https://pkg.go.dev/html/template

// 模板文件 templates/index.tmpl
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <h1>{{ .title}}</h1>
</body>
</html>

// main.go
package main

import (
	"fmt"
	"github.com/gin-gonic/gin"
	"net/http"
	"os"
	"path/filepath"
)

func main() {
	router := gin.Default()

	// 会将指定目录下的文件加载好 相对目录
	// goland 运行 main.go 文件时并没有生成执行文件
	dir, _ := filepath.Abs(filepath.Dir(os.Args[0]))
	fmt.Println(dir) // 因为会在一个临时目录中 因此模板文件并没有加载到这二
	// 使用相对路径在goland 中会报错  在加载静态文件时 在控制台直接 go run
	router.LoadHTMLFiles("templates/index.tmpl")
	router.GET("/index", func(c *gin.Context) {
		c.HTML(http.StatusOK, "index.tmpl", gin.H{
			"title": "klcc",
		})
	})
	router.Run(":8080")
}
```

#### 返回多个模板文件

`templates/defaults/index.tmpl`

```
{{define "myindex.tmpl"}}
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>Title</title>
    </head>
    <body>
    <h1>{{ .title}}</h1>
    </body>
    </html>
{{end}}
```

`templates/goods/list.tmpl`

```
{{define "goods/list.tmpl"}}
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>商品列表</title>
    </head>
    <body>
        <h1>{{.title}}</h1>
    </body>
    </html>
{{end}}
```

`templates/user/list.tmpl`

```
{{define "user/list.tmpl"}}
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>用户列表</title>
    </head>
    <body>
    <h1>{{ .title }}</h1>
    </body>
    </html>
{{end}}
```

`main.go`

```go
package main

import (
	"github.com/gin-gonic/gin"
	"net/http"
)

func main() {
	router := gin.Default()
	router.LoadHTMLGlob("templates/**/*")  // ** 是目录 * 是代表文件 不会加载  templates 下的文件了
	router.GET("/index", func(c *gin.Context) {
		c.HTML(http.StatusOK, "myindex.tmpl", gin.H{
			"title": "klcc",
		})
	})

	router.GET("/user/list", func(c *gin.Context) {
		c.HTML(http.StatusOK, "user/list.tmpl", gin.H{  //  可以使用define指定的别名 来进行区分
			"title": "用户",
		})
	})

	router.GET("/goods/list", func(c *gin.Context) {
		c.HTML(http.StatusOK, "goods/list.tmpl", gin.H{
			"title": "商品",
		})
	})

	router.Run(":8080")
}
```

### 静态文件处理

`static/css/style.css`

```go
h1{
    background-color: red;
}
```

`templates/goods/list.tmpl`

```
{{define "goods/list.tmpl"}}
    <!DOCTYPE html>
    <html lang="en">
    <link rel="stylesheet" href="/static/css/style.css">
    <head>
        <meta charset="UTF-8">
        <title>商品列表</title>
    </head>
    <body>
        <h1>{{.title}}</h1>
    </body>
    </html>
{{end}}
```

`main.go`

```go
package main

import (
	"github.com/gin-gonic/gin"
	"net/http"
)

func main() {
	router := gin.Default()
	router.Static("/static", "./static")
	router.LoadHTMLGlob("templates/**/*")
	router.GET("/goods/list", func(c *gin.Context) {
		c.HTML(http.StatusOK, "goods/list.tmpl", gin.H{
			"title": "商品",
		})
	})

	router.Run(":8080")
}

```

### `gin`的优雅退出

```go
// https://gin-gonic.com/docs/examples/graceful-restart-or-stop/

package main

import (
	"fmt"
	"github.com/gin-gonic/gin"
	"net/http"
	"os"
	"os/signal"
	"syscall"
)

func main() {
	// 当关闭程序的时候应该做的后续处理
	// 微服务 启动之前或之后 会将当前的服务 ip 地址和端口号注册到服务中心
	// 服务停止之后并没有告知注册中心

	router := gin.Default()
	router.GET("/", func(c *gin.Context) {
		c.JSON(http.StatusOK, gin.H{
			"message": "pong",
		})
	})
	go func() {
		router.Run(":8080")
	}()

	// 如果想要接收到信号 kill -9 是不能收到消息的
	quit := make(chan os.Signal)
	signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
	<-quit

	// 处理后续的逻辑
	fmt.Println("关闭服务中....")
	fmt.Println("注销服务")
}
```
