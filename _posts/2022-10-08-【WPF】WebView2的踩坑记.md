---
title: "WebView2的踩坑记"
author: 胡承
date: 2022-10-08 09:12:3 +0800
CreateTime: 2022-10-08 09:12:3 +0800
categories: C# WPF
---

对于桌面应用，用来显示Web页面的方式有很多种。比如说：WebBrowser、Cef Sharp、NWJS、WebView2等。CEF应该是最常用的一种页面嵌入方案。

<!-- more -->

微软原来推出过WebView，实在是过于拉跨，于是现在有了WebView2。

WebView2已经在项目上用上了，这里记录下它目前存在的几个坑。

### 一、Win7系统下，兼容性不够好

1. Win7系统下，不支持背景透明，背景设置透明会抛出异常。
1. Win7系统下，不支持父级窗口透明，否则会出现页面渲染不出来的问题。
1. 反馈页面：https://github.com/MicrosoftEdge/WebView2Feedback/issues/2782

### 二、以管理员权限运行
1. 无法打开上传对话框


### 三、WebView2的层级问题
1. WebView2的层级是置顶的，到导致内嵌WebView2只能在最顶层

### EnsureCoreWebView2Async 一直处于等待状态
1. 当WebView2不在视觉树上时，该方法将一直阻塞。WebView2的进程也不会创建出来。

### 因为ssl证书不可用导致页面无法访问问题

1. 注册 `CoreWebView2.ServerCertificateErrorDetected += CoreWebView2_ServerCertificateErrorDetected;`事件，在事件中，允许访问
```cs
        private void CoreWebView2_ServerCertificateErrorDetected(object sender, CoreWebView2ServerCertificateErrorDetectedEventArgs e)
        {
            //此处可以加一些条件判断
            e.Action = CoreWebView2ServerCertificateErrorAction.AlwaysAllow;
        }
```
2. 关闭智能拦截器
```cs
    //该属性默认为True，该属性只有高版本才支持
    CoreWebView2Settings.IsReputationCheckingRequired =false;
```

### 一些奇怪的异常
1. `异常来自 HRESULT:0x8007139F Microsoft.Web.WebView2.Core.Raw.ICoreWebView2Controller.MoveFocus(COREWEBVIEW2_MOVE_FOCUS_REASON reason) `

    尝试解决方法：
    ```cs
        protected override IntPtr WndProc(IntPtr hwnd, int msg, IntPtr wParam, IntPtr lParam, ref bool handled)
        {
            if (!isCoreWebView2Initialized) return IntPtr.Zero;
            return base.WndProc(hwnd, msg, wParam, lParam, ref handled);
        }
    ```
1. `(异常来自 HRESULT:0x8007139F) Microsoft.Web.WebView2.Core.Raw.ICoreWebView2Controller.set_Bounds(tagRECT Bounds) `

    尝试解决方法：
    ```cs
        protected override void OnWindowPositionChanged(Rect rcBoundingBox)
        {
            try
            {
                //如果没有初始化，就不执行操作
                // isCoreWebView2Initialized 是表示 EnsureCoreWebView2Async 方法是否已经执行完成
                if (!isCoreWebView2Initialized) return;

                //已经释放了，就不要再执行了
                if (!IsDisposed && IsLoaded)
                {
                    base.OnWindowPositionChanged(rcBoundingBox);
                }
            }
            catch (InvalidCastException ex)
            {
                if (ex.HResult == -2147467262)
                {
                    //do something
                }
            }
            catch (COMException ex2)
            {
                if (ex2.HResult == -2147019873)
                {
                    //do something
                }
            }
        }
    ```
1. `The WebView control is no longer valid because the browser process crashed.To work around this, please listen for the CoreWebView2.ProcessFailed event to explicitly manage the lifetime of the WebView2 control in the event of a browser failure.https://docs.microsoft.com/en-us/dotnet/api/microsoft.web.webview2.core.corewebview2.processfailed(异常来自 HRESULT:0x80131509) Microsoft.Web.WebView2.Wpf.WebView2.VerifyBrowserNotCrashed() `

    目前观察该问题不会有什么影响，WebView可以自行恢复。

1. `WebView2.Dispose`引发 调用线程无法访问此对象，因为另一个线程拥有该对象。
    因为WebView内部存在异步调用，要考虑线程的问题。

**欢迎转载分享，如若转载，请标注署名。**

博客链接1：https://huchengv5.gitee.io/

博客链接2：https://huchengv5.github.io/

微信公众号：

![承哥技术交流小作坊](https://i.loli.net/2021/09/27/FmsaLU1Oo7tX8kl.jpg)