---
title: BeautifulSoup4的使用
lastmod: 2021-06-03T16:43:23+08:00
date: 2021-06-02T11:52:03+08:00
tags:
  - crawler
categories:
  - crawler
url: post/crawler-02.html
toc: true
---

### 爬取梨视频

```python
requests 模拟发送 http 请求
解析库进行解析，有 re bs4 lxml
requests-html: 发送请求+解析xml

视频去水印: fmmpeg模块 加水印，拼接裁剪，抠图，转码
```

<!-- more -->

```python
import re
import requests

# res = requests.get("https://www.pearvideo.com/")
# print(res.text)

# 页面中发送有 Ajax 请求得到存放视频地址的页面
# https://www.pearvideo.com/category_loading.jsp?reqType=5&categoryId=59&start=240&mrd=0.4791462124097654&filterIds=1761446,1761399,1761415,1745287,1745362,1745246,1745234,1745171,1745100,1745122,1745112,1744971,1744958,1744950,1744934
# https://www.pearvideo.com/category_loading.jsp?reqType=5&categoryId=59&start=0

# res = requests.get("https://www.pearvideo.com/category_loading.jsp?reqType=5&categoryId=59&start=0")
# print(res.text)

# 取出视频 video
# <a href="video_1761446" class="vervideo-lilink actplay">
# https://www.pearvideo.com/video_1761399


res = requests.get("https://www.pearvideo.com/category_loading.jsp?reqType=5&categoryId=59&start=0")
video_list = re.findall('<a href="(.*?)" class="vervideo-lilink actplay">', res.text)
# print(video_list)
# 得到每个视频页面的地址
for video in video_list:
    video_url = 'https://www.pearvideo.com/' + video
    # print(video_url)
    # # 该页面又向后端发送了 Ajax请求 https://www.pearvideo.com/videoStatus.jsp?contId=1760940&mrd=0.07016044407010624
    # res_video = requests.get(video_url)
    # print(res_video.text)  # 拿到的没有页面中 mp4 的url信息 该文章已下线

    # 第一层反爬需要加 Referer
    video_id = video_url.split('_')[-1]
    header = {
        "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/101.0.4951.54 Safari/537.36",
        "Referer": video_url
    }
    res_video = requests.get(f"https://www.pearvideo.com/videoStatus.jsp?contId={video_id}", headers=header).json()
    mp4_url = res_video.get("videoInfo").get("videos").get("srcUrl")

    # 第二层反爬 需要将 url 变形
    # https://video.pearvideo.com/mp4/short/20220509/  1652081176661  -15876181-hd.mp4
    # https://video.pearvideo.com/mp4/short/20220509/  cont-1760940   -15876181-hd.mp4
    mp4_url = mp4_url.replace(mp4_url.split("/")[-1].split("-")[0], f'cont-{video_id}')
    # print(mp4_url)
    print(mp4_url)
    # 保存本地
    video_detail = requests.get(mp4_url)
    with open(f'./video/{video_id}.mp4', "wb") as f:
        for line in video_detail.iter_content(512):
            f.write(line)
```

### `BS4`爬汽车之家新闻

```python
# pip3 install beautifulsoup4
# pip3 install lxml
# https://www.autohome.com.cn/news/

# https://www.autohome.com.cn/news/1/#liststart
```

```python
import json
import requests
from bs4 import BeautifulSoup

news_list = []
info = {"car_new": {"news": news_list}}

res = requests.get("https://www.autohome.com.cn/news/1/#liststart")
# print(res.text)
# html.parser 是 bs4 的默认解析库
soup = BeautifulSoup(res.text, 'html.parser')
ul_list = soup.find_all(name='ul', class_='article')
# print(len(ul_list))
for ul in ul_list:
    li_list = ul.find_all(name='li')
    # print(li_list)
    for li in li_list:
        h3 = li.find(name='h3')
        if h3:
            title = h3.text
            desc = li.find(name='p').text
            img = li.find(name='img')['src']
            if not img.startswith("http"):
                img = "https://" + img
            url = "https:" + li.find(name='a')['href']
            news_dict = {
                "新闻标题": title,
                "新闻摘要": desc,
                "新闻图片": img,
                "新闻地址": url,
            }
            news_list.append(news_dict)
            res_img=requests.get(img)
            img_name=img.split('/')[-1]
            with open('./img/%s'%img_name,'wb') as f:
                for line in res_img.iter_content(1024):
                    f.write(line)
            # 可以将信息保存至数据库中

with open("news.json", "a") as f:
    json.dump(info, f, ensure_ascii=False)
```

### `BS4`遍历文档树

