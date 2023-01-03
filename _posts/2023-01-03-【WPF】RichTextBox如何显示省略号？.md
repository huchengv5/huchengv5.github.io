---
title: "RichTextBox如何显示省略号？"
author: 胡承
date: 2023-01-03 09:12:3 +0800
CreateTime: 2023-01-03 09:12:3 +0800
categories: C# WPF
---

RichTextBox是个很强大的文本组件，它可以用于显示各种复杂的富文本。

<!-- more -->
通常用富文本显示内容，通常是可以支持滚动显示，所以常规场景下是用不到省略号的。但是也不乏有些场景需要用到 省略号。比如：文本内容的简述。

下面来分享一种比较简单的实现方法，来实现富文本省略号逻辑。

我们都知道`WPF`里面有`TextBlock`组件，它是一个相对比较轻量的文本组件，使用频率也是比较高的。它可以支持直接文本，也可以直接内联（Inlines)文本,如：`Run`标签。`TextBlock`本身也支持`TextTrimming`,可以用来显示省略号。

所以呢，要让`RichTextBox`来支持省略号的思路就比较简单，只需要在它的段落里面，内嵌`TextBlock`即可。

处理前的流文本示例：
```xml

<FlowDocument xml:space="preserve" xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation">
    <Paragraph>        
            <Run Foreground="red">[红包]</Run>
            <Run Foreground="green">恭喜发财,大吉大利</Run>       
    </Paragraph>
</FlowDocument>
```

处理后的流文本示例：

```xml
<FlowDocument xml:space="preserve" xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation">
    <Paragraph>
        <TextBlock TextTrimming="CharacterEllipsis" Foreground="blue">
            <Run Foreground="red">[红包]</Run>
            <Run Foreground="green">恭喜发财,大吉大利</Run>            
        </TextBlock>
    </Paragraph>
</FlowDocument>
```

以上方法，就可以简单的实现省略号的功能了，后面我会再分享一个组件，用来支持 `html`转`RichTextBox`。

**欢迎转载分享，如若转载，请标注署名。**

博客链接1：https://huchengv5.gitee.io/

博客链接2：https://huchengv5.github.io/

微信公众号：

![承哥技术交流小作坊](https://i.loli.net/2021/09/27/FmsaLU1Oo7tX8kl.jpg)