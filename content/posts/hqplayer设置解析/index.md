---
title: hqplayer设置解析
id: bd98f916-37cf-4be5-9ede-493473a66ced
date: 2025-09-21 05:44:30
auther: squaly
featured_image_preview: Pasted%20image%2020261023204647.png
featured_image: Pasted%20image%2020261023204647.png
summary: 1、设置Backend（后端） 如果需要本地播放选择ALSA，如果需要通过网络进行NAA推送播放，则选择Network Audio 2、设置Output mode（音频输出模式） Auto是根据音频本身的格式，输出PCM或者SDM。但如果希望强制音频转换成PCM或者SDM，可以选择下面两项强制输出。
permalink: /archives/FdvCMgQt
categories:
  - media
tags:
  - hqplayer
---


1、设置Backend（后端）

![](Pasted%20image%2020261023204230.png)


如果需要本地播放选择ALSA，如果需要通过网络进行NAA推送播放，则选择Network Audio

## 2、设置Output mode（音频输出模式）
![](Pasted%20image%2020261023204312.png)


Auto是根据音频本身的格式，输出PCM或者SDM。但如果希望强制音频转换成PCM或者SDM，可以选择下面两项强制输出。

## 3、Fixed volume（绝对音量）

![68cf2e2c37ed8.png](https://photo.aaza.top:666/i/2025/09/21/68cf2e2c37ed8.png)

该选项可以将音量锁定在某一个响度，非必要选项。如果不希望数字音量调节，可以勾选“enabled”（开启）并填入数字“0”。

## 4、Max volume/Min volume（最大音量/最小音量）

![](Pasted%20image%2020261023204333.png)


该选项可以自行决定音量响度的刻度尺，设定最小和最大两个数值，后面就可以在这个规定的范围内进行调节。如果打开了上面的选项，则此选项不生效。

## 5、Startup volume （启动音量）

![](Pasted%20image%2020261023204346.png)


本选项相当于音乐开头提供一个衰减，以降低对人耳的刺激，对音质无影响，可以不设置。

## 6、Adaptive volume（适应性音量）

![](Pasted%20image%2020261023204359.png)

自动平衡各音频的响度，保证每首曲子的音量基本一致，可能对音质有负面影响，建议不开启。

## 7、PCM gain compensation（PCM增益补偿）

![](Pasted%20image%2020261023204420.png)

播放PCM音频时，为每首曲子固定增加的音量，，可能对音质有负面影响，建议不开启。

## 8、Channels（声道）

![](Pasted%20image%2020261023204449.png)

选择输出声道数量，一般填写“2”，即两声道。如果是多声道音频，在硬件条件允许情况下，需要对此进行配对设置。

## 9、Options（其它设置）

![](Pasted%20image%2020261023204501.png)

Auto rate family：根据音频原采样率进行倍数升频，如果不选，则是根据设置的最高采样率进行升频。建议一定要开启，以保证不会出现SRC偏差。

Quick pause：快速暂停，允许在播放音频时迅速启动暂停/重新启动播放的功能。

Short buffer：短缓冲，可以有效降低播放延迟，但可能影响最终音质。除非是观看视频时出现明显音画不同步的情况，一般听音乐不选择此选项。

## 10、Log file（日志文件）

记录HQE系统运行时的日志文件，一般用户无需理会该选项，甚至可以选择关闭。

## 11、DSD sources（DSD信号源）

![](Pasted%20image%2020261023204517.png)

该部分为DSD信号源部分的设置：

Direct SDM：直接输出SDM信号，关闭升频。

Gain +6 dB：为所有DSD音频信号默认追加6dB响度。

Intergrator：低通滤波器，当前提供了5种可选，请根据听感自行选择。

SDM-SDM Conversion：DSD升频滤波设置，当前有wide和narrow两种选择。

![68cf2fb78077a.png](https://photo.aaza.top:666/i/2025/09/21/68cf2fb78077a.png)

Noise filter：噪声滤波器，当前有9种可选，请根据实际听感选择。

SDM-PCM Conversion：DSD升频PCM滤波选择，当前有14个可选，请根据实际听感选择。

## 12、PCM settings/SDM settings（PCM播放设置/SDM播放设置）

![](Pasted%20image%2020261023204535.png)

12-3 Filter Nx意思是原生HIRES级别音源采用的滤波。

12-4 Dither是抑制抖动的选项，选择听感合适的就行。

12-5 Sample rate是PCM采样率设置，根据自己解码最高数据进行设置。

12-6 Oversampling 1x是DSD64 不升频时使用的滤波，根据自己听感选择喜欢的就行，听不到区别的就别纠结了。

12-7 Oversampling Nx意思是DSD升频时使用的滤波 ，根据自己听感选择喜欢的就行，听不到区别的就别纠结了。另外升频会占用大量CPU资源，而CPU在极高频运作的时候，会明显增大jitter，简单来说，CPU不够强的就量力而行。

12-8 Modulator 同Dither类似，选择合适的选项就行。

12-9 Bit rate是DSD采样率设置，根据自己解码最高数据进行设置。

## 13、ALSA backend（ALSA后端设置/本地播放设置）

![](Pasted%20image%2020261023204549.png)

Device：输出设备选择

Channel offset：通道偏移，不知道有什么用，默认为0，请勿改动。

DAC bits：DAC的转换位深，根据硬件实际性能选择，如果不知道填数字“0”

Buffer time：缓冲时间，默认为100ms，可以根据硬件和体验实际进行调整。

DoP：将SDM信号打包成PCM信号输出

48k DSD：强制PCM升频DSD时采用48k整数倍升频（44.1Khz进行DSD升频为非整数倍升频，SRC可能导致失真），一般不开启，因为不推荐将PCM升频至DSD。

## 14、Network Audio backend（网络后端设置/NAA播放设置）

![](Pasted%20image%2020261023204602.png)

IPv6：打开对IPv6的IP地址支持，需要路由支持。
