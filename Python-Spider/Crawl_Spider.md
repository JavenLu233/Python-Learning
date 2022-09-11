# Crawl_Spider

# 爬虫

学习视频:https://www.bilibili.com/video/BV1TT4y1e7nB?p=1

<br>

## 初步了解

#### 分类

**1.通用爬虫**

爬取整个页面

**2.聚焦爬虫**

爬取指定部分

**3.增量式爬虫**

爬取更新的数据

#### 反爬与反反爬

**robots.txt 协议**

君子协议，规定哪些数据可以被爬取

<br>

### http协议

超文本传输协议

#### 请求头信息（Request Headers）

**User-Agent：**

请求载体的身份标识

浏览器就是一种发出请求的载体

**Connection：**

请求完毕后断开连接/保持连接

#### 响应头信息

**Content-Type：**

服务器响应回客户端的数据类型



<br>

### https

安全的http协议（security），传输过程中进行数据加密

#### 加密方式

1.**对称密钥加密**

客户端自己指定方式加密数据发送给服务器端

服务器使用客户端提供的密钥解密，得到原文数据

（有安全隐患）

2.**非对称密钥加密**

服务器先设定一种加密方式，将加密方式发送给客户端

客户端用服务器的加密方式加密，然后将密文发送给服务器

服务器制定好的

加密方式叫做 公钥

解密方式叫做 私钥

（避免密文和密钥同时传输，但效率低，且如果公钥被拦截和篡改，

客户端也有安全隐患）

3.**证书密钥加密**（https常用）

证书认证机构，第三方中间机构

服务器端先把公钥提交到证书认证机构，机构进行审核，

通过后作数字签名，然后封装到证书中，一并发送给客户端





<br>

<br>

## Request

Python中原生的基于网络请求的模块

功能强大且使用简单

作用：模拟浏览器发送请求

编码流程：

 -指定url

 -发起请求

 -获取响应数据

 -持久化存储

### 01.获取一个页面的文本

```Python
import requests

# Step_1:指定一个url
url = 'https://cn.bing.com/'

# Step_2:发起请求
response = requests.get(url=url)

# Step_3:获取响应数据
page_text = response.text
# print(page_text)

# Step_4:持久化存储
with open('./bing.html', 'w', encoding='utf-8') as fp:
    fp.write(page_text)
print(url, "爬取成功")

fp.close
```

### 02.获取一个关键词搜索的页面文本

```python
import requests

# UA伪装
headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/101.0.4951.54 Safari/537.36 Edg/101.0.1210.39'
}

# 处理url参数
url = 'https://cn.bing.com/search?'

kw = input("enter the word for searching:\n")
param = {
    'q': kw
}

# 发起请求
response = requests.get(url=url, params=param, headers=headers)
page_text = response.text

# 持续化存储
fileName = kw + '.html'

with open(fileName, 'w', encoding='utf-8') as fp:
    fp.write(page_text)

print(fileName, "爬取成功")

fp.close
```

### 03.获取百度翻译更新后的一个词条文本

```python
import requests
import json

url_1 = 'https://fanyi.baidu.com/sug'  # 必须是 /sug 才行

# 进行UA伪装
headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/101.0.4951.54 Safari/537.36 Edg/101.0.1210.39'
}

# post参数处理
kw = input("请输入要翻译的单词：\n")
data = {
    'kw': kw
}  # 通过火狐浏览器“编辑并重发” post请求 可知道这个变量名为 kw

# 发送请求
response = requests.post(url=url_1, data=data, headers=headers)

# 获取响应数据
# json返回一个对象,需要确认服务器响应数据的类型式json才能使用
# 从响应请求参数里的 Content-Type 可以得知响应数据的类型
dic_obj = response.json()
print(dic_obj)

# 持续化存储
fileName = kw + '.json'
filePath = './word/'
fp = open(filePath + fileName, 'w', encoding='utf-8')
json.dump(dic_obj, fp=fp, ensure_ascii=False)
# 需要import json; 由于含有中文,需要关闭ascii码编码

fp.close

```

#### post请求

利用抓包工具的 网络-XHR 得到ajax数据

主要获取url和携带的参数

### 04.获取豆瓣电影排行

```Python
import requests
import json
import csv

# 输入url
url = 'https://movie.douban.com/j/chart/top_list?'

# get请求携带参数
param = {
    "type": 10,  # 获取类型
    "interval_id": "100:90",  # 一开始冒号之后有空格，一直获取到空json
    "action": "",
    "start": 0,  # 获取的起始位置
    "limit": 20,  # 获取电影个数
}

# UA伪装参数
headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/101.0.4951.54 Safari/537.36 Edg/101.0.1210.39'
}

# 获取信息
response = requests.get(url=url, params=param, headers=headers)

# 将json格式数据转化为对象
data_list = response.json()
print(data_list)  # 输出一下数据，检查

# 打开文件路径
fp = open('./douban_list.json', 'w', encoding='utf-8')

# 将json持久化存储
json.dump(data_list, fp=fp, ensure_ascii=False)

# 提示爬取结束
print('爬取结束！')

# 关闭文件
fp.close()


# 把电影名，上映时间和评分提取到csv表格中

# f = open("豆瓣电影喜剧排行榜.csv", mode="w")
# writer = csv.writer(f)

# for movie in data_list:
#     movie_name = movie['title']
#     movie_time = movie['release_date']
#     movie_score = movie['score']
#     writer.writerow([movie_name, movie_time, movie_score])

# f.close()


```

### 05.获取肯德基餐厅数据

```Python
import requests
import json

# 获取url
# _url = 'http://www.kfc.com.cn/kfccda/ashx/GetStoreList.ashx?' 没有op=获取不到数据
_url = 'http://www.kfc.com.cn/kfccda/ashx/GetStoreList.ashx?op=keyword'

# UA伪装参数
_headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/101.0.4951.54 Safari/537.36 Edg/101.0.1210.39'
}

# 提示键入关键词
kw = input("请输入查询的关键词:\n")
pg = 1
end_page = -1
data_text = ''
# post请求参数
_params = {
    'cname': '',
    'pid': '',
    'keyword': kw,
    'pageIndex': pg,
    'pageSize': 10,
}

while(True):
    # 获取数据
    response = requests.post(url=_url, params=_params, headers=_headers)

    # 将数据暂时存储
    data_temp = response.text
    # 空页为47个字符,如果字符数小于50,直接跳出循环
    if(len(data_temp) < 50):
        end_page = pg
        break

    # print(_params['pageIndex'])

    # 添加每页前置提示,方便后期查阅
    data_text = data_text + '**Page:' + str(pg) + '**'
    # 将下一页的信息追加在data_text后
    data_text = data_text + data_temp + '\n'

    # 页数加一
    pg = pg + 1
    _params['pageIndex'] = pg


# with open('.\canteens.txt', 'w', encoding='utf-8') as fp:
#     fp.write(data_text）

# 打开文件路径，并持久化存储
fileName_json = './肯德基餐厅信息/' + kw + '.json'
fileName_txt = './肯德基餐厅信息/' + kw + '.txt'

_fp = open(fileName_json, 'w', encoding='utf-8')
_fp2 = open(fileName_txt, 'w', encoding='utf-8')
_fp2.write(data_text)
json.dump(data_text, fp=_fp, ensure_ascii=False)

# 提示爬取结束
print("爬取结束！")
print("空页在第", end_page, "页")


# 关闭文件
_fp.close
_fp2.close

```

### 06.获取药监总局化妆品许可证信息

