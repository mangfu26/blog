---
title: selenium -- 等待网页加载完成
index_img: /images/post/selenium-cover.jpg
date: 2022-06-17 17:35:12
categories:
- 开发
- 技巧
tags: 
- selenium
- 爬虫
---


# selenium -- 等待网页加载完成

使用elenium做项目时候通常需要等待网页完全加载后再执行某些操作

例如需要爬取一个动态表格的数据，需要等待JS完全加载后从后端拿到数据渲染

> 环境:
> 系统: Windows10
> Python: Python 3.8.7
> Selenium: 4.2.0


## 1、Selenium等待方式

> 详情请查看官方文档: [https://www.selenium.dev/zh-cn/documentation/webdriver/waits/](https://www.selenium.dev/zh-cn/documentation/webdriver/waits/)


### 1.1、显式等待(Explicit wait)

根据等待条件进行等待, 只有等待条件为 `true` 时再继续执行，反正将一直等待直到超时


### 1.2、隐式等待(Implicit wait)

全局等待, 只有网页完全加载时才继续执行, 否则将一直等待直到超时


### 1.3、流畅等待(Fluent Wait)

多配置版等待，可以配置 最长等待时间, 轮询频率, 忽略等待时出现的特定类型的异常(例如在页面上搜索元素时出现的 `NoSuchElementException` 异常)等



## 2、等待页面加载完成的方法

### 2.1、使用隐式等待

了解了Selenium的各种等待方法后，很自然地选择了 `隐式等待` ，因为它是全局地，并且会循环检查页面元素是否加载，否则将会一直等待，直到超时或者加载完成

```python
driver = Firefox()

# 等待页面加载, 超过10秒抛出异常
driver.implicitly_wait(10)

# 后续操作
driver.get("https://www.baidu.com")
my_dynamic_element = driver.find_element(By.ID, "myDynamicElement")
```

### 2.2、使用显式等待 + JS动态插入元素检查

还有一种方法是利用显式等待配置JS动他插入元素地方式来实现, 原理：

使用JS动态插入一个div元素作为标记元素, 然后使用显式等待, 等待条件为标记元素出现 

```python
import uuid

# 为标记元素生成一个UUID
markElementId = uuid.uuid1().hex

# 在Body元素下插入标记元素
jsCode = '''
let div = document.createElement('div');
div.setAttribute('id', '@@ID@@');
div.setAttribute('style', 'width: 0px; height: 0px;');
let body = document.getElementsByTagName('body')[0];
body.appendChild(div);
'''.replace('@@ID@@', markElementId)

# 等待页面加载完成
try:
    # 执行JS
    self.brower.execute_script(jsCode)
    # 如果检测到标注元素, 视为页面加载完成, 否则一直等待, 直到超时
    wait = WebDriverWait(self.brower, 10)
    wait.until(lambda b : b.find_element(By.ID, markElementId))
except Exception as err:
    print('页面加载超时')
```

