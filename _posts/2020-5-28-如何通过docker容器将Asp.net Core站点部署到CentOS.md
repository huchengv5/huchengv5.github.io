---
title: "如何通过docker容器将ASP.NET Core站点部署到CentOS"
author: 胡承
date: 2020-05-28 20:35:3 +0800
CreateTime: 2020-05-28 20:35:3 +0800
categories: C# asp.net core linux
---

上一篇我们讲了如何将asp.net core部署到linux系统上，这次我们以docker容器的方式把我们的站点部署到linux系统中

<!-- more -->

使用Docker来部署，其实比我们常规的部署方式更简单。因为Docker的存在，本来就是把复杂的事情简单化，让我们可以“一次部署，处处可用”。

如果你是使用windows操作系统，首先记得先安装好`docker for windows`，不清楚怎么安装可以自行查阅资料。

## 创建dockerfile

在上一次我们创建的asp.net core web项目中，我们在发布的目标文件夹根目录中创建一个`Dockerfile`的文件（就是一个文本文件没有扩展名），具体内容如下：

```dockerfile
#从docker hub中拉取asp.net runtime基础镜像
FROM mcr.microsoft.com/dotnet/core/aspnet:3.1
#将当前目录下的文件，拷贝到容器中的app目录下
COPY . /app 
#设置app为工作目录
WORKDIR /app
#暴露TCP端口5100
EXPOSE 5100/tcp
#设置程序启动入口函数库名，第一个参数表示执行命令的程序名，第二个函数表示带有主函数入口文件名
ENTRYPOINT ["dotnet","LinuxWebForTest.dll"]
```
**特别注意：程序入口函数库文件名，`一定是以dll作为入口函数库，不能以exe文件或者不带扩展名的文件名称作为入口函数库`**

如果入口函数指定错误，则会出现以下提示：

      ```txt
      It was not possible to find any installed .NET Core SDKs
      Did you mean to run .NET Core SDK commands? Install a .NET Core SDK from:
            https://aka.ms/dotnet-download
      ```

## 通过Dockerfile构建镜像

1. 我们打开asp.net core web的根目录（也就是dockerfile所在目录），在资源管理器上方输入`cmd`命令，打开`cmd命令窗口`

2. 登录docker仓库服务器，以上次构建的私有仓库为例

      ```
      > docker login 192.168.1.110:5000
      //之前已经登录过，所以直接从缓存里面查找用户和密码信息，自动登录
      Authenticating with existing credentials...
      Login Succeeded
      //如果没有登录过，输入用户名，密码进行登录。可参见：https://huchengv5.github.io//post/%E6%90%AD%E5%BB%BAdocker%E7%A7%81%E6%9C%89%E4%BB%93%E5%BA%93.html 中的登录部分。
      ```
