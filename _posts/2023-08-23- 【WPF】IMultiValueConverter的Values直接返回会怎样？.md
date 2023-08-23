---
title: "IMultiValueConverter的Values直接返回会怎样？"
author: 胡承
date: 2023-08-23 09:12:3 +0800
CreateTime: 2023-08-23 09:12:3 +0800
categories: C# WPF
---

IMultiValueConverter的Values直接返回会怎样？

<!-- more -->

IMultiValueConverter接口，我们在写wpf程序的时候应该是比较熟悉了，这个接口主要用于实现多路数据绑定转换器。

如果我们写了一个转换器，继承至IMultiValueConverter,直接将传入的值返回，会出现什么结果？示例代码如下：

```cs
//...省略... 转换器礼逻辑
public object Convert(object[] values, Type targetType, object parameter, CultureInfo culture)
{
  return values;
}

//...省略...  绑定逻辑      
<Button.CommandParameter>
  <MultiBinding Converter="{StaticResource MultiValueConverter}">
  <Binding Path="P1"/>
  <Binding Path="P2"/>
  <Binding Path="P3"/>
  </MultiBinding>
</Button.CommandParameter>
//...省略...

```

当我们从CommandParameter获取相关的参数值的时候，我们发现参数的数量为0!

我们的参数明明没有做任何修改，为什么会变成了0? 正常不应该是数组长度为3的数组吗?

通过查看源代码发现，原来转换器里面的Values被清空了。

源代码位置：MultiBindingExpression -> TransferValue() 方法。

部分源代码如下所示：

```cs
   //...省略...  this._tempValues 表示原始的数据 obj2 表示转换后的数据
   obj2 = this.Converter.Convert(this._tempValues, base.TargetProperty.PropertyType, this.ParentMultiBinding.ConverterParameter, culture);
   //...省略...  这里把原始数据清理掉了
   Array.Clear(this._tempValues, 0, this._tempValues.Length);
   //……省略……
```

所以要解决这个问题，我们可以修改下转换器：

```cs
//...省略... 转换器礼逻辑
public object Convert(object[] values, Type targetType, object parameter, CultureInfo culture)
{
  return values.Clone();
}
```

这样就可以解决上述问题了。