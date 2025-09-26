---
layout: post
title: "Python实用技巧-网页抓取基础"
date: 2018-11-30 
description: "BeautifulSoup、lxml、requests 组合"
tag: Python 

---

### BeautifulSoup 基础

>```python
>from bs4 import BeautifulSoup
>import requests
>
>response = requests.get('https://example.com')
>soup = BeautifulSoup(response.text, 'html.parser')
>title = soup.find('title').text
>```

### 选择器使用

>```python
>links = soup.find_all('a', href=True)
>for link in links:
>    print(link['href'])
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python实用技巧-网页抓取基础](http://zhouzhiyang.cn/2018/11/Python_Tips_Web_Scraping/) 

