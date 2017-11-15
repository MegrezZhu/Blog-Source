---
title: 提取Windows Spotlight提供的漂亮壁纸
date: 2017-11-11 10:53:10
tags:
	- Windows Spotlight
	- Node.js
---

## Windows Spotlight

Windows Spotlight（在中文系统里叫Windows聚焦）是一个系统集成的Feature，可以在锁屏界面上显示不同的背景图像并且定期自动更新。与Bing首页的背景图片差不多，Windows Spotlight提供的壁纸的质量都比较高，因此我们经常会希望能把这些壁纸保存下来用作桌面壁纸。

<!-- more -->

![](1.jpg)

---

## Naive的做法

在Google上找了一些解决方案，其基本做法都是到这个路径中去提取：

`%localappdata%/Packages/Microsoft.Windows.ContentDeliveryManager_cw5n1h2txyewy/LocalState/Assets`

![](2.jpg)

可以看到，在这个文件夹下的文件都是没有后缀名的，从文件大小上来看也不难发现仅有一部分的文件是图片。至于如何区分呢？一种比较Naive的方法是通过类似于`ren *.* *.jpg`这样的命令批量增加后缀名之后用肉眼把图片筛选出来。

但这样还是比较麻烦的，毕竟Windows Spotlight里面的资源图片是不定期更新的，如果每次都要走这个流程大概也没几个人能坚持下来吧。（魔方这个应用似乎是有一键提取图片的功能，但也是需要经常性地去使用这个功能，稍稍麻烦）

## Spotlight Watcher！

所以基于这样的需求，我用Node.js写了一个工具，包含这几样功能：

* 定期触发（能够注册成一个系统服务，开机自动运行）


* 自动到Windows Spotlight的资源目录下面扫描，筛选出横屏的图片（每张壁纸都有提供横屏/竖屏两种状态）并保存到指定目录下
* 过滤重复的图片

源代码放在GitHub：[Here](https://github.com/MegrezZhu/spotlight-watcher)

也发布到了NPM：[Here](https://www.npmjs.com/package/spotlight-watcher)

### 安装

前置要求：

* [Node.js](https://nodejs.org/en/) >= 8.0.0
* Windows 10 1607 或更高（才能找到Windows Spotlight）

然后只需要在命令行输入`npm i -g spotlight`即可。

### 配置与使用

通过`spotlight config`进行初始化设置：

![](3.jpg)

然后就可以通过`spotlight update`手动触发一次提取，当然更推荐用`spotlight install`直接注册一个系统服务自动运行，以后只需要时不时去config时设置的目标文件夹就能收获一堆新壁纸了。

当然也可以直接把系统的壁纸幻灯片文件夹设为目标文件夹，更省心。

如果不想使用了，可以通过`spotlight uninstall`把之前安装的服务卸载掉。

---

![](4.jpg)



