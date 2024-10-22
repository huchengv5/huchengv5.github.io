---
title: "使用FFMpeg.AutoGen播放视频"
author: 胡承
date: 2024-10-22 09:12:3 +0800
CreateTime: 2024-10-22 09:12:3 +0800
categories: C# WPF
---

前面我们已经简单介绍过FFMpeg.AutoGen，今天来看下怎么使用它来实现MP4视频格式的播放。

<!-- more -->
#### 说明：ffmpeg软解码和硬解码

软解码：指的是使用CPU进行对封装的音视频数据做解码。其优势是有更好的兼容性，不依赖GPU的算力，也不存在内存和显存直接的数据交换，在CPU的性能比较好的机器上能有更好的综合性能表现。如果代码实现过程中需要对解码出来的数据做二次处理，那么更推荐使用软解码。

硬解码：指的是使用GPU进行对封装的音视频数据做解码。使用硬解码过程中，主要消耗GPU资源。其优势是能充分使用显卡资源，提升视频播放的流畅度。如果GPU性能比较差的机器，用软解码或能获得更好的视频播放体验。

### 环境准备

1. 安装FFMpeg.AutoGen nuget包，版本只需要跟FFMpge版本号对应上就好。ffmpeg版本需要注意x86和x64的版本。

2. 开启"允许使用unsafe编译代码"

本示例是基于"FFmpeg.AutoGen 6.1.0.1"和"FFmpeg 6.0"版本，以软解码实现方式进行演示

### 实现步骤

1. 初始化解码上下文

      由于ffmpeg是非托管代码实现，其中的内存需要我们手动释放。为了方便的管理这些对象，我们自定义一个解码上下文的类，主要用于内存复用和释放。具体定义如下：
      
```cs

/// <summary>
/// 解码上下文
/// </summary>
public unsafe class MediaCodecContext
{
    /// <summary>
    /// 视频播放总时长，单位为秒
    /// </summary>
    public double Duration;
    /// <summary>
    /// 时间基
    /// </summary>
    public AVRational AVStreamTimeBase;
    /// <summary>
    /// 帧率
    /// </summary>
    public int Fps;
​
    /// <summary>
    /// 缩放比
    /// </summary>
    public double Scale;
​
    /// <summary>
    /// 
    /// </summary>
    public int Width;
​
    /// <summary>
    /// 
    /// </summary>
    public int Height;
​
    /// <summary>
    /// 视频流的索引
    /// </summary>
    public int VideoStreamIndex;
​
    /// <summary>
    /// 音频流的索引
    /// </summary>
    public int AudioStreamIndex;
​
    /// <summary>
    /// 
    /// </summary>
    public int PixelFormat;
​
    /// <summary>
    /// 音/视频上下文
    /// </summary>
    public AVFormatContext* FormatContext;
​
    /// <summary>
    /// 视频解码器上下文
    /// </summary>
    public AVCodecContext* VideoCodecContext;
​
    /// <summary>
    /// 音频解码上下文
    /// </summary>
    public AVCodecContext* AudioCodecContext;
​
    /// <summary>
    /// 复用数据包
    /// </summary>
    public AVPacket* Packet;
​
    /// <summary>
    /// 加载的文件路径
    /// </summary>
    public string FileName = string.Empty;
​
    /// <summary>
    /// 当前播放的索引
    /// </summary>
    public int CurrentFrameIndex;
​
    /// <summary>
    /// 释放资源占用,重置上下文
    /// </summary>
    public void Free()
    {
        var packet = Packet;
        if (packet != null)
        {
            ffmpeg.av_packet_unref(packet);
            ffmpeg.av_packet_free(&packet);
            Packet = packet;
        }
​
        var videoCodecContext = VideoCodecContext;
        if (videoCodecContext != null)
        {
            ffmpeg.avcodec_free_context(&videoCodecContext);
            VideoCodecContext = videoCodecContext;
        }
​
        var audioCodecContext = AudioCodecContext;
        if (audioCodecContext != null)
        {
            ffmpeg.avcodec_free_context(&audioCodecContext);
            AudioCodecContext = audioCodecContext;
        }
​
        var formatContext = FormatContext;
        if (formatContext != null)
        {
            ffmpeg.avformat_close_input(&formatContext);
            ffmpeg.avformat_free_context(formatContext);
            FormatContext = formatContext;
        }
        CurrentFrameIndex = 0;
        AudioStreamIndex = -1;
        VideoStreamIndex = -1;
    }

```