```Python
import requests
import json

url = ''
headers = {
    
}

all_data_list = []#存储详情信息
id_list = []#用于存储企业的id

for page in range(1,6):
    data_1 = {
        'page':page,
        'pageSize':'15'
    }




    json_ids = requests.post(url= ,headers = ,data = ).json()

    for dic in json_ids['list']:
        id_list.append(dic['ID'])
    

post_url = ''
for id in id_list:
    data_2 = {
		'id':id
    }
    detail_json = request.post(url = ,headers = ,data = )
    print(detail_json)
    all_data_list.append(detail_json)
    
fp = open('./allData.json','w',encoding='utf-8')    
json.dump(all_data_list,fp = fp,ensure_ascii = False)

print('over!')

fp.close
    
```

网站无法登录..

#### 动态加载数据

对于动态获取到的信息,一般不是从网站的url获取,

应该使用抓包工具找到ajax请求的url

有时候详情页的超链接没有给出,但是会有id,可以进入详情页对url域名进行观察

<br>

<br>

## 聚焦爬虫

- 编码流程
- - 指定url
  - 发起请求
  - 获取响应数据
  - 数据解析
  - 持久化存储

### 数据解析

#### 分类:

-正则

-bs4

-xpath(重点)

#### 原理:

- 解析的局部文本内容都会在标签或者标签对应属性值进行存储
- 1.进行指定标签的定位
- 2.对应属性进行提取

<br>

<br>

## 正则数据解析

### 01.获取一张图片

```Python
import requests

url = 'https://cdn-hz.skypixel.com/uploads/cn_files/photo/image/e21681c4-3907-42f3-9523-2529953953ca.jpg@!1920'

# .content返回二进制数据
img_data = requests.get(url=url).content

with open('./图片爬取/.jpg', 'wb') as fp:
    fp.write(img_data)

```

### 02.获取多张图片

此处是自己找的网站,与视频中不同

```Python
import requests
import re
import os

folderName = input("请输入文件夹名称:\n")
filePath = './skypixel图片爬取/' + folderName + '/'
# 如果没有指定文件夹就创建文件夹
if not os.path.exists(filePath):
    os.mkdir(filePath)

# 用抓包工具获取url
_url = 'https://www.skypixel.com/api/v2/tags/2468ca60-5292-46e5-8914-bfadcfcdc1fc/works?'

# 进行UA伪装
_headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/101.0.4951.54 Safari/537.36 Edg/101.0.1210.39'
}

# offset_now = 0
# while(True):  可以每次获取15张然后循环获取,也可以直接把limit改成你想获取图片张数
_limit = eval(input("请输入想要获取图片的张数:\n"))
_offset = eval(input("请输入获取图片的起始序号:\n"))
# get请求的参数
_params = {
    'lang': 'zh-Hans',
    'platform': 'web',
    'device': 'desktop',
    'sort': 'popular',
    'cycle': '30d',
    'limit': _limit,  # 获取图片数量
    'offset': _offset,  # 起始图片序号
}

page_text = requests.get(url=_url, headers=_headers,
                         params=_params).text  # 原来是get不是post = =
# print(page_text)


# 聚焦爬虫

# ex_url = '"image":{.*?"large":"(.*?)"},"equipment"'  #不可,因为有些image后面跟了是location

# 使用正则表达式提取图片的url和名称
ex_url = '"image":{.*?"large":"(.*?)"}'
ex_name = '"title":"(.*?)"'

data_img_url_list = re.findall(ex_url, page_text, re.S)
data_img_name_list = re.findall(ex_name, page_text, re.S)

# print(data_img_url_list)
# print(data_img_name_list)

# 循环次数
img_num = _params['limit']
for i in range(0, img_num):
    img_url = data_img_url_list[i]
    # 把名称里的一些文件名不能出现的特殊符号,例如|,替换成空格
    img_name = re.sub(r'[^\w\s]', '', data_img_name_list[i])

    #out = re.sub(r'[^\w\s]', '', s)

    img_data = requests.get(url=img_url, headers=_headers).content

    fileName = img_name + '.jpg'

    with open(filePath + fileName, 'wb') as fp:
        fp.write(img_data)
        print("第", i, "张:", fileName, "下载成功!")


print("本次爬取结束!")

```

```Python
#视频中的一个分页操作

url = ".../page/%d/..."
for pageNum in (1,10):
    new_url = format(url%pageNum)
```

re.s 是将字符串作为整体查找，如果没有加，只会逐行逐行查找

<br>

<br>

## bs4

仅Python可用

#### 原理：

- 1.实例化一个BeautifulSoup对象，并将页面源码数据加载到该对象中
- 2.通过调用BeautifulSoup对象中相关的属性或者方法进行标签定位和数据提取

#### 环境安装：

pip install bs4

pip install lxml

不用进入python，直接pip install ...

解压包安装:

cmd打开setup.py所在路径

然后 python setup.py install

#### 可用的国内镜像

https://blog.csdn.net/wukai0909/article/details/62427437

```
国内源：
新版ubuntu要求使用https源，要注意。

清华：https://pypi.tuna.tsinghua.edu.cn/simple

阿里云：http://mirrors.aliyun.com/pypi/simple/

中国科技大学 https://pypi.mirrors.ustc.edu.cn/simple/

华中理工大学：http://pypi.hustunique.com/

山东理工大学：http://pypi.sdutlinux.org/ 

豆瓣：http://pypi.douban.com/simple/

临时使用：
可以在使用pip的时候加参数-i https://pypi.tuna.tsinghua.edu.cn/simple

例如：pip install bs4 -i https://pypi.tuna.tsinghua.edu.cn/simple pyspider，这样就会从清华这边的镜像去安装pyspider库。
```

#### 查看已经安装的模块

```
pip list
```

```
Python
help('modules')
```

#### 如何使用

- from bs4 import BeautifulSoup
- 对象实例化:
- - 1.将本地的html文档中的数据加载到该对象中
  - - fp = open('./test.html','r',encoding='utf-8')
    - soup = BeautifulSoup(fp,'lxml')
  - 2.将互联网上获取的页面源码加载到该对象中
  - - page_text = response.text
    - soup = BeautifulSoup(page_text,'lxml')
- 提供实例化的方法和属性

**soup.tagName**

返回html中第一次出现的tagName标签

**soup.find('tagName')**

1.同soup.tagName

2.属性定位:

​	如soup.find('div',class_='song')   class后下划线表示这个class是一个参数而不是一个关键字

**soup.find_all('tagName')**

返回一个列表,包含所有符合的标签

**soup.select()**

1.使用一个id/类/标签选择器,返回一个列表

​	如soup.select('.tang')

2.层级选择器

​	如soup.select('.tang > ul > li > a')[0]

​	>表示下一个层级, 只用空格表示多个层级

**获取标签之间的文本数据**

- soup.a.text/string/get_text()
- text/get_text():可以获取某一个标签中所有的文本,即使不是直系的(例如子/孙标签下的内容)
- string:可以获取该标签下直系的内容

**获取标签中属性值**

- soup.a['href']

### 03.获取小说

以下展示部分html文件

