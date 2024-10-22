---
title: "适用于.NET的FFMpeg.AutoGen简介"
author: 胡承
date: 2024-10-18 09:12:3 +0800
CreateTime: 2024-10-18 09:12:3 +0800
categories: C# WPF
---

ffmpeg在音视频领域还是有种非常重要的作用，主要用于对音视频的编解码。

如果我们需要处理一些音频，视频文件的话，那很有可能需要用到它了。
<!-- more -->
ffmpeg在音视频领域还是有种非常重要的作用，主要用于对音视频的编解码。  
如果我们需要处理一些音频，视频文件的话，那很有可能需要用到它了。  
FFMpeg.AutoGen是基于LGPL的开源协议，可以为C#语言，提供与C++等价的编码体验，其封装的ffmpeg相关的api的命名规范也完全照搬了C++相关的Api。

这大大降低了api的使用难度和学习成本。  
优势：提供了丰富的API，使用起来灵活方便，功能强大。  
劣势：需要对ffmpeg相关的api有所了解，并且能熟练使用C# unsafe 语法的使用。

FFMpeg.AutoGen项目地址：Ruslan-B/FFmpeg.AutoGen: FFmpeg auto generated unsafe bindings for C#/.NET and Core (Linux, MacOS and Mono). (github.com)。

FFMpeg.AutoGen 提供了不同版本的nuget包，如：6.1.0，5.1.2.3，该主体版本号是对应ffmpeg的版本号。

所以我们在开发的过程中，需要安装与ffmpeg版本对应的FFMpeg.AutoGen nuget包（ffmpeg版本也需要考虑x86，x64的平台相关性），这点非常重要，否则会导致ffmpeg的相关api无法正常调用。

下期我们来讲讲，如何通过使用FFMpeg.AutoGen，实现mp4的视频播放功能。


博客地址：https://huchengv5.github.io/

微信公众号：

![承哥技术交流小作坊](https://i.loli.net/2021/09/27/FmsaLU1Oo7tX8kl.jpg)

**欢迎转载分享，如若转载，请标注署名。**

