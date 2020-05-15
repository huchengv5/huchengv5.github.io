---
title: "UWP InkCanvas 基本知识"
author: 胡承
date: 2020-05-15 20:35:3 +0800
CreateTime: 2020-05-15 20:35:3 +0800
categories: C# .net WPF UWP
---

WPF中有InkCanvas，UWP中也一样存在InkCanvas，但是两者的API设计方面有所差异！UWP的InkCanvas的API设计上会比WPF版本的更难使用一些，同时功能也更强大一些！

<!-- more -->


InkCanvas通常称为墨迹或者笔迹，通常我们在写书写笔迹功能或者做一些标注，涂鸦等功能会用到它。它包含基本的笔迹书写，笔迹擦除，笔迹选择，笔迹识别，笔迹保存等功能。
下面先简单介绍下UWP版本下的InkCanvas基本概念。

## 基本概念 ##

在UWP版本中的笔迹，包含有两种：湿笔迹（Wet）和干笔迹(Dry)。  
湿笔迹：其实就是动态笔迹，书写过程中绘制的笔迹。因动态笔迹不需要额外的交互功能，所以动态笔迹的绘制通常都是采用最高性能的方式绘制，不支持交互。笔迹变干后，湿笔迹就会消失，干笔迹就会代替湿笔迹，并支持交互。
干笔迹：干笔迹就是动态笔迹落笔后生成的笔迹，干笔迹一旦生成，湿笔迹就会被删除或者不再渲染。干笔迹支持交互，渲染上的性能要求相比湿笔迹要低。

## 开始书写 ##

默认情况下，我直接将InkCanvas控件拖放到我们MainPage里面，直接运行程序，我们会发现，此时的InkCanvas并不能直接开始书写，因为我们没有给InkCanvas初始化相关参数。  
具体参数设置如下：  
```cs

//告诉InkCanvas，接收哪些类型的输入设备。包含此声明后，即可开始书写，默认书写（Inking）模式，颜色黑色。
InkCanvas.InkPresenter.InputDeviceTypes = Windows.UI.Core.CoreInputDeviceTypes.Mouse | Windows.UI.Core.CoreInputDeviceTypes.Pen | Windows.UI.Core.CoreInputDeviceTypes.Touch;

```
## 多指书写 ##

InkCanvas也是支持多指书写的，只是多指书写的实现过程会复杂一些。要启用多笔书写，就必现启用`自定义干`的操作，即：自定义将笔迹从湿笔迹转成干笔迹。
```cs
//启用自定义干
var inkSynchronizer = InkCanvas.InkPresenter.ActivateCustomDrying();

//开启多指触摸功能
InkCanvas.InkPresenter.SetPredefinedConfiguration(InkPresenterPredefinedConfiguration.SimpleMultiplePointer);

//自定义干如何使用，下篇再讲
//……
```

## 启动擦除 ##

InkCanvas仅提供两种模式切换：笔迹模式和擦除模式。

InkCanvas的擦除模式，只支持线相交模式擦除。

启用擦除模式方法如下：
```cs
//切换到擦除模式
InkCanvas.InkPresenter.InputProcessingConfiguration.Mode = InkInputProcessingMode.Erasing;
```
模式切换主要有三种：`None`,`Inking`,`Erasing`。

## 基本设置 ##

InkCanvas提供一些基础的设置方法，比如：笔迹的颜色，粗细，是否高亮，宽度，高度等。这些属性的设置都在`InkDrawingAttributes`对象中。可通过`InkCanvas.InkPresenter.CopyDefaultDrawingAttributes();`获取默认参数设置，然后根据`InkCanvas.InkPresenter.UpdateDefaultDrawingAttributes(drawingAttributes);`进行修改。

值得注意的是,PenTip可用来设置笔尖显示效果，如下设置：
```cs
    //设置笔迹使用圆形
    DrawingAttributes.PenTip = PenTipShape.Circle;
    //通过矩阵变换来设置笔尖形状
    DrawingAttributes.PenTipTransform = Matrix3x2.CreateSkew(4, 4);
```

UWP InkCanvas的基本用法就先介绍到这儿，有了这些功能就可以完成一些基础的功能开发了，后面会再介绍一些更高级的用法。
