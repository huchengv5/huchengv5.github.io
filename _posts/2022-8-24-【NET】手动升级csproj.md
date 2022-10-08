---
title: "手动升级csproj踩坑"
author: 胡承
date: 2022-08-24 09:12:3 +0800
CreateTime: 2022-08-24 09:12:3 +0800
categories: C# WPF
---

因为项目原因，还使用着比较原始的`.NET Framework`框架，但因为某种原因，暂时不让升级到.NET 6。为了能够解锁更多`Visual Studio 2022`的功能，尝试手动修改`csproj`文件。

<!-- more -->

这个过程中，也会遇到不少坑，再次做个记录。

### **一、类库中包含了 `App.xaml`**

错误输出：

<font color=red>

C:\Program Files\Microsoft Visual Studio\2022\Community\MSBuild\Current\Bin\amd64\Microsoft.Common.CurrentVersion.targets(2302,5): warning MSB3270: 所生成项目的处理器架构“MSIL”与引用“D:\work\project\xxx\Utils\xxx\\..\..\Debug\xxx.dll”的处理器架构“x86”不匹配。这种不匹配可能会导致运行时失败。请考虑通过配置管理器更改您的项目的目标处理器架构，以使您的项目与引用间的处理器架构保持一致，或者为引用关联一个与您的项目的目标处理器架构相符的处理器架构。
1>C:\Program Files\dotnet\sdk\6.0.301\Sdks\Microsoft.NET.Sdk.WindowsDesktop\targets\Microsoft.WinFX.targets(223,9): error MC1002: 库项目文件无法指定 ApplicationDefinition 元素。
1>C:\Program Files\dotnet\sdk\6.0.301\Sdks\Microsoft.NET.Sdk.WindowsDesktop\targets\Microsoft.WinFX.targets(223,9): error BG1003: 项目文件包含无效的属性值。

</font>  


解决方法：

<font color=green>
删除类库中的 App.xaml，因为`Microsoft.NET.Sdk`中已经包含了对它编译方式的定义。
</font>

<br/>

### **二、AssemblyAttribute 提示重复**

<br/>

因为 `Microsoft.NET.Sdk` 中已经包含了对它的定义，我们只需要在`csproj`中，将自动生成`AssemblyInfo`文件关闭。

    <GenerateAssemblyInfo>false</GenerateAssemblyInfo>

### **三、生成事件批处理报错**

事件编译，新旧不一样，需要修改。把bat批处理复制到IDE里面自动生成即可

旧的`csproj`形式
```xml
  <PropertyGroup>
    <PreBuildEvent>if  "$(PlatformName)" == "x64"  (xcopy /E /Y "$(ProjectDir)libs\x64\*.*" "$(ProjectDir)libs\") else (xcopy /E /Y "$(ProjectDir)libs\x86\*.*" "$(ProjectDir)libs\")</PreBuildEvent>
  </PropertyGroup>
```

新的`csproj`方式

```xml
	<Target Name="PreBuild" BeforeTargets="PreBuildEvent">
	  <Exec Command="if  &quot;$(PlatformName)&quot; == &quot;x64&quot;  (xcopy /E /Y &quot;$(ProjectDir)libs\x64\*.*&quot; &quot;$(ProjectDir)libs\&quot;) else (xcopy /E /Y &quot;$(ProjectDir)libs\x86\*.*&quot; &quot;$(ProjectDir)libs\&quot;)" />
	</Target>
```
主意看两端批处理的表示方式，可以很明显的看出来，相关的配置已经完全不一样了。
 
### **四、命令行无法构建x64的原因**

错误提示

<font color=red>
...省略... 

project.assets.json 没有 net452/win7-x64 的目标 

...省略...

可能需要在项目 RuntimeIdentifiers 中包括 “win7-x64”.
</font>

这个错误的主要原因是因为nuget包还原的时候，默认是按 `win7-x86` 来还原的。我们可以在`project.assets.json`文件里面找到这个配置项。要解决这个问题，我们就需要在还原`nuget`包的时候，告诉编译器我们是按`win7-x64`来做编译。

具体如下：
```cmd
dotnet restore -r win7-x64
```

### **五、.NET Framework v4.5.2 基于netstandar无法编译通过**
*<font color=green>
netstandar 不支持.net framework v4.5.2，升级到.net framework v4.6.1 以上即可。
</font>*

### **六、动态库的引用差异**

1. .NET6中不需要额外引用`System.Web.dll`，直接`Using System.Web`的命名空间即可
1. `System.Windows.Form.MenuItem` 使用 `ToolStripMenuItem` 代替
1. `System.Windows.Interactivity.dll`使用`System.Windows.Interactivity.WPF` nuget包代替
1. `System.Drawing.dll` 使用 `System.Drawing.Common` nuget包代替
1. `NotifyIcon.ContextMenu` 使用 `NotifyIcon.ContextMenuStrip` 代替
1. `ContextMenu` 使用 `ContextMenuStrip` 代替


关于手动升级到新版的`csproj`文件格式，网上有较多教程，我这边就不列出来了。
主要有两种方式：
1. 通过工具升级
1. 手动修改`csproj`文件

本博客会持续更新~

<a href="https://huchengv5.gitee.io/">博客链接1</a>

<a href="https://huchengv5.github.io/">博客链接2</a>

![承哥技术交流小作坊](https://i.loli.net/2021/09/27/FmsaLU1Oo7tX8kl.jpg)