---
title: "PropertyPath的表示形式"
author: 胡承
date: 2022-06-10 09:10:3 +0800
CreateTime: 2022-06-10 09:10:3 +0800
categories: C# WPF
---

WPF的PropertyPath设计虽然灵活，但是不得不说，真的很不好用！今天就纯粹做下备忘~

<!-- more -->

`PropertyPath`一般我们在写动画或者绑定的时候会用的比较多，因为它可以接受一个字符串的参数，所以我们可以给它指定任意的字符串。

也正因为如此，所以它的输入合不合法，我们就不好去做检查了！

再此，例举几种常用的方法示例:

1. `(UIElement.RenderTransform).(TransformGroup.Children)[0].(TranslateTransform.X)`
1. `(UIElement.RenderTransform).(TranslateTransform.X)`


![承哥技术交流小作坊](https://i.loli.net/2021/09/27/FmsaLU1Oo7tX8kl.jpg)