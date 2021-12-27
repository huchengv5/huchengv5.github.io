---
title: "认识asyncawait，如何自己动手编写可等待的任务？"
author: 胡承
date: 2021-12-27 19:10:3 +0800
CreateTime: 2021-12-27 19:10:3 +0800
categories: C# WPF
---

C#里面的`async/await`语法简直好用的不要不要的。但是，如果我想让这种异步状态机，能工作在我自定义的线程里面要怎么办呢？

<!-- more -->

话不多说，先看代码。
为了简化描述，详细讲解，请看注释描述。

```cs

    /// <summary>
    /// 
    /// </summary>
    /// <typeparam name="T"></typeparam>
    public interface IAwaitable<T> : INotifyCompletion
    {
        bool IsCompleted { get; }
        T Result { get; }
        T GetResult();
        void ReportCompleted(T result);
    }

    /// <summary>
    /// 
    /// </summary>
    public interface IAwaitable : INotifyCompletion
    {
        bool IsCompleted { get; }
        void GetResult();
        void ReportCompleted();
    }

    /// <summary>
    /// 
    /// </summary>
    public interface IAwaiter
    {
        IAwaitable GetAwaiter();
    }

    /// <summary>
    /// 
    /// </summary>
    public interface IAwaiter<T>
    {
        IAwaitable<T> GetAwaiter();
    }

```

```cs
    public class AwaitableTask : IAwaiter
    {
        public AwaitableTask()
        {
            AwaitableObject = new AwaitableObject();
        }

        public IAwaitable GetAwaiter()
        {
            return AwaitableObject;
        }

        public AwaitableObject AwaitableObject { get; private set; }
    }

    public class AwaitableTask<T> : IAwaiter<T>
    {
        public AwaitableTask()
        {
            AwaitableObject = new AwaitableObject<T>();
        }

        public IAwaitable<T> GetAwaiter()
        {
            return AwaitableObject;
        }

        public AwaitableObject<T> AwaitableObject { get; private set; }
    }
```

```cs

    public class AwaitableObject : Awaitable, IAwaitable
    {
        public void GetResult()
        {

        }
    }

    public class AwaitableObject<T> : Awaitable, IAwaitable<T>
    {
        public T Result { get; private set; }

        public T GetResult()
        {
            return Result;
        }

        public void ReportCompleted(T result)
        {
            Result = result;
            ReportCompleted();
        }
    }

    public abstract class Awaitable
    {
        private Action _continuation;

        private readonly object _locker = new object();

        public bool IsCompleted { get; private set; }

        public void OnCompleted(Action continuation)
        {
            //因为此处调用和ReportCompleted，可能不在一个线程上，会出现时机问题
            lock (_locker)
            {
                if (IsCompleted)
                {
                    continuation?.Invoke();
                }
                else
                {
                    _continuation += continuation;
                }
            }
        }

        public void ReportCompleted()
        {
            lock (_locker)
            {
                IsCompleted = true;
                var continuation = _continuation;
                _continuation = null;
                continuation?.Invoke();
            }
        }

        public void Reset()
        {
            lock (_locker)
            {
                _continuation?.Invoke();
                IsCompleted = false;
                _continuation = null;
            }
        }
    }

```
未完待续……

欢迎转载分享，请关注微信公众号，将同步更新博客，方便查看！

![承哥技术交流小作坊](https://i.loli.net/2021/09/27/FmsaLU1Oo7tX8kl.jpg)