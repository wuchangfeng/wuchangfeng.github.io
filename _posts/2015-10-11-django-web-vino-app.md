---
layout: post
title: Django Web 开发之 Vino 小站
date: 2015-10-11 20:22:08 +0800
categories: Django
---

pythonanywhere 免费部署额度应该快到期了，下一次准备自己买个阿里云或者别的服务器了。之前写了好多个应用部署在上面，一旦到期估计也没有了。这里写篇文章记录一下，以防止以后还需要相关的知识。

主页就是这个样子啦：

<img src="http://ww3.sinaimg.cn/large/b10d1ea5jw1fbbe2nbc7wj21kw0zk46v.jpg"/>

文章详情页面：

<img src="http://ww4.sinaimg.cn/large/b10d1ea5jw1fbbe32pprqj21kw0zkdst.jpg"/>

后台管理页面：

![](http://ww4.sinaimg.cn/large/b10d1ea5jw1fbbe3bsxugj21kw0zk44z.jpg)

## 开发

- 开发平台 Mac os
- 开发工具 Pycharm
- 部署平台 pythonanywhere
- 开发环境 python3.5 + Django 1.9 

## 复现

- Install Python 3.5 ，Django 1.9，PyCharm
- Clone this Project
- Install virtualenv,create virtualenv,and activate virtualenv,in Mac,you can create and activate virtualenv like:
  - $ python3 -m vent myvenv
  - $ source myvenv/bin/activate
- pip install -r requirements.txt
- python manage.py migrate
- create super user,means that a username and secert for admin
- python manage.py runserver
- In your chrome see :127.0.0.1

## 参鉴

- DjangoStudyTeam的[教程](https://github.com/djangoStudyTeam/DjangoBlog/tree/blog-tutorial)
- 部署参考[djangogirls](https://tutorial.djangogirls.org/zh/deploy/)
- CSS、HTML效果参考[0v0.link-blog](https://github.com/7sDream/0v0.link-blog)
- 开发框架参考[ChenBlog](https://github.com/woodcoding/ChenBlog)

最后这个实例其实代码写的很糟糕，相对于 HTML CSS 以及 JS 来说，不建议参考。