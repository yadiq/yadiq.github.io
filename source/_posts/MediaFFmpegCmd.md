---
title: FFmpeg命令和语法
date: 2022-08-22 21:26:21
tags: 音视频
categories: 音视频
---

## 1.基础命令
ffprobe /home/gyd/test1.mp4 //解析一个媒体文件
ffmpeg -i test.265 //检查 test.265 媒体信息 
ffplay test.265 //播放
ffplay rtmp://media.weihuhao.cn:1935/aaaa/12345 //播放
ffplay -fflags nobuffer -i rtmp://zlm.aerial-photography.live.epplink.com/aaaaa/cccccc //播放 低延迟
ffplay -f hevc udp://192.168.10.126:10000 //播放 
ffmpeg -i test.265 -acodec copy -vcodec copy test_h265.mp4 //封装到mp4
ffmpeg -i input.mp4 output.webm //转换文件格式
ffmpeg -i input.mp4 -c:v libvpx -c:a libvorbis output.webm //转换文件且指定编解码器

//推流视频文件 (推流到斗鱼服务器 ok)
ffmpeg -re -i /home/test/test1.mp4 -vcodec libx264 -acodec aac -f flv "rtmp://sendtc3.douyu.com/live/1697710rdLDqrlFg"
  -re 控制读取 AVpacket 的速度，按照帧率速度读取文件
  -i：表示输入视频文件，后跟视频文件路径/URL
  -vcodec：指定视频解码器，后跟解码器名称，copy表示不作解码；
  -acodec：指定音频解码器，后跟解码器名称。an代表acodec none就是去掉音频的意思。
  -f：强制ffmpeg采用某种格式，后跟对应的格式

//推流音频文件
ffmpeg -re -i test1.mp3 -c copy -f flv "rtmp://media.weihuhao.cn/aaaa/12345?sign=5b0d581dce68c30985bd6b67a87824ad"

## 2.拉流rtp
//拉流rtsp
ffplay rtsp://admin:12345@192.168.10.138:8554/streaming/live/1

//拉流rtp
//  不使用sdp文件
ffplay -protocol_whitelist "file,udp,rtp" -i rtp://192.168.10.213:10000
//  使用sdp文件
ffplay -protocol_whitelist file,udp,rtp -i test.sdp
//  sdp文件，详见 RTP推流&sdp文件
v=0
o=- 0 0 IN IP4 192.168.10.213
s=No Name
c=IN IP4 192.168.10.213
t=0 0
a=tool:libavformat 56.15.102
m=video 10000 RTP/AVP 96
a=rtpmap:96 H265/90000

## 3.拉流低延迟
https://stackoverflow.com/questions/16658873/how-to-minimize-the-delay-in-a-live-streaming-with-ffmpeg
-analyzeduration 分析媒体文件的持续时间，默认5s
-probesize 分析媒体文件时读取的数据大小,预设值为50,000字节
-fflags nobuffer 探测的数据包不缓存 ok
-flags low_delay  降低延迟 有效
-framedrop 默认开启 如果视频不同步，则丢弃视频帧
-rtsp_transport tcp rtp不排序

ffplay -fflags nobuffer -flags low_delay -framedrop -protocol_whitelist file,udp,rtp -i test1.sdp

ffplay -i rtsp://media.weihuhao.cn:554/rtp/34020000001320000077_34020000001320000001 -fflags nobuffer -analyzeduration 1000000 -rtsp_transport tcp

ffplay -i rtmp://media.weihuhao.cn:1935/aaaa/12345 -fflags nobuffer 

## 4.高级命令
//枚举设备
ffmpeg -f dshow -list_devices true -i dummy
  "Integrated Camera"
  "麦克风阵列 (Realtek High Definition Audio)"
//展示摄像头
ffplay -f dshow -video_size 1280x720 -i video="Integrated Camera"
ffplay -f vfwcap -i 0
//播放音频
ffplay -f dshow -i audio="麦克风阵列 (Realtek High Definition Audio)"

//查询本机音视频设备
ffmpeg -list_devices true -f dshow -i dummy
//摄像头推流
ffmpeg -f dshow -i video="Integrated Camera" -vcodec libx264 -f rtsp rtsp://127.0.0.1:8554/video
//摄像头推流-低延迟
ffmpeg -f dshow -i video="Integrated Camera" -vcodec libx264 -preset:v ultrafast -tune:v zerolatency -f rtsp rtsp://127.0.0.1:8554/video

## 5.rtp推流 sdp文件
//rtp推流 播流
ffmpeg -re -i rayna.h264 -vcodec copy -f rtp rtp://30.40.37.23:3000>test.sdp
ffplay -i test.sdp -protocol_whitelist file,udp,rtp

//推流
ffmpeg -re -i mm.mp4 -vcodec libx264 -an -f rtp rtp://127.0.0.1:20001
vim b.sdp
https://blog.csdn.net/MACMACip/article/details/106425380

//播放
ffplay a.sdp -protocol_whitelist file,crypto,udp,rtp

## 6.推流服务器
1. 方法
https://blog.csdn.net/EthanCo/article/details/125321957
2. 安装rtsp服务器：
https://github.com/bluenviron/mediamtx/releases
运行rtsp-simple-server.exe
3. 关闭防火墙
4. ffmpeg推流&app拉流