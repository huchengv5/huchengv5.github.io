---
title: "UWP InkCanvas 高级用法"
author: 胡承
date: 2020-05-16 20:35:3 +0800
CreateTime: 2020-05-16 20:35:3 +0800
categories: C# .net WPF UWP
---

前面已经介绍了InkCanvas基础用法，现在来看看InkCanvas的一些高级用法！

<!-- more -->
这里先介绍下InkCanvas的一些处理对象和事件：

属性`InkCanvas.InkPresenter`:InkCanvas关联的核心对象，包含事件的输入处理，笔迹的绘制参数设置，事件传递等操作。

属性`InkCanvas.InkPresenter.IsInputEnabled`：InkPresenter是否处理输入事件。如果为`true`,则自动处理事件，事件将不会传递给`Pointer`相关系列事件。

属性`InkCanvas.InkPresenter.InputDeviceTypes`:设置需要处理的设备类型，如鼠标，触控笔，手等。

属性`InkCanvas.InkPresenter.StrokeContainer`：`InkStroke`容器，在未启用自定义干操作时，该容器会自动实例化，并将笔迹填充到该容器。如果启用自定义干操作时，该属性为`null`,并且不允许实例化。

属性`InkCanvas.InkPresenter.StrokeInput`：笔迹输入处理对象，提供Stroke绘制过程中的事件处理。

事件`InkCanvas.InkPresenter.StrokesCollected`:当湿笔迹生成完成后触发（即鼠标抬起或者触摸抬起时触发）。我们可以通过该事件，将湿笔迹转成干笔迹。

事件`InkCanvas.PointerPressed`：当指针按下时触发。当现在的模式是由InkCanvas自己处理书写逻辑（即未启用自定义干操作）并且设置了`InkCanvas.InkPresenter.IsInputEnabled = true`时，事件不会被触发。如果启用了自定义干操作或者设置`InkCanvas.InkPresenter.IsInputEnabled = false`时，事件将会被触发。

事件`InkCanvas.PointerMoved`：当指针按下时触发。当现在的模式是由InkCanvas自己处理书写逻辑（即未启用自定义干操作）并且设置了`InkCanvas.InkPresenter.IsInputEnabled = true`时，事件不会被触发。如果启用了自定义干操作或者设置`InkCanvas.InkPresenter.IsInputEnabled = false`时，事件将会被触发。

事件`InkCanvas.PointerReleased`：当指针按下时触发。当现在的模式是由InkCanvas自己处理书写逻辑（即未启用自定义干操作）并且设置了`InkCanvas.InkPresenter.IsInputEnabled = true`时，事件不会被触发。如果启用了自定义干操作或者设置`InkCanvas.InkPresenter.IsInputEnabled = false`时，事件将会被触发。

事件`WetStrokeStarting`:当湿笔迹准备绘制时触发。该事件需要通过`var coreWetStrokeUpdateSource = CoreWetStrokeUpdateSource.Create(InkCanvas.InkPresenter)`方法获取可注册事件的对象。该事件可以用于处理一些业务逻辑，比如动态笔迹绘制前，是否智能启用擦除模式。

事件`WetStrokeCanceled`:当湿笔迹绘制被取消时触发。该事件需要通过`var coreWetStrokeUpdateSource = CoreWetStrokeUpdateSource.Create(InkCanvas.InkPresenter)`方法获取可注册事件的对象。暂未找到取消方法，在`WetStrokeStarting`事件中设置`args.Disposition = CoreWetStrokeDisposition.Canceled;`却未生效。

事件`WetStrokeContinuing`:当湿笔迹绘制被取消时触发。该事件需要通过`var coreWetStrokeUpdateSource = CoreWetStrokeUpdateSource.Create(InkCanvas.InkPresenter)`方法获取可注册事件的对象。

事件`WetStrokeStopping`:当湿笔迹绘制停止时时触发。该事件需要通过`var coreWetStrokeUpdateSource = CoreWetStrokeUpdateSource.Create(InkCanvas.InkPresenter)`方法获取可注册事件的对象。该事件触发早于`WetStrokeCompleted`。

事件`WetStrokeCompleted`:当湿笔迹绘制完成时触发。该事件需要通过`var coreWetStrokeUpdateSource = CoreWetStrokeUpdateSource.Create(InkCanvas.InkPresenter)`方法获取可注册事件的对象。该事件触发晚于`WetStrokeStopping`。