```Python
<ul class="paiban">
      <li class="menu-item"><a href="https://sanguo.5000yan.com/965.html">第一回 宴桃园豪杰三结义 斩黄巾英雄首立功</a></li>
<li class="menu-item"><a href="https://sanguo.5000yan.com/966.html">第二回 张翼德怒鞭督邮 何国舅谋诛宦竖</a></li>
<li class="menu-item"><a href="https://sanguo.5000yan.com/967.html">第三回 议温明董卓叱丁原 馈金珠李肃说吕布</a></li>
<li class="menu-item"><a href="https://sanguo.5000yan.com/968.html">第四回 废汉帝陈留践位 谋董贼孟德献刀</a></li>
<li class="menu-item"><a href="https://sanguo.5000yan.com/969.html">第五回 发矫诏诸镇应曹公 破关兵三英战吕布</a></li>
<li class="menu-item"><a href="https://sanguo.5000yan.com/970.html">第六回 焚金阙董卓行凶 匿玉玺孙坚背约</a></li>
<li class="menu-item"><a href="https://sanguo.5000yan.com/971.html">第七回 袁绍磐河战公孙 孙坚跨江击刘表</a></li>
<li class="menu-item"><a href="https://sanguo.5000yan.com/972.html">第八回 王司徒巧使连环计 董太师大闹凤仪亭</a></li>
<li class="menu-item"><a href="https://sanguo.5000yan.com/973.html">第九回 除暴凶吕布助司徒 犯长安李傕听贾诩</a></li>
<li class="menu-item"><a href="https://sanguo.5000yan.com/974.html">第十回 勤王室马腾举义 报父仇曹操兴师</a></li>
<li class="menu-item"><a href="https://sanguo.5000yan.com/975.html">第十一回 刘皇叔北海救孔融 吕温侯濮阳破曹操</a></li>
<li class="menu-item"><a href="https://sanguo.5000yan.com/976.html">第十二回 陶恭祖三让徐州 曹孟德大战吕布</a></li>
```

```Python

import requests
from bs4 import BeautifulSoup
import os

if not os.path.exists('./三国演义'):
    os.mkdir('./三国演义')

url = 'https://sanguo.5000yan.com/'

headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/101.0.4951.54 Safari/537.36 Edg/101.0.1210.39'
}


response = requests.get(url=url, headers=headers)
# 由于这个网页的html中没有指定编码，request会自动使用ISO-8859-1编码,不更改中文会乱码
# 第一种方法，把response的编码改为utf-8
# print(response.encoding)
response.encoding = 'utf-8'
page_text = response.text

# page_text = page_text.encode('raw_unicode_escape').decode()  # 第二种方法把乱码变回中文

# print(page_text)

soup = BeautifulSoup(page_text, 'lxml')

li_list = soup.select('.paiban > li')

# print(chapter_list)

fp = open('.\三国演义\三国演义.txt', 'w', encoding='utf-8')

for li in li_list:
    title = li.a.string
    detail_page_url = li.a['href']
    #print(title, detail_page)
    detail_response = requests.get(url=detail_page_url, headers=headers)
    detail_response.encoding = 'utf-8'
    detail_page = detail_response.text

    soup_detail = BeautifulSoup(detail_page, 'lxml')
    div_tag = soup_detail.find('div', class_="grap")
    content = div_tag.text
    # print(content)

    fp.write(title + ':\n' + content + '\n')
    print(title, '已写入')


fp.close

```

<br>

<br>

## xpath

最常用，最便捷，最通用

#### 解析原理

1.实例化一个etree的对象，且需要将被解析的页面源码数据加载到该对象中。

2.调用etree对象中的xpath方法结合着xpath表达式实现标签的定位和内容的捕获。

#### 环境安装

pip install lxml

#### 实例化etree对象

```Python
from lxml import etree
```

1.加载本地html文档

```Python
tree = etree.prase(filePath)
```

2.加载互联网上获取的源码数据

```Python
tree = etree.HTML(page_text)
#page_text 不用加引号啊啊啊啊啊啊
```

#### xpath表达式

```Python
xpath('xpath表达式')
#返回对象列表,对象是定位到标签所在行文本的抽象

xpath('xpath01 | xpath02')
#获取满足01或02表达式的标签

r1 = tree.xpath('/html/body/div') #/表示根目录
r2 = tree.xpath('/html//div') #//表示跨越多个层级
r3 = tree.xpath('//div')	#源码中所有的div

# #属性定位 tag[@attrName="attrvalue"]
r4 = tree.xpath('//div[@class="song"]') 

# #索引定位 
r5 = tree.xpath('//div[@class="song"]/p[3]')
#获取到第三个p标签,计数从1开始


# #获取文本
r6 = tree.xpath('//div[]@class="tang"//li[5]/a/text()')[0]
# /text() 获得直系的文本的列表  [0]得到列表第一个元素

# //text() 获取该标签下所有文本内容的列表

# #获取属性值
r7 = tree.xpath('//div[@class="song"]/img/@src')
# /@attrName   

#当前etree下记得加 （'./'）


```

#### 解决中文乱码

1.直接对html的编码格式进行修改

```Python
response = requests.get()
response.encoding = 'utf-8'
```

2.对获取到的文本单独重新编码

```Python
img_name = img_name.encode('iso-8859-1').decode('gbk')
```

### 04.爬取冰海战记漫画

```python
import os
import re
import requests
from lxml import etree

# 获取目录html文本
if not os.path.exists('./冰海战记漫画'):
    os.mkdir('./冰海战记漫画')

url = 'https://www.mhkan.com/manhua/binghaizhanji/'

headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/101.0.4951.64 Safari/537.36 Edg/101.0.1210.47',
}

# 解除报错的一些操作，见网上
requests.adapters.DEFAULT_RETRISE = 5
s = requests.session()
s.keep_alive = False
requests.packages.urllib3.disable_warnings()

#response = requests.get(url=url, headers=headers, )
# 报错:SSLError  HTTPSConnectionPool(host='www.mhkan.com', port=443): Max retries exceeded with url: /manhua/binghaizhanji (Caused by SSLError(SSLCertVerificationError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: unable to get local issuer certificate (_ssl.c:997)')))

response = requests.get(url=url, headers=headers, verify=False)
page_text = response.text
# print(page_text)

# 生成etree对象
tree = etree.HTML(page_text)
# print(tree)

# 获取a标签的列表
a_list = tree.xpath(
    '/html/body/div[4]/div[1]/div/div[3]/div[1]/div[3]/div[3]/div[2]/ul/li/a')

# print(a_list)

# 对每一话漫画进行所有图片的下载
for a in a_list:
    comic_name = a.xpath('./span/text()')
    comic_name = comic_name[0]
    if not os.path.exists('./冰海战记漫画/' + comic_name):
        os.mkdir('./冰海战记漫画/' + comic_name)

    # print(comic_name)
    comic_url = 'https://www.mhkan.com/' + a.xpath('./@href')[0]
    # print(comic_url)

    comic_response = requests.get(url=comic_url, headers=headers, verify=False)
    comic_text = comic_response.text

    # with open('./comic.txt', 'w', encoding='utf-8') as fp:
    #     fp.write(comic_text)

    c_tree = etree.HTML(comic_text)
    all_img_url = c_tree.xpath('//body/script[1]/text()')

    # 获取图片地址
    ex_img = '\\\/images(.*?).jpg'
    img_list = re.findall(ex_img, str(all_img_url[0]), re.S)

    img_num = 0
    for img_url in img_list:
        img_num += 1
        img_url = 'https:\/\/images' + str(img_url) + '.jpg'
        img_url = img_url.replace('\\', '')
        # print(img_url)
        img_data = requests.get(url=img_url, headers=headers).content
        # print("访问成功")
        with open('./冰海战记漫画/' + comic_name + '/' + str(img_num) + '.jpg', 'wb') as fp:
            fp.write(img_data)
            print(comic_name, img_num, '下载成功')

```

```Python
出现的问题：

1.SSLError
加上以下代码：
response = requests.get(url=url, headers=headers, verify=False)
page_text = response.text

修改requests.get(,verify=False)

2.一直爬取不到详细标签列表
etree.HTML(page_text)  不用加引号！！

3.获取到的图片url列表要转化为字符串类型

4.把url中的反斜杠删去

5.with open 没写 ‘wb’和encoding='utf-8'

6.文件夹后要加/，不然就连成一个文件名了

7.爬取速度太慢，后面要改进为多线程

8.如果展示的文本太长，vscode的控制台会省略一些标签，实际上是完整的

```

