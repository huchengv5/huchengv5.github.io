---
title: "使用BaGet快速搭建nuget服务"
author: 胡承
date: 2023-03-07 09:12:3 +0800
CreateTime: 2023-03-07 09:12:3 +0800
categories: C# WPF
---

`BaGet`是基于`asp.net core`编写的一个轻量级的`nuget`管理服务，安装部署非常简单。

<!-- more -->
## 环境准备
1. 下载 [BaGet安装包](https://loic-sharma.github.io/BaGet/)
1. 下载 [aspnetcore-runtime](https://dotnet.microsoft.com/zh-cn/download/dotnet/thank-you/runtime-aspnetcore-3.1.32-windows-x64-installer)
1. 下载 [dotnet-runtime](https://dotnet.microsoft.com/zh-cn/download/dotnet/thank-you/runtime-3.1.32-windows-x64-installer)

## 命令终端运行（自我寄宿）
1. 解压`BaGet.zip`
1. 打开`cmd`命令行，输入`dotnet BaGet.dll`即可运行

## 以服务的方式运行
1. [使用nssm进行安装，点击查看具体操作方法](https://blog.csdn.net/liyou123456789/article/details/123094277)
1. 通过windows任务计划

    **这里推荐第一种方式**

## 相关配置

找到BaGet目录下的`appsettings.json`配置文件。

```json
{
  "ApiKey": "",
  //开启硬删除
  "PackageDeletionBehavior": "HardDelete",
  "AllowPackageOverwrites": true,

  "Database": {
    "Type": "Sqlite",
    "ConnectionString": "Data Source=baget.db"
  },

  "Storage": {
    "Type": "FileSystem",
    "Path": ""
  },

  "Search": {
    "Type": "Database"
  },

  "Mirror": {
    "Enabled": false,
    // Uncomment this to use the NuGet v2 protocol
    //"Legacy": true,
    //用于配置到visual studio中
    "PackageSource": "https://api.nuget.org/v3/index.json"
  },

  // Uncomment this to configure BaGet to listen to port 8080.
  //如果需要配置为Https,
  //"Https": {
  //      "Url": "https://*:5004",
  //      "Certificate": {
  //        "Path": "<path to .pfx file>",              （.pfx文件路径）
  //        "Password": "<certificate password>"          （证书密码）
  //          }
  //      }
  // See: https://docs.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel?view=aspnetcore-3.1#listenoptionsusehttps
  
  "Kestrel": {
    "Endpoints": {
      "Http": {
        "Url": "http://192.168.1.221:5000"
      }
    }
  },

  "Logging": {
    "IncludeScopes": false,
    "Debug": {
      "LogLevel": {
        "Default": "Warning"
      }
    },
    "Console": {
      "LogLevel": {
        "Microsoft.Hosting.Lifetime": "Information",
        "Default": "Warning"
      }
    }
  }
}

```

## 配置Visual Studio中的Nuget配置
在Visual Studio 包管理器中，添加`http://192.168.1.221:5000/v3/index.json`这样子的链接。IP地址可以根据实际部署的`IP`或者`域名`进行修改。

## 使用浏览器打开Nuget管理页面

通过浏览器，输入我们配置的Url  http://192.168.1.221:5000/，打开即可。

微信公众号：

![承哥技术交流小作坊](https://i.loli.net/2021/09/27/FmsaLU1Oo7tX8kl.jpg)

**欢迎转载分享，如若转载，请标注署名。**