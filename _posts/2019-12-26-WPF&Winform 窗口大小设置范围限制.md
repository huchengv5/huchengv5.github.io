---
title: "WPF & Winform 窗口大小设置范围限制"
author: 胡承
date: 2019-12-26 22:23:3 +0800
CreateTime: 2019-12-26 22:23:3 +0800
categories: .net
---

有个有趣的事情，当我们创建一个WPF窗体时，我们将窗体的大小设置为Width=90,Height=160。在设计器模式下，窗体比例看着很和谐，如下图所示：  

![待插图]()  

示例代码：

```xml

<Window x:Class="WpfApp1.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        mc:Ignorable="d"
        Title="MainWindow" WindowStyle="SingleBorderWindow" Left="0" Top="0" Height="160" Width="90">
    <Grid>
        <Image Source="Images/test.png" Stretch="Fill"/>
    </Grid>
</Window>

```

运行以后，我们发现窗口中的图片居然变形了，look!    

![待插图]()  

WPF的窗体高宽设置有bug么？不是所见即所得吗？  
通过snoop看到窗体的ActualWidth并不是90，而是131，此时窗体的Height属性依然是90！

然后我们把WindowStyle设置成None时，窗口的大小能够顺利的应用90 * 160 的分辨率。嗯，看到这里，好像问题已经解决了，因为WindowStyle非None时，有标题栏。

但是又继续尝试，将分辨率设置成768 * 1366，同样的，在设计器模式下，图片没有变形，显示正常。  

在运行模式下，图片发现又变形了！同样打开snoop，查看窗体的高宽，结果发现高度相比原有值更小了，只有1095。

难道真的是WPF的窗体在计算的时候出现了bug？  

于是在窗体里面重写了以下两个方法:

```csharp
        protected override Size ArrangeOverride(Size arrangeBounds)
        {
            var size = base.ArrangeOverride(arrangeBounds);
            return size;
        }

        protected override Size MeasureOverride(Size availableSize)
        {
            var size = base.MeasureOverride(availableSize);
            return size;
        }

```
通过断点我们发现，arrangeBounds 和 availableSize 并不等于我们之前设定的Height和Width值。

那么还不行，使用DNspy做源代码调试，一步一步跟进查看问题的根源在哪里。

其调用堆栈：发现GetWindowRect方法被调用，于是使用MoveWindow的方法尝试，结果依然。

最后考虑到窗体存在超出显示区的最大值的情况，比如说任务栏，于是通过日志跟踪法，将其窗体的大小打印出来。

```cs

            Task.Run(() =>
            {
                while (true)
                {
                    Thread.Sleep(1000);
                    Dispatcher.Invoke(()=> Console.WriteLine($"awid: {this.ActualWidth} ah:{this.ActualHeight} maxh:{ SystemParameters.MaximumWindowTrackHeight } maxwid:{ SystemParameters.MaximumWindowTrackWidth } minh:{SystemParameters.MinimumWindowTrackHeight} minwid:{SystemParameters.MinimumWindowTrackWidth}"));
                }                
            });

```
根据以上打印的结果发现：  
窗口的高度最大值是`SystemParameters.MaximumWindowTrackHeight`  
窗口的宽度最大值是`SystemParameters.MaximumWindowTrackWidth`  
窗口的高度最小值是`SystemParameters.MinimumWindowTrackHeight`  
窗口的宽度最小值是`SystemParameters.MinimumWindowTrackHeight`。

特别注意：该窗口的最小值限制不适用于`WindowStyle.None`的情况。

以上问题同样在Winform中验证过，存在相同的问题。并且在win7和win10不同的操作系统上运行过。

总结：导致窗口在设计器模式和运行模式表现不一样的原因是因为因为Windows操作系统的机制原因，限制的窗口的最大值和最小值。而设计器模式时，窗体属于VisualStudio的内部组件，不受窗体大小的限制，所以渲染出来的效果会存在以上比较诡异的差异。