<br>

<br>

## 验证码

一种反爬机制

识别验证码，模拟登录：

1.人工肉眼识别

2.第三方自动识别

### 云打码（收费）

- 注册普通用户和开发者用户
- 使用模板

```Python
response = requests.get()
response.status_code  #如果是200则表示f成功
```

### Cookie

http/https 协议特性：无状态

cookie用来让服务器端记录客户端的相关状态

1.手动处理

从浏览器复制cookie，附加在headers

（效率低，且有些网站cookie动态变化）

2.自动处理

创建session对象,以自动保存请求得到的cookie

```python
#创建session对象
session = requests.Session()

#使用session进行post请求的发送
response = session.post(url = url,headers = headers)

#使用携带cookie的session进行get请求的发送
detail_page_text = session.get(url = ,headers= ).text
```

### 超级鹰

可以验证坐标计算类的验证码

模拟登录12306

<br>

<br>

## 代理

破解封IP这种反爬机制

代理的作用:

- 突破自身IP访问的限制
- 隐藏自身真实的IP

代理相关的网站:

快代理

西祠代理

www.goubanjia.com

<br>

<br>

## 高性能异步爬虫

### 多线程/多进程

优势:	为阻塞操作单独开启线程或者进程

劣势:	无法无限制开启多线程/进程

### 进程池/线程池

优势:	降低系统对进程或者线程创建和销毁的频率

弊端:	池中线程/进程数量有上限

```Python
import time
from multiprocessing.dummy import Pool
#导入线程池模块对应的类

#使用线程池方式执行
start_time = time.time()

def get_page(url):
    print()
    time.sleep(2)
    print()
    
name_list = ['xiaozi','aa','bb','cc']


#实例化一个线程池对象
pool = Pool(4)
#将列表中每一个元素传递给get_page进行处理
pool.map(get_page,name_list)

end_time = time.time()
print(end_time - start_time)

pool.close()
pool.join()

```

原则:线程池处理的是阻塞且耗时的操作

并不是所有页面数据都来自url,有些是ajax

script 代码并不是标签,来自json,可以用正则解析

<br>

### 02-04-01 使用线程池爬取冰海战记漫画

```Python
import os
import re
import requests
from lxml import etree
from multiprocessing.dummy import Pool
import time

# 定义一个获取一话漫画所有图片的方法，方便pool调用


def get_comic(dic):

    img_url = dic['url']
    img_fp = './冰海战记漫画/' + dic['name']
    response = requests.get(url=img_url, headers=headers,
                            verify=False)

    # print(response.status_code, '\n')
    # print(img_url, '\n')
    img_data = response.content

    # 如果已经下载了就跳过
    if(not os.path.exists(img_fp)):
        with open(img_fp, 'wb') as fp:
            fp.write(img_data)
            print(dic['name'], '下载成功')

    else:
        print(dic['name'], "已存在")


start_time = time.time()

if not os.path.exists('./冰海战记漫画'):
    os.mkdir('./冰海战记漫画')

url = 'https://www.mhkan.com/manhua/binghaizhanji/'

headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/101.0.4951.64 Safari/537.36 Edg/101.0.1210.47',
    'Connection': 'close'
}

# 解除报错的一些操作，似乎并不用，
# requests.adapters.DEFAULT_RETRISE = 5
# s = requests.session()
# s.keep_alive = False
requests.packages.urllib3.disable_warnings()

#response = requests.get(url=url, headers=headers, )
# 报错:SSLError  HTTPSConnectionPool(host='www.mhkan.com', port=443): Max retries exceeded with url: /manhua/binghaizhanji (Caused by SSLError(SSLCertVerificationError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: unable to get local issuer certificate (_ssl.c:997)')))

# 解决方法：
# 1. headers里加入'Connection': 'close'
# 2. requests.packages.urllib3.disable_warnings()
# 3. requests.get 加入 verify=False

response = requests.get(url=url, headers=headers, verify=False)
page_text = response.text
# print(page_text)

# 生成etree对象
tree = etree.HTML(page_text)
# print(tree)

# 获取a标签的列表
a_list = tree.xpath(
    '/html/body/div[4]/div[1]/div/div[3]/div[1]/div[3]/div[3]/div[2]/ul/li/a')

# print(a_list)

# 对每一话漫画进行所有图片的下载
for a in a_list:
    comic_name = a.xpath('./span/text()')[0]
    if not os.path.exists('./冰海战记漫画/' + comic_name):
        os.mkdir('./冰海战记漫画/' + comic_name)

    # print(comic_name)
    comic_url = 'https://www.mhkan.com/' + a.xpath('./@href')[0]
    # print(comic_url)

    comic_response = requests.get(url=comic_url, headers=headers, verify=False)
    comic_text = comic_response.text

    # with open('./comic.txt', 'w', encoding='utf-8') as fp:
    #     fp.write(comic_text)

    c_tree = etree.HTML(comic_text)
    all_img_url = c_tree.xpath('//body/script[1]/text()')

    # 获取图片地址
    ex_img = '\\\/images(.*?).jpg'
    img_list = re.findall(ex_img, str(all_img_url[0]), re.S)

    dic_list = []
    img_num = 0
    for img_url in img_list:
        img_num += 1
        img_url = 'https:\/\/images' + str(img_url) + '.jpg'
        img_url = img_url.replace('\\', '')

        dic = {
            'url': img_url,
            'name': comic_name + '/' + str(img_num) + '.jpg'
        }

        dic_list.append(dic)

    # 相当于将dic_list的每个dic都传给get_comic
    # 使用线程池更加快捷
    pool = Pool(len(dic_list))
    pool.map(get_comic, dic_list)
    pool.close()
    pool.join()

end_time = time.time()

print("本次漫画爬取已结束！")
print("本次爬取用时", end_time - start_time, "秒")



```

**遇到的问题**：

```Python
0.报错:SSLError  HTTPSConnectionPool(host='www.mhkan.com', port=443)
解决：
headers里加入'Connection': 'close'
requests.packages.urllib3.disable_warnings()
requests.get 加入 verify=False

1.不明白pool是相当于将dic_list的每个dic都传给get_comic
解决：要将dic append 到 dic_list 中，而不是把一个dic给pool

2.下载的图片损坏
解决：404报错，一开始以为是访问过快
后来找到bug，是img_url写错了，加多了一个img_url
img_url = 'https:\/\/images' + str(img_url) + '.jpg'
写成了 img_url = 'https:\/\/images' + img_url +str(img_url) + '.jpg'
```

### 单线程+异步协程

**协程基础知识**

```python
import asyncio

#async修饰的函数，调用之后不会马上执行，会返回一个协程对象
async def request(url):
    print('正在请求的url是',url)
    print('请求成功',url)
    return url
    
c = request('www.baidu.com')
```

```Python

#创建一个事件循环对象
loop = asyncio.get_event_loop()

#将携程对象注册到loop中,然后启动loop
loop.run_until_complete(c)

```

```Python
#task 的使用
loop = asyncio.get_event_loop()

#基于loop创建一个task对象
task = loop.create_task(c)
print(task)

loop.run_until_complete(task)
print(task)

```

```Python
#future 的使用
loop = asyncio.get_event_loop()

#基于asyncio创建一个task对象
task = asyncio.ensure_future(c)
print(task)

loop.run_until_complete(task)
print(task)


```

```Python
#绑定回调
# request(url)->c->task(result)->callback_func
def callback_func(task):
	print(task.result())
    
    
loop = asyncio.get_event_loop()
task = asyncio.ensure_future(c)

#将回调函数绑定到任务对象中
task.add_done_callback(callback_func)

loop.run_until_complete(task)

```

