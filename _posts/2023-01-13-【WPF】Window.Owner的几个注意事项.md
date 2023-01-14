---
title: "Window.Owner的几个注意事项"
author: 胡承
date: 2023-01-13 09:12:3 +0800
CreateTime: 2023-01-13 09:12:3 +0800
categories: C# WPF
---

Window.Owner的几个注意事项！

<!-- more -->

1. Owner不能设置自己，否则会引发异常
2. ShowDialog的话，关闭的时候，需要将Owner设置为null，否则会出现闪一下的问题。
3. 关闭父级窗口时候，子窗口也同步关闭，但不会触发Closeing事件

**欢迎转载分享，如若转载，请标注署名。**

博客链接2：https://huchengv5.github.io/

微信公众号：

![承哥技术交流小作坊](https://i.loli.net/2021/09/27/FmsaLU1Oo7tX8kl.jpg)