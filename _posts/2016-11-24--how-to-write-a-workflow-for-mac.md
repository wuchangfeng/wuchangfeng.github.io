---
layout: post
title: 如何去写一个第三方的 workflow
date: 2016-11-24 19:05:35 +0800
categories: Mac
---

学习一下 Mac 下的 workflow 开发流程。地址在[这里](https://github.com/wuchangfeng/Vino-Workflow)。

### 更新

* workflow for Gank 获取 Gank 当日开发干货
* workflow for YouDao 获取中文对应的英文翻译
* workflow for V2ex 获取 V2ex 当日最热 10 条帖子

### 准备

- 购买了 Powerpack 的 Alfred 或者破解版的 Alfred workflow


- 利用 Python 编写脚本的教程：[Alfred-Workflow](http://www.deanishe.net/alfred-workflow/index.html)

### 说明

我们跟随上面提供的教程可以进行一个初步的学习，并且上述的教程图片和流程都挺清晰的，可是教程却选择了一个付费的示例，使得不能继续跟下去学习，所以在了解了大概开发流程之后，并没有选择教程所提供的示例练习。

想起来之前一个[潇大](https://github.com/hujiaweibujidao)写过 Gank 的 workflow，就下载下来，使用了一下，发现确实可以用，但是有可能接口平台的原因，数据有时候能显示，有时候却不能。但是由于道理是相通的，所以作为入门级教程，我们首先就模拟一下 Gank 的 workflow 的开发过程。潇大的 workflow 地址在这里：[Gank-workflow](https://github.com/hujiaweibujidao/Gank-Alfred-Workflow)。学习成果的 workflow 在这里：[Vino-workflow](https://github.com/wuchangfeng/Vino-Workflow)

### 开始

如下图所示选择到指定位置，点击 Help 按钮：

![](http://ww2.sinaimg.cn/large/b10d1ea5jw1fa2cyj6j3bj21kw0y1djl.jpg)

选择 Blank Workflow,并填写好相关信息：

![](http://ww1.sinaimg.cn/large/b10d1ea5jw1fa2d0ahqu0j21kw0u2dl7.jpg)

点击 Create 之后，如下图所示，右击添加一个 workflow object：

![](http://ww4.sinaimg.cn/large/b10d1ea5jw1fa2d0mnjraj21kw0xtaei.jpg)

右击之后选择 Input 选项中的 Script Filter 选项：

![](http://ww1.sinaimg.cn/large/b10d1ea5jw1fa2d11xgmmj21kw0xxgrf.jpg)

然后我们继续在下图填写相关信息，keyword 为搜索触发词，接下来三个选项就不是很重要了，一些描述性内容。接着选择 Language 选项，我们选择 /bin/bash,故名思议，能够执行脚本性语言，接着在同一行，我们选择 with input as {query}。这里我猜想是需要输入查询关键词所需要的选项吧。在 Script 中输入 `pythongank.py "{query}"`：

![](http://ww3.sinaimg.cn/large/b10d1ea5jw1fa2d4xjabfj21kw0yewny.jpg)

完成上述步骤后，如下图所示，右击，选择在 Finder 中打开：

![](http://ww1.sinaimg.cn/large/b10d1ea5jw1fa2d4qowgqj21kw0jkgql.jpg)

会发现默认情况下存在 info.plist 文件。接着我们创建 gank.py 文件，没错，这个就是前面输入的名称，一定要确保一致性。然后，我们在 [Alfred-workflow](https://github.com/deanishe/alfred-workflow/releases/tag/v1.24) 这个页面下载最新的文件，解压复制其中的 workflow 文件夹到并且粘贴到下面的目录中，另外为了让 workflow 更好看，可以自定义图片 Icon，当然可以随着 Item类型不同选择多种类型的 Item Icon：

![](http://ww2.sinaimg.cn/large/b10d1ea5jw1fa2d4tv19fj21kw0ty0yo.jpg)

紧接着，我们就编写 gank.py 文件了。**这个在最后的步骤贴出**。

为了能够在 workflow 的 list 中，跳转到指定 Item 所对应的 URL 页面中，我们还需要给 script 添加一个Actions URL，如下所示，选择 Open URL。当然在这一步，我们有更为丰富的选择，如，我们可以选择复制 Item 的内容到剪贴板，就可以选择 Outputs 中的相应的选项：

![](http://ww3.sinaimg.cn/large/b10d1ea5jw1fa2d4y5blrj21kw0y0q9u.jpg)

添加之后，默认配置如下即可，点击 Save ：

![](http://ww4.sinaimg.cn/large/b10d1ea5jw1fa2d4rwylaj21kw0i7jxe.jpg)

接下来，我们就要连接工作流啦，如下图所示，用一条线连接 Script Filter 和 Open URL ：

![](http://ww1.sinaimg.cn/large/b10d1ea5jw1fa2d4qnuywj21kw0jqn1f.jpg)

如上，基本整个流程就完毕了，双击 command 打开 Alfred ，输入触发词，即可：

![](http://ww4.sinaimg.cn/large/b10d1ea5jw1fa2d4waa9ej20xa0aamzv.jpg)

### 代码

如下即为 Gank workflow 的核心代码，我们分段解读一下：

```python
def search(query):
    # search the ganks from gank.io
    url = 'http://gankio.herokuapp.com/search'
    # query 即为查询关键词，指定 API 接口返回 Json 数据
    params = dict(keyword=query)
    r = web.post(url, params)
    # throw an error if request failed, Workflow will catch this and show it to the user
    r.raise_for_status()
    return r.json()
```

程序的主函数如下所示：

```python

def main(wf):
   	
    # 获取查询关键字
    query = wf.args[0]
    # Search ganks or load from cached data, 10 mins
    def wrapper():
        return search(query)
    # 对于搜索接口存在缓存时间
    ganks = wf.cached_data(query, wrapper, max_age=600)
     
    # Parse the JSON returned by pinboard and extract the ganks
    for gank in ganks:
        wf.add_item(title=gank['title'],
                     subtitle=gank['source'],
                     arg=gank['url'], 
                     valid=True,
                     icon=ICON_DEFAULT)
    
    # Send output to Alfred. You can only call this once.
    # Well, you *can* call it multiple times, but Alfred won't be listening
    # any more...
    wf.send_feedback()
```

对于 Python 文件的编写，在我们引入了 Python 相应的类库之后，就变得模式很固定化了，甚至可以说，只是去解析一些 Json 数据，填写一些 Item 而已。

### 进阶

如上作者的版本已经得到证明接口不稳定了，而 Gank 官方也提供了更好的  API 接口，我们就来磨磨刀，练练手吧。

- 选择 Gank 的今日干货接口 http://gank.io/api/day/2015/08/07 点击查看一下 Json 数据，熟悉下格式；
- 此处跟上面的搜索 wf 不一样，这里只需要触发词：gt 即可。我们在 keyword 之后选择：No Argument 。这样在 wf 输入框内输入：gt 之后即可列出今日 gank 的干货列表：

![](http://ww2.sinaimg.cn/large/b10d1ea5jw1fa3ch96g6yj21kw0m8ah0.jpg)

- 其余创建流程都跟上面一样的，重点还是 Gank.py 文件的编写;

  请求数据以及解析的函数如下所示：

  ```python
  # 加载今天的干货列表
  def today():
      ganks = []
      url = 'http://gank.io/api/day/' + datetime.datetime.now().strftime("%Y/%m/%d)
      r = web.get(url)
      # 错误提示
      r.raise_for_status()
      # 返回 json 数据                                                                   
      data = r.json()
      # 解析数据
      results = data['results']
      categories = data['category']
      results = data['results']
      for category in categories:
          for gank in results[category]:
              ganks.append(gank)
      return ganks
  ```

  Main 函数如下所示：

  ```python
  def main(wf):
      # 请求数据
      ganks = wf.cached_data('today', today, max_age=60)
      # 请求时机不对
      if len(ganks) <= 0:
              wf.add_item(title=u'今天还没发干货', valid=True, icon=ICON_DEFAULT)
      # 添加 item 到 workflow 列表
      for gank in ganks:
          sub =  gank['type'] 
          wf.add_item(title=gank['desc'],
                      subtitle=sub,
                      # 注释一
                      arg=gank['url'],
                      valid=True,
                      icon=ICON_DEFAULT)
   	# feedback
      wf.send_feedback()
  ```

- 使用方式，如下图所示，输入触发词 gt，即可获取当天的 Gank 干货列表：

  ![](http://ww2.sinaimg.cn/large/b10d1ea5jw1fa3e0wyodjj20w80r6gua.jpg)

- 为了能跳转到指定干货页面，我们再添加一个 workflow 连线以及结合上述代码注释一处：

  ![](http://ww3.sinaimg.cn/large/b10d1ea5jw1fa3e4x2iupj21b20iegmu.jpg)

好了，以上就是开发一个 workflow 的大概流程，我们选择的语言是 Python，当然其他语言也是可以的，我在网上看了看，我写的这篇文章应该是比较全面的中文开发 workflow 的教程。虽然过程很简单，嘿嘿。

**Thanks for reading ！！！**

Ps:[有没有人要收实习啊？](http://allenwu.itscoder.com/resume)

