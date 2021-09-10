---
title: "如何在Canvas上实现元素按中心点坐标作为相对坐标"
author: 胡承
date: 2021-09-10 11:10:3 +0800
CreateTime: 2021-09-10 11:10:3 +0800
categories: C# WPF
---

这个问题，相信大部分同学都知道，实现这个效果，只需要对`Canvas`上的子元素进行坐标换算。

<!-- more -->

先看下常规的做法，可能是：

```cs
//frameworkElement
Canvas.SetLeft(frameworkElement,left - frameworkElement.ActualWidth/2);
Canvas.SetTop(frameworkElement,top - frameworkElement.ActualHeight/2);
```

但是这种方法，虽然可以实现，但是它不太好放到`xaml`上去实现，也没有那么方便。

当然微软也会遇到这样的一个问题，所以他们偷偷的实现了一种方法来实现这种逻辑，但是：`就不告诉你`。

没关系，微软不告诉你，我告诉你。请看具体实现：

前台`xaml`演示代码

```xml
    <Canvas>
        <Rectangle Height="100" Width="150" Fill="Red" Margin="-100000"/>
    </Canvas>
```

显示效果：

![效果图](http://)

聪明的小伙伴可以发现，其实实现这个功能非常简单，只需要在`Margin`上设置一下属性值为`-100000`即可。

是不是很黑科技呢？