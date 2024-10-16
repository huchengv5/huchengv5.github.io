---
title: "WPF如何实现“不用写转换器的Binding"
author: 胡承
date: 2024-10-14 09:12:3 +0800
CreateTime: 2024-10-14 09:12:3 +0800
categories: C# WPF
---

Xaml中的Binding语法是WPF的中的灵魂，但是很多业务场景下，我们都需要通过IValueConverter接口来实现一个转换器。

<!-- more -->

这种实现方式会导致我们编码的过程中会存在越来越多的转换器，每写一个转换器，命名或许都是个头疼的问题。尤其是针对多路Binding的时候，xaml的代码编写量会比较多，严重降低编码效率。

为了优化转化器的使用，我们可以通过Xaml扩展，自定义一个Binding的扩展方法。暂且命名为“XBinding”。

## 具体实现思路如下：

我们自定义一个Binding，该Binding本质还是原来的那个Binding，只是我们在原来的Binding的基础上做了一些逻辑精简。Binding所使用的转换器为IMultiValueConverter。考虑到Button按钮还存在Command的Binding场景，所以在XBinding中也增加了对Command的支持。

具体实现看代码：

### XBinding扩展代码实现
```cs
[ContentProperty("Bindings")]
public class XBinding : MarkupExtension
{
    public XBinding()
    {
​
    }
    public string Method { get; set; }
​
    [ConstructorArgument("Path")]
    /// <summary>
    /// 
    /// </summary>
    public string Path { get; set; }
​
    /// <summary>
    /// 多路绑定使用
    /// </summary>
    public MultiBinding Bindings { get; set; }
​
    public override object ProvideValue(IServiceProvider serviceProvider)
    {
        if (Method == null)
        {
            throw new ArgumentNullException("method");
        }
​
        IProvideValueTarget provideValueTarget = (IProvideValueTarget)serviceProvider.GetService(typeof(IProvideValueTarget));
        var rootObjectProvider = (IRootObjectProvider)serviceProvider.GetService(typeof(IRootObjectProvider));
        var rootObject = rootObjectProvider?.RootObject;
        if (rootObject != null)
        {
            MethodInfo method = rootObject.GetType().GetMethod(Method);
            bool isCommand = false;
            var targetObj = provideValueTarget.TargetObject;
            if (targetObj is CommandBinding)
            {
                throw new NotSupportedException("XBinding不支持CommandBinding");
            }
            else
            {
                var targetProperty = provideValueTarget.TargetProperty as DependencyProperty;
                if (targetProperty != null && targetProperty.PropertyType == typeof(ICommand))
                {
                    isCommand = true;
                }
            }
​
            if (Bindings != null)
            {
                Bindings.Converter = new XBindingMultiValueConverter() { Target = rootObject, ConvertMethod = method, IsCommand = isCommand };
                return Bindings.ProvideValue(serviceProvider);
            }
            else
            {
                var multiBinding = new MultiBinding() { Converter = new XBindingMultiValueConverter() { Target = rootObject, ConvertMethod = method, IsCommand = isCommand } };
                multiBinding.ConverterParameter = ConverterParameter;
                string[] propertyPaths;
                if (string.IsNullOrWhiteSpace(Path))
                {
                    multiBinding.Bindings.Add(new Binding());
                }
                else
                {
                    propertyPaths = Path.Split(new char[] { ',', ' ' }, StringSplitOptions.RemoveEmptyEntries);
                    foreach (var propertyPath in propertyPaths)
                    {
                        multiBinding.Bindings.Add(new Binding(propertyPath));
                    }
                }
​
                return multiBinding.ProvideValue(serviceProvider);
            }
        }
​
        return null;
    }
​
    #region Binding Properties
​
    [DefaultValue(BindingMode.Default)]
    public BindingMode Mode { get; set; }
​
    [DefaultValue("")]
    public RelativeSource RelativeSource { get; set; }
​
    public object Source { get; set; }
​
    [DefaultValue("")]
    public string ElementName { get; set; }
​
    [DefaultValue("")]
    public string StringFormat { get; set; }
​
    [DefaultValue("")]
    public object ConverterParameter { get; set; }
​
    #endregion
}
```

### 多路绑定的转换器实现：

