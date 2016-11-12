---
title: 用 Pygal 生成漂亮的 SVG 图像并显示在 Web 页面上
date: 2016-10-24 20:58:02
tags: Pygal
---

本篇文章讲述如何用 Pygal 来生成漂亮的 SVG 图表，并能够利用 Python 中的 Flask 框架来显示你的 SVG 图像。

<!--more-->

SVG可以算是目前最最火热的[图像文件格式](http://baike.baidu.com/view/3413757.htm)了，它的英文全称为Scalable Vector Graphics，意思为可缩放的矢量图形。它是基于XML（Extensible Markup Language），由World Wide Web Consortium（W3C）联盟进行开发的。严格来说应该是一种开放标准的矢量图形语言，可让你设计激动人心的、高分辨率的Web图形页面。用户可以直接用代码来描绘图像，可以用任何文字处理工具打开SVG图像，通过改变部分代码来使图像具有交互功能，并可以随时插入到HTML中通过浏览器来观看。

## 零.First Head in Pygal

首先安装 pygal 啦：

`pip install pygal`

如果你要把生成格式设为除了 svg 之外的格式，如 png，jpg 之类，就要安装底下几个库了：

`pip install lxml`

在 Ubuntu 中按照如下提示安装即可：

```python
sudo apt-get install libxml2-dev libxslt1-dev python-dev
sudo apt-get install python-lxml
```

`pip install cairosvg`

安装该库原理同上：

```python
sudo apt-get install python-cairosvg
```
如下两个库，只需正常 pip 安装即可：

`pip install tinycss`

`pip install cssselect`

## 一.Hello SVG

```python
import pygal                                                      
bar_chart = pygal.Bar()                                           
bar_chart.add('Fibonacci', [0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55])  
bar_chart.render_to_file('Hello.svg')                         
```

生成的是黑色的 Hello.svg 文件，因为是 svg 格式的，一般的话直接是不能打开的，选择默认的浏览器打开吧，看到就是底下这个样子：

![](http://7xrl8j.com1.z0.glb.clouddn.com/svg.gif)



更加骚气点的图：

```python
import pygal
line_chart = pygal.Line()
line_chart.title = 'Browser usage evolution (in %)'
line_chart.x_labels = map(str, range(2002, 2013))
line_chart.add('Firefox', [None, None,    0, 16.6,   25,   31, 36.4, 45.5, 46.3, 42.8, 37.1])
line_chart.add('Chrome',  [None, None, None, None, None, None,    0,  3.9, 10.8, 23.8, 35.3])
line_chart.add('IE',      [85.8, 84.6, 84.7, 74.5,   66, 58.6, 54.7, 44.8, 36.2, 26.6, 20.1])
line_chart.add('Others',  [14.2, 15.4, 15.3,  8.9,    9, 10.4,  8.9,  5.8,  6.7,  6.8,  7.5])
line_chart.render_to_file('Hello_line_chart.svg')
```

生成的图就是下面这个样子：

![](http://ww1.sinaimg.cn/large/b10d1ea5jw1f93qej26pij20os0ljgr7.jpg)



## 二.Hello PNG

有时候，我们不需要 svg，只需要 png 格式的图表，没关系，pygal 也能够做到：

```python
import pygal
bar_chart = pygal.Bar()
bar_chart.add('Fibonacci', [0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55])
bar_chart.render_to_file('bar_chart.svg')
# 生成 png 格式图表
bar_chart.render_to_png(filename='bar_chart.png')
```

注意绿色的标示，成功生成 png 格式的图片啦：

![](http://ww3.sinaimg.cn/large/b10d1ea5jw1f93n2b9euxj20ra0o4q7j.jpg)

## 三.Hello Flask and Pygal

让 Pygal 生成的 svg 格式图片中，显示在你的网页上呗，我们选择 flask 来提供 web 支持：

```python 
pip install flask
```

核心代码如下，没错就是这么短：

```Python
import pygal
from flask import Flask, Response

app = Flask(__name__)

@app.route('/')
def index():
    """ 在html文件中渲染 svg """
    return """
<html>
    <body>
        <h1>hello pygal and flask</h1>
        <figure>
        <embed type="image/svg+xml" src="/hellosvg/" />
        </figure>
    </body>
</html>'
"""

@app.route('/hellosvg/')
def graph():
    """ render svg graph """
    bar_chart = pygal.Bar()
    bar_chart.add('Fibonacci', [0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55])
    return Response(response=bar_chart.render(), content_type='image/svg+xml')

if __name__ == '__main__':
    app.run()
    
```

打开 127.0.0.1:5000 就能看到下面的样子咯：

![](http://ww4.sinaimg.cn/large/b10d1ea5jw1f93na6fa1nj20rl0oltfp.jpg)



当然咯，你还可以做出如下骚气的 svg 图像，不过这一切都是 pygal 的用法啦：

![](http://ww3.sinaimg.cn/large/b10d1ea5jw1f93ngn536yj20kg0efwgq.jpg)

移步 [pygal](http://pygal.org/en/stable/documentation/configuration/value.html) 官方文档吧。