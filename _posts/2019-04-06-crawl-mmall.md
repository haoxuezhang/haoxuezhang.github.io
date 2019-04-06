---
layout: post
title: FTP网站带目录递归爬取
categories: 爬虫
description: FTP网站带目录递归爬取
keywords: python, 爬虫, requests
---

明明半个小时就可以手动下载，却非要花一个小时写一个爬虫(/手动狗头)

---
## 引子
学习java web部署的时候找到一个[网站](http://learning.happymmall.com/)，这个网站有很多部署java项目的配置文件，下载就直接可以用，并且还有不少值得学习的资料，所以就想把这个网站的文件下载下来，网站的文件还是有点多的，并且这个网站下载速度很慢，手动点击下载的话非常慢，所以就想写一个爬虫帮助我下载文件，

## 分析网站
网站是一个典型的FTP站点，如图：
![](/images/posts/20190406_1.png)
该ftp网站中有文件也有文件夹，爬取时需要判断是文件还是文件夹，并且在本地创建相同的目录结构。a标签下有文件的地址以及目录跳转链接
![](/images/posts/20190406_2.png)
看网站中所有的a标签就是我们需要的链接地址，以`/`结尾的是文件夹地址，其他的就直接是需要爬取的文件。另外文件的地址href标签都被参数化，不方便建立文件夹，将被参数化后的部分替换为text，发现也是可以直接访问的，
```url
http://learning.happymmall.com/QQ%E5%AD%A6%E4%B9%A0%E7%BE%A4%E5%A4%A7%E5%AE%B6%E5%85%B1%E4%BA%AB%E7%9A%84useful%E6%96%87%E6%A1%A3/Intellij%20IDEA%E6%95%99%E7%A8%8B.pdf
```
转为
```url
http://learning.happymmall.com/QQ学习群大家共享的useful文档/Intellij IDEA教程.pdf
```
需要在本地建立与网站相同目录的文件夹，将上面URL以`/`进行split分割，取`[3:-1]`位置的即为需要建立的文件夹名。
有了文件夹后即可通过`requests`库进行文件递归爬取。

## 代码
1. 首先引入需要的包
```python
from pyquery import PyQuery as pq
import requests
import os
```

2. 要爬取的网站地址
```python
base_url = 'http://learning.happymmall.com/'
```

3. 根据URL以及目录下载文件
```python
def get_file(url, path):
    if os.path.exists(path):
        print('文件已经存在： ', path)
    else:
        response = requests.get(url)
        with open(path, 'wb') as f:
            f.write(response.content)
            f.close()
        print('下载完成： ', path)
```

4. 递归爬取文件，注意要去除`./`和`../`这连个目录
```python
def get_url(page_url):
    doc = pq(url=page_url)
    # response = requests.get(url)
    a_links = doc.find('a')
    hrefs = []
    for a in a_links.items():
        href = a.attr('href')
        name = a.text()
        if href == '../' or href == './':
            continue
        else:
            if href[-1] == '/':
                new_url = os.path.join(page_url, name)
                dirs = new_url.split('/')[3:-1]
                dir = '/'.join(dirs)
                if os.path.exists(dir)==False:
                    os.makedirs(dir)
                get_url(new_url)
            else:
                path = os.path.join(page_url, name)
                name = path[31:]
                get_file(path, name)
                # print(os.path.join(page_url, href))
```

5. 运行程序
```python
get_url(base_url)
print('success')
```

### 到此为止所有的文件已经爬取到我们自己的电脑上面了，下面是代码和爬取下来的文件结构，done.
![](/images/posts/20190406_3.png)
代码比较短，就不上传`github`了，把上面五部分全部复制下来即可。

> 从开始提出想法到写完代码不到一个小时，还有很多需要改进的地方，例如在爬取的时候可以用异步方法加速爬取，这个网站比较简单，没有反爬策略，如果有反爬的话还需要考虑加代理。