```cs

/// <summary>
    /// 多路绑定
    /// </summary>
    internal class XBindingMultiValueConverter : IMultiValueConverter
    {
        /// <summary>
        /// 要执行多路转换的实例对象
        /// </summary>
        internal object Target { get; set; }
​
        /// <summary>
        /// 转换的方法
        /// </summary>
        internal MethodInfo ConvertMethod { get; set; }
​
        /// <summary>
        /// 反转转换的方法
        /// </summary>
        internal MethodInfo ConvertBackMethod { get; set; }
​
        /// <summary>
        /// 是否为Command
        /// </summary>
        internal bool IsCommand { get; set; }
​
        public object Convert(object[] values, Type targetType, object parameter, CultureInfo culture)
        {
            if (ConvertMethod != null)
            {
                if (IsCommand)
                {
                    return new XCommand((p) =>
                    {
                        if (p is object[] objs)
                        {
                            Execute(objs, parameter, targetType);
                        }
                        else
                        {
                            Execute(new object[1] { p }, parameter, targetType);
                        }
                    });
                }
                else
                {
                    return Execute(values, parameter, targetType);
                }
            }
            else
            {
#if DEBUG
                //throw new ArgumentNullException(nameof(ConvertMethod));
#endif
                return DependencyProperty.UnsetValue;
            }
        }
​
        private object Execute(object[] values, object parameter, Type targetType)
        {
            var parameterCount = ConvertMethod.GetParameters().Length;
            if (parameterCount == 0)
            {
                return ConvertMethod.Invoke(Target, new object[] { });
            }
            else
            {
                var count = Math.Min(parameterCount, values.Length + 1);
                var objs = new object[count];
                if (values == null)
                {
                    return ConvertMethod.Invoke(Target, objs);
                }
                else
                {
                    for (int i = 0; i < count; i++)
                    {
                        if (i == values.Length)
                        {
                            objs[i] = parameter;
                        }
                        else if (i > values.Length)
                        {
                            //默认null
                        }
                        else
                        {
                            if (values[i] != DependencyProperty.UnsetValue)
                            {
                                objs[i] = values[i];
                            }
                        }
                    }
​
                    if (DesignerProperties.GetIsInDesignMode(new DependencyObject()))
                    {
                        if (targetType.IsValueType)
                        {
                            return Activator.CreateInstance(targetType);
                        }
                        else
                        {
                            return null;
                        }
                    }
                    else
                    {
                        //无法转换类型
                        return ConvertMethod.Invoke(Target, objs);
                    }
                }
            }
        }
​
        public object[] ConvertBack(object value, Type[] targetTypes, object parameter, CultureInfo culture)
        {
            if (ConvertBackMethod != null)
            {
                var objs = new object[] { value };
                return (object[])ConvertBackMethod.Invoke(Target, objs);
            }
            else
            {
                throw new ArgumentNullException(nameof(ConvertBackMethod));
            }
        }
    }

```

### ICommand的实现：

```cs
/// <summary>
/// 表示XBinding所支持的ICommand
/// </summary>
public class XCommand : ICommand
{
    private readonly Action<object> _execute;
​
    private readonly Func<bool> _canExecute;
​
    public XCommand(Action<object> execute) : this(execute, null)
    {
​
    }
​
    public XCommand(Action<object> execute, Func<bool> canExecute)
    {
        if (execute == null)
        {
            throw new ArgumentNullException("execute");
        }
​
        _execute = execute;
        _canExecute = canExecute;
    }
​
    public event EventHandler CanExecuteChanged;
​
    public bool CanExecute(object parameter)
    {
        return _canExecute == null ? true : _canExecute();
    }
​
    public void Execute(object parameter)
    {
        if (_execute != null)
        {
            _execute(parameter);
        }
    }
}

```

具体使用方法：
```cs

    // 参考语法
    // 单路绑定：
    PlayStatus="{XBinding Path=IsSpeaking,Method=GetExpectPlayStatus}"
    // 多路绑定：
     PlayStatus="{XBinding Path=IsSpeaking Sex,Method=GetExpectPlayStatus}"
    <SvgaPlayerControl.SourceUri>
      <XBinding Method = "GetSpeakingSvga" >
        < MultiBinding >
            < Binding Path="IsSpeaking"/>
            <Binding Path = "Player.Sex" />
            < Binding RelativeSource="{RelativeSource AncestorType=SvgaPlayerControl}"/>
        </MultiBinding>
      </XBinding>
     </SvgaPlayerControl.SourceUri>

```
对应的Mothod需要在xaml对应的cs代码中编写。

结合以上代码，XBinding就可以应用到我们Xaml中了，对开发效率的提升还是有不少的。当然了，如果可以梳理一些通用转换器，那效率会更高。针对灵活处理的转换器，那么可以试试XBinding。

博客地址：https://huchengv5.github.io/

微信公众号：

![承哥技术交流小作坊](https://i.loli.net/2021/09/27/FmsaLU1Oo7tX8kl.jpg)

**欢迎转载分享，如若转载，请标注署名。**

