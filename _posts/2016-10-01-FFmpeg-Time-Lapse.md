---
title: 使用 FFmpeg 合成延时视频
---

最近玩[延时摄影](https://en.wikipedia.org/wiki/Time-lapse_photography "Time-lapse photography")，需要一个简单的将几百张照片合成延时视频的工具。

开源免费的 [FFmpeg](https://www.ffmpeg.org/ "FFmpeg") 是不二选择。

使用方法也很简单，只需要在你存放照片的目录下执行以下命令即可：

{% highlight shell %}
# -r 帧率，在这里取每秒 24 帧
# -s 视频分辨率，缩放比例与原照片分辨率比例一致
# -i 照片名字的编码格式，我的照片格式是 DSC*****.jpg 五位数字
# -start_number 照片的起始编号
# 最后输入视频名称，在这里是 video.mp4
$ ffmpeg -r 24 -s 1200x800 -i DSC%05d.jpg -start_number 2931 video.mp4 {% endhighlight %}