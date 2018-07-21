---
layout:     post
title:      Wechat Auto-chat Robot
subtitle:   When you are busy...
date:       2018-07-21
author:     CodingMrWang
header-img: img/post-bg-robot.jpg
catalog: true
tags:
    - Python
    - Robot
    - Wechat
---


> This post is first created by [CodingMrWang](http://codingmrwang.github.io), 作者 [@Zexian Wang](http://github.com/codingmrwang) ,please keep the original link if you want to repost it.

# WechatRobot

Let's create a wechat robot to auto reply some intelligent words when you are busy.

Firstly, install itchat package

```
sudo pip install itchat
```

# Create a Tuling Robot

Go to [Tuling Offical Website](http://www.tuling123.com/).

Set up an account and login in. Then you can create your own robot, select the character you want the robot be and set the age and name. 
After that, enter the robot and get your Api key.

![Api key](https://ws3.sinaimg.cn/large/006tNc79ly1ftfh0xtfegj317s11qtct.jpg)

Also, get your user id on the top right corner.

Replace Api key and user id in the code:

```python
"userInfo": {
     "apiKey": "yourapikey",
     "userId": "youruserid"
}
```
Finally, run

```
python path/to/WecharRobot.py
```

Code:

```python
# -*- coding=utf-8 -*-
import requests
import itchat
import pdb
import json


def get_response(msg):
    apiUrl = 'http://openapi.tuling123.com/openapi/api/v2'

    data = {
	"reqType":0,
    "perception": {
        "inputText": {
            "text": msg
        },
        "inputImage": {
            "url": "imageUrl"
        },
    },
    "userInfo": {
        "apiKey": "yourapikey",
        "userId": "youruserid"
    }
}

    try:
        r = requests.post(url=apiUrl, json=data)
        body = json.loads(r.text)
        return body.get('results', {})[0].get('values', {}).get('text')
    except:
        return u'你好，我在忙，稍后回复'

@itchat.msg_register(itchat.content.TEXT)
def tuling_reply(msg):
    defaultReply = 'I received: ' + msg['Text']
    reply = get_response(msg['Text'])
    return reply or defaultReply


itchat.auto_login()
itchat.run()
```


It will pop up a QR code, scan the code, 10 seconds later, your robot will start work.

This is a sample, it is really intelligent, if you want a more intelligent one, you can upgrade your robot. 
This robot can also help you search all questions like weather, time and so on.

![Sample](https://ws4.sinaimg.cn/large/006tNc79ly1ftfh9w5oe2j30u01hcn0u.jpg)

Enjoy your wechat Robot
