---
title: "Inno Setup基本用法"
author: 胡承
date: 2020-07-07 21:35:3 +0800
CreateTime: 2020-07-07 21:35:3 +0800
categories: C# Setup Inno
---

Inno Setup 是个简单又强大的打包工具，能通过向导方式生成安装脚本，也可通过自定义编写脚本的方法增加强调的功能。今天我们就先讲讲，通过向导的方式来进行打包。

<!-- more -->

通过向导来生成安装脚本

![](http://image.acmx.xyz/hucheng%2F202077152313690.jpg)

开始制作安装脚本

![](http://image.acmx.xyz/hucheng%2F2020771523327422.jpg)

设置软件基本信息，包含：软件名称，版本号，公司名称，公司站点

![](http://image.acmx.xyz/hucheng%2F2020771524264734.jpg)

设置安装的默认目录位置，是否需要自定义目录。默认是在`Program Files`目录下

![](http://image.acmx.xyz/hucheng%2F2020771525194896.jpg)

选择需要添加到安装包的文件列表，并指定启动文件路径

![](http://image.acmx.xyz/hucheng%2F202077152610561.jpg)

创建快捷方式

![](http://image.acmx.xyz/hucheng%2F202077152794782.jpg)

添加许可条例内容

![](http://image.acmx.xyz/hucheng%2F2020771527459325.jpg)

设置以什么权限运行安装脚本，默认以管理员权限运行

![](http://image.acmx.xyz/hucheng%2F202077152825553.jpg)

选择语言，可惜就是`没有`简体中文

![](http://image.acmx.xyz/hucheng%2F2020771529555633.jpg)

设置输出目录

![](http://image.acmx.xyz/hucheng%2F2020771530334423.jpg)

使用预定义宏，开启就好

![](http://image.acmx.xyz/hucheng%2F202077153118804.jpg)

向导完成，可以进入到脚本页面了，这个时候如果需要更加强大的功能，可以自己修改脚本文件

![](http://image.acmx.xyz/hucheng%2F2020771531447778.jpg)


根据向导，我们就生成好了安装脚本。我们只需要点击菜单栏上的`编译[Build -> Compile]`进行打包，通过`Run`来设定安装还是卸载来执行安装或者卸载操作。

Inno Setup 提供了丰富的帮助文档，可以在`Help`菜单中查看。

通过向导的方式进行打包非常简单，大家只需要根据向导的步骤，一步一步操作就可以完成打包了。如果针对一些额外的操作，如写注册表，环境变量，安装服务等，这个时候我们可以通过修改脚本来实现。

这里我简单列举几个项给予说明：

[Languages]
表示安装界面要显示的语言。

[Tasks]
表示安装过程中，要执行的操作。如桌面快捷方式，开始菜单栏等。

[Files]
需要添加到安装包中的文件集合

[Run]
表示安装的过程中，需要执行的操作。这里可以执行我们自己编写的安装脚本或者程序。

[UninstallRun]
表示卸载时执行。这里可以执行我们自己编写的卸载脚本或者程序。