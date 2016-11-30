---
layout: post
title: Python：浅叙 Python Web 开发中的框架对比
date: 2016-11-14 22:34:34 +0800
categories: Python
---

最近学习和了解了如何用 `Django` 开发` Web App` 的流程，也能做出一些博客类型的 `App`。加上去年年末学习的 `Flask` 框架，大概也知道 `Python Web` 开发是怎么一个回事，这里对比、梳理一下基于这两个框架开发 `Python Web App` 的异同。只能算是浅显的对比。因为并没有深入的研究进去，仅仅作为一个框架使用者说下感受。

## Flask框架

由于 `Flask` 不需要引导工具，所以开始 `Hello world` 级别的编程十分容易，7 行代码即可：

```python
from flask import Flask
app = Flask(__name__)
 
@app.route("/") 
def hello():
    return "Hello World!"
 
if __name__ == "__main__":
    app.run()
```

Flask 的一些优缺点如下所示：

- 容易入门，国内翻译以及官方文档资料全面。
- 轻量级，`Micro Framework` ，使用简单，并且扩展性强。
- 容易与 `NOSQL` 类型数据库结合，并且与关系型数据库结合的能力也不差。
- 容易提供出 `RESTFUL` 的 `WEB API`。
- 由于其轻量级，导致框架自身携带的功能没有 `Django` 多,而由此出现众多第三方库。
- 不需要去考虑 `MVC` 或者 `MTV` 设计，`Flask` 中有蓝图的设计，按功能划分模块。
- 可以提供由 `Django` 衍生出来的模板引擎 `Jinja` 来渲染模板。
- 提供与 `ORM` 功能相近的 `SQLAlchemy`。
- 代码量约 6000 行左右，够简洁，够 `Pythonic`，`Python` 学习的好资料。

## Django框架

由于 `Django` 有自己的引导工具，所以即使建立一个很简单的 `Hello world` 界面，也需要建立一个相对 `复杂的`目录结构。启动引导( 内置在 django-admin 中 )：

```python
django-admin startproject mysite
django-admin startapp polls
```

项目基本的目录结构：

```python
mysite/
    manage.py
    mysite/
        __init__.py
        settings.py
        urls.py
        wsgi.py
 polls/
    __init__.py
    admin.py
    migrations/
        __init__.py
    models.py
    urls.py
    tests.py
    views.py
```

在 `views.py` 里面输入如下内容：

```python
from django.http import HttpResponse
def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")
```

在 **polls/urls.py** 中，输入如下代码：

```python
# polls/urls.py
from django.conf.urls import url
from . import views

urlpatterns = [
    url(r'^$', views.index, name='index'),
]
```

在最外层的 urls.py 中写入如下语句，包含进 polls.py 的 url：

```python
# mysite/urls.py
from django.conf.urls import include, url
from django.contrib import admin

urlpatterns = [
    url(r'^polls/', include('polls.urls')),
    url(r'^admin/', include(admin.site.urls)),
]
```

Django 的一些优缺点如下所示：

- 活跃开源的社区、大量的实例 `App`、开发文档。
- Django 自带模板引擎和 `ORM` 方便开发，但是同时导致 Django 看起来有点厚重。
- Django 与关系型数据库结合非常好，但是与非关系型数据库结合就没有 `Flask` 那么好了。
- Django 后台管理及其方便，容易设定角色，几乎写很少的代码，就可以获得很丰富的后台功能权限。
- Django 由于存在年限的问题，成熟，稳定，靠谱。

## 入门推荐

个人学习 `Python Web` 开发入门的教程是 《Flask Web 开发-基于 Python 的应用开发实战》，极力推荐这本书，非常好，并且每一个该记录的版本，作者都已标签的形式上传到 `GitHub`。如果你对该细节不理解，你可以迁出对应的代码教程。书籍从入门到设计到开发以及最后的测试、部署。整个过程都十分的详细。

看完上面这本书之后，你可以试着做一些自己的应用了。我当时兴趣使然，选择了 `MongoDB` 作为后台数据库，并且设计了 `WEBAPI`，最后选择部署在 `Heroku` 平台上。在这个过程中，必须要不停的 `Google` 以及看 `Flask` 的中文官方文档。并且伴随着 `App` 的需求，你也要不停的 `GitHub` 上寻找相应的类库。

对于 `Django Web` 框架，入门的选择就多了。我先看的是 `Django1.9` 入门的官方文档前 7 小节。然后选择一个称为 Djangogrils 的实例教程边学边做，最后改一些，加上一些自己的代码，模拟了部署过程。

再后来选择做一个博客类型的 `App`，也是看了大量的第三方成型 `App`，逐渐了解到整个框架就那么回事，并且 Django 后台的特性为你节省了很多的工作。你要做的工作，其实真的很少。甚至连数据库中的表之间的关联，由于 ORM 的特性你也可以很轻松的搞定。你的大部分时间，其实花在前端 `UI` 上。

最后，如果你是 `Web` 新手，比较建议你从 `Flask` 入门。并且 `Flask` 入门你可以完成很多酷酷的功能，部署到 `LeanCloud` 、与 `MongoDB` 联通、搭建 `API` 接口等等。入门之后，你再来学习 `Django` ，感觉一个星期就可以入门了。但是说这么多最重要的还是大家都在说的，项目驱动学习，做一个 `Web App` 吧，并且也要多看看人间优秀的源码，想想看人家这个功能怎么实现的，能否复现到我的 `App` 上。

## 工具及部署

- `PyCharm` 开发工具的不二选择
- `Django1.9 & Flask & Python3.4`
- 虚拟环境以及 `Git` 和 `GitHub` 账号
- 免费的部署平台国外 `Heroku` 和 `pythonanywhere`
- 国内免费部署的平台 `lecnCloud` 提供 `Flask` 部署示例
- 自动化部署 `WebHook`