2. 查找并打开解码器


```cs

/// <summary>        
/// 打开文件，读取音/视频流，获取解码器，打开解码器
/// </summary>
/// <param name="filePath"></param>
public unsafe bool Open()
{
    //分配解码上下文，该上下文贯穿整个解码过程，包含封装格式，音视频流信息，时长等
    var formatContext = ffmpeg.avformat_alloc_context();
    //打开文件，并读取文件头部
    var res = ffmpeg.avformat_open_input(&formatContext, _context.FileName, null, null);
    if (res != 0)
    {
        //打开失败，释放资源
        ffmpeg.avformat_free_context(formatContext);
        //Log.Error($"avformat_open_input:{res}");
        return false;
    }
​
    //查找音/视频流
    if ((res = ffmpeg.avformat_find_stream_info(formatContext, null)) != 0)
    {
        ffmpeg.avformat_free_context(formatContext);
        //Log.Error($"avformat_find_stream_info:{res}");
        return false;
    }
​
    //枚举音/视频流
    AVStream* videoStream = null, audioStream = null;
    for (var i = 0; i < formatContext->nb_streams; i++)
    {
        var codecpar = formatContext->streams[i]->codecpar;
        if (codecpar->codec_type == AVMediaType.AVMEDIA_TYPE_VIDEO)
        {
            videoStream = formatContext->streams[i];
            _context.VideoStreamIndex = i;
        }
        else if (codecpar->codec_type == AVMediaType.AVMEDIA_TYPE_AUDIO)
        {
            audioStream = formatContext->streams[i];
            _context.AudioStreamIndex = i;
        }
    }
​
    if (videoStream == null && audioStream == null)
    {
        ffmpeg.avformat_free_context(formatContext);
        //Log.Warn($"ffmpeg:检测不到音视频流");
        return false;
    }
​
    if (audioStream != null)
    {
        // 初始化音频解码器
        if (!InitializeAudioDecoder(audioStream->codecpar))
        {
            ffmpeg.avformat_free_context(formatContext);
            //Log.Error($"ffmpeg:初始化音频解码器失败");
            return false;
        }
    }
​
    if (videoStream != null)
    {
        // 初始化视频解码器
        if (!InitializeVideoDecoder(videoStream->codecpar))
        {
            ffmpeg.avformat_free_context(formatContext);
            //Log.Error($"ffmpeg:初始化视频解码器失败");
            return false;
        }
    }
    
    //初始化解码过程中的复用数据包，为av_packet申请内存，用于存放每一帧的封装的数据包
    var packet = ffmpeg.av_packet_alloc();
    var fps = videoStream->avg_frame_rate.num / videoStream->avg_frame_rate.den;
​
    #region 初始化上下文
​
    _context.FormatContext = formatContext;
    _context.Duration = videoStream->duration * ffmpeg.av_q2d(videoStream->time_base);
    _context.Packet = packet;
    _context.Fps = fps;
    _context.Scale = 1;
​
    #endregion
​
    return true;
}
​
/// <summary>
/// 初始化视频解码器
/// </summary>
/// <param name="codecParameter"></param>
/// <returns></returns>
private unsafe bool InitializeVideoDecoder(AVCodecParameters* codecParameter)
{
    //查找解码器
    var codec = ffmpeg.avcodec_find_decoder(codecParameter->codec_id);
    //为解码器分配一个上下文
    var codecContext = ffmpeg.avcodec_alloc_context3(codec);
    //填充解码器上下文参数
    var res = ffmpeg.avcodec_parameters_to_context(codecContext, codecParameter);
    if (res != 0)
    {
        //释放上下文资源
        ffmpeg.avcodec_free_context(&codecContext);
        return false;
    }
    //打开解码器
    res = ffmpeg.avcodec_open2(codecContext, codec, null);
    if (res != 0)
    {
        ffmpeg.avcodec_free_context(&codecContext);
        return false;
    }
​
    _context.VideoCodecContext = codecContext;
    _context.Width = codecParameter->width;
    _context.Height = codecParameter->height;
    _context.PixelFormat = codecParameter->format;
​
    return true;
}
​
//初始化音频解码器(暂不做实现)
private unsafe bool InitializeAudioDecoder(AVCodecParameters* codecParameter)
{
    //音频暂不做实现
    return true;
}

```

