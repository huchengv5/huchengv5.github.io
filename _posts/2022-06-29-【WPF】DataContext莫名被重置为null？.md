---
title: "FrameworkElement.DataContext莫名被重置为null"
author: 胡承
date: 2022-06-29 09:10:3 +0800
CreateTime: 2022-06-29 09:10:3 +0800
categories: C# WPF
---

当我们在开发自定义的组件的时候，有没有遇到过这种场景，我们明明没有给`FrameworkElement.DataContext`属性赋`null`值，为什么他变成`null`了？

<!-- more -->
出现这种问题，可能有以下几种场景：
1. 调试的对象不是同一个对象
1. 异步代码出现不按常规顺序赋值
1. 数据绑定，更改了其数值
1. 数据模版或者样式应用，导致数据被重置

前面几种很好理解，调试的过程中也比较容易发现。这次就简单的来说下第四种场景。

先来看个堆栈：
![](https://s2.loli.net/2022/06/29/NhIY4AWrDcEUKjC.jpg)

从堆栈信息里面，我们可以发现，`MeasureCore`里面调用了`ApplyTemplate`的代码。

这个是`ContentPresenter`的部分源代码：
```code
        private void EnsureTemplate()
        {
            DataTemplate oldTemplate = Template;
            DataTemplate newTemplate = null;

            for (_templateIsCurrent = false; !_templateIsCurrent; )
            {                
                _templateIsCurrent = true;
                newTemplate = ChooseTemplate();
                
                if (oldTemplate != newTemplate)
                {
                    Template = null;
                }

                if (newTemplate != UIElementContentTemplate)
                {                    
                    this.DataContext = Content;
                }
                else
                {
                    this.ClearValue(DataContextProperty);
                }
            }

            Template = newTemplate;
            if (oldTemplate == newTemplate)
            {
                StyleHelper.DoTemplateInvalidations(this, oldTemplate);
            }
        }
```

在首次应用模版的时候，DataContext会被重新赋值。所以就会出现不符合预期的现象。

解决这个问题的办法也很简单，就是在FrameworkElement.ApplyTemplate后，再给DataContext赋值就可以了。

![承哥技术交流小作坊](https://i.loli.net/2021/09/27/FmsaLU1Oo7tX8kl.jpg)