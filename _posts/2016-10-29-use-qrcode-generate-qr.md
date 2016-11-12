---
title: 用 Qrcode 来快速生成二维码
date: 2016-10-29 11:59:25
tags: qrcode


---

本篇文章讲述如何在 Python 环境中使用 qrcode 模块来生成个性化二维码图片。

<!--more-->

qrcode GitHub 地址：https://github.com/lincolnloop/python-qrcode

### 安装模块

在安装 qrcode 之前，我们需要首先安装 PIL, 但是在我们尝试安装时候，却会报错，在 Linux 系统下很少会出现安装一个模块出错的情况，网上 Goolge 之后，发现 PIL 对于 Python 2.7 以及 Python 3.X 都还尚未支持，替代的解决方案是安装 pillow:

```python
pip install pillow
```

安装好预备模块之后，按照 qrcode 吧，也是很简单：

```pyth
pip install qrcode
```

安装好上述之后，我们简单的进入 Python 环境检测一下，即：

```python
import qrcode
```

### 初次尝试

没有错误即验证安装成功。紧接着我们按照官方文档提示，初次实验一下，复制如下命令进入命令行：

```shell
qr "Just a test" > test.png
```

当然我们也可以不借助于作者提供的 qr 脚本，直接写入 Python 文件中：

```python
import qrcode
img = qrcode.make('Just a test')
img.save("test.png")
```

运行完成之后，顺利在当前工作目录下生成一张名为 test.png 的二维码，如下图所示：

![](http://ww4.sinaimg.cn/large/b10d1ea5jw1f98zm9gntvj20t50gun19.jpg)

接着我们当然用微信扫一扫这张二维码图片啦，顺利会在你的手机界面上显示： Just a test。

### 个性化定制

我们来尝试一下为你的网站生成一张定制的二维码呗：

```python
#coding=utf-8
import qrcode
qr = qrcode.QRCode(
	version=2,# 取值 1-40 相应大小也不同
	error_correction=qrcode.constants.ERROR_CORRECT_L,# 指的是纠错容量，
    #这就是为什么二维码上面放一个小图标也能扫出来，纠错容量有四个级别
	box_size=10,# 指的是生成图片的像素
	border=1 # 表示二维码的边框宽度，4是最小值
)
qr.add_data("http://allenwu.itscoder.com/")
qr.make(fit=True)
img = qr.make_image()
img.save("allenwu.png")
```

生成二维码如下所示，当你用手机扫一扫之后，就能跳转到指定的网站啦：

![](http://ww2.sinaimg.cn/large/b10d1ea5jw1f98zzbaan1j20t20gu450.jpg)

### 支持 SVG

QRCODE 非常牛逼的支持了 SVG 格式的图标生成, 按照官方文档你可以按照如下三个方式生成：

```shell
qr --factory=svg-path "Some text" > test.svg
qr --factory=svg "Some text" > test.svg
qr --factory=svg-fragment "Some text" > test.svg
```

相应的 Python 代码如下：

```python
import qrcode
import qrcode.image.svg

if method == 'basic':
    # Simple factory, just a set of rects.
    factory = qrcode.image.svg.SvgImage
elif method == 'fragment':
    # Fragment factory (also just a set of rects)
    factory = qrcode.image.svg.SvgFragmentImage
else:
    # Combined path factory, fixes white space that may occur when zooming
    factory = qrcode.image.svg.SvgPathImage

img = qrcode.make('Some data here', image_factory=factory)
```

