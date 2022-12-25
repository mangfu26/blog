---
layout: post
title: 转义XML标签内容中的特殊字符
index_img: /images/post/code-cover.png
date: 2022-12-25 22:20:00
categories:
- 开发
- 日常
tags: 
- 开发
- Python
---

## 1、前言

因为公司业务需求，需要经常爬取 [CNVD](https://www.cnvd.org.cn/) 和 [CNNVD](https://www.cnnvd.org.cn/) 上的漏洞信息，但 CNVD 用了知道创宇的 CND 加速乐进行内容分发，这玩意反爬太厉害了，一直绕不过，又不想上无头浏览器，恰好这两个漏洞库都有增量 xml 文件发布，所以打算在内网自建一个漏洞数据库，定期导入增量漏洞数据保持同步即可，这样在内网想怎么查就怎么查，快乐~。

感兴趣的可以在两个漏洞库的官网下载增量 xml 文件自己导入：

[https://www.cnnvd.org.cn/home/dataDownLoad](https://www.cnnvd.org.cn/home/dataDownLoad)

[https://www.cnvd.org.cn/shareData/list](https://www.cnvd.org.cn/shareData/list)


## 2、遇到的导入问题

在导入的过程中，遇到了一个比较奇葩的问题，在导入 CNNVD 的 XML 文件时报了以下错误：

```shell
C:\Users\mangfu> python
Python 3.8.7 (tags/v3.8.7:6503f05, Dec 21 2020, 17:59:51) [MSC v.1928 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license" for more information.
>>> from xml.etree import ElementTree
>>> with open(r'C:\Users\mangfu\Downloads\1月_979a496e-9222-4eac-937f-9707b4d875b8.xml', encoding='utf-8') as f:
...    root_element = ElementTree.parse(f).getroot()
...
Traceback (most recent call last):
  File "<stdin>", line 2, in <module>
  File "E:\python3.8.7\lib\xml\etree\ElementTree.py", line 1202, in parse
    tree.parse(source, parser)
  File "E:\python3.8.7\lib\xml\etree\ElementTree.py", line 595, in parse
    self._root = parser._parse_whole(source)
xml.etree.ElementTree.ParseError: not well-formed (invalid token): line 207, column 178
>>>
```

报了个解析错误，看报错消息是 xml 文件的 207 行导致的报错，看下 207 行的内容

![](images/post/2b04777f846311edbae82c337aee98b9.png)

```xml
<vuln-descript>Digia Qt是芬兰Digia公司的一套跨平台的C++应用程序开发框架。该框架可用于开发GUI程序。 Digia Qt 5.0.0 到 5.15.2 和 6.0.0 到 6.2.1 中的 Qt SVG 存在缓冲区错误漏洞，该漏洞源于在 QtPrivate::QCommonArrayOps<QPainterPath::Element>::growAppend（从 QPainterPath::addPath 和 QPathClipper 调用）中有一个越界写入。</vuln-descript>
```

CNNVD 的 XML 中的标签内容并没有转义，而且粗略过了一遍 xml 文件，发现未转义的标签内容还有很多

![](images/post/7872491c846511edaa732c337aee98b9.gif)

没办法，只能写个脚本转义特殊字符了

## 3、编写转义脚本

### 3.1、xml 文件处理函数

```python
import re

def replace_xml_special_characters(file_content: str) -> str:
    '''
    替换 XML 标签内容中的特殊字符为转移字符

    Args:
        file_conetnt: XML 文件内容

    Returns:
        处理后的 XML 文件内容
    '''
    def _rep(matched):
        '''
        替换处理函数, 将特殊字符替换为转义字符

        Args:
            matched: 正则匹配结果
        '''
            
        # 截取匹配到的标签内容
        start_index, end_index = matched.span()
        s = file_content[start_index:end_index]
        # 由于正则表达式匹配了标签的闭合字符
        # 例如 <vuln-descript>Wolfssl（CyaSSL）...... WolfSSL wolfMQTT 1.9。</vuln-descript>
        # 将会匹配到 >Wolfssl（CyaSSL）...... WolfSSL wolfMQTT 1.9。<
        # 需要去除内容首尾的 > < 剩下的才是真正要处理的内容
        s = s.lstrip('>').rstrip('<')
        # 替换特殊字符为转移字符
        s = s.replace('&', '&amp;').replace('<', '&lt;').replace('>', '&gt;')
        # 返回前添加上 > < 让标签闭合
        return f'>{s}<'

    # 匹配包含特殊字符 & < > 的标签内容
    p = r'>.*<.*<|>.*>.*<|>.*&.*<'
    file_content = re.sub(p, _rep, file_content)

    return file_content
```

### 3.2、调用函数处理 xml 文件

为了方便批量处理，xml 路径可以接收一个 xml 文件路径或 xml 目录

```python
import sys
from pathlib import Path


if __name__ == '__main__':
    # 从命令行参数读取 xml 或者 xml 目录路径
    if len(sys.argv) < 2:
        print('[-] 请指定 xml 文件或者 xml 目录路径')
        sys.exit(1)

    xml_path = Path(sys.argv[1])
    if not xml_path.exists():
        print(f'[-] "{sys.argv[1]}" 不是有效的路径')
        sys.exit(1)

    # 处理 xml 文件
    if xml_path.is_file():
        file_content = replace_xml_special_characters(xml_path.read_text())
        xml_path.write_text(file_content)

    # 处理 xml 目录
    elif xml_path.is_dir():
        for sub_xml_path in xml_path.glob("*.xml"):
            file_content = replace_xml_special_characters(sub_xml_path.read_text())
            sub_xml_path.write_text(file_content)

    else:
        print(f'[-] "{sys.argv[1]}" 不是有效的路径')
        sys.exit(1)
```

### 3.3、完整脚本源码

```python
import re
import sys
from pathlib import Path


def replace_xml_special_characters(file_content: str) -> str:
    '''
    替换 XML 标签内容中的特殊字符为转移字符

    Args:
        file_conetnt: XML 文件内容

    Returns:
        处理后的 XML 文件内容
    '''
    def _rep(matched):
        '''
        替换处理函数, 将特殊字符替换为转义字符

        Args:
            matched: 正则匹配结果
        '''
            
        # 截取匹配到的标签内容
        start_index, end_index = matched.span()
        s = file_content[start_index:end_index]
        # 由于正则表达式匹配了标签的闭合字符
        # 例如 <vuln-descript>Wolfssl（CyaSSL）...... WolfSSL wolfMQTT 1.9。</vuln-descript>
        # 将会匹配到 >Wolfssl（CyaSSL）...... WolfSSL wolfMQTT 1.9。<
        # 需要去除内容首尾的 > < 剩下的才是真正要处理的内容
        s = s.lstrip('>').rstrip('<')
        # 替换特殊字符为转移字符
        s = s.replace('&', '&amp;').replace('<', '&lt;').replace('>', '&gt;')
        # 返回前添加上 > < 让标签闭合
        return f'>{s}<'

    # 匹配包含特殊字符 & < > 的标签内容
    p = r'>.*<.*<|>.*>.*<|>.*&.*<'
    file_content = re.sub(p, _rep, file_content)

    return file_content


if __name__ == '__main__':
    # 从命令行参数读取 xml 或者 xml 目录路径
    if len(sys.argv) < 2:
        print('[-] 请指定 xml 文件或者 xml 目录路径')
        sys.exit(1)

    xml_path = Path(sys.argv[1])
    if not xml_path.exists():
        print(f'[-] "{sys.argv[1]}" 不是有效的路径')
        sys.exit(1)

    # 处理 xml 文件
    if xml_path.is_file():
        file_content = replace_xml_special_characters(xml_path.read_text())
        xml_path.write_text(file_content)

    # 处理 xml 目录
    elif xml_path.is_dir():
        for sub_xml_path in xml_path.glob("*.xml"):
            file_content = replace_xml_special_characters(sub_xml_path.read_text())
            sub_xml_path.write_text(file_content)

    else:
        print(f'[-] "{sys.argv[1]}" 不是有效的路径')
        sys.exit(1)
```