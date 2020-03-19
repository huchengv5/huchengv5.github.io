---
title: "WPF迁移至WPF Core 之 生成操作"
author: 胡承
date: 2020-03-19 15:35:3 +0800
CreateTime: 2020-03-19 15:35:3 +0800
categories: C# .net Core
---

.net core 3.x开始，已经支持WPF了。我们可以将原有的小型项目迁移到.net core平台了。迁移过程中可能会遇到各种问题，本次我们只分享资源文件可能遇到的问题。

<!-- more -->

.net framework和.net core之间还是存在一些差异的，最明显的就是csproj文件了。在.net core里面，我们能发现csproj文件变得清爽了很多。

为什么会变得如此清爽？因为繁杂的配置都丢`<Project Sdk="Microsoft.NET.Sdk.WindowsDesktop">`里面了。

原理参见：<https://blog.lindexi.com/post/WPF-%E8%AE%B2%E8%AE%B2-Microsoft.NET.Sdk.WindowsDesktop-%E7%9A%84%E5%8E%9F%E7%90%86.html>。

包括`AssemblyInfo.cs`文件也变得非常简单了，因为它部分内容也放到公共部分了。（这些本次不讨论）

实际上迁移过程中，绝大部分的文件，我们都可以直接迁移过来，当我们在编译的时候，就会发现有些编译不通过，比较常见的问题就是类找不到，通常只需要寻找nuget包即可解决，vs会有提示。

我们也会遇到一些资源文件迁移后，会出现资源找不到的情况，如下图所示：


其实原因很简单，因为文件放到wpf core里面，默认生成操作是`无`，只需要将资源文件的生成操作改成`资源`即可。
**改完以后切记要重新编译，不然会导致修改无效的假象**

就结束了？这也太少了吧！好吧，再来回忆下生成操作！
生成操作是Visual Studio针对不同的文件类型所做的处理方式。如：编译，资源，内容，嵌入的资源等等。有些操作是针对整个.net平台适用，有些是针对WPF适用，则有些是针对WF适用。

可选项：
![](http://image.acmx.xyz/hc%2F20203191555353138.jpg)

具体说明：

![](http://image.acmx.xyz/hc%2F20203191556451552.jpg)
![](http://image.acmx.xyz/hc%2F2020319155712577.jpg)