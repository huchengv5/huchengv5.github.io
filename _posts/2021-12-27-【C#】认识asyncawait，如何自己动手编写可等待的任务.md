---
title: "认识asyncawait，如何自己动手编写可等待的任务？"
author: 胡承
date: 2021-12-27 19:10:3 +0800
CreateTime: 2021-12-27 19:10:3 +0800
categories: C# WPF
---

C#里面的`async/await`语法简直好用的不要不要的。但是，如果我想让这种异步状态机，能工作在我自定义的线程里面要怎么办呢？

<!-- more -->

要让编译器识别async await 语法首先需要具备以下条件：  
1、 可等待的对象，需要实现`T GetAwaiter();`方法。注意：`T`可以是任何类型，不一定是`object`类型。

可等待对象`无返回值`需要满足的条件：
1）、继承接口`INotifyCompletion`或实现`void OnCompleted(Action continuation);`方法。  
2）、实现`bool IsCompleted { get; }`属性。
3）、实现`void GetResult();`方法。
        
可等待对象`带返回值`需要满足的条件：
1）、继承接口`INotifyCompletion`或实现`void OnCompleted(Action continuation);`方法。  
2）、实现`bool IsCompleted { get; }`属性。
3）、实现`T GetResult();`方法。`T`可以是任意类型。
        
具备上述条件，你的代码就可以顺利编译通过了。但是此时的代码，还不能工作，因为你还没有给状态机增加状态！

为了增加这个状态，我们增加一个方法`void ReportCompleted(T result);`，用来更改状态机的状态值，表示我们要执行的异步操作是否已完成。

具体代码，如下所示：

**`相关的接口定义:`**

```cs

    /// <summary>
    /// 带返回值的可等待对象
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
    /// 不带返回值的可等待对象
    /// </summary>
    public interface IAwaitable : INotifyCompletion
    {
        bool IsCompleted { get; }
        void GetResult();
        void ReportCompleted();
    }

    /// <summary>
    /// 获取可等待者
    /// </summary>
    public interface IAwaiter
    {
        IAwaitable GetAwaiter();
    }

    /// <summary>
    /// 获取可等待者
    /// </summary>
    public interface IAwaiter<T>
    {
        IAwaitable<T> GetAwaiter();
    }

```

```cs

    /// <summary>
    /// 不带返回值的可等待的任务
    /// </summary>
    public class AwaitableTask : IAwaiter
    {
        public AwaitableTask()
        {
            AwaitableObject = new AwaitableObject();
        }

        /// <summary>
        /// 获取可等待的对象
        /// </summary>
        public IAwaitable GetAwaiter()
        {
            return AwaitableObject;
        }

        /// <summary>
        /// 实现相关接口的可等待的对象
        /// </summary>
        public AwaitableObject AwaitableObject { get; private set; }
    }

    /// <summary>
    /// 带返回值的可等待的任务
    /// </summary>
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

    /// <summary>
    /// 不带返回值的可等待的状态对象
    /// </summary>
    public class AwaitableObject : Awaitable, IAwaitable
    {
        public void GetResult()
        {

        }
    }

    /// <summary>
    /// 带返回值的可等待的状态对象
    /// </summary>
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

    /// <summary>
    /// 可等待的抽象类，状态机的具体实现
    /// </summary>
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

        //用于通知，当前任务已经完成。
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

        //重置状态，让状态机恢复可等待
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

示例：

```cs

    public class TestElement : UIElement
    {
        //实例化一个可等待任务
        AwaitableTask _awaitableTask = new AwaitableTask();

        /// <summary>
        /// 指定的资源的路径
        /// </summary>
        public string SourceUri
        {
            get { return (string)GetValue(SourceUriProperty); }
            set { SetValue(SourceUriProperty, value); }
        }

        private ImageSource gifSource;

        public static readonly DependencyProperty SourceUriProperty = DependencyProperty.Register(nameof(SourceUri), typeof(string), typeof(TestElement), new PropertyMetadata(string.Empty, async (s, e) =>
        {
            var loc = s as TestElement;
            loc.gifSource = await LoadGifSourceAsync(e.NewValue as string);
            loc._awaitableTask.AwaitableObject?.ReportCompleted();
        }));

        public static async Task<ImageSource> LoadGifSourceAsync(string url)
        {
            await Task.Delay(500);
            return new BitmapImage(new Uri(url));
        }

        /// <summary>
        /// 等待源数据加载完成，对外开放的方法
        /// </summary>
        /// <returns></returns>
        public async Task WaitForLoaded()
        {
            await _awaitableTask;
        }

        public bool Do()
        {
            if (gifSource != null)
                return true;            
            return false;
        }
    }

    public class TestClass
    {
        public TestClass()
        {
            TestForNotWait();
            TestForWait();
        }


        public async void TestForWait()
        {
            TestElement testElement = new TestElement
            {
                SourceUri = @"D:\Users\huc\Pictures\Saved Pictures\1.jpeg"
            };
            await testElement.WaitForLoaded();
            Console.WriteLine("TestForWait Result:" + testElement.Do());
        }

        public void TestForNotWait()
        {
            TestElement testElement = new TestElement
            {
                SourceUri = @"D:\Users\huc\Pictures\Saved Pictures\1.jpeg"
            };

            Console.WriteLine("TestForNotWait Result:" + testElement.Do());            
        }
    }
        
```

欢迎转载分享，请关注微信公众号，将同步更新博客，方便查看！

![承哥技术交流小作坊](https://i.loli.net/2021/09/27/FmsaLU1Oo7tX8kl.jpg)