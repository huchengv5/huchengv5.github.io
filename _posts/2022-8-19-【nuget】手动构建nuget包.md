---
title: "手动构建nuget包"
author: 胡承
date: 2022-08-19 09:10:3 +0800
CreateTime: 2022-08-19 09:10:3 +0800
categories: C# WPF
---

在.net core 的版本上，visual studio 已经提供了自动打包的功能。当然，在.NET Framework的版本上也是可以的，不过需要收到修改proj文件才行，这里先不做讨论。

<!-- more -->

在Visual Studio不具备一键打包的功能时，我们可以使用第三方软件来实现nuget包的打包。

在Microsoft Store商城，搜索安装`nuget package explorer`。

![主界面](http://image.acmx.xyz/hucheng%2F20228191022514879.jpg)

创建一个新的包，`Create a new package`

![右键菜单](http://image.acmx.xyz/hucheng%2F20228191023205534.jpg)

通过右键菜单增加对应的目录和文件。


目录参考以下方式：

```txt

├─lib
│  ├─net462
│  └─netstandard1.3
└─runtimes
    ├─win-arm64
    │  └─native
    ├─win-x64
    │  └─native
    └─win-x86
        └─native

```

![左边配置栏](http://image.acmx.xyz/hucheng%2F20228191027232912.jpg)

左侧是可视化的包信息配置，按需求填写即可，依赖性也可以在这里添加。

添加好后，保存就可以了。

![承哥技术交流小作坊](https://i.loli.net/2021/09/27/FmsaLU1Oo7tX8kl.jpg)