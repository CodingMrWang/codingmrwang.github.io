---
layout:     post
title:      Mobile packages spider with mitmproxy
subtitle:   Powerful crawl tool mitmproxy
date:       2018-07-13
author:     CodingMrWang
header-img: img/post-bg-swift.jpg
catalog: true
tags:
    - Python
    - Mitmproxy
    - IOS
    - Mobile
    - Spider
---


> This post is first created by [CodingMrWang](http://codingmrwang.github.io), 作者 [@Zexian Wang](http://github.com/codingmrwang) ,please keep the original link if you want to repost it.

### Mitmproxy

- Mitmproxy is a packet capture program that supports HTTP and HTTPS.

***related links***

* Offical website：[https://mitmproxy.org](https://mitmproxy.org)
* GitHub: [https://github.com/mitmproxy/mitmproxy](https://github.com/mitmproxy/mitmproxy)
* Offical document：[http://docs.mitmproxy.org](http://docs.mitmproxy.org)
* download address：[https://github.com/mitmproxy/mitmproxy/releases](https://github.com/mitmproxy/mitmproxy/releases)

### Install with pip

The easiest way to install mitmproxy is to install it with pip, you can directly install with the following Command
```
sudo pip3 install mitmproxy
```

### Certificate configuration

If you want to catch HTTPS requests, you have to configure the certificates, after you install mitmproxy, it will create some certificates for you, only if you trust the certificates, you could catch HTTPS requests, or you cannot catch them.

Fristly, create the certificates:
```
mitmdump
```
Under the .mitmproxy directory, you can find six certificates.

![.mitmproxy](https://ws3.sinaimg.cn/large/006tNc79ly1ft8e3sbc5uj30zy0hc76q.jpg)

Double click mitmproxy-ca-cert.pem, then find mitmproxy Certificate, open the settings and set always trust.

![certificate](https://ws3.sinaimg.cn/large/006tNc79ly1ft8e79vezbj31cs0xatj8.jpg)


### IOS:

use your Iphone connect to the same wifi with your computer, open the setting of wifi, switch http proxies to manual, then enter your computer's ip address and port number 8080. After that, open your brower on your Iphone, go to mitm.it, click the apple icon and download the certificate, then trust it.

![phone](https://ws1.sinaimg.cn/large/006tNc79ly1ft8e9zi5m5j30u01hcmz8.jpg)


After that, run the mitmproxy:

```
mitmproxy
```
Then you can see all requests to your Iphone would be shown on your terminal.

![mitmproxy](https://ws4.sinaimg.cn/large/006tNc79ly1ft8ebd63ylj31kw0y11cr.jpg)
Some Hot Keys:

* q: quit
* z: clean the screen
* f: filter

### Handle packages with self designed python code
Besides that, you can catch all requests and handle them with python code:

```python
# -*- coding=utf-8 -*-
import os
import hashlib
import re
import MySQLdb

def response(flow):
    request = flow.request
    response = flow.response

    # feeds data
    if 'xxx' in request.url:
        file_name = re.findall(r'\d+', request.url)[0]
        try:
            insertIntodb(**kwargs)
        except Exception as e:
            print(e)

def insertIntodb(**kwargs):
    db = MySQLdb.connect(
        host = 'xx.xx.xx.xx',
        user = 'xxx',
        passwd = 'xxxx',
        db = 'xxx',
        port = xxx,
        charset='utf8')
    cursor = db.cursor()
    # SQL 插入语句
    sql = "Your sql command"
    try:
       # 执行sql语句
       cursor.execute(sql)
       # 提交到数据库执行
       db.commit()
    except Exception as e:
       # Rollback in case there is any error
        print (e)
        print('db error')
        db.rollback()
    # 关闭数据库连接
    db.close()
```
After finishing your python code, run:

```
mitmdump -s /path/to/yourcode.py
```
