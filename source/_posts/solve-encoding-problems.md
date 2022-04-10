---
title: 解决俄语软件乱码问题（不影响系统）
date: 2021-09-17 20:14:52
categories:
tags:
---

新学期，新课程，遇到了新的问题

<!-- more -->



在使用乌克兰老师提供的软件时，部分**俄语无法显示**，出现乱码：

{% asset_img Snipaste_2021-09-17_14-49-35.png %}



更改 Windows **地区**和**区域格式**后问题仍然**没有解决**

经过一番折腾，终于解决了这个问题。而且这个方法只会改变软件的编码，**不会影响系统编码**



需要借助第三方软件 **Locale Emulator** （这貌似是游戏党为了解决游戏编码问题开发的）



现在总结一下整个操作步骤，备忘

### 1. 在 [这里](https://pc.qq.com/detail/1/detail_24061.html) 下载软件，解压后运行 **LEInstaller.exe**

{% asset_img Snipaste_2021-09-17_14-57-09.png %}

### 2. 选择 Install for all users

{% asset_img Snipaste_2021-09-17_14-59-29.png %}

如果安装成功会出现以下弹窗，选择**确定**

{% asset_img Snipaste_2021-09-17_15-00-46.png %}

### 3. 右击需要运行的软件，选择 Locale Emulator -> 修改此程序的配置

{% asset_img Snipaste_2021-09-17_15-02-17.png %}



### 4. 在弹出的窗口中修改预置配置为俄语后，勾选以 管理员身份运行 ，点击左上角的保存

{% asset_img Snipaste_2021-09-17_15-50-10.png %}





### 5.如果一切顺利的话，软件现在已经正常运行了

{% asset_img Snipaste_2021-09-17_15-08-46.png %}



而且原来的文件夹中会出现一个 **KSS.exe.le.config** 配置文件

{% asset_img Snipaste_2021-09-17_15-09-02.png %}



### 注意：以后启动软件时，要 右击 -> Locale Emulator -> 以此程序配置运行

{% asset_img Snipaste_2021-09-17_15-53-19.png %}





### Enjoy yourself！



