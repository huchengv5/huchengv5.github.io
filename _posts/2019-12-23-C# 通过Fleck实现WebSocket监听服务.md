---
title: "C# 通过Fleck实现WebSocket监听服务"
author: 胡承
date: 2019-12-23 16:23:3 +0800
CreateTime: 2019-12-23 16:23:3 +0800
categories: .NET
---

Fleck是一个websocket开源框架，通过Fleck，我们可以很轻松的实现WebSocket服务端；  
GitHub地址：<https://github.com/statianzo/Fleck.git>。  
现在我们就用它来实现WebSocket基本通讯。
<!-- more -->
我们首先在nuget上面搜索Fleck，并安装到我们的工程中,然后贴上以下关键代码即可：


``` csharp

                var _webSocketServer = new WebSocketServer($"ws://0.0.0.0:{port}")
                {
                    RestartAfterListenError = true
                };

                _webSocketServer.Start(socket =>
                {
                    socket.OnOpen = () =>
                    {
                        //建立新的Socket请求时发生
                    };

                    socket.OnClose = () =>
                    {
                       //关闭Socket时发生
                    };

                    socket.OnMessage = data =>
                    {
                        //收到字符串数据时发生
                    };

                    socket.OnBinary = data =>
                    {
                       //收到二进制数据时发生
                    };
                });

```

我们通过以上方法，就可以轻松的实现WebSocket服务端了。