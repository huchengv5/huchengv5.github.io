---
title: "WPF 依赖项属性的AddOwner与OverrideMetadata区别"
author: 胡承
date: 2020-09-28 13:35:3 +0800
CreateTime: 2020-09-28 13:35:3 +0800
categories: C# .NET WPF
---

WPF中的依赖项属性的AddOwner与OverrideMetadata有何区别？什么情况下使用AddOwner?什么情况下使用OverrideMetadata呢？

<!-- more -->

依赖项属性是WPF框架中非常核心的属性，相比传统属性提供了丰富的功能。如：内置绑定支持，渲染行为支持，动画支持等等。

1. `DependencyProperty.AddOwner`

顾名思义，`AddOwner`，就是给当前的依赖项属性添加“所有者”或者“拥有者”。也就是说，它的主要作用是给新的类型，定义相同的依赖项属性，从而省去了我们自己通过`DependencyProperty.Registry`方式进行注册。实际上两者的效果是差不多的，都是给指定的类型定义一个依赖项属性。

`AddOwner` 接受的参数是 `Type` ,通过AddOwner方法，只需要传入我们注册依赖项属性的类型就可以了。

举例：

假设我们要给`TestClass`定义一个`Background`依赖项属性。而`TestClass`也是继承自`FrameworkElement`，同时假设他的`Background`属性的作用和`Border`的`Background`属性是相同的。

那么这个时候，我们完全没有必要重新去注册一个依赖项属性，直接用`Border.BackgroundProperty.AddOwner(typeof(TestClass))`即可。

所以我们在使用`AddOwner`方法时，是我们想要注册的属性，在现有的依赖项属性中已经存在注册了，并且两者的行为功能一致的情况下即可使用。

2. `DependencyProperty.OverrideMetadata`

同样的，`OverrideMetadata`从字面上看，是重写元数据。它主要是用于现有的依赖项属性的元数据进行修改。

实际上，`OverrideMetadata`内部的实现是一个委托链，当我们在重写它的时候，它只是在委托链上增加了一个委托方法。当我们方法在执行的时候，我们最终的实现方法，会覆盖部分已定义的功能实现。

所以，`OverrideMetadata`主要使用场景在于，我们要修改依赖项属性的一些默认行为。

我们在使用`OverrideMetadata`方法的过程中，需要注意，元数据需要跟积累的元数据类型相同。

如基类元数据类型是`FrameworkPropertyMetadata`,那我们重写的时候也必须是这个类型。默认情况下是`PropertyMetadata`。这种情况下会导致错误，需要注意一下。

3. 两者区别

`AddOwner`是`OverrideMetadata`和添加`PropertyFromName`哈希表的合集，`OverrideMetadata`是把当前元数据和继承体系中最近的父类的依赖属性的元数据`PropertyChangedCallback`进行合并，`CoerceCallback`进行取舍。`AddOwner`调用的过程中会调用`OverrideMetadata`方法。

部分参考代码：

```cs
        //AddOwner部分代码
        public DependencyProperty AddOwner(Type ownerType, PropertyMetadata typeMetadata)
        {
            if (ownerType == null)
            {
                throw new ArgumentNullException("ownerType");
            }
            FromNameKey key = new FromNameKey(this.Name, ownerType);
            lock (Synchronized)
            {
                if (PropertyFromName.Contains(key))
                {
                    throw new ArgumentException(MS.Internal.WindowsBase.SR.Get("PropertyAlreadyRegistered", new object[] { this.Name, ownerType.Name }));
                }
            }
            if (typeMetadata != null)
            {
                this.OverrideMetadata(ownerType, typeMetadata);
            }
            lock (Synchronized)
            {
                PropertyFromName.set_Item(key, this);
            }
            return this;
        }


        public void OverrideMetadata(Type forType, PropertyMetadata typeMetadata)
        {
            //省略
            //部分代码
            DependencyObjectType dType = DependencyObjectType.FromSystemType(forType);
            PropertyMetadata baseMetadata = this.GetMetadata(dType.BaseType);
            //合并PropertyMetaChanged的事件
            if (baseMetadata.PropertyChangedCallback != null)
            {
                Delegate[] invocationList = baseMetadata.PropertyChangedCallback.GetInvocationList();
                if (invocationList.Length > 0)
                {
                    System.Windows.PropertyChangedCallback a = (System.Windows.PropertyChangedCallback) invocationList[0];
                    for (int i = 1; i < invocationList.Length; i++)
                    {
                        a = (System.Windows.PropertyChangedCallback) Delegate.Combine(a, (System.Windows.PropertyChangedCallback) invocationList[i]);
                    }
                    a = (System.Windows.PropertyChangedCallback) Delegate.Combine(a, this._propertyChangedCallback);
                    this._propertyChangedCallback = a;
                }
            }
            if (this._coerceValueCallback == null)
            {
                //委托直接覆盖
                this._coerceValueCallback = baseMetadata.CoerceValueCallback;
            }
            if (this._freezeValueCallback == null)
            {
                this._freezeValueCallback = baseMetadata.FreezeValueCallback;
            }
            //省略
        }

```

总的来讲，如果是需要给依赖项属性添加新的数据类型，就可以直接用`AddOwner`;如果是想要更改默认的行为，那么可以用`OverrideMetadata`;如果要定义的依赖项属性框架没有提供，那么就重新注册一个即可。