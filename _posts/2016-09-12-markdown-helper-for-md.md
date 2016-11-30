---
title: Python：使用 MarkDownHelper 来简化你的写作
toc: true
date: 2016-06-22 21:55:35
tags: python markdown
categories: About Python
description:
feature:
---

一个 Python 脚本用来简化 MarkDown 写作中上传图片的过程以及压缩图片比例。

<!--more-->

MarkDown 已经成为技术人员写博客的标配了。

自然, 博客中经常要插入图片或者 GIF，对于这些，我们经常选择七牛或者 LeanCloud 来作为图传。但这个过程也稍微有点繁琐，要打开浏览器，寻找图片路径，点击上传图片，复制 URL。

利用 Python 的 Drop ，Handle 思想，写出了该脚本。如下 GIF 所示，你只需要将图片或者 GIF拖动到脚本上，即可生成 MarkDown 格式 URL 在你 TXT 文档和你的黏贴板中，直接右键黏贴即可。省去了不少步骤。

**[具体的配置步骤](https://github.com/wuchangfeng/MarkDownHelper)**

**核心代码:**

```python 
# upload your img to qiniu
def upload_img(bucket_name,file_name,file_path):
    # generate token
    token = q.upload_token(bucket_name, file_name, 3600)
    info = put_file(token, file_name, file_path)
    return
# get md_url 
def get_img_url(bucket_url,file_name):

    img_url = 'http://%s/%s' % (bucket_url,file_name)
    # generate md_url
    md_url = "![%s](%s)\n" % (file_name, img_url)
    return md_url
# save to txt
def save_to_txt(bucket_url,file_name):
    # get url
    url_before_save = get_img_url(bucket_url,file_name)
    # save to clipBoard
    addToClipBoard(url_before_save)
    
    # save md_url to txt
    with open(md_url_result, "a") as f:
        f.write(url_before_save)
    return
# save to clipboard
def addToClipBoard(text):
	command = 'echo ' + text.strip() + '| clip'
	os.system(command)
	
if __name__ == '__main__':

    q = Auth(access_key, secret_key)
    bucket = BucketManager(q)

    imgs = sys.argv[1:]
    # drop more than one img to .py file and
    # it will auto run and generate txt file
    for img in imgs:
        up_filename = os.path.split(img)[1]
        upload_img(bucket_name,up_filename,img)
        save_to_txt(bucket_url,up_filename)
```



**效果图**

![](http://7xrl8j.com1.z0.glb.clouddn.com/MarkDownHelper.gif)