```python
from bs4 import BeautifulSoup

html_doc = """
<html><head><title>The Dormouse's story</title></head>
<body>

<p class="title" id="id_p"> test <b>The Dormouse's story</b></p>

<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1">Elsie</a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>

<p class="story">...</p>
"""

# html.parser 内置的，速度一般,容错能力强
# soup = BeautifulSoup(html_doc, 'html.parser')

# lxml    第三方，速度快,容错能力强
# pip3 install lxml
soup = BeautifulSoup(html_doc, 'lxml')

# print(soup.prettify())  # 会对 html 进行美化 将对标签补齐等

# 遍历文档树 . 遍历 速度快
# print(soup.title)
# print(soup.body)
# print(soup.body.p)
# print(soup.body.p.b)
# print(soup.body.a)

# 获取标签名称
# print(soup.title.name)
# print(soup.body.name)
# print(soup.p.name)

# 获取标签属性
# print(soup.body.p)
# print(soup.body.p['class'])  # ['title'] 因为 class 可能有多个 所以是列表
# print(soup.body.p['id'])  # id_p  全局唯一
# print(soup.body.p.attrs)  # {'class': ['title'], 'id': 'id_p'} 所有属性放到字典中

# 获取标签的文本内容
# print(soup.p.text)  # 将标签中的文本内容和所有子标签的内容拼接在一起
# print(soup.p.string)  # 当前标签只有文本或只有一个子有文本才拿出来，如果有其他标签，返回None
# print(list(soup.p.strings))  # 将文本内容和子标签中的文本内容放到 generator

# 嵌套选择
# print(soup.html.head)
# print(soup.head.title.string)  # 可以一直 . 下去

# 子节点 子孙节点
# print(soup.p.contents)  # p下所有子节点，放到列表中
# print(list(soup.p.children))  # 得到一个迭代器,包含p下所有子节点,跟contents本质一样，只是节约内存
# print(list(soup.p.descendants))  # 获取子孙节点,p下所有的标签都会选择出来  子子孙孙
# for i, child in enumerate(soup.p.children):
#     print(i, child)
# for i, child in enumerate(soup.p.descendants):
#     print(i, child)

# 父节点 祖先节点
# print(soup.a.parent)  # p 标签
# print(list(soup.a.parents))  # p body html 等

# 兄弟节点
# print(soup.a.next_sibling)  # 下一个兄弟
# print(soup.a.previous_sibling)  # 上一个兄弟
# print(list(soup.a.next_siblings))  # 下面的兄弟们=>生成器对象
# print(list(soup.a.previous_siblings))  # 上面的兄弟们=>生成器对象

'''
重点记住
.                     遍历
[] attrrs.get()       取属性
text string strings   取文本
'''
```

### `BS4`搜索文档树

```python
from bs4 import BeautifulSoup

html_doc = """
<html><head><title>The Dormouse's story</title></head>
<body>

<p class="title" id="id_p"> test <b>The Dormouse's story</b></p>

<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1">Elsie</a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>

<p class="story">...</p>
"""

soup = BeautifulSoup(html_doc, 'lxml')

# 五种过滤器 字符串 正则表达式 列表 True 方法
# find      找第一个
# find_all  找所有
# res = soup.find_all(name='p')
# res = soup.find(id="id_p")
# res = soup.find_all(class_="story")
# res = soup.find_all(name='p', class_='story')  # and 条件
# res = soup.find(name='a', id='link2').text
# res = soup.find(name='a', id='link2').attrs.get("href")  # attrs 将 属性: 值的字典
# res = soup.find(attrs={'id': 'link2', 'class': 'sister'}).attrs.get('href')  # 上面的第二种写法
# print(res)


# 正则表达式 value 是正则表达式
# import re
# res = soup.find_all(name=re.compile('^b'))  # 以 b 开头的 标签
# res = soup.find_all(href=re.compile('^http'))  # href 以 http 开头的
# res = soup.find_all(class_=re.compile('^s'))  # class 以 s 开头的
# print(res)


# 列表 value 值是列表
# res = soup.find_all(name=['body', 'a'])
# res = soup.find_all(class_=['sister', 'story'])
# res = soup.find_all(id=['link2', 'link3'])
# print(res)

# True value 值是True
# res = soup.find_all(name=True)
# res = soup.find_all(id=True)
# res = soup.find_all(href=True)
# print(res)

# 方法
# def has_class_but_no_id(tag):
#     return tag.has_attr('class') and not tag.has_attr('id')
# print(soup.find_all(name=has_class_but_no_id))  # 有class但是没有id的标签


# html页面中，只要有的东西，通过bs4都可以解析出来
# 遍历文档树+搜索文档树混用
# def has_class_but_no_id(tag):
#     return tag.has_attr('class') and not tag.has_attr('id')
# print(soup.find(name=has_class_but_no_id).a.text)


# find_all的其他参数limit:限制取几条  recursive：是否递归查找
# def has_class_but_no_id(tag):
#     return tag.has_attr('class') and not tag.has_attr('id')
# res = soup.find_all(name=has_class_but_no_id, limit=1)
# print(res)
# res = soup.find_all(name='a', recursive=False)  # 不递归查找,速度快，只找一层
# print(res)
```

### `CSS`选择器

```python
# css xpath选择器是通用的，基本所有的解析库(bs4,lxml,pyquery,selenium的解析库)，都支持css选择器，css在前端通用

from bs4 import BeautifulSoup

html_doc = """
<html><head><title>The Dormouse's story</title></head>
<body>

<p class="title" id="id_p"> test <b>The Dormouse's story</b></p>

<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1">Elsie</a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>

<p class="story">...</p>
"""
soup = BeautifulSoup(html_doc, 'lxml')
'''
div      找div标签
div>a    找div下的紧邻的a
div a    找div下的子子孙孙的a
.sister  找类名为sister的标签
#id_p    找id为id_p的标签
'''

# soup.select()  # 找所有
# soup.select_one()  # 找一个

# res = soup.select('p')  # 找所有的 p 标签
# res = soup.select('#id_p')  # 找 id 是 id_p 的标签
# res = soup.select('.sister')  # 找类是 sister 的标签
# res = soup.select_one('.story>a').attrs.get('href')  # .story 类下的 a 标签 生成字典形式 获取 href
# print(res)

# 终极大招 CV 浏览器中 找到需要查找的地方 选择Copy ---> Copy selector
import requests
response=requests.get('https://www.runoob.com/cssref/css-selectors.html')
soup=BeautifulSoup(response.text,'lxml')
res=soup.select_one('#content > table > tbody > tr:nth-child(2) > td:nth-child(3)').text
print(res)


# 只要页面中有的通过bs4都能解析出来
```