3. 读取帧数据，并进行解码和转码

```cs

/// <summary>
/// 读取视频帧数据
/// </summary>
/// <returns></returns>
private unsafe VideoFrame ReadVideoFrame()
{
    var data = new VideoFrame();
    //将av_packet数据发送给解码器
    var res = ffmpeg.avcodec_send_packet(_context.VideoCodecContext, _context.Packet);
    //为av_frame结构体分配内存
    var frame = ffmpeg.av_frame_alloc();
    frame->width = _context.Width;
    frame->height = _context.Height;
    frame->format = _context.PixelFormat;
    //根据像素的宽高，给av_frame分配实际占用的内存
    ffmpeg.av_frame_get_buffer(frame, 0);
    //将解码后的数据填充到av_frame
    res = ffmpeg.avcodec_receive_frame(_context.VideoCodecContext, frame);
    if (res != 0)
    {
        data.State = FrameState.Continue;
        //减少av_frame的引用计数，防止内存泄露
        ffmpeg.av_frame_unref(frame);
        //释放av_frame的内存占用
        ffmpeg.av_frame_free(&frame);
        //减少av_packet的引用计数，防止内存泄露
        ffmpeg.av_packet_unref(_context.Packet);
        return data;
    }
​
    _context.CurrentFrameIndex++;
​
    var scale = _context.Scale;
    //将原始帧转码成BGRA，方便使用WPF进行渲染
    AVFrame* originalFrame = ConvertToBGRA(frame, (int)(scale * _context.Width), (int)(scale * _context.Height));
    //将av_frame转成二进制，传递给wpf
    var pixels = CropFrameToPixels(originalFrame, originalFrame->linesize[0], new Int32Rect(0, 0, originalFrame->width, originalFrame->height));
    //显示用的二进制像素数据    
    data.Pixels = pixels;
    //每一行像素所占用的字节数
    data.Pitch = originalFrame->linesize[0];
    //像素宽度
    data.Width = originalFrame->width;
    //像素高度
    data.Height = originalFrame->height;
    data.State = FrameState.OK;
    //释放av_frame的内存占用
    ffmpeg.av_frame_unref(frame);
    ffmpeg.av_frame_free(&frame);
    ffmpeg.av_frame_unref(originalFrame);
    ffmpeg.av_frame_free(&originalFrame);
    ffmpeg.av_packet_unref(_context.Packet);
    return data;
}
​
/// <summary>
/// 视频帧数据结构
/// 定义为结构体是为了提升性能
/// </summary>
public struct VideoFrame : IMediaFrame
{
    /// <summary>
    /// 渲染数据的宽度
    /// </summary>
    public int Width;
​
    /// <summary>
    /// 渲染数据的高度
    /// </summary>
    public int Height;
​
    /// <summary>
    /// 渲染数据的像素信息
    /// 视频数据表示 Pixels，音频数据表示 pcm
    /// </summary>
    public byte[] Pixels;
​
    /// <summary>
    /// 每行占用的字节数
    /// </summary>
    public int Pitch;
​
    /// <summary>
    /// 当前帧的状态
    /// </summary>
    public FrameState State { get; set; }
}

```

4. 转码成可显示的数据格式

大部分音视频数据解码出来后都是YUV格式的，我们需要将他们转码成BRGA格式，方便我们使用WPF进行渲染。

