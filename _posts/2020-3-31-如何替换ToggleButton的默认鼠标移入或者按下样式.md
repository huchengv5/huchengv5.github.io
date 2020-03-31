---
title: "如何替换ToggleButton的默认鼠标移入或者按下样式"
author: 胡承
date: 2020-03-31 20:35:3 +0800
CreateTime: 2020-03-31 20:35:3 +0800
categories: C# .net Core
---

WPF有段时间没有做界面了，温故而知新，重新复习下模版相关知识。这里只以ToggleButton的模版举例！

<!-- more -->

代码不清真不重要，本博客主要是想演示下，如何修改Button在鼠标移入或者按下时的样式。  
直接看代码：

```xml
<Style x:Key="ToggleButtonStyle" TargetType="ToggleButton">
                <Style.Setters>
                    <Setter Property="Template">
                        <Setter.Value>
                            <ControlTemplate>
                                <Border Background="{TemplateBinding Property=Background }" BorderBrush="Transparent">
                                    <Image Source="{Binding RelativeSource={RelativeSource AncestorType=UserControl}, Path=CurrentIcon}"/>
                                </Border>
                            </ControlTemplate>
                        </Setter.Value>
                    </Setter>
                    <Setter Property="Background" Value="Transparent"/>
                </Style.Setters>
                <Style.Triggers>
                    <Trigger Property="IsMouseOver" Value="True">
                        <Setter Property="Background" Value="Transparent"/>
                    </Trigger>
                    <Trigger Property="IsPressed" Value="True">
                        <Setter Property="Background" Value="Transparent"/>
                    </Trigger>
                </Style.Triggers>
            </Style>           
```

代码其实很简单，我们给ToggleButton定义了一个Style样式，重写了Template的模版属性。

这里我们重点看下`Border`的`Background`属性中，我们是使用`TemplateBinding`。这样我们在后面的`Style.Triggers`中，才可以修改`IsMouseOver`和`IsPressed`的颜色。

控件默认的颜色设置是通过`VisualStateManager`来修改的，如果我们不使用`TemplateBinding`来绑定`Background`的话，我们就无法修改到原有的依赖项属性值，这也就导致我们的修改无效。

