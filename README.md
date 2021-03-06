本文告诉大家如何使用这个博客主题搭建自己的博客。这个主题是由 [吕毅 - walterlv](https://walterlv.github.io/ )基于[hcz-jekyll-blog](https://codeasashu.github.io/hcz-jekyll-blog/) 修改出来的，可以用于手机端和pc端。

<!--more-->
<div id="toc"></div>
<!-- csdn -->

博客支持搜索，适配手机访问，支持评论和类别。修改很是简单，只需要修改一个属性就可以搭建。

本文搭建博客使用 oschina 代码托管为例，实际上的其他网站搭建也一样。

## 创建项目

第一步是创建一个项目，我下面创建一个叫 Foo 的项目。

![](http://7xqpl8.com1.z0.glb.clouddn.com/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F20171015103919.jpg)

注意不要选择`使用Readme文件初始化这个项目`

## 下载博客

博客通过[https://github.com/iip-easi/Theme](https://github.com/iip-easi/Theme)进行下载

当然复制下载需要使用 git 或者直接点击压缩包，下面告诉大家使用 git 的方法

```csharp
cd xx 到新建博客的文件夹
git clone https://github.com/iip-easi/Theme.git
```

然后可以看到下载好了文件夹

接着在 git 删除远程，使用下面的代码，假设你创建的项目地址是 `https://gitee.com/lindexi/Foo.git` ，请把代码的 https://gitee.com/lindexi/Foo.git 修改为你创建项目的地址

```csharp
git remote remove origin

git remote add origin https://gitee.com/lindexi/Foo.git
```

## 启动服务

接下来就是做一些修改让自己的博客可以跑，打开`_config.yml`可以看到`baseurl: "/Theme"`，尝试把`/Theme`修改为自己创建项目的名称，这里使用是`Foo`。如果项目有大写，那么还需要把大写字符转小写

```csharp
baseurl: "/foo"
```

然后提交，打开 git 输入

```csharp
git add .
git commit -m "添加博客"
git push origin master
```

打开 gitee 可以看到服务里有 Page ，点击他

![](http://7xqpl8.com1.z0.glb.clouddn.com/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F20171015104927.jpg)

![](http://7xqpl8.com1.z0.glb.clouddn.com/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F20171015105014.jpg)

请等待一下就可以看到搭建好了，尝试访问一下。可以看到博客可以访问，如果修改了还出现无法访问，那么请联系我

如果出现样式找不到，那么检查一下自己的网站，项目是否因为字符大小写错误。

## 修改信息

接下来就是修改自己的信息

把`title` `author` 都换成自己的，这样就好了，其中`logo` 就是网站图片，请把图片修改为自己的地址

除了这些之外，其他暂时可以不用修改，直接把自己的博客写在 `_post` 文件夹。可以看着我里面的文件就知道文件格式。

这时可以看到首页还有一些地方没有修改，请打开`index.html`等进行修改，把我的名字修改为你的名字。

 - footer.html cnzz统计，以及版权

 - index.html 首页显示

 - social.json 社交账号，包括推特、github还有其他

## 评论

博客使用的评论是 [disqus](https://disqus.com) ，请自己去申请账号，然后在`_config.yml`写自己的名字，这样就好啦。

如果有什么问题，欢迎联系我 237059917@qq.com 

参见：[胡承 - huchengv5](https://huchengv5.github.io/ )

[简单搭建自己的博客](https://lindexi.gitee.io/lindexi//post/%E7%AE%80%E5%8D%95%E6%90%AD%E5%BB%BA%E8%87%AA%E5%B7%B

