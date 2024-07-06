---
title: Selenium的使用和xpath使用
lastmod: 2021-06-03T16:43:23+08:00
date: 2021-06-02T11:52:03+08:00
tags:
  - crawler
categories:
  - crawler
url: post/crawler-03.html
toc: true
---

### `Selenium`的使用

```python
requestsm模块，可以发送http请求，但是有的页面是由render+ajax渲染完的，如果只使用requestes，它只能执行render的请求，拿回数据，执行ajax的请求，需要你再去分析，再去发请求

selenium，控制浏览器，操作浏览器，完成人的行为-->自动化测试工具
本质是python通过代码，借助于浏览器驱动，操作浏览器，真正的实现了，可见即可爬

安装模块
pip3 install selenium

下载谷歌浏览器驱动(版本最接近的也可以)
https://registry.npmmirror.com/binary.html?path=chromedriver/

```

<!-- more -->

#### 基本使用

```python
import time
from selenium import webdriver

# chromedriver 在同级目录下 Mac
bro = webdriver.Chrome(executable_path="./chromedriver")

# 在地址栏输入地址
bro.get("https://www.baidu.com")

# 找到输入框
search = bro.find_element_by_id("kw")

# 在输入框中输入 python
search.send_keys("python")

# 找到百度一下按钮
button = bro.find_element_by_id("su")

# 点击按钮
button.click()

time.sleep(3)
print(bro.page_source)  # 打印当前页面的 html
with open('baidu.html', 'w', encoding='utf-8') as f:
    f.write(bro.page_source)  # 包含redner+ajax
bro.close()
```

#### 无头浏览器

```python
# 不显示浏览器 在后台运行 完成
from selenium import webdriver
from selenium.webdriver.chrome.options import Options

# 得到一个配置对象
chrome_options = Options()
chrome_options.add_argument('window-size=1920x3000')  # 指定浏览器分辨率
chrome_options.add_argument('--disable-gpu')  # 谷歌文档提到需要加上这个属性来规避bug
chrome_options.add_argument('--hide-scrollbars')  # 隐藏滚动条, 应对一些特殊页面
chrome_options.add_argument('blinfk-settings=imagesEnabled=alse')  # 不加载图片, 提升速度
chrome_options.add_argument('--headless')  # 浏览器不提供可视化页面. linux下如果系统不支持可视化不加这条会启动失败

bro = webdriver.Chrome(executable_path='./chromedriver', options=chrome_options)
bro.get('http://www.cnblogs.com')
print(bro.page_source)
bro.close()
```

#### 获取元素位置、属性、大小

```python
# 一般用在验证码破解上

import time
from selenium import webdriver

bro = webdriver.Chrome(executable_path="./chromedriver")
bro.get("https://www.jd.com/")

# 找到图片
img = search = bro.find_element_by_css_selector("a.logo_tit_lk")
print(img.size)  # 图片大小  通过位置和大小可以唯一确定这张图，通过截图可以把图截出来
print(img.location)  # 图片位置 {'x': 105, 'y': 41}
print(img.id)  # selenium提供的id号，忽略
print(img.tag_name)  # a

location = img.location
size = img.size
time.sleep(5)
bro.save_screenshot('./jd.png')  # 把整个页面保存成图片

# pip3 install pillow
from PIL import Image

# pillow抠图，把图标抠出来
# 第一个参数 开始截图的x坐标
# 第二个参数 开始截图的y坐标
# 第三个参数 结束截图的x坐标
# 第四个参数 结束截图的y坐标
img_tu = (
    int(location['x']), int(location['y']), int(location['x'] + size['width']), int(location['y'] + size['height']))
# 使用pillow打开截图
img = Image.open('./jd.png')
# 从截图中按照位置扣除验证码
code_img = img.crop(img_tu)
# 把扣出来的图，保存到本地
code_img.save('./code.png')
bro.close()

'''
补充：标签位置和大小:size和location
一般用来扣验证码图片：可能会由于分辨率问题导致扣出的图不一致，通过修改分辨率，实现正确抠图
验证码是img--->src 自己加载就能拿到验证码，保存到本地即可(requests)更简单
'''
```

#### 等待元素被加载

```python
# 代码操作，速度非常快，可能标签还没有加载出来，代码就去取标签操作，所以找不到标签，报错
# 等待标签加载完成再取
    显示等待：每个标签都要写等待逻辑
    隐式等待：任何要取的标签都遵循这个逻辑，只需要写一次（推荐用）
    bro.implicitly_wait(10) # 取这个标签，如果取不到就等待，直到标签加载完成或10s到了
```

