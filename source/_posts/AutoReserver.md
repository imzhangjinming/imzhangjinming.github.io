---
title: AutoReserver体育场地自助预约软件
date: 2022-01-23 22:28:31
categories:
- code
tags:
---

2022年寒假使用Python开发的一款小工具

<!--more-->

# AutoReserver 自助预约软件

## 1. AutoReserver 是何方神圣？

AutoReserver 是使用python语言开发的自助预约小软件，用户通过简单的设置即可定时预约体育场地。

{% asset_img Snipaste_2022-01-15_18-14-26.png %}



## 2. 为什么要开发它？

* 为了磨练和提高技术，把学习到的知识应用到实践中

* 为了用户能在懒觉后享受体育运动的快乐:grin:



## 3. 软件的基本结构

{% asset_img IMG_FF36705D89AC-1.jpeg %}

小工具采用客户端和服务端结合的基本结构。由客户端采集用户预约任务信息并以加密文本的形式上传到个人云服务器，服务器最终执行预约动作。



## 4. 技术要点总结

### 4.1 客户端

### 4.1.1 使用 PyQt6 设计 GUI 界面

python 貌似没有原生的GUI设计软件，所以我使用了 Qt Creator 搭配 PyQt6 设计GUI。

首先创建 Qt Creator 项目并可视化地进行界面设计

{% asset_img qtCreator.png %}

创建完成后，文件夹中会出现以 `.ui` 为后缀的文件，这个文件包含了界面的所有信息。但是python没办法直接使用 `.ui` 文件，我们需要用 PyQt 包中的 `pyuic6` 工具将 `.ui` 文件转换成对应的 `.py` 文件，命令如下

```shell
pyuic6 -o ./ui_ReserveForm.py ./FirstQtProject/reserveForm.ui
```

到这里界面就生成好了



### 4.1.2 QThread 解决界面卡死问题

PyQt 是单线程运行的，所以在执行耗时任务时界面会处于卡死状态，非常影响使用体验。

解决办法是将耗时任务放在子线程中执行，主线程负责更新界面信息。

具体代码：

```python
class LoginThread(QtCore.QThread):
    loginResult = QtCore.pyqtSignal(bool,str)

    def __init__(self,name,password,parent=None):
        logger.debug('creating instance of LoginThread')
        # 初始化函数
        super(QtCore.QThread, self).__init__()    
        self.name = name
        self.password = password

    def run(self):
        try:
            isLoginSuccess,raw_json_str = new_yuyue_client.login(self.name,self.password)
        except Exception as e:
            logger.exception('LoginThread.run exception')
            self.loginResult.emit(False,'')
        else:
            logger.debug('LoginThread.run return')
            self.loginResult.emit(isLoginSuccess,raw_json_str)
```



### 4.1.3 客户端登录 selenium 设置

```python
options = ChromeOptions()
options.headless = True # 无头模式，不显示浏览器界面
prefs = {"profile.managed_default_content_settings.images": 2,
         "intl.accept_languages":"en"} # 不加载图片，提高速度
options.add_experimental_option("prefs", prefs)
options.add_experimental_option('excludeSwitches', ['enable-automation'])
options.add_experimental_option('useAutomationExtension', False)
options.add_argument("disable-blink-features=AutomationControlled")#就是这一行告诉chrome去掉了webdriver痕迹
options.add_argument("--incognito") # 无痕模式
```



## 4.2 服务器端

### 4.2.1 inotifywait 监视 ftp 文件夹

客户端上传预约信息后，服务器端程序需要在第一时间监测到文件变化并创建预约任务。为此使用了Linux系统命令inotifywait监视接收ftp文件的文件夹。shell 脚本如下：

```shell
#!/bin/bash
exec >>../AutoReserver.log                                                                    
exec 2>&1
source ../shell-logger/etc/shell-logger

info watchFile start

MONITOR_DIR="/var/run/vsftpd/AutoReserver"
PYTHON_PATH="/usr/bin/python3"
FILE_WATCH_DOG_PATH="/var/run/vsftpd/AutoReserver/fileWatchDog.py"
info MONITOR_DIR:$MONITOR_DIR
info PYTHON_PATH:$PYTHON_PATH
info FILE_WATCH_DOG_PATH:$FILE_WATCH_DOG_PATH

while read path action file datetime; do
    info "The file '$file' appeared in directory '$path' via '$action' @ '$datetime'"
    info executing $PYTHON_PATH $FILE_WATCH_DOG_PATH $file
    $PYTHON_PATH $FILE_WATCH_DOG_PATH $file
done < <(inotifywait  -mr --timefmt '%F_%T' --format '%w %e %f %T' -e close_write $MONITOR_DIR)
```



### 4.2.2 at 命令和 apscheduler 库结合以执行定时预约

`inotifywait` 监测到文件变化后会调用 `fileWatchDog.py` 脚本

在 `fileWatchDog.py` 脚本里读取用户上传的信息并使用 `at` 命令创建定时任务

`at` 创建的任务会在预约时间前5分钟调用脚本登录网站，等待最终预约



### 4.2.3 systemd 系统服务

服务器端的监听程序需要在后台运行，并且在程序崩溃或系统重启后重新运行，这可以用 systemd 实现

```shell
[Unit]
Description=AutoReserver daemon
After=network.target vsftpd.target
ConditionPathIsDirectory=/var/run/vsftpd/AutoReserver

[Service]
Type=simple
ExecStart=/home/programs/AutoReserver/watchFile.sh
ExecReload=/bin/kill -HUP $MAINPID
Restart=always

[Install]
WantedBy=multi-user.target
```

  

## 参考博客

* [LINUX添加自定义系统服务[Systemd]](https://www.lemonsys.cn/tech_107/)
* [[PyQt入门教程 ] PyQt5中多线程模块QThread使用方法](https://www.cnblogs.com/linyfeng/p/12239856.html)



## 参考书籍

[1]Ryan Mitchell.Python网络数据采集[M]. 北京:人民邮电出版社, 2016

[2]韦世东.Python3反爬虫原理与绕过实战[M]. 北京:人民邮电出版社, 2020

[3]王维波,栗宝鹃,张晓东.Python Qt GUI与数据可视化编程[M]. 北京:人民邮电出版社, 2019

[4]Richard Blum, Christine Bresnahan.Linux命令行与shell脚本编程大全[M]. 北京:人民邮电出版社, 2016

[5]Matt Frisbie.JavaScript高级程序设计[M]. 北京:人民邮电出版社, 2020

[6]Michael Fitzgerald.学习正则表达式[M]. 北京:人民邮电出版社, 2013

[7]结城浩.图解密码技术[M]. 北京:人民邮电出版社, 2016

[8]Eric Matthes.Python编程：从入门到实践[M]. 北京:人民邮电出版社, 2016
