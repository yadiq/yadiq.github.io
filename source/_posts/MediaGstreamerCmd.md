---
title: Gstreamer命令和语法
date: 2022-08-16 21:26:21
tags: 音视频
categories: 音视频
---

> [Gstreamer](https://gstreamer.freedesktop.org)是一个非常强大的、多平台的用于开发流媒体应用程序的框架。GStreamer框架的许多优点都来自于它的模块化。

## 1.三个命令行工具
st-inspect-1.0，gst-discoverer-1.0，gst-launch-1.0

//列出Element的详细信息
gst-inspect-1.0 jpegdec

//查看媒体文件的编码，帧率等信息
gst-discoverer-1.0 test1.mp4

//执行Pipline，快速的检查Pipeline中各个元素是否能够正确的连接起来。
//例如，播放测试视频
gst-launch-1.0 videotestsrc ! videoconvert ! autovideosink

## 2.常用命令
1. 播放
//音频
gst-launch-1.0 audiotestsrc ! audioconvert ! audioresample ! autoaudiosink
//视频
gst-launch-1.0 videotestsrc ! warptv ! videoconvert ! autovideosink
//本地视频
gst-play-1.0 test1.mp4
gst-launch-1.0 filesrc location=test1.mp4 ! qtdemux ! queue ! decodebin ! autovideosink
//网络视频 playbin
gst-launch-1.0 playbin uri=https://www.freedesktop.org/software/gstreamer-sdk/data/media/sintel_trailer-480p.ogv
gst-launch-1.0 playbin uri=udp://192.168.10.126:25000
//无人机rtsp
gst-launch-1.0 playbin uri=rtsp://admin:12345@192.168.10.138:8554/streaming/live/1
//服务端车视频udp转发到pc(明确：rtp协议 H265编码)
gst-launch-1.0 udpsrc port=10000 ! application/x-rtp, media=video, clock-rate=90000, encoding-name=H265 ! rtpjitterbuffer latency=1000 drop-on-latency=true ! decodebin ! videoconvert ! autovideosink sync=false

2. 直播
//rtsp推流
gst-launch-1.0 videotestsrc ! x264enc ! rtph264pay name=pay0 pt=96
//rtsp拉流
gst-launch-1.0 playbin uri=rtsp://127.0.0.1:8554/test
//rtsp拉流
gst-launch-1.0 rtspsrc location=rtsp://admin:12345@192.168.10.191:8554/streaming/live/1 ! decodebin ! videoconvert ! autovideosink sync=false

3. 分支两路
tee分支元素。将数据拆分为多个pads端口
queue队列元素。在每个分支中使用，为每个分支创建单独的线程。

例如：播放音频，并使用Goom元素渲染可视化
 gst-launch-1.0 filesrc location=test1.mp3 ! decodebin 
 ! tee name=t ! queue ! audioconvert ! audioresample ! autoaudiosink 
 t. ! queue ! audioconvert ! goom ! videoconvert ! autovideosink

例如：播放并录制
udpsrc port=10000 ! application/x-rtp, media=video, clock-rate=90000, encoding-name=H265 ! rtpjitterbuffer latency=1000 drop-on-latency=true 
 ! tee name=t ! queue ! decodebin3 ! videoconvert ! autovideosink sync=false
 t. ! queue ! rtph265depay ! mpegtsmux ! filesink location = ExternalStorageDirectory/name.mp4

## 3.语法
pipeline 管道：
  source=>filter=>sink
  特殊的容器bin, 用于包含其它元素，负责时钟和消息传递
element 元素：
  特殊的GObject,包含属性
  source 生产数据，例如 Videotestsrc
  sink  消费数据，例如 autovideosink
  //palybin 单个元素组成的管道
  audioconvert 音频格式转换
  audioresampe 音频重采样

管道总线 
  Gstbug 管道总线
  Gstmessage 信息结构体，不同类型提供不同的解析功能
  gst_bus_timed_pop_filtered() 同步地从总线提取消息
  gst_bus_add_watch() 总线观察者
  gst_bus_add_signal_watch()
  GSignals 信号
  g_signal_connect() 异步地为GObject连接信号和回调
  pad_added_handler() 信号处理函数，回调
元素通信端口
  GstPad 通信端口
  demuxer 分离器

其它：
pa

函数：
0：
  回调需要提前声明
demo1:例子
  gst_parse_launch("",NULL) 快捷使用管道
  gst_element_set_state(pipe, state) 修改管道状态
  gst_element_get_bus(pipe) 检索管道的总线
demo2:概念
  gst_element_factory_make(type,name) 创建元素
  gst_pipeline_new() 创建空管道
  gst_bin_add_many() 将元素添加到管道
  gst_element_link() 连接元素
  g_object_set() 设置元素属性
demo3:动态管道，手动连接端口
  gst_element_get_static_pad() 获取端口
  gst_pad_link() 连接两个端口
demo7:多线程 
source tee 
  queue AudioConvert AudoResample AudioSink
  queue WaveScope VideoConvert VideoSink
demo1:自定义playbin，选择音轨
  g_main_loop_new() 为了交互创建了Loop,可调用回调
    g_bus_add_watch() 解析文件信息
    g_io_add_watch() 检测输入字符