#### 元素操作

```python
import time
from selenium import webdriver

'''
查找标签的方式: selenium:find_element_by_xx,find_elements_by_xx
find_element_by_id                 # 通过id找
find_element_by_link_text          # 通过a标签文字
find_element_by_partial_link_text  # 通过a标签文字模糊找
find_element_by_tag_name           # 通过标签名找
find_element_by_class_name         # 通过类名找
find_element_by_name               # 通过name属性找
find_element_by_css_selector       # css选择器
find_element_by_xpath              # xpath选择器
'''

bro = webdriver.Chrome(executable_path="./chromedriver")
bro.get("https://www.baidu.com/")
bro.implicitly_wait(10)

# 查找a标签文本内容是登陆的
login_a = bro.find_element_by_link_text('登录')
login_a.click()  # 点击 a 标签 点击登录

# 找到账号登录点击
login_pwd_btn = bro.find_element_by_id('TANGRAM__PSP_11__changePwdCodeItem')
login_pwd_btn.click()

# 找到用户名输入框填写
username = bro.find_element_by_name('userName')
username.send_keys('xxxx@qq.com')

# 清空用户名重新填写
time.sleep(3)
username.clear()
username.send_keys('xxxx@qq.com')

# 找到密码框填写密码
pwd = bro.find_element_by_css_selector('#TANGRAM__PSP_11__password')
pwd.send_keys('xxxx')

# 找到登录按钮点击
submit = bro.find_element_by_css_selector('#TANGRAM__PSP_11__submit')
submit.click()
# 手动操作解决出现的验证识别

input()
time.sleep(5)
bro.close()

'''
登录的目的是为了拿到cookie发送请求
可以批量操作账号操作网页的功能 自动回复点赞等
'''
```

#### 执行`js`

```python
普遍常用，在本地页面中直接执行js代码
第一种情况，控制操作页面滑动
第二种情况，使用当前页面中的一些变量，执行页面中的函数
```

```python
import time
from selenium import webdriver

bro = webdriver.Chrome(executable_path="./chromedriver")
bro.get("https://www.pearvideo.com/category_9")
bro.implicitly_wait(10)

# 执行js
# bro.execute_script("alert('hello')")

# 操作页面滑动
bro.execute_script('window.scrollBy(0, document.body.scrollHeight)')
time.sleep(1)
bro.execute_script('window.scrollBy(0, document.body.scrollHeight)')
time.sleep(1)
bro.execute_script('window.scrollBy(0, document.body.scrollHeight)')
time.sleep(1)


# 使用当前页面中的一些变量 执行页面中的函数
bro.execute_script('alert(md5_vm_test())')

time.sleep(5)
bro.close()
```

#### 切换选项卡

```python
import time
from selenium import webdriver

browser = webdriver.Chrome(executable_path='./chromedriver')
browser.get('https://www.baidu.com')

# 打开选项卡
browser.execute_script('window.open()')
print(browser.window_handles)  # 获取所有的选项卡

browser.switch_to.window(browser.window_handles[1])
browser.get('https://www.taobao.com')
time.sleep(2)

browser.switch_to.window(browser.window_handles[0])
browser.get('https://www.sina.com.cn')

browser.close()  # 关闭当前选项卡
browser.quit()  # 退出浏览器
```

#### 模拟前进后退

```python
import time
from selenium import webdriver

browser = webdriver.Chrome(executable_path='./chromedriver')
browser.get('https://www.baidu.com')
browser.get('https://www.taobao.com')
browser.get('http://www.sina.com.cn/')

browser.back()  # sina 回退到 taobao
time.sleep(2)
browser.forward()  # taobao 前进到 sina
browser.close()
```

#### 异常处理

```python
from selenium import webdriver
from selenium.common.exceptions import TimeoutException, NoSuchElementException, NoSuchFrameException

browser = webdriver.Chrome(executable_path='./chromedriver')
try:
    browser.get('http://www.runoob.com/try/try.php?filename=jqueryui-api-droppable')
    browser.switch_to.frame('iframssseResult')

except TimeoutException as e:
    print(e)
except NoSuchFrameException as e:
    print(e)
finally:
    browser.close()
```