#### 多任务异步协程

```Python
import asyncio
import time

async def request(url):
    print('正在下载',url)
    
    #在异步协程中如果出现了同步模块相关的代码,就无法实现异步
    # time.sleep(2)
    
    
    #当在asycio中遇到阻塞操作必须进行手动挂起
    await asyncio.sleep(2)  #asyncio中自带的延时函数
  
    print('下载结束',url)
    
start_time = time.time()
urls = [
    'www.baidu.com',
    'www.sogou.com',
    'www.goubanjia.com'
]

#创建任务列表
tasks = []



for url in urls:
    c = request(url)
    task = async.ensure_future(c)
	tasks.append(task)

loop = async.get_event_loop()
#需要将任务列表封装到wait中
loop.run_until_complete(asyncio.wait(tasks))

end_time = time.time()
print(end_time - start_time)

```

#### aiohttp

```Python
import aiohttp
import asyncio

async def get_page(url):
    async with aiohttp.ClientSession() as session:
        # get() / post()
        # headers / params(get) /data(post)
        # proxy='http://ip:port' (是字符串而不是字典)
        async with await session.get(url) as response
			#text()返回字符串类型的响应数据
            #read()返回二进制类型的响应数据
            #json()返回json对象
            #注意:获取响应数据之前一定要使用await进行手动挂起
            page_text = await response.text()
            print(page_text)
```

<br>

<br>

## selenium

基于浏览器自动化的一个模块

与爬虫之间的关联：

- 便捷地获取网站中动态加载的数据
- 便捷地实现模拟登录

### 使用流程：

**环境安装**

pip install selenium

**下载浏览器的驱动**

http://chromedriver.storage.googleapis.com/index.html

**实例化浏览器对象**

```Python
from selenium import webdriver
from time import sleep
dr = webdriver.Chrome(executable_path='./chromedriver')
dr.get(url) #对url发送get请求
page_text = dr.page_source #获取页面源码数据

#因为是使用浏览器打开网页,所以动态加载的数据也会在页面源码中

#然后再用其他解析方法(bs4或者xpath)获取想要的数据
```

遇到的问题: vscode中创建浏览器对象之后会自动关闭

解决的方法:用右键，选择在终端运营Python，浏览器就不会关。点触摸栏上面的小三角浏览器就一定会关闭.

#### 一些其他操作的实现

**搜索**

```Python
#淘宝

#标签定位
#dr.find_element 有一系列方法
search_input = dr.find_element_by_id('q')

#标签交互
search_input.send_keys('Iphone')
#<button class="btn-search tb-bg"> 空格两侧任选一个作为class属性值
btn = dr.find_element_by_css_selector('.btn-serch')  #类选择器,后面跟类的属性
btn.click()

sleep(5)

#关闭浏览器
dr.quit()

```

**滚轮**

```Python
#js代码实现滚轮
#window.scrollTo(0,document.body.scrollHeight) #向下滚动相当于一个屏幕的像素

dr.execute_script('window.scrollTo(0,document.body.scrollHeight)')
#有两个参数(x,y)
```

**回退**

```Python
dr.get("https://www.baidu.com")
dr.back()
dr.forward()
```

#### 处理iframe

iframe 是一个子页面,嵌套到当前页面

如果定位的标签是存在于iframe标签之中,需要进行如下操作

```Python
#切换作用域到子页面
dr.switch_to.frame(id)
div = bro.find_element_by_id('draggable')
```

**动作链**

```Python
from selenium.webdriver import ActionChains
#创建动作链实例化对象
action = ActionChains(dr)
div = bro.find_element_by_id('draggable')

#点击长按指定的动作
action.click_and_hold(div)

#拖动
for i in range(5):
    #perform()表示立即执行动作链操作 
    action.move_by_offset(17,0).perform() #有两个参数(x,y)
    sleep(0.3)
 
#释放动作链对象
action.release()

```

#### 模拟登录

```python
from selenium import webdriver
from time import sleep

dr = webdriver.Chrome('./chromedriver.exe')

dr.get('https://qzone.qq.com/')
dr.switch_to.frame('login_frame')

sleep(0.5)

change_bt = dr.find_element_by_id('switcher_plogin')
change_bt.click()

UserName = dr.find_element_by_id('u')
Password = dr.find_element_by_id('p')

sleep(1)
UserName.send_keys('2499128801')
sleep(1)
Password.send_keys('')

sleep(1)
Login_bt = dr.find_element_by_id('login_button')
Login_bt.click()


sleep(5)

dr.quit()

```

#### 无头浏览器

隐藏可视化界面,防止干扰用户 (直接粘贴代码)

```Python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options

chrome_options = Options()
chrome_options.add_argument('--headless')
chrome_options.add_argument('--disable-gpu')

###
dr = webdriver.Chrome(executable_path='./chromedriver.exe',chrome_options=chrome_options)#加一个参数
```

#### selenium规避检测风险

 (直接粘贴代码)

```Python
from selenium import webdriver
from selenium.webdriver import ChromeOptions

option = ChromeOptions()
option.add_experimental_option('excludeSwitches',['enable-automation'])

dr = webdriver.Chrome(executable_path='./chromedriver.exe',options=option)
```

<br>

<br>


## scrapy框架

框架:继承了很多功能且具有很强通用性的一个项目模板

学习:框架封装的功能和用法,学习到一定程度后再去理解源码和原理

scrapy:爬虫中封装好的一个明星框架

功能:高性能的持久化存储,异的书话剧下载,高性能的数据解析,分布式

**环境安装**

windows:

pip install wheel