3. 构建镜像

      ```
      > docker build -t 192.168.1.110:5000/aspnetcoreapp:v1 .

      //执行过程
      Sending build context to Docker daemon  4.631MB
      Step 1/5 : FROM mcr.microsoft.com/dotnet/core/aspnet:3.1
      ---> bc877ac43e02
      Step 2/5 : COPY . /app
      ---> eaa31c882445
      Step 3/5 : WORKDIR /app
      ---> Running in f8db2aff9c73
      Removing intermediate container f8db2aff9c73
      ---> 97f999072fc2
      Step 4/5 : EXPOSE 5100/tcp
      ---> Running in ddeb5c939269
      Removing intermediate container ddeb5c939269
      ---> fbabdcf7447b
      Step 5/5 : ENTRYPOINT ["dotnet","LinuxWebForTest.dll"]
      ---> Running in 0d60745c61ea
      Removing intermediate container 0d60745c61ea
      ---> d2517be5dbd8
      Successfully built d2517be5dbd8
      Successfully tagged 192.168.1.110:5000/aspnetcoreapp:v1

      ```

      我们可以通过`docker images`来查看我们构建的镜像。

      ```
      > docker images
      REPOSITORY                             TAG                 IMAGE ID            CREATED             SIZE
      192.168.1.110:5000/aspnetcoreapp       v1                  26070ec7870f        18 hours ago        212MB
      linuxwebfortest                        dev                 8485a72fcf26        25 hours ago        207MB
      docker101tutorial                      latest              c57c605dedaa        3 days ago          25.1MB
      192.168.1.110:5000/python              test                f4df7f234e59        7 days ago          78.9MB
      python                                 alpine              f4df7f234e59        7 days ago          78.9MB
      mcr.microsoft.com/dotnet/core/aspnet   3.1                 bc877ac43e02        8 days ago          207MB
      mcr.microsoft.com/dotnet/core/aspnet   3.1-buster-slim     bc877ac43e02        8 days ago          207MB
      node                                   12-alpine           7a48db49edbf        4 weeks ago         88.7MB
      nginx                                  alpine              89ec9da68213        4 weeks ago         19.9MB
      registry                               latest              708bc6af7e5e        4 months ago        25.8MB
      ```

      ### 命令说明：

      ```
      docker build 构建镜像

      -t (tag) 指定镜像tag

      192.168.1.110:5000 镜像指定的仓库地址，会作为镜像名称的一部分，但这部分不能乱填

      aspnetcoreapp 镜像名称，可以任意填写

      v1 对应的标签，这里是表示仓库 192.168.1.110:5000 下的 aspnetcoreapp 镜像的 v1 版本

      最后面有个 . ，表示的是当前目录下的dockerfile文件，也可以通过 -f 显示指定dockerfile的文件路径
      ```

4. 推送镜像

      镜像构建好后，我们需要将镜像推送到我们的私有仓库中

      ```
      > docker push 192.168.1.110:5000/aspnetcoreapp:v1

      ```
      基本语法：docker push <镜像完整名称>

5. 启动容器

      镜像推送到服务器以后，我们需要进入到服务器，将镜像运行到容器中。（镜像就相当于是原始文件，容器可以理解为加载镜像文件后运行的程序实例）

      ```
      docker run -p 8000:5100 192.168.1.110:5000/aspnetcoreapp:v1
      ```
      执行成功效果如下：

      ```
      warn: Microsoft.AspNetCore.DataProtection.Repositories.FileSystemXmlRepository[60]
            Storing keys in a directory '/root/.aspnet/DataProtection-Keys' that may not be persisted outside of the container. Protected data will be unavailable when container is destroyed.
      warn: Microsoft.AspNetCore.DataProtection.KeyManagement.XmlKeyManager[35]
            No XML encryptor configured. Key {e0fd53e8-940f-4e70-b192-2b16c0f684dc} may be persisted to storage in unencrypted form.
      warn: Microsoft.AspNetCore.Server.Kestrel[0]
            Overriding address(es) 'http://+:80'. Binding to endpoints defined in UseKestrel() instead.
      info: Microsoft.Hosting.Lifetime[0]
            Now listening on: http://0.0.0.0:5100
      info: Microsoft.Hosting.Lifetime[0]
            Application started. Press Ctrl+C to shut down.
      info: Microsoft.Hosting.Lifetime[0]
            Hosting environment: Production
      info: Microsoft.Hosting.Lifetime[0]
            Content root path: /app

      ```

      参数说明：

      ```
      docker run：启动一个容器实例
      -p 8000:5100：将本地端口8000映射到容器端口5100
      192.168.1.110:5000/aspnetcoreapp:v1： 表示完整的镜像路径

      ```

      我们也可以通过 `-d` 参数启动后台运行，也可以通过设置 `--restart=always` 使其自动启动，小伙伴们可以自行尝试下。

6. 使用浏览器，外部验证是否可以正常访问

      使用浏览器打开：http://192.168.1.110:8000。如果能正常打开，那么恭喜你已经成功了。

      Docker的最简易的部署就到这里了，基本思路有了，更复杂点的部署也会有个基本的思路了。