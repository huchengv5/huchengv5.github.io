---
title: "诡异的Visual Studio编译错误"
author: 胡承
date: 2023-03-02 09:12:3 +0800
CreateTime: 2023-03-02 09:12:3 +0800
categories: C# WPF
---

使用`Visual Studio 2022 17.5.1`版本编译项目的时候，意外发现时不时会出现`System.ArgumentException: 路径中具有非法字符`的编译错误。但后续编译一切正常，重启`Visual Studio`后，发现有的项目无法正常加载了！

<!-- more -->

除了这种情况以外，还会提示其它错误：`“Persistence = ProjectFileWithInterceptionViaSnapshot”没有项目属性提供程序。`。

通过检查代码，一路狂飙，看着都没有啥毛病啊！

最终在对比文件修改区别，逐个排除最终发现：`csproj`文件经过`xml`格式化工具格式化过，格式化以后，visual studio在解析的时候不能正常工作，导致内部出现异常，从而引发奇奇怪怪的问题。

最终发现是因为格式化的过程中，导致标签内容出现了较多不需要的空格和换行。如：

```xml
        <Content Include="$(MSBuildThisFileDirectory)..\..\runtimes\win-x86\native\CefSharp.dll">
            <CopyToOutputDirectory>
                PreserveNewest
            </CopyToOutputDirectory>
            <Link>
                runtimes\win-x86\native\CefSharp.dll
            </Link>
            <Visible>
                false
            </Visible>
        </Content>
```

修正后

```xml
        <Content Include="$(MSBuildThisFileDirectory)..\..\runtimes\win-x86\native\CefSharp.dll">
            <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
            <Link>runtimes\win-x86\native\CefSharp.dll</Link>
            <Visible>false</Visible>
        </Content>
```

按修正后的重新编译，一切恢复正常！

以上问题尚不确定到底是编译器的BUG，还是什么原因，如果有遇上，可以尝试通过上述方法尝试解决！

微信公众号：

![承哥技术交流小作坊](https://i.loli.net/2021/09/27/FmsaLU1Oo7tX8kl.jpg)

**欢迎转载分享，如若转载，请标注署名。**