下载安装twisted (http://www.lfd.uci.edu/~gohlke/pythonlibs/#twisted)

pip install pywin32

pip install scrapy

#### 使用:

- 创建一个工程

在终端中录入scrapy指令

```
scrapy startproject xxxPro
```

- cd xxxPro
- 在spiders子目录中创建一个爬虫文件

```
scrapy genspider My_spiderName www.xxx.com
```

- 编写代码
- 执行工程

```
scrapy crawl spiderName
```

创建后的源代码文件

```Python
import scrapy


class FirsttestSpider(scrapy.Spider):
    #爬虫文件名称:爬虫源文件的唯一标识
    name = 'FirstTest'
    
    ##一般要注释掉
    #允许的域名:限定start_urls列表中属于以下域名的才能被自动请求
    #allowed_domains = ['www.baidu.com']
    
    
    #起始的url列表:该列表中存放的url会被scrapy自动进行请求的发送
    start_urls = ['http://www.baidu.com/','https://www.sougou.com']

    
    #用作于数据解析:response参数表示的就是请求成功后的响应对象
    #每次只传入一个,会自动按照列表中个数多次调用
    def parse(self, response):
        pass



```

**取消遵从robot协议**

```Python
#settings.py 中 下一行的 True改为False
ROBOTSTXT_OBEY = False  
```

**在配置文件中 修改User-Agent**

**删除域名限定**

```Python
# ./spiders/XXX.py
	   # allowed_domains = ['www.xxx.com']
```


**取消显示日志**

```Python
scrapy crawl xxxpro --nolog #完全不显示日志

#settings.py配置文件中加入:
LOG_LEVEL = 'ERROR' #只显示指定类型的日志信息
```

#### 数据解析

```Python

	#解析写在parse方法下
	def parse(self,response):
        #解析：返回的是列表，元素是Selector对象
        div_list = response.xpath()
    
        #存储所有解析的数据
        all_data = []
    
        for div in div_list:
        
            author = div.xpath()[0].extract()
            #亦可 div.xpath().extract_first(),返回的是字符串
	    # 而extract()返回的是列表
        
       		#注：列表也可以调用extract()，表示将每一个对象中的data数据中提取出来,如：
        	content = div.xpath().extract() 
        	content = ''.join(content) #转化为字符串
    
			div = {
                'author':author,
                'content':content
            }
        
            all_data.append(dic)
        
        
        return all_data
  
```

#### 持久化存储

**基于终端指令：**

要求：只可以将parse方法的返回值存储到本地的文本文件中（即不可以存到数据库中）

```
scrapy crawl SpiderName -o filePath
```

只能存储为以下类型文件：

json, jsonlines, jl, csv, xml, marshal, pickle

好处：简介高效便捷

缺点：局限性强（数据只能存储到指定后缀的文本文件中）

**基于管道：**

编码流程：

- 数据解析
- 在item类中定义相关的属性

```Python
##在items.py中

class XXXItem(scrapy.Item):
    author = scrapy.Field()
    content = scrapy.Field()
```

- 数据封装到 item类型对象
- 将item对象提交给管道进行持久化存储的操作

```Python
from XXX.items import XXXItem

	#解析写在parse方法下
	def parse(self,response):
        #解析：返回的是列表，元素是Selector对象
        div_list = response.xpath()
        
        #存储所有解析的数据
        all_data = []
        
        for div in div_list:
            
            author = div.xpath()[0].extract()
            #亦可 div.xpath().extract_first()
            
       		#注：列表也可以调用extract()，表示将每一个对象中的data数据中提取出来,如：
        	content = div.xpath().extract() 
        	content = ''.join(content) #转化为字符串
        
			
		###
        	#实例化item对象,并将数据封装
			item = XXXItem()
            item['author'] = author
            item['content'] = content
            
            yield item #将item提交给了管道
            
	
```

- 在管道类的process_item中要将其接受到的item对象中的数据进行持久化存储

```Python
##在pipelines.py中

class XXXPipeline(object):
    
    fp = None
    
    #重写父类的一个方法:该方法只在开始爬虫的时候被调用一次
    def open_spider(self, spider):
        print('开始爬虫.......')
        self.fp = open('./qiubai.txt','w',encoding = 'utf-8')
    
    #专门用来处理item类型对象
	#该方法可以接收爬虫文件提交过来的item对象
    #该方法每接收一个item就会被调用一次
    
    def process_item(self, item, spider):
        author = item['author']
        content = item['content']
        
        self.fp.write(author+':'+content+'\n')
        
        return item #传递给下一个即将被执行的管道类
    
    #重写父类的一个方法:该方法只在结束爬虫的时候被调用一次
    def close_spider(self, spider):
        printf('结束爬虫......')
        self.fp.close()
        
       

```

- 在配置文件中开启管道

```Python
##在配置文件settings.py中

#取消注释
ITEM_PIPELINES = {
    'xxxx.pipelines.xxxPipeline':300,
    #300表示优先级,数值越小优先级越高
}
```

- 处理TypeError

好处:通用性强

#### 定义多个管道类

管道文件中一个管道类对应将一组数据存储到一个平台或者载体中

```python
##导入到数据库中
import pymysql

class mysqlPipeline(object):
	conn = None #链接对象
    cursor = NOne #游标对象
    def open_spider(self,spider):
        self.conn = pymysql.Connect(host='127.0.0.1', port=3306, user='root', password='123456', db='qiubai',charset='utf8') #创建连接对象,db是存储项目名
	
    def process_item(self,item,spider):
        self.cursor = self.conn.cursor()
        
        
        try:
        	self.cursor.execute('insert into qiubai values("%s","%s")'%(item["author"],item["content"]))
            #注意格式化拼接,%()应该放在引号外面
            
            self.conn.commit()
        except Exception as e:
            print(e)
            self.conn.rollback()
            
    #关闭
    def close_spider(self,spider):
        self.cursor.close()
        self.conn.close()
```

配置文件开启管道类

```Python
ITEM_PIPELINES = {
    'qiubaiPro.pipelines.QiubaiproPipeline':300,
    'qiubaiPro.pipelines.mysqqlPipeLine':301,
}
```

爬虫文件提交的item类型的对象最终只会给优先级最高的管道类

可以在优先管道类中添加(最好在编码过程中养成习惯无论有无下一个管道了都加上)

```Python
 return item #传递给下一个即将被执行的管道类
```

#### 基于Spider的全站数据爬取

全站数据爬取：将网站中某板块下的全部页码对应的页面数据进行爬取

- 注释allow_domains
- 解析数据
- 写出通用url模板
- 手动请求发送

```python
#spiders文件夹 下的 xiaohua.py

import scrapy

class XiaohuaSpider(scrapy.Spider):
    name = 'xiaohua'
    # allowed_domains = [xxx]
    start_urls = ['xxx']
    
    #生成一个通用的url模板(不可变)
    url = 'xxx%dxxx'
	page_num = 2
    
    #手动请求发送:callback回调函数是专门用作数据解析
    #递归
    if self.page_num <= 11:
        new_url = format(self.url%self.page_num)
        self.page_num += 1
        yield scrapy.Request(url=new_url,callback=self.parse)


```

#### 五大核心组件

**爬虫Spider:**

​	①产生url,对url进行请求

​	②数据解析

**引擎Scrapy:**

​	Spider将请求对象提交给引擎,引擎再提交给调度器

**调度器Scheduler:**

​	分为两部分:过滤器和队列

​	过滤器去重,然后加入队列

​	调度器再将新的请求对象交给引擎

**下载器Downloader:**

​	引擎将新的请求对象给下载器,下载器负责通过互联网进行数据下载

​	Response数据交给引擎

​	引擎再交给Spider

​	Spider调用parse进行数据解析

​	封装到item中并交给引擎

**管道Pipeline:**

​	引擎将数据交给管道对item进行持久化存储

*下载器使用twist模块进行异步下载

*引擎对数据流进行处理,并可以触发事务(对象实例化,方法调用)

#### 请求传参

使用场景:如果爬取解析的数据不在同一张页面中,需要在下一级页面才能获取(深度爬取)

基于管道进行持久化存储就必须使用请求传参

即 parse内调用一个新的parse_2,但是item是定义在parse内的局部变量,因此需要将parse中的item传递给parse_2  的过程

```Python
	yield scrapy.Request(url=detail_url, callback=self.parse_detail, meta={'item':item})
    
    
    ##
    def parse_2(self,response):
        item = response.meta['item']
        .....
        item['job_desc'] = job_desc
        
        yield item
```

#### 爬取壁纸

问题:Pause in Debugger 已在调试程序中暂停

解决:打开 Activate Breakpoint (Ctrl+F8),然后点击开始调试

#### ImagesPipeline

图片数据爬取的管道类

只需要将img的src的属性值进行解析，提交到管道，管道就会对图片的src进行请求发送获取图片的二进制类型的数据，且还会进行持久化存储

字符串类型与图片类型 数据爬取的区别？

- 字符串：只需要基于xpath进行解析并提交管道进行持久化存储
- 图片：xpath解析出图片src的属性值，之后单独对地址发起请求获取图片二进制类型的数据

**使用流程**

*爬取站长素材中的高清图片*

图片懒加载

img标签下的属性值为src2（伪属性），只有图片进入可视化区域才会加载出来，此时属性值变更为src

所以在爬虫文件中解析时要使用伪属性

**流程**

- 数据解析(图片地址)
- 将存储地址的item提交到制定的管道类

```Python
##xxx.py

import scrapy
form imgsPro.items import ImgsproItem #自带的管道类

class ImgSpider(scrapy.Spider):
    name = 'img'
    
    start_urls = ['xxx']
	
    def parse(self, response):
        div_list = response.xpath('//div[@id="container"]/div')
        for div in div_list:
            #使用伪属性
			src = div.xpath('./div/a/img/@src2').extract_first()
            item = ImgsproItem()
            item['src'] = src

            yield item
```

- 重写管道类

```Python
 ##在//spider/pipelines.py 改一下管道类
 from scrapy.pipelines.images import ImagesPipeline
 
 class imgsPipeLine(ImagesPipeline):
     
     #根据图片地址进行图片数据的请求
 	def get_media_requests(self, item, info):
 		yield scrapy.Request(item['src'])
         
     #指定图片存储的路径
     def file_path(self, request, response=None, info=None):
         imgName = request.url.split('/')[-1] #以斜杠分割并取最后一个字符串作为名称
         return imgName
     	
         ##需要在settings.py中指定存储路径
         ##加入: IMAGES_STORE = './imgs'  会存放在imgs文件夹中 
     
     
     def item_completed(self, results, item, info):
         return item #返回给下一个即将被执行的管道类
     
     	##在settings.py 中将ITEM_PIPELINES这一段取消注释且将管道类的名称更换
     
     
```

- 在配置文件中指定文件存储目录和开启自定管道



#### 中间件

引擎和下载器之间的中间件叫 下载中间件

middlewares.py 中的类 XXXSpiderMiddleware(object)

引擎和爬虫之间的中间件叫 爬虫中间件

middlewares.py中的类 XXXDownloaderMiddleware(object)


**下载中间件**

作用：批量拦截整个工程中所有的请求和响应

拦截请求：

* UA伪装（局部的，对于下载的）: 写在process_request
* 代理IP的设定: 写在process_exception:return request

拦截响应：

* 篡改响应数据，响应对象


删除类 XXXSpiderMiddleware(object)

删除类 XXXDownloaderMiddleware(object)中的方法: from_crawler(父类方法) 和 spider_opened(打印日志)

重点处理以下方法:

```Python
import random


class XXXDownloaderMiddleware(object):
# 封装UA池(装有大量的UA)
user_agent_list = []

# 代理池 (可以在www.goubanjia.com查询)
PROXY_http = [
	'158.180.102.104:80',
	'195..208.131.189:56055',

]
PROXY_https = [
	'120.83.49.90:9000',
	'95.189.122.214.35508',
]

# 拦截请求
def process_request(self, request, spider):


	# UA伪装
	request.headers['User-Agent'] = random.choice(user_agent_list)


# 拦截所有响应
def process_response(self, request, exception, spider):


# 拦截发生异常的请求
def process_exception(self, request, exception, spider):
	if request.url.split(':')[0] == 'http':
		# 代理(异常才使用代理)
		request.meta['proxy'] = 'http://' + random.choice(self.PROXY_http)
	else:
		request.meta['proxy'] = 'https://' + random.choice(self.PROXY_https)

	return request # 将修正之后的请求对象进行重新的请求发送
```


**到settings.py中开启下载中间件**

```Python
# Enable or disable downloader middlewares
# See https://docs.scrapy.org/en/latest/topics/downloader-middleware.html
DOWNLOADER_MIDDLEWARES = {
    'firstblood.middlewares.FirstbloodDownloaderMiddleware': 543,
}
```



需求:爬取网易新闻中的新闻数据(标题+内容)

1.通过网易新闻主页面获取各大板块的详情页url

```Python
import scrapy


class WangyiSpider(scrapy.Spider):
    name = 'wangyi'
    #allowed_domains = ['www.xxx.com']
    start_urls = ['https://news.163.com/']
    models_urls = [] # 存储板块对应详情页的url

    # 解析五大板块对应详情页的url
    def parse(self, response):
        li_list = response.xpath('//*[@id="index2016_wrap"]/div[2]/div[2]/div[2]/div[2]/div/ul/li')
        alist = [1,2,4]
        for index in alist:
            model_url = li_list[index].xpath('./a/@href').extract_first()
            self.models_urls.append(model_url)

        # 依次对每一个板块对应的页面进行请求
        for url in self.models_urls:
            yield scrapy.Request(url, callback=self.parse_model)

  
    # 每一个板块对应的新闻标题相关的内容都是动态加载的
    def parse_model(self, response): # 解析每一个板块页面中对应新闻的标题和新闻详情页的url
        # response.xpath()
        pass
```


2.每一个板块对应的新闻标题都是动态加载的(中间件操作的重点)

```Python
# Define here the models for your spider middleware
#
# See documentation in:
# https://docs.scrapy.org/en/latest/topics/spider-middleware.html

from urllib import response
from scrapy import signals

# useful for handling different item types with a single interface
from itemadapter import is_item, ItemAdapter
from scrapy.http import HtmlResponse




class WangyiproDownloaderMiddleware:
    # Not all methods need to be defined. If a method is not defined,
    # scrapy acts as if the downloader middleware does not modify the
    # passed objects.

    # 通过该方法拦截五大板块对应的响应对象，进行篡改
    def process_request(self, request, spider):
      
        return None

    def process_response(self, request, response, spider): #spider指爬虫对象
        # 挑选出指定的响应对象进行篡改
        # 通过url指定request
        # 通过request指定response
        if request.url in spider.models_urls:
            # response # 五大板块对应的响应对象
            # 针对定位到的response进行篡改
            # 实例化一个新的响应对象（符合需求：包含动态加载出的新闻数据），替代原来旧的不满足需求的响应对象
            # 如何获取出动态加载的新闻数据？
                # 基于selenium 便捷的获取动态加载数据
              
            new_response = HtmlResponse(url,body,encoding,request)
      
            return new_response
        else:
            response # 其他请求对应的响应对象


        return response

    def process_exception(self, request, exception, spider):
        # Called when a download handler or a process_request()
        # (from other downloader middleware) raises an exception.

        # Must either:
        # - return None: continue processing this exception
        # - return a Response object: stops process_exception() chain
        # - return a Request object: stops process_exception() chain
        pass



```

在爬虫文件中实例化一个浏览器对象

```Python
#import scrapy
from selenium import webdriver

# class WangyiSpider(scrapy.Spider):
#     name = 'wangyi'
#     #allowed_domains = ['www.xxx.com']
#     start_urls = ['https://news.163.com/']
#     models_urls = [] # 存储板块对应详情页的url

	#实例化一个浏览器对象
    def __init__(self):
        self.bro = webdriver.Chrome(executable_path='D:\Desktop\Python\Crawl Spider\chromedriver.exe')


#     # 解析五大板块对应详情页的url
#     def parse(self, response):
#         li_list = response.xpath('//*[@id="index2016_wrap"]/div[2]/div[2]/div[2]/div[2]/div/ul/li')
#         alist = [1,2,4]
#         for index in alist:
#             model_url = li_list[index].xpath('./a/@href').extract_first()
#             self.models_urls.append(model_url)

#         # 依次对每一个板块对应的页面进行请求
#         for url in self.models_urls:
#             yield scrapy.Request(url, callback=self.parse_model)

  
#     # 每一个板块对应的新闻标题相关的内容都是动态加载的
#     def parse_model(self, response): # 解析每一个板块页面中对应新闻的标题和新闻详情页的url
#         # response.xpath()
#         pass
```



3.通过解析出每一条新闻详情页的url获取详情页的页面源码,解析出新闻内容

```Python
from time import sleep

    def process_response(self, request, response, spider): #spider指爬虫对象
	bro = spider.bro
        # 挑选出指定的响应对象进行篡改
        # 通过url指定request
        # 通过request指定response
        if request.url in spider.models_urls:
            bro.get(request.url) # 五个板块对应的url进行请求
            sleep(2)
            page_text = bro.page_source # 包含了动态加载的新闻数据

            # response # 五大板块对应的响应对象
            # 针对定位到的response进行篡改
            # 实例化一个新的响应对象（符合需求：包含动态加载出的新闻数据），替代原来旧的不满足需求的响应对象
            # 如何获取出动态加载的新闻数据？
                # 基于selenium 便捷的获取动态加载数据

            new_response = HtmlResponse(url=request.url,body=page_text,encoding='utf-8',request=request)

  
            return new_response
        else:
            return response # 其他请求对应的响应对象
```


```python
# import scrapy
from selenium import webdriver
from wangyiPro.items import WangyiproItem


# class WangyiSpider(scrapy.Spider):
#     name = 'wangyi'
#     #allowed_domains = ['www.xxx.com']
#     start_urls = ['https://news.163.com/']
#     models_urls = []  # 存储板块对应详情页的url

#     # 实例化一个浏览器对象
#     def __init__(self):
#         self.bro = webdriver.Chrome(
#             executable_path='D:\Desktop\Python\Crawl Spider\chromedriver.exe')

#     # 解析五大板块对应详情页的url

#     def parse(self, response):
#         li_list = response.xpath(
#             '//*[@id="index2016_wrap"]/div[2]/div[2]/div[2]/div[2]/div/ul/li')
#         alist = [1, 2, 4]
#         for index in alist:
#             model_url = li_list[index].xpath('./a/@href').extract_first()
#             self.models_urls.append(model_url)

#         # 依次对每一个板块对应的页面进行请求
#         for url in self.models_urls:
#             yield scrapy.Request(url, callback=self.parse_model)

#     # 每一个板块对应的新闻标题相关的内容都是动态加载的

    def parse_model(self, response):  # 解析每一个板块页面中对应新闻的标题和新闻详情页的url
        # response.xpath()
        div_list = response.xpath(
            '/html/body/div/div[3]/div[3]/div[1]/div[1]/div/ul/li/div/div')
        for div in div_list:
            title = div.xpath('./div/h3/a/text()').extract_first()
            news_detail_url = div.xpath('./div/h3/a/@href').extract_first()

            item = WangyiproItem()
            item['title'] = title

            # 对新闻详情页的url发起请求
            yield scrapy.Request(url=news_detail_url, callback=self.parse_detail, meta={'item': item})

    def parse_detail(self, response):  # 解析新闻内容
        content = response.xpath('//*[@id="content"]/div[2]//text()').extract()
        content = ''.join(content)
        item = response.meta['item']
        item['content'] = content

        yield item

    #重写父类方法，会在结束时自动调用一次，用来关闭无头浏览器
    def closed(self,spider):
        self.bro.quit()
```


```Python
# items.py中

import scrapy

class WangyiproItem(scrapy.Item):
    # define the fields for your item here like:
    title = scrapy.Field()
    content = scrapy.Field()
  
```

管道类中进行持久化存储


开启管道

```Python
# setting.py
ITEM_PIPELINES = {
   'wangyiPro.pipelines.WangyiproPipeline': 300,
}
```


> 遇到的问题：selenium打开无头浏览器后闪退
>
> 解决：没在settings开启下载中间件= =
>
> 同时可以忽略ERROR:device_event_log_impl.cc(214)
>

<br>


### CrawlSpider

一个类，Spider的子类


全站数据爬取：

一个板块下所有页码的数据进行爬取

* 基于Spider：手动请求
* 基于CrawlSpider


**CrawlSpider的使用**

* 创建一个工程
* cd XXX
* 创建爬虫文件（CrawlSpider）：

  * scrapy genspider -t crawl xxx www.xxxx.com
  *


**rules**

```Python
rules = (
	# Rule: 规则解析器
	# LinkExtractor: 链接提取器
	Rule(LinkExtractor(allow=r'Items/'),callback='parse_item',follow=True),
)


```

* **链接提取器**

	根据指定规则 (allow="正则") 进行指定链接的提取

	默认去重

* **规则解析器**

	将链接提取器提取到的链接进行指定规则 (callback) 的数据解析

一个链接提取器对应一个规则解析器

* **follow**

  可以将链接提取器继续作用到链接提取器提取到的链接所对应的页面


* xpath中不能出现tbody标签



用一个Rule分别解析页面编号和标题,另一个Rule解析详情页编号和内容

Rule各有一个parse解析方法

用一个item存储页面编号和标题,另一个item存储详情页编号和内容

两个item分别提交给管道

* 如何判定item类型:

```Python
# pipeline中

	if item.__class__.__name__ == 'DetailItem':
	……
	else:
	……
```

* 如何保持数据的一致性

  两个编号相同时插入到一起

<br>


### 分布式爬虫

* 概念：需要搭建一个分布式的机群（多台电脑），对一组资源进行分布联合爬取
* 作用：提升爬取数据的效率


**如何实现分布式**

* 安装scrapy-redis组件

* 原生的scrapy不可以实现分布式爬虫（组件之间没有关联），必须要让scrapy结合scrapy-redis组件一起实现分布式爬虫

  * 调度器不可以被分布式机群共享
  * 管道不可以被分布式机群共享


**搭建分布式爬虫**

* 创建一个工程
* 创建一个基于CrawlSpider的爬虫文件
* 修改当前爬虫文件

  * 导包： from scrapy_redis.spiders import RedisCrawlSpider
  * 将start_urls和allowed_domains进行注释
  * 添加一个新属性：redis_key = 'xxx' （可以被共享的调度器队列名称）
  * 编写数据解析相关操作
  * 将当前爬虫类的父类改成RedisCrawlSpider
* 修改配置文件settings：

  * 指定使用可以被共享的管道：

    * ```Python
      # 指定管道
      ITEM_PIPELINES = {
      	'scrapy_redis.pipelines.RedisPipeline':400
      }

      # 指定调度器
      #  增加一个去重容器类的配置，作用是用Redis的set集合来存储请求的指纹数据，从而实现请求去重的持久化存储
      DUPEFILTER_CLASS = 'scrapy_redis.dupefilter.RFPDupeFilter'
      #  使用scrapy_redis组件自己的调度器
      SCHEDULER = 'scrapy_redis.scheduler.Scheduler'
      #  配置调度器是否要持久化，也就是当爬虫结束了，要不要清空Redis中请求队列和去重指纹的set
      SCHEDULER_PERSIST = True

      ```
  * 指定radis服务器

    * ```python
      # 指定redis
      REDIS_HOST = ''
      REDIS_PORT = 
      ```

* redis相关操作配置

  * 配置redis的配置文件

    * linux或者mac：redis.conf
    * windows:redis.windows.conf
    * 打开配置文件修改：将    bind 127.0.0.1（某个ip）  注释
    * 关闭保护模式： 将 protected-mode no(由yes改为no）
  * 结合配置文件开启redis服务

    * ```vim
      redis-server +配置文件
      ```
  * 启动客户端：

    * redis-cli
* 执行工程：

  * scrapy runspider xxx.py
* 向调度器的队列中放入一个 起始url：

  * 调度器的队列在redis的客户端中

    * lpush xxx www.xxx.com

* 爬取到的数据存储在了redis的proName:items这个数据结构中



<br>

<br>


## 增量式爬虫

* 概念: 检测网站数据更新的情况,只会爬取网站最新更新出来的数据

* 核心: 检测url之前有没有请求过

  * 将爬取过的url存储

    * 存储到redis的set数据结构(原生set也行)中

```Python
def parse_item(self, response):

	ex = self.conn.sadd('urls', detail_url)
	if ex == 1:
		……
	else:
```




---

b站课程已观看完毕
