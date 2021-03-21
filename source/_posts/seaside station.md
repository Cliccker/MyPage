---
title: 抓取某网站的摄影作品
categories: 实践
img: 'https://my-picbed.oss-cn-hangzhou.aliyuncs.com/img/20200826140039.jpg'
tags:
  - 爬虫
  - python
  - 摄影
abbrlink: 25022
date: 2020-08-26 13:12:23
---

最近逛知乎找到了一个神奇的网站[SEASIDE STATION IN JAPAN](https://seaside-station.com/)



这个网站记录了大大小小182个靠海车站的景色

![网站首页](https://my-picbed.oss-cn-hangzhou.aliyuncs.com/img/20200826131956.png)

其中不乏一些我比较喜欢的图片，比如

<img src="https://my-picbed.oss-cn-hangzhou.aliyuncs.com/img/20200826132210.png" alt="wabuka站" style="zoom: 50%;" />

<img src="https://my-picbed.oss-cn-hangzhou.aliyuncs.com/img/20200826132255.png" alt="umishibaura站" style="zoom: 50%;" />

许多图片我都想保存下来，但是那样做实在太麻烦了，所以选择用爬虫的方式，代码如下：

```python
import requests
import re
import time
from tqdm import tqdm


def get_url(index):
    wb_date = requests.get(index).text
    res = re.compile(r'***"') # 保护一下这个网站
    reg = re.findall(res, wb_date)
    print(reg)
    return reg


def get_picture(url):
    wb_date = requests.get(url).text
    res = re.compile(r'src="(http.+?jpg)"')  # 正则表达式匹配图片
    reg = re.findall(res, wb_date)
    nun = 0
    for i in tqdm(reg):  # 遍历
        a = requests.get(i)
        name = re.sub(r"***", "", i)
        f = open('all_station/' + name, 'wb')
        f.write(a.content)
        f.close()
        nun = nun + 1


urls = get_url('***')
for i, urls in tqdm(enumerate(urls)):
    try:
        get_picture(urls)
    except:
        continue
    time.sleep(5)
        
```

如果你也喜欢这些图片，可以在评论里索取，千万不要学我！！！



