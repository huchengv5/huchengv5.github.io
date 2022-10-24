---
title: "WebView2的踩坑记"
author: 胡承
date: 2022-10-08 09:12:3 +0800
CreateTime: 2022-10-08 09:12:3 +0800
categories: C# WPF
---

对于桌面应用，用来显示Web页面的方式有很多种。比如说：WebBrowser、Cef Sharp、NWJS、WebView2等。CEF应该是最常用的一种页面嵌入方案。

<!-- more -->

微软原来推出过WebView，实在是过于拉跨，于是现在有了WebView2。

WebView2已经在项目上用上了，这里记录下它目前存在的几个坑。

### 一、Win7系统下，兼容性不够好

1. Win7系统下，不支持背景透明，背景设置透明会抛出异常。
1. Win7系统下，不支持父级窗口透明，否则会出现页面渲染不出来的问题。
1. 反馈页面：https://github.com/MicrosoftEdge/WebView2Feedback/issues/2782

### 二、以管理员权限运行
1. 无法打开上传对话框


### 三、WebView2的层级问题
1. WebView2的层级是置顶的，到导致内嵌WebView2只能在最顶层

未完待续~

**欢迎转载分享，如若转载，请标注署名。**

博客链接1：https://huchengv5.gitee.io/

博客链接2：https://huchengv5.github.io/

微信公众号：

![承哥技术交流小作坊](https://i.loli.net/2021/09/27/FmsaLU1Oo7tX8kl.jpg)