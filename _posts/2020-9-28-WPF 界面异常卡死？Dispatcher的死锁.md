---
title: "WPF 界面异常卡死？Dispatcher的死锁"
author: 胡承
date: 2020-09-28 14:35:3 +0800
CreateTime: 2020-09-28 14:35:3 +0800
categories: C# .NET WPF
---

前面我们有提到过多线程UI，不知道小伙伴们在用的时候有没有遇到过界面卡死的情况？或者在做其他业务的时候，使用过`Dispatcher.Invoke`导致卡死？

<!-- more -->

针对上面的问题，我们来举个导致卡死的例子！

就以我们此前写的多线程UI为例，贴上部分代码：

```cs
            //启动主线程以外的UI线程
            DispatcherUIThread dispatcherUIThread = new DispatcherUIThread();
            dispatcherUIThread.Start();
            //调度到UI线程上执行某个操作
            dispatcherUIThread.Dispatcher.Invoke(() =>
            {
                int j = 00;
                //再次调度到主线程，执行某个操作
                Dispatcher.Invoke(() =>
                {
                    int i = 0;
                });
            });
```

在程序启动的时候，我们调用上述代码，即可导致窗体卡死。

## 为什么会导致卡死？

这个问题的原因还有点复杂，为了简化描述，这里不会讲的太详细，大致讲下思路。

我们先来看下整个调用过程：

主线程 -> UI线程 -> 主线程。

以上代码会涉及到两个线程，并且会做线程切换。

WPF的线程调度中，有个上下文同步对象 `DispatcherSynchronizationContext`，继承自`SynchronizationContext`。

当我们在调用`Dispatcher.Invoke`的时候，代码内部首先会做上下文切换（注意，这个上下文切换是WPF内部实现的）。

这里贴上部分代码：

```cs

                SynchronizationContext oldSynchronizationContext = SynchronizationContext.Current;

                try
                {
                    DispatcherSynchronizationContext newSynchronizationContext;
                    if(BaseCompatibilityPreferences.GetReuseDispatcherSynchronizationContextInstance())
                    {
                        newSynchronizationContext = _defaultDispatcherSynchronizationContext;
                    }
                    else
                    {
                        if(BaseCompatibilityPreferences.GetFlowDispatcherSynchronizationContextPriority())
                        {
                            newSynchronizationContext = new DispatcherSynchronizationContext(this, priority);
                        }
                        else
                        {
                            newSynchronizationContext = new DispatcherSynchronizationContext(this, DispatcherPriority.Normal);
                        }
                    }
                    SynchronizationContext.SetSynchronizationContext(newSynchronizationContext);

                    callback();
                    return;
                }
                finally
                {
                    SynchronizationContext.SetSynchronizationContext(oldSynchronizationContext);
                }

```

切换完成以后，开始执行`InvokeImpl`，

`InvokeImpl`执行过程中，将会把`Dispatcher`对象以`DispatcherOperation`对象封送过去

并通过异步方式`InvokeAsyncImpl`将委托加入到执行队列中。

如下所示：

```cs
//operation 为 DispatcherOperation 对象。
operation._item = this._queue.Enqueue(operation.Priority, operation);

//省略…
```
封送完成后，接下来将会等待操作执行完成

```cs

operation.Wait();

```

走到这里，代码` int j = 00;`已经可以执行了。

接下来，我们发现又出现了一个`Dispatcher`,按照上面的步骤，再执行一遍。

最后，主线程和UI线程均处于 Wait() 状态。也就是说，主线程在等UI线程返回结果，UI线程再等主线程返回结果。于是就一直等下去，然后就没有然后了……

## 如何解决？


要解决这个问题，我们有两种方法：

1、避免递归出现上下文切换。

2、如果一定要这么调用，那么就需要在Invoke的地方修改成异步调用方法。

修改上述代码，再次执行：

```cs
            //启动主线程以外的UI线程
            DispatcherUIThread dispatcherUIThread = new DispatcherUIThread();
            dispatcherUIThread.Start();
            //调度到UI线程上执行某个操作
            dispatcherUIThread.Dispatcher.Invoke(() =>
            {
                int j = 00;
                //再次调度到主线程，执行某个操作
                //此处修改为异步队列，异步队列是不需要等待返回，那么函数会先返回到原有的上下文以后，再执行异步队列中的函数
                Dispatcher.InvokeAsync(() =>
                {
                    int i = 0;
                });
            });
```

程序可正常启动。