```cs

/// <summary>
/// 转码成BGRA
/// </summary>
/// <param name="input"></param>
/// <param name="outputWidth"></param>
/// <param name="outputHeight"></param>
/// <returns></returns>
public unsafe AVFrame* ConvertToBGRA(AVFrame* input, int outputWidth, int outputHeight)
{
    //创建转码上下文
    SwsContext* swsContext = ffmpeg.sws_getContext(input->width, input->height, (AVPixelFormat)input->format, outputWidth, outputHeight, AVPixelFormat.AV_PIX_FMT_BGRA, ffmpeg.SWS_FAST_BILINEAR, null, null, null);
    AVFrame* output = ffmpeg.av_frame_alloc();
    output->format = (int)AVPixelFormat.AV_PIX_FMT_BGRA;
    output->width = outputWidth;
    output->height = outputHeight;
    var res = ffmpeg.av_frame_get_buffer(output, 0);
    //开始转码
    res = ffmpeg.sws_scale(swsContext, input->data, input->linesize, 0, input->height, output->data, output->linesize);
    //释放资源
    ffmpeg.sws_freeContext(swsContext);
    return output;
}
​
/// <summary>
/// 裁剪帧到像素集合
/// </summary>
/// <param name="srcFrame">原始BGRA帧</param>
/// <param name="targetLineSize">裁剪后，一行包含多少像素</param>
/// <param name="region">裁剪的区域大小</param>
/// <returns></returns>
public unsafe byte[] CropFrameToPixels(AVFrame* srcFrame, int targetLineSize, Int32Rect region)
{
    int frameStride = srcFrame->linesize[0];
    var size = targetLineSize * region.Height;
    byte[] pixels = new byte[size];
    for (int height = 0; height < region.Height; ++height)
    {
        int srcStartIndex = (height + region.Y) * frameStride + region.X * 4;
        int targetStartIndex = height * targetLineSize;
        var srcStartPointer = &srcFrame->data[0][srcStartIndex];
        Marshal.Copy((IntPtr)srcStartPointer, pixels, targetStartIndex, targetLineSize);
    }
    return pixels;
}

```
5. 渲染播放

在WPF中，我们可以通过使用WriteableBitmap来实现对BGRA数据格式的显示。

```cs

//自定义播放器
public MediaPlayer()
{
    _child = new DrawingVisual();
    AddVisualChild(_child);
}
​
protected override Visual GetVisualChild(int index)
{
    return _child;
}
​
protected override int VisualChildrenCount => 1;
​
//需要渲染的帧，这里复用提升性能
private WriteableBitmap _frame;
​
/// <summary>
/// 绘制视频帧
/// </summary>
/// <param name="data"></param>
private void Draw(VideoFrame data)
{
    using (var drawingContext = _child.RenderOpen())
    {
        if (_frame == null || _frame.Width != data.Width || _frame.Height != data.Height)
        {
            _frame = new WriteableBitmap(data.Width, data.Height, 96, 96, PixelFormats.Pbgra32, null);
        }
​
        if (_frame.TryLock(new Duration(TimeSpan.FromMilliseconds(50))))
        {
            try
            {
                _frame.WritePixels(new Int32Rect(0, 0, data.Width, data.Height), data.Pixels, data.Pitch, 0);
                drawingContext.DrawImage(_frame, new Rect(0, 0, data.Width, data.Height));
            }
            catch (Exception ex)
            {
​
            }
            finally
            {
                _frame.Unlock();
            }
        }
    }
}

```


6. 播放控制

要想视频动起来，我们就需要写一个循环，让他针对每一帧进行播放，示例代码如下：

```cs

Task.Factory.StartNew(() =>
{
    _decoder.Load(filename);
    if (!_decoder.Open())
    {
        _decoder.Close();
        return;
    }
    IsPlaying = true;
    var startTime = Environment.TickCount;
    var delaySpan = 1000 / _decoder.Fps;
​
    while (IsPlaying)
    {
        _decoder.SetDisplaySize(RenderSize.Width, RenderSize.Height);
        var data = _decoder.ReadFrame();
        if (data.State == FrameState.OK)
        {
            if (data is VideoFrame videoFrame)
            {
                Dispatcher.InvokeAsync(() =>
                {
                    Draw(videoFrame);
                });
            }
            else
            {
                continue;
            }
        }
        else if (data.State == FrameState.Over)
        {
            if (data is VideoFrame videoFrame)
            {
                Dispatcher.InvokeAsync(() =>
                {
                    Draw(videoFrame);
                });
            }
            else
            {
                break;
            }
        }
        else
        {
            continue;
        }
​
        var decodeTime = Environment.TickCount - startTime;
        var delay = Math.Max(0, delaySpan - decodeTime);
        Thread.Sleep(delay);
        startTime = Environment.TickCount;
    }
    _decoder.Close();
    IsPlaying = false;
});

```


写在最后

示例代码并非完整代码，大家在实现视频播放功能的时候，还是需要考虑不少细节，上述代码仅供参考！过程中还有什么疑问可以在评论区留言。


博客地址：https://huchengv5.github.io/

微信公众号：

![承哥技术交流小作坊](https://i.loli.net/2021/09/27/FmsaLU1Oo7tX8kl.jpg)

**欢迎转载分享，如若转载，请标注署名。**