#### `selenium`登录`cnblogs`获取 `cookie`

```python
使用 selenuim 半自动登录 cnblogs 取出 cookie 存到本地
下次使用 selenuim 访问 cnblogs 加载 cookie 状态就是登录状态了
```

##### 登录获取`cookie`

```python
import time
import json
from selenium import webdriver

bro = webdriver.Chrome(executable_path='./chromedriver')
try:
    bro.get('http://www.cnblogs.com')
    bro.implicitly_wait(10)
    sub_a = bro.find_element_by_link_text("登录")
    sub_a.click()
    username = bro.find_element_by_id("mat-input-0")
    username.send_keys("1771617190@qq.com")
    password = bro.find_element_by_id("mat-input-1")
    password.send_keys("19976102zjx")
    submit = bro.find_element_by_class_name("mat-button-wrapper")
    submit.click()
    input()  # 手动输入验证码 程序框回车一下
    # 将 cookie 保存到本地
    print(bro.get_cookies())
    with open('cnblogs.json', 'w', encoding='utf-8') as f:
        json.dump(bro.get_cookies(), f)

except Exception as e:
    print(e)
finally:
    bro.close()
```

##### 使用`cookie`访问

```python
import time
import json
from selenium import webdriver

bro = webdriver.Chrome(executable_path='./chromedriver')
try:
    bro.get('http://www.cnblogs.com')
    bro.implicitly_wait(10)
    with open('cnblogs.json', 'r', encoding='utf-8') as f:
        cookie_dic = json.load(f)
    for item in cookie_dic:
        bro.add_cookie(item)
    bro.refresh()
    time.sleep(2)
except Exception as e:
    print(e)
finally:
    bro.close()

```

#### 抽屉半自动点赞

```python
使用selenium半自动登陆(纯自动登陆，不好登)，可以登陆上很多小号，拿到cookie保存到redis(保存到本地)
再使用requests+cookie池中的某个cookie---》刷评论，刷赞
```

##### 登录获取 cookie

```python
import time
import json
from selenium import webdriver

bro = webdriver.Chrome(executable_path='./chromedriver')

try:
    bro.get('https://dig.chouti.com/')
    login_btn = bro.find_element_by_link_text('登录')
    login_btn.click()
    # bro.execute_script('arguments[0].click();', login_btn)
    username = bro.find_element_by_name("phone")
    username.send_keys("13088229550")

    password = bro.find_element_by_name("password")
    password.send_keys("asd123...")

    submit = bro.find_element_by_css_selector(
        'body > div.login-dialog.dialog.animated2.scaleIn > div > div.login-footer > div:nth-child(4) > button')
    time.sleep(2)
    submit.click()

    input()
    with open('chouti.json', 'w', encoding='utf-8') as f:
        json.dump(bro.get_cookies(), f)

    time.sleep(2)
except Exception as e:
    print(e)
finally:
    bro.close()
```

##### 使用 cookie 访问

```python
import json
import requests
from bs4 import BeautifulSoup

header = {
    'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/101.0.4951.54 Safari/537.36'
}

res = requests.get('https://dig.chouti.com/', headers=header)

soup = BeautifulSoup(res.text, 'lxml')
div_list = soup.find_all(class_='link-item')
for div in div_list:
    article_id = div.attrs.get('data-id')
    print(article_id)
    if article_id:
        data = {
            'linkId': article_id
        }
        # 导入 cookie
        cookie = {}
        with open('chouti.json', 'r') as f:
            res = json.load(f)
        for item in res:
            # selenium的cookie和requests模块使用的cookie不太一样，requests只要name和value
            cookie[item['name']] = item['value']
        res = requests.post('https://dig.chouti.com/link/vote', headers=header, data=data, cookies=cookie)
        print(res.text)
```

### 打码平台

```python
# 验证码---》简单的数字字母验证码---》选择(选出所有草莓)---》计算类--》滑动类--》点选类---》
# 第三方平台破解验证码---》花钱买服务，把图片给人家，人家帮你解开，返回来
# 云打码，超级鹰

# 超级鹰
  开发文档：python示例代码
  应用案例：爬虫采集技术，自动智能化、人工化多种模式兼备
  价格体系：1元=1000分
```

