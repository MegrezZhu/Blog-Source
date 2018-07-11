---
title: TypeScript初战Chrome插件：Bilibili弹幕热度
date: 2018-06-07 00:59:28
tags:
	- Chrome
	- TypeScript
---

## TL;DR

这是一个用来在B站视频进度条上方创建显示弹幕热度的Chrome插件，以弹幕数量-时间的直方图显示，在高能处（定义为短时间内有大量弹幕出现的时间点）有明显的峰值，可以用来直观地看视频中的热点，也可以拿来作为空降（跳跃快进）的指示。

已经上架Chrome插件市场，[在这里](https://chrome.google.com/webstore/detail/danmaku/amabhfielmdcjgahcjkipdhlffejidjo)或者搜【Danmaku】就能找到。

源代码在[GitHub](https://github.com/MegrezZhu/danmaku)，欢迎Star。

![Chrome Webstore](1528304851281.png)

<!-- More -->

## 背景

其实做一个这样的插件的想法已经在脑海里盘桓好一阵了（创意来源于某Hub），直到最近才有空（摸鱼）写了出来，正好拿来作为TypeScript跟Chrome Extension的练手项目。

## 实现

插件的实现主要分为如下几个部分：

* 代码注入
* 获取av号及分P
* 获取弹幕
* 可视化
* 监控网页重载

基于[chrome-extension-typescript-starter](https://github.com/chibat/chrome-extension-typescript-starter)的脚手架。

### 比较核心的库

* TypeScript
  * 类型系统用来静态检查/代码提示的效率提升还是十分显著的，也减少了一些很蠢的bug的发生
* echarts
  * 百度家的图表库，用来生成直方图
* rxjs
  * 观察者 + 迭代器模式的实现
* jquery
  * 简易的DOM操作

### 代码注入

依靠Chrome Extension的`manifest.json`文件可以指定在bilibili的页面中执行指定的代码文件。

### 获取av号、分P

av号跟分P信息一般可从url中直接获得（形如`https://www.bilibili.com/video/avXXXXX/?p=XXXX`），但事实上B站的视频格式分很多种，目前光我见到的就有下面几种：

* 普通视频`/video/avXXXXX/?p=XXXX`
* 稍后再看`/watchlater/#/avXXXXXX/pXXXX`
* 从历史观看中进入`/video/avXXXXXX/index_XXXXX.html`
* 番剧
  * 具体的某一集`/bangumi/play/epYYYYY/`
  * 从【番剧】分区中直接点进`/bangumi/play/ssZZZZZ`

因此需要有不同的处理，特别是番剧，url中没有av号信息，需要从DOM中获取。

### 获取弹幕

B站每个视频都有一个av号，但由于每个视频有可能有多个分P，因此B站还有一个隐含的cid用以索引一个具体的视频（以及弹幕），通过分析找到了这个API：

`https://api.bilibili.com/x/player/pagelist?aid=AV_ID&jsonp=jsonp `

> 其实cid可以有多种方法获得，包括网页DOM、原网页HTML等等，但由于B站视频分类众多（普通视频、番剧、稍后再看、历史观看）且网页DOM都有不同，因此还是用这个API比较优雅

可以通过av号拿到该视频每个分P的cid以及视频长度，然后通过API

`https://comment.bilibili.com/CID.xml `

能够获取xml格式的弹幕数据，告一段落

> 弹幕数据的解析参考自[这篇博客](http://ju.outofmemory.cn/entry/146571 )

### 生成直方图

这个倒简单，处理一下弹幕数据调用echarts的API即可。

### 监控网页重载

在用户点击切换分P或者其他操作的时候，B站前端用的是HTML5的History API（也有的是hashchange），因此不能光用window.onhashchange事件来监控。而对于H5 History的监控，标准上又只有onpopstate事件，没有onpushstate（这一点是我比较困惑的，望有人解惑）。

因此只能使用折中的方案：利用插件的background script以及webNavigation权限监控所有B站选项卡的history更新（通过chrome提供的onHistoryStateUpdated事件），并且建立content_script与background_script的连接进行单点通知，触发页面中的url更新事件。