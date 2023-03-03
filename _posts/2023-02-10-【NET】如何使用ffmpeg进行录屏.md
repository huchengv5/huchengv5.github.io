---
title: "如何使用ffmpeg进行录屏"
author: 胡承
date: 2023-03-03 09:12:3 +0800
CreateTime: 2023-03-03 09:12:3 +0800
categories: C# WPF
---

录屏软件，我们去网上下载，发现有很多软件都是要收费的！但是录屏功能很难做吗？为啥都需要收费呢？

<!-- more -->

于是我整了个小demo，用于实现基础的屏幕录制功能。

思路很简单，考虑到`FFMpeg.exe`是一个非常成熟的音视频处理软件，而且是开源的。我们完全可以基于`FFMpeg.exe`来实现我们的要求。

使用`FFMpeg`录屏通常有两种方式：
1. 基于 `gdigrab` (使用CPU录制)
1. 基于`DirectShow` （使用显卡录制）

考虑到`gdigrab`方式实现更简单，更轻量，兼容性也更好，所以我们采用`gdigrab`方案。`DirectShow`方案需要额外下载`screen-capture-recorder`!

基于该方法，主要核心就是`ffmpeg`的命令编写，具体命令如下：

```bat

ffmpeg -f gdigrab -framerate 30 -offset_x 0 -offset_y 0 -video_size 1920x1080 -i desktop -r 30 -vcodec libx264 -pix_fmt yuv420p -y "output.mp4"

```

命令解说：

```cs

//注意：ffmpeg 的命令输入，也需要考虑输入顺序的。先指定输入参数，再指定输出参数。

// -f 表示指定文件格式或采集数据的设备，如果是 DirectShow 方案，就需要指定为 screen-capture-recorder。
    //对于输入，如果不指定-f, ffmpeg 会根据输入数据，来判断数据的封装格式.
    //对于输出，如果不指定-f， ffmpeg 也可以通过输出文件名进行推导.
    //ffmpeg -formats 可以列出所有的formats

// -framerate 指定录制的视频帧率

// -offset_x -offset_y 指定录制的区域左上角坐标

// -video_size 指定录制的视频尺寸

// -i 指定从哪儿采集数据，它是一个文件索引号。这里是指定输入的视频源。 （i 表示input，不一定是表示视频文件，表示输入。如也可以是音频文件输入等）

// -r 指定帧率

// -vcodec 指定编码格式。 libx264 这种格式录制出来的视频，清晰度更清晰，占用的文件大小更小，推荐使用。

// -pix_fmt 像素格式，常规使用yuv420p

// -y 表示强制覆盖，如果输出文件已经存在，就覆盖

// 输出文件路径。这里需要注意下：输出文件路径带上 双引号 ，防止不合法路径将命令拆分了。

```

为了更方便代码实现相关编码，下面贴上核心代码（附加音视频合并的示例代码）：

```cs
    /// <summary>
    /// 视频录制命令构建类
    /// </summary>
    public class FFMpegScreenRecordArgs : FFMpegArgs
    {
        public int Framerate { get; set; }

        public int Rate { get; set; }

        public Rect Region { get; set; }

        public VideoCodec Codec { get; set; }

        public PixelFormat PixelFormat { get; set; }

        public string SavePath { get; set; }

        public override string ToArgs()
        {
            ArgsBuilder.SetRecordDevice();
            ArgsBuilder.SetFramerate(Framerate);
            ArgsBuilder.SetRecordRect((int)Region.Left, (int)Region.Top);
            ArgsBuilder.SetVideoSize((int)Region.Width, (int)Region.Height);
            ArgsBuilder.SetInput();
            ArgsBuilder.SetRate(Rate);
            ArgsBuilder.SetCodec(Codec);
            ArgsBuilder.SetPixFarmat(PixelFormat);
            return $"{ArgsBuilder.GetArgs()} -y {SavePath}\"";
        }
    }

    /// <summary>
    /// 音视频合成构建类
    /// </summary>
    public class FFMpegAVSynthesisArgs : FFMpegArgs
    {
        public string InputAudioPath { get; set; }
        public string InputVideoPath { get; set; }
        public string OutputVideoPath { get; set; }

        public override string ToArgs()
        {
            this.ArgsBuilder.SetInput(InputAudioPath);
            this.ArgsBuilder.SetInput(InputVideoPath);
            this.ArgsBuilder.SetCodec(AudioCodec.copy);
            this.ArgsBuilder.SetCodec(VideoCodec.copy);

            return $"{ArgsBuilder.GetArgs()} \"{OutputVideoPath}\"";
        }
    }

    //基础命名构建类
    public class FFmpegArgsBuilder
    {
        protected readonly List<string> Args = new List<string>();
        public FFmpegArgsBuilder AddArg<T>(string Key, T Value)
        {
            Args.Add($"-{Key} {Value}");

            return this;
        }

        public virtual string GetArgs()
        {
            return string.Join(" ", Args);
        }

        public void Reset()
        {
            Args.Clear();
        }

        #region args

        public FFmpegArgsBuilder SetRecordRect(int x, int y)
        {
            AddArg("offset_x", x);
            AddArg("offset_y", y);
            return this;
        }

        public FFmpegArgsBuilder SetVideoSize(int width, int height)
        {
            AddArg("video_size", $"{width}x{height}");
            return this;
        }

        public FFmpegArgsBuilder SetInput(string input = "desktop")
        {
            AddArg("i", input);
            return this;
        }

        public FFmpegArgsBuilder SetFramerate(int framerate)
        {
            AddArg("framerate", framerate);
            return this;
        }

        public FFmpegArgsBuilder SetRate(int rate)
        {
            AddArg("r", rate);
            return this;
        }

        public FFmpegArgsBuilder SetRecordDevice(string device = "gdigrab")
        {
            AddArg("f", device);
            return this;
        }

        public FFmpegArgsBuilder SetCodec(VideoCodec codec)
        {            
            AddArg("vcodec", codec);
            return this;
        }

        public FFmpegArgsBuilder SetCodec(AudioCodec audioCodec)
        {
            AddArg("acodec", audioCodec);
            return this;
        }

        public FFmpegArgsBuilder SetPixFarmat(PixelFormat x)
        {
            AddArg("pix_fmt", x);
            return this;
        }
        
        #endregion
    }

    //像素格式
    public enum PixelFormat
    {
        ///<summary>
        /// yuv420p
        ///</summary>
        yuv420p,
        //……省略……
    }

    //编码格式
    public enum VideoCodec
    {
        ///<summary>
        ///      libx264
        ///</summary>
        libx264,

        ///<summary>
        ///      copy
        ///</summary>
        copy,
        //……省略……
    }

    /// <summary>
    ///     Audio codec ("ffmpeg -codecs")
    /// </summary>
    public enum AudioCodec
    {
        ///<summary>
        ///      copy
        ///</summary>
        copy,
    }

```


其它文献：
1. https://www.cnblogs.com/tuyile006/p/12572656.html
1. https://www.jianshu.com/p/f15fe6f7202f
1. https://blog.csdn.net/a457636876/article/details/107857674

微信公众号：

![承哥技术交流小作坊](https://i.loli.net/2021/09/27/FmsaLU1Oo7tX8kl.jpg)

**欢迎转载分享，如若转载，请标注署名。**