```python
import requests
from hashlib import md5

class ChaojiyingClient():

    def __init__(self, username, password, soft_id):
        self.username = username
        password =  password.encode('utf8')
        self.password = md5(password).hexdigest()
        self.soft_id = soft_id
        self.base_params = {
            'user': self.username,
            'pass2': self.password,
            'softid': self.soft_id,
        }
        self.headers = {
            'Connection': 'Keep-Alive',
            'User-Agent': 'Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 5.1; Trident/4.0)',
        }

    def PostPic(self, im, codetype):
        """
        im: 图片字节
        codetype: 题目类型 参考 http://www.chaojiying.com/price.html
        """
        params = {
            'codetype': codetype,
        }
        params.update(self.base_params)
        files = {'userfile': ('ccc.jpg', im)}
        r = requests.post('http://upload.chaojiying.net/Upload/Processing.php', data=params, files=files, headers=self.headers)
        return r.json()

    def PostPic_base64(self, base64_str, codetype):
        """
        im: 图片字节
        codetype: 题目类型 参考 http://www.chaojiying.com/price.html
        """
        params = {
            'codetype': codetype,
            'file_base64':base64_str
        }
        params.update(self.base_params)
        r = requests.post('http://upload.chaojiying.net/Upload/Processing.php', data=params, headers=self.headers)
        return r.json()

    def ReportError(self, im_id):
        """
        im_id:报错题目的图片ID
        """
        params = {
            'id': im_id,
        }
        params.update(self.base_params)
        r = requests.post('http://upload.chaojiying.net/Upload/ReportError.php', data=params, headers=self.headers)
        return r.json()

if __name__ == '__main__':
    chaojiying = ChaojiyingClient('306334678', 'lqz12345', '903641')  # 用户中心>>软件ID 生成一个替换 96001
    im = open('./b.png', 'rb').read()  # 本地图片文件路径 来替换 a.jpg 有时WIN系统须要//
    print(chaojiying.PostPic(im, 6001))  # 1902 验证码类型  官方网站>>价格体系 3.4+版 print 后要加()
    # print chaojiying.PostPic(base64_str, 1902)  #此处为传入 base64代码
```

### `xpath`使用

```python
# css 和 xpath 和自己的
# xpath：XML路径语言（XML Path Language），它是一种用来确定XML文档中某部分位置的语言

'''
.	         # 选取当前节点。
..	       # 选取当前节点的父节点
/          # 表示当前路径
//         # 表示任意路径，子子孙孙
nodename   # a img p 节点名字
	—举例
	//div    # //div 在当前html的任意路径下找div
	/div     # 只找本层的div
*          # 任意标签
@href      # 取这个标签的属性
/text()    # 获取标签的文本
'''
```

```python
from lxml import etree

doc = '''
<html>
 <head>
  <base href='http://example.com/' />
  <title>Example website</title>
 </head>
 <body>
  <div id='images'>
   <a href='image1.html'>Name: My image 1 <br /><img src='image1_thumb.jpg' /></a>
   <a href='image2.html'>Name: My image 2 <br /><img src='image2_thumb.jpg' /></a>
   <a href='image3.html'>Name: My image 3 <br /><img src='image3_thumb.jpg' /></a>
   <a href='image4.html'>Name: My image 4 <br /><img src='image4_thumb.jpg' /></a>
   <a href='image5.html' class='li li-item' name='items'>Name: My image 5 <br /><img src='image5_thumb.jpg' /></a>
   <a href='image6.html' name='items'><span><h5>test</h5></span>Name: My image 6 <br /><img src='image6_thumb.jpg' /></a>
  </div>
 </body>
</html>
'''

html = etree.HTML(doc)

# 所有节点
a = html.xpath('//*')

# 指定节点 结果为列表
a = html.xpath('//head')

# 子节点 子孙节点
a = html.xpath('//div/a')
a = html.xpath('//body/a')  # 无数据
a = html.xpath('//body//a')

# 父节点
a = html.xpath('//body//a[@href="image1.html"]/..')
a = html.xpath('//body//a[1]/..')
a = html.xpath('//body//a[1]/parent::*')

# 属性匹配
a = html.xpath('//body//a[@href="image1.html"]')

# 文本获取
a = html.xpath('//body//a[@href="image1.html"]/text()')

# 属性获取
a = html.xpath('//body//a/@href')
# 注意从1 开始取（不是从0）
a = html.xpath('//body//a[1]/@href')

# 属性多值匹配
# a 标签有多个class类，直接匹配就不可以了，需要用contains
a = html.xpath('//body//a[@class="li"]')
a = html.xpath('//body//a[contains(@class,"li")]')
a = html.xpath('//body//a[contains(@class,"li")]/text()')

# 多属性匹配
a = html.xpath('//body//a[contains(@class,"li") or @name="items"]')
a = html.xpath('//body//a[contains(@class,"li") and @name="items"]/text()')
a = html.xpath('//body//a[contains(@class,"li")]/text()')

# 按序选择
a = html.xpath('//a[2]/text()')
a = html.xpath('//a[2]/@href')
a = html.xpath('//a[last()]/@href')  # 取最后一个
a = html.xpath('//a[position()<3]/@href')  # 位置小于3的
a = html.xpath('//a[last()-2]/@href')  # 倒数第二个

# 节点轴选择
'''
ancestor           祖先节点
使用了 *            获取所有祖先节点
attribute          属性值
child              直接子节点
descendant         所有子孙节点
following          当前节点之后所有节点
following-sibling  当前节点之后同级节点
'''
a = html.xpath('//a/ancestor::*')
a = html.xpath('//a/ancestor::div')  # 获取祖先节点中的div
a = html.xpath('//a[1]/attribute::*')
a = html.xpath('//a[1]/child::*')
a = html.xpath('//a[6]/descendant::*')
a = html.xpath('//a[1]/following::*')
a = html.xpath('//a[1]/following::*[1]/@href')
a = html.xpath('//a[1]/following-sibling::*')
a = html.xpath('//a[1]/following-sibling::a')
a = html.xpath('//a[1]/following-sibling::*[2]')
a = html.xpath('//a[1]/following-sibling::*[2]/@href')

print(a)

# 终极大招 ---> 复制
# //*[@id="maincontent"]/div[5]/table/tbody/tr[2]/td[2]
```