事件`InkCanvas.InkPresenter.StrokeInput.StrokeStarted`：湿笔迹已启动绘制时发生，此事件触发要晚于`WetStrokeStarting`。

事件`InkCanvas.InkPresenter.StrokeInput.StrokeCanceled`：湿笔迹取消绘制时发生。暂未验证！

事件`InkCanvas.InkPresenter.StrokeInput.StrokeContinued`：湿笔迹持续绘制时发生。此事件触发要晚于`WetStrokeContinuing`。

事件`InkCanvas.InkPresenter.StrokeInput.StrokeEnded`：湿笔迹完成绘制时发生。此事件触发要晚于`WetStrokeStarting`。

事件触发顺序如下：
```txt

WetStrokeStarting
StrokeStarted
WetStrokeContinuing
StrokeContinued
WetStrokeContinuing
StrokeContinued
----省略----
WetStrokeContinuing
StrokeContinued
WetStrokeStopping
StrokeEnded
WetStrokeCompleted

wetStrokeStarting
StrokeStarted
WetStrokeContinuing
StrokeContinued
----省略----
WetStrokeContinuing
StrokeContinued
WetStrokeContinuing
StrokeContinued
StrokeEnded
WetStrokeStopping
WetStrokeCompleted

WetStrokeStarting
StrokeStarted
WetStrokeContinuing
StrokeContinued
----省略----
WetStrokeContinuing
StrokeContinued
WetStrokeStopping
StrokeEnded
WetStrokeCompleted

```

#### 这里值得一提的是，我们发现WetStrokeStopping和StrokeEnded事件触发顺序可能会出现不一致的情况。这是因为CoreWetStrokeUpdateSource下的事件和StrokeInput下的事件并不是在同一个线程，CoreWetStrokeUpdateSource的事件在触摸线程中触发，而StrokeInput事件是在主线程上触发。####

核心的事件都了解了，我们围绕这些事件就可以扩展各种业务逻辑了。

这里以动态笔迹转静态笔迹为例：

```cs

        private InkSynchronizer _inkSynchronizer; 
        private InkStrokeContainer _inkStrokeContainer = new InkStrokeContainer();
        
        public void Init()
        {
            //开启自定义干操作
            _inkSynchronizer = InkCanvas.InkPresenter.ActivateCustomDrying();
            //设置基础参数
            var drawingAttributes = InkCanvas.InkPresenter.CopyDefaultDrawingAttributes();
            drawingAttributes.Color = Colors.Red;
            drawingAttributes.Size = new Size(3, 3);
            drawingAttributes.PenTip = PenTipShape.Circle;
            drawingAttributes.PenTipTransform = Matrix3x2.CreateSkew(4, 4);
            InkCanvas.InkPresenter.UpdateDefaultDrawingAttributes(drawingAttributes);
            //告诉InkCanvas处理那些设备类型的消息
            InkCanvas.InkPresenter.InputDeviceTypes = Windows.UI.Core.CoreInputDeviceTypes.Mouse | Windows.UI.Core.CoreInputDeviceTypes.Pen | Windows.UI.Core.CoreInputDeviceTypes.Touch;
            //开启多指触控，多指触控之前需要先开启完全控制
            InkCanvas.InkPresenter.SetPredefinedConfiguration(InkPresenterPredefinedConfiguration.SimpleMultiplePointer);
            //多指触摸必须要设置的选项
            InkCanvas.InkPresenter.InputProcessingConfiguration.RightDragAction = InkInputRightDragAction.LeaveUnprocessed;
            //注册湿笔迹收集的事件
            InkCanvas.InkPresenter.StrokesCollected += InkPresenter_StrokesCollected;
        }

        private void InkPresenter_StrokesCollected(InkPresenter sender, InkStrokesCollectedEventArgs args)
        {
            _inkSynchronizer.BeginDry();
            //收集笔迹
            _inkStrokeContainer.AddStrokes(args.Strokes);
            //todo:添加自己的操作，需要如何处理收集到的笔迹。
            _inkSynchronizer.EndDry();
        }

```


其他的案例这里就不多写了，了解以上事件就可以自定义很多业务了。