---
title: "Bitmap与BitmapSource的互转"
author: 胡承
date: 2020-09-29 11:35:3 +0800
CreateTime: 2020-09-29 11:35:3 +0800
categories: C# .NET
---

使用WPF过程中，有些时候需要调用系统的一些接口，必须传入GDI+所支持的图片类型，也是winform支持的图片类型，这个时候我们就需要做一个转换了。

<!-- more -->

请看主要代码，图片格式看情况自行设定。通常使用jpg比较好，省空间；如果支持透明就用png吧。

```cs

        public static Bitmap BitmapSourceToBitmap(BitmapSource source)
        {
            if (source == null) return null;
            using var stream = new MemoryStream();
            var jpegBitmapEncoder = new JpegBitmapEncoder();
            jpegBitmapEncoder.Frames.Add(BitmapFrame.Create(source));
            jpegBitmapEncoder.Save(stream);
            var bmp = new Bitmap(stream);
            return bmp;
        }

        public static BitmapSource BitmapToBitmapSource(Bitmap bitmap)
        {
            if (bitmap == null) return null;
            var ms = new MemoryStream();
            bitmap.Save(ms, ImageFormat.Png);
            var image = new BitmapImage();
            image.BeginInit();
            //有些格式你会发现打不开，因为像素有丢失，没关系加上这个配置即可。
            //image.CreateOptions=BitmapCreateOptions.IgnoreColorProfile;
            image.StreamSource = ms;
            image.EndInit();
            return image;
        }
```

