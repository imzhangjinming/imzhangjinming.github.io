---
title: python 实现 Win 和 Linux 电脑剪切板共享
date: 2021-08-18 04:02:44
categories:
- code
tags:
---


{% asset_img clipboard-307332_1280.png %}

前段时间买了一块树莓派4B（花了我 1000+ 大洋），正好手头有一套闲置的外设（显示器，键盘和鼠标)，就把它当作了一台备用电脑。

平时在 Windows 电脑上查资料，看书，在树莓派上敲代码。有时遇到问题，在 Win 上查到了答案（能解决问题的代码），却没有办法直接 `ctrl C`  `ctrl V` ，非常不舒服。所以今天用 python 写了这个 共享剪切板（Shared Clipboard） 小程序。



<!-- more -->

代码很短，思路就是 每 0.1 秒检查 Win 电脑剪切板内容是否变化，如果变化，就把新内容通过 TCP 协议传到 树莓派上，树莓派上的程序负责接收并把剪切板内容更新。

目前只能让 树莓派 的剪切板内容与 Win 电脑一致，反之则不能。我也不想让 树莓派 剪切板内容的变化影响我的主力机。



下面直接上代码

服务器端（树莓派）

```python
#coding:utf-8
# python 2 默认编码不是 UTF-8 ,中文会出现乱码。所以这里需要特别设置
import sys
if sys.version[0] == '2':
    reload(sys)
    sys.setdefaultencoding("utf-8")


from socket import *
import pyperclip

serverPort = 12000
serverSocket = socket(AF_INET,SOCK_STREAM)
serverSocket.bind(('',serverPort))
serverSocket.listen(1)      # 只监听一个连接
print('The server is ready to receive')
connectionSocket,addr=serverSocket.accept()

while True:
    sentence = connectionSocket.recv(1024).decode()
    pyperclip.copy(sentence)    # 使用接收到的内容更新剪切板
    #print("clipboard content changed: %s" % str(sentence)[:20])

connectionSocket.close()
```



客户端（Windows 电脑）

```python
from socket import *
import pyperclip
import time

serverName = '192.168.0.103'    # 树莓派 IP 地址
serverPort = 12000              # 使用 12000 端口
clientSocket = socket(AF_INET,SOCK_STREAM)
clientSocket.setsockopt(SOL_SOCKET, SO_KEEPALIVE, 1) #在客户端开启心跳维护
clientSocket.connect((serverName,serverPort))

recent_value=pyperclip.paste()

# 每 0.1 秒检查一次剪切板内容是否改变，如果改变则将新内容发送至服务器端（树莓派）
while True:
    tmp_value=pyperclip.paste()
    if tmp_value!=recent_value:
        recent_value=tmp_value
        clientSocket.sendall(recent_value.encode())
        print("Value changed: %s" % str(recent_value))
    time.sleep(0.1)
```

