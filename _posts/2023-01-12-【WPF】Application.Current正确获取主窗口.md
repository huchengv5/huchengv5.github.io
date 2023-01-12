---
title: "Application.Current正确获取主窗口"
author: 胡承
date: 2023-01-12 09:12:3 +0800
CreateTime: 2023-01-12 09:12:3 +0800
categories: C# WPF
---

`Application.Current.MainWindow`用于获取主窗口，相信不少同学有这么用过。但是这种方式，可靠吗？

<!-- more -->

正常情况下，这种方式是可以获取到主窗口的。因为常规的操作，应用程序启动的时候，就会启动一个默认的MainWindow（当然我们也可以手动启动这个主窗口）。

这个时候，通过`Application.Current.MainWindow`获取这个主窗口，并不会有什么问题。

但是，如果我们的主窗口存在关闭，再打开其它的窗口，这个时候去获取主窗口就不一定是它了。

因为主窗口我们通常知道它是什么数据类型，所以我们可以通过指定窗口类型的方式来获取我们需要的主窗口。

```cs
        public static Window GetWindow<T>() where T : Window
        {
            if (Application.Current == null || Application.Current.Windows == null)
                return null;
            for (int i = 0; i < Application.Current.Windows.Count; i++)
            {
                if (Application.Current.Windows[i] is T window)
                {
                    return window;
                }
            }
            return null;
        }
```

通过上述方法，我们就可以获取指定类型的窗口了。用这种方式获取对应的窗口，相对来讲会比较稳妥。但是如果有多个相同类型的窗口，那就需要改造一下了。


**欢迎转载分享，如若转载，请标注署名。**

博客链接1：https://huchengv5.gitee.io/

博客链接2：https://huchengv5.github.io/

微信公众号：

![承哥技术交流小作坊](https://i.loli.net/2021/09/27/FmsaLU1Oo7tX8kl.jpg)