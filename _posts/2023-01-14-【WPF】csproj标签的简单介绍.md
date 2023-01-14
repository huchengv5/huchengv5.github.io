---
title: "csproj标签的简单介绍"
author: 胡承
date: 2023-01-14 09:12:3 +0800
CreateTime: 2023-01-14 09:12:3 +0800
categories: C# WPF
---

介绍几个常用的csproj Property!

<!-- more -->

```xml
<!--csproj框架引用-->
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <!--支持的编译平台，配置了此选项，可以在配置管理里面配置它到底是x86还是x64或者AnyCPU-->
    <Platforms>AnyCPU;x86;x64</Platforms>
    <!--当前项目按什么来编译-->
    <PlatformTarget>x86</PlatformTarget>
    <!--版本，打nuget包也会认这个版本号-->
    <Version>1.0.0</Version>
    <FileVersion>1.0.0</FileVersion>
    <!--编译输出类型：Library 类库，WinExe 应用程序-->
    <OutputType>Library</OutputType>
    <!--是否启用WPF，Winform同理-->
    <UseWPF>true</UseWPF>
    <!--是否生成 AssemblyInfo.cs，如果需要手动添加就设置为False。因为 Microsoft.NET.Sdk 里面已经包含了该配置-->
    <GenerateAssemblyInfo>False</GenerateAssemblyInfo>
    <!--打nuget包时可能会用到该选项。如果csproj没有显示的包含Compile的项，提示重复的Compile。可以尝试添加此项-->
    <IncludePackageReferencesDuringMarkupCompilation>False</IncludePackageReferencesDuringMarkupCompilation>
    <!--目标框架类型-->
    <TargetFramework>net452</TargetFramework>
    <!--基础输出目录-->
    <BaseOutputPath>..</BaseOutputPath>
    <!--输出目录-->
    <OutputPath>$(BaseOutputPath)\$(Configuration)\</OutputPath>
    <!--是否将framework版本追加到目录，如 自动创建 net452 文件夹-->
    <AppendTargetFrameworkToOutputPath>false</AppendTargetFrameworkToOutputPath>
    <!--是否将运行标识追加到目录，如 自动创建 x86 文件夹-->
    <AppendRuntimeIdentifierToOutputPath>false</AppendRuntimeIdentifierToOutputPath>
    <!--应用程序图标-->
    <ApplicationIcon>ico_test_48.ico</ApplicationIcon>
  </PropertyGroup>
  <!--框架引用-->
  <ItemGroup>
    <Reference Include="System" />
    <Reference Include="System.Core" />
    <Reference Include="System.Xml.Linq" />
    <Reference Include="System.Data.DataSetExtensions" />
    <Reference Include="Microsoft.CSharp" />
    <Reference Include="System.Data" />
    <Reference Include="System.Net.Http" />
    <Reference Include="System.Xml" />
  </ItemGroup>
  <!--项目引用-->
  <ItemGroup>
    <ProjectReference Include="..\A\B.csproj" />
  </ItemGroup>
  <!--包引用-->
  <ItemGroup>
    <PackageReference Include="System.Data.SQLite">
      <Version>1.0.115.5</Version>
    </PackageReference>
  </ItemGroup>
</Project>
```

微信公众号：

![承哥技术交流小作坊](https://i.loli.net/2021/09/27/FmsaLU1Oo7tX8kl.jpg)

**欢迎转载分享，如若转载，请标注署名。**