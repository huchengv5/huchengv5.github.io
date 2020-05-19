---
title: "使用U盘安装Linux（CentOS）操作系统"
author: 胡承
date: 2020-05-19 20:35:3 +0800
CreateTime: 2020-05-19 20:35:3 +0800
categories: C# .net core
---

随着.NET推出了.net core以后，开始支持跨平台，拥有一台linux的服务器似乎变得很有必要。今天我们来看看，如何在我们的PC电脑上，安装Linux操作系统。

<!-- more -->
本文以Linux CentOS 7 为例!

安装过Windows操作系统的小伙伴们都知道，想要通过U盘安装Linux，首先就必须得有个启动U盘。启动U盘需要事先安装引导程序才可安装操作系统。

常用的U盘启动工具有：大白菜、老毛桃、UltraISO（软碟通）等。本文以软碟通为例！

准备工作：

1. 下载 CentOS 7 操作系统ISO镜像
1. 下载UltraISO U盘启动制作工具
1. 准备一个大于4GB的USB 2.0 U盘（3.0或以上的U盘会导致`无法检测到USB驱动器`的问题）

先下载U盘启动工具UltraISO，并打开，打开界面如下：

![](https://i.loli.net/2020/05/19/BzpdRbJFu67mwcG.jpg)

准备一个的U盘，并插入到电脑中，并将本地目录选中到`CentOS ISO`镜像

![](https://i.loli.net/2020/05/19/H4twc9xzG3fQdeN.jpg)

点击上方菜单`启动`->`写入硬盘映像`,弹出制作配置对话框：

![](https://i.loli.net/2020/05/19/yAKfCim9vxLWoDc.jpg)

写入方式选择 USB-HDD 或 USB-HDD+

点击格式化，先将U盘格式化成FAT32格式

点击写入即可完成启动盘制作，如果刻录成功，消息栏中会显示已成功刻录。

U盘制作好后，将U盘插入到需要重装系统的电脑中，进入BIOS，设置U盘启动（不同电脑BIOS设置不一样，这里省略）

通过USB引导，进入到安装界面

![](https://i.loli.net/2020/05/19/xWdJS4iIrkPKCQu.jpg)

按Tab键或者字母 e 进入编辑页面

![](https://i.loli.net/2020/05/19/UXuphPD16nejHdR.jpg)

修改USB驱动器的名称，改成我们U盘的名称。
vmlinuz initrd=initrd.img inst.stage2=hd:LABEL=CentOS\x207\x20x86_64 rd.live.check quiet

改成

vmlinuz initrd=initrd.img inst.stage2=hd:LABEL=[U盘名称，区分大小写] quiet

如果不更改U盘名称，引导系统无法找到操作系统，导致无法进行安装，U盘名称区分大小写。

如果不知道U盘名称，可以修改为：`vmlinuz initrd=initrd.img linux dd quiet`,退出该页面后，会进入选择硬盘驱动器的界面。

![](https://i.loli.net/2020/05/19/4ESxrbWIUzdKLMG.jpg)

**该示例是无法找到USB驱动器，是因为USB是3.0接口，引导程序不支持3.0接口，需要用USB2.0的U盘重新制作启动盘。切记切记！！**

设置好后，我们就可以进入linux安装程序界面了

![](https://i.loli.net/2020/05/19/6LiglJMEZDNkByr.jpg)

选择简体中文

![](https://i.loli.net/2020/05/19/rfsF4jJlbPwaYi8.jpg)

设置安装位置

![](https://i.loli.net/2020/05/19/n9fsKEwcaVFJYWq.jpg)

如果磁盘空间不足，勾选`我想让额外空间可用`，删除不需要的文件即可。

![](https://i.loli.net/2020/05/19/pVJd25OflioBvtr.jpg)

设置IP地址

![](https://i.loli.net/2020/05/19/CtdvYyXaclMkBqO.jpg)

点击“配置“按钮

![](https://i.loli.net/2020/05/19/q25NPVdny1AwusO.jpg)

设置开机的时间网卡自动启动，点击“常规”---选中“可用时自动链接到这个网络”

![](https://i.loli.net/2020/05/19/gikmFtYNlCuS6hb.jpg)

选择“IPV4 设置“，在方法（M）：中选择”手动“，然后点击”Add”按钮

![](https://i.loli.net/2020/05/19/2QdVa8u6HmkPETp.jpg)

配置好后点击完成

![](https://i.loli.net/2020/05/19/nCJXcPDpBoK7hru.jpg)

选择自动分区，此时就可以点击`开始安装`进行安装了

![](https://i.loli.net/2020/05/19/n6z8oewtGhs3OdL.jpg)

安装完成后，设置Root密码

![](https://i.loli.net/2020/05/19/gs2DnUBVAd4Pa75.jpg)

输入密码后，点击完成

![](https://i.loli.net/2020/05/19/bOcR3Bm9l4suxef.jpg)

最后重启系统即可进入linux系统了

![](https://i.loli.net/2020/05/19/3RevnJoVACspKEd.jpg)

现在就可以尽情的玩耍了

![](https://i.loli.net/2020/05/19/HRkQMAurT8YFEXK.jpg)


Linux安装首次告捷，本文到此结束！
