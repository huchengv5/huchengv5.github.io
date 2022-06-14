---
title: "git用ssh的方式进行连接"
author: 胡承
date: 2022-06-14 09:10:3 +0800
CreateTime: 2022-06-14 09:10:3 +0800
categories: C# WPF
---
git做代码管理用Http的方式有诸多限制，用SSH的方式就比较省事。


<!-- more -->
## git如何开启SSH？

方法很简单，关键的命令：`ssh-keygen -t rsa -C “配置自己邮箱”`

1. 通过git brash ，打开 git 命令行工具，输入以上命令，一路回车。工具会在当前用户下创建三个ssh的授权文件，分别是：`id_rsa`，`id_rsa.pub`，`known_hosts`。
1. pub结尾是公钥，我们用记事本打开，并复制里面的内容。
1. 在`gitlab`中，我的头像那儿点开，找到`Settings`，再找到`SSH Keys`选项卡，将刚才生成的公钥粘贴进去。
1. 接下来就可以直接用ssh的方式进行拉取推送了


内容比较简单，纯粹做个备忘！

![承哥技术交流小作坊](https://i.loli.net/2021/09/27/FmsaLU1Oo7tX8kl.jpg)