### 爬取京东商品信息

```python
from pymongo import MongoClient
from urllib.parse import quote_plus


class Mongo:
    def __init__(self):
        user = ''
        pwd = ''
        host = ''
        port = 27017
        uri = "mongodb://%s:%s@%s" % (quote_plus(user),
                                      quote_plus(pwd),
                                      host)

        self.client = MongoClient(uri, port)
        self.col = self.client.pydata.jd_goods  # pydata 库下 jd_goods 集合

    def save_many(self, data):
        self.col.insert_many([data])  # 插入多条
```

```python
from selenium import webdriver
from selenium.webdriver.common.keys import Keys  # 键盘按键操作
from pymongo_client import Mongo  # 上面的 py 模块
import time



def get_goods(driver):
    try:
        # 找到所有类名叫gl-item的标签
        goods = driver.find_elements_by_class_name('gl-item')
        for good in goods:
            detail_url = good.find_element_by_tag_name('a').get_attribute('href')
            p_name = good.find_element_by_css_selector('.p-name em').text.replace('\n', '')
            price = good.find_element_by_css_selector('.p-price i').text
            p_commit = good.find_element_by_css_selector('.p-commit a').text
            img = good.find_element_by_css_selector('div.p-img img').get_attribute('src')
            if not img:
                img = 'http:' + good.find_element_by_css_selector('div.p-img img').get_attribute('data-lazy-img')
            # msg = '''
            #     商品 : %s
            #     链接 : %s
            #     图片 : %s
            #     价钱 ：%s
            #     评论 ：%s
            #     ''' % (p_name, detail_url, img, price, p_commit)
            #
            # print(msg, end='\n\n')

            msg_dict = {
                "商品名称": p_name,
                "链接地址": detail_url,
                "商品图片": img,
                "商品价钱": price,
                "商品评论": p_commit,
            }
            # print(msg_dict)
            mongo = Mongo()
            mongo.save_many(msg_dict)

        button = driver.find_element_by_partial_link_text('下一页')
        button.click()
        time.sleep(1)
        get_goods(driver)
    except Exception:
        pass


def spider(url, keyword):
    driver = webdriver.Chrome(executable_path='./chromedriver')
    driver.get(url)
    driver.implicitly_wait(3)  # 使用隐式等待
    try:
        input_tag = driver.find_element_by_id('key')
        input_tag.send_keys(keyword)
        input_tag.send_keys(Keys.ENTER)  # 敲回车
        get_goods(driver)
    finally:
        driver.close()


if __name__ == '__main__':
    spider('https://www.jd.com/', keyword='精品内衣')
```
