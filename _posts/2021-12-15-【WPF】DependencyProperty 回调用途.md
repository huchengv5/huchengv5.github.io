---
title: "【WPF】DependencyProperty 回调用途"
author: 胡承
date: 2021-12-15 09:10:3 +0800
CreateTime: 2021-12-15 09:10:3 +0800
categories: C# WPF
---

当我们在定义一个依赖属性的时候，通常是需要重写属性元数据，以便做我们想做的工作。

<!-- more -->

声明一个依赖属性的示例：

```cs
    public static readonly DependencyProperty MyProperty = DependencyProperty.Register(nameof(My), typeof(string), typeof(MyControl), new PropertyMetadata(null,     
    PropertyChangedCallback, OnCoerceValueCallback)，ValidateValueCallback);
```

`PropertyChangedCallback` 表示属性变更后的回调，通常用于属性值被改变后，执行对应的业务逻辑。
`CoerceValueCallback` 表示属性变更前的回调，通常用来做数据类型转换。（当然也可以做些其它的事情）
`ValidateValueCallback` 表示数据验证回调，通常用来验证数据的有效性。

调用顺序：`ValidateValueCallback` -> `OnCoerceValueCallback` -> `PropertyChangedCallback`。

针对 `CoerceValueCallback` 举个例子：

我们通常在`xaml`代码中，会给`Image.Source`赋个值，并且是纯`字符串`。但是，我们的`Image`控件都能很好的把字符串转成我们要到对象`ImageSource`。转换的过程，总不可能把这个工作交给`xaml`解析器吧，那不然这个解析器，还不得炸锅了~

所以呢，依赖属性元数据里面，使用 `CoerceValueCallback`，可以帮助我们在数据改变之前，做一些想做的操作，通常可以用它来做一些数据类型转换。

欢迎转载分享，请关注微信公众号，将同步更新博客，方便查看！

![承哥技术交流小作坊](https://i.loli.net/2021/09/27/FmsaLU1Oo7tX8kl.jpg)