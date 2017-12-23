---
layout: post
title: 如何学习新技术：从 Docker 谈起
date: 2017-12-21 15:01:10 +0800
categories: 
---

**Why Docker**

了解和使用Docker是很早之前就有的想法，应该是学习Go语言那会儿，知道Docker是Go的杀手级应用。去知乎上看了很多相关信息，也大概知道了Docker的一些用处，后来自己还专门去查了好些资料。之后因为自己有许多事情要忙就不了了之，也相当于零基础。为什么最近要拾起来Docker呢，主要还是自己想了解的更多，类似Linux、shell和服务器部署之类的知识，刚好前段时间自己也买了一台VPS。所以就开始研究起来了，写下这篇文章第一个是总结和理清一下学习Docker的过程。另外也可以表述一下自己是如何学习和应用一个全新的概念和知识的，自己也是冲着使用的目的去了解Docker这个工具的，不会做深入到源码级别的研究分析，边使用、边摸索、边总结。本篇文章侧重于学习的过程和方法，而不侧重于具体概念细节，请知晓。

**需求和入门**

为什么有这个需求？从哪里了解Docker的？为什么需要Docker？这是我对自己学习Docker这个工具的自我提问。首先了解Docker这方面知识或者说知道Docker这个概念存在，是因为自己从各种零散的知识碎片中获取的，对于一些关键字留一个心眼，之后在别的地方在接触到这个概念，保持关注一下，有空就点进帖子看一下。潜移默化的你就知道这个玩意了。之后谈谈为什么有这个需求？之前写过Python Web，类似于Flask和Django都或多或少接触过，本地写完了，总想部署到云服务器上跑起来，不得不说这一点还是很吸引人的，技术人总想把自己的东西让更多的人看到，或者说做一个可视化的东西。但是服务器部署就显得很难受了，教程倒是有许多，跟着一步一步做其实也没什么大问题。但是最严重的问题就是教程和各种依赖配置版本的不同步性，这样开发者就把本该花在理解配置流程和各模块的作用上的时间，花在了去解决各种版本号不一致导致各种的坑上，虽然解决“坑”也能锻炼一个人的解决问题的能力或者说“坑”也是不可避免的，但是这样对于时间有限或者成本较大的人来说，多少有点本末倒置。因此有了想应用之前了解的Docker技术来解决问题的想法。好了这个流程需求就走起来了。下面可以开始流程学习了。

**做好前提准备**

首先要去选一份好的入门文档，有一定经验的技术开发者应该很容易就能判断什么样的文档值得学习，无非是全面、结构完整、知识循序渐进。官方文档对于有能力的人无疑是最好的选择。很多人避免官方文档最主要的原因还是因为语言的原因，其实用心就会发现，技术角度的语言大部分都是言简意赅的，并没有想象中的难以入手。文档选择好之后，针对实际情况配置本地Docker开发环境，然后跟着教程学习完基础的知识。该背的就背一下，不该记住的先了解一下，然后知道有这样一个概念即可。当然，如果你的本机配置或者环境不太足以支持Docker，可以考虑在云服务器上玩耍Docker。无非是选择一个合适的SSH客户端，然后都是在命令行中操作，跟本地没有什么区别。知识的学习应该是循序渐进和循环巩固理解的。对于本篇博文即如何学习Docker这篇文章,个人选择的教程是[docker_practice](https://yeasy.gitbooks.io/docker_practice/content/container/),作为基础准备，我认真并且按照实例完成了Docker简介、基本概念、安装Docker、使用镜像、操作容器这四章。学习完这之后，我感觉大致入了门，但是随之而来的就是知识的深入以及自己的跟不上，此时我的办法就是实战-应用Docker技术跑Web应用。

**流程跑起来**

跑流程是刺激继续深入学习Docker的一个必备过程。前面说了新知识学习到一定的层次，就有了疲劳感和“跟不上”的感觉。这个时候实战就能帮助你保持继续学习。首先同样提出需求，对于我来说需求很明确：Docker部署Flask程序。首先就是Google这个两个关键词。通读前五篇文章，通读过程中也要有取舍。细节概念不可细扣，最重要的是流程即人家怎么完成这个部署过程的。选择其中最好或者最完整的一篇文章进行边阅读边操作学习，中间能够克服各种困难。类似于教程文档是在Linux上实现的，你需要在Mac上实现；教程文档将写好的Docker服务跑在本地，而你需要跑在VPS上等等，这些东西都是刺激你进一步学习或者探索的动力。这个阶段完成之后，回想一下之前学习过的基础概念，能够将那些理论与文章实际操作过程一一对应，简单来说知道某个东西原来是这样用的。这个时候会有一种奇妙的感觉，you know it.上面这个过程并没有完全结束，在阅读文章时候，你必不可免的接触到一些关于Docker新的概念，比如Docker Hub、数据卷、Compose等等。这个时候就是该你更深一步学习教程文档的动力了。
我个人学习这块知识实际操作过程是首先在Mac本地安装好Docker开发环境，然后上GitHub上搜索关键字：Docker Flask Nginx。然后选择根据Readme完成程度，选择一两个[实例](https://github.com/danriti/nginx-gunicorn-flask)进行实战,本地跑起来之后，在我的VPS也部署上了，第一个步骤肯定是在VPS上部署Docker环境，如[Centos6安装Docker](http://www.jianshu.com/p/3eb556b47918)，当然在这之前一定要能在你的本机上通过SSH控制你的VPS，之后的流程就是跟本地一模一样了。最终你能够通过你的VPS IP地址访问你的应用程序了。
另外一个很重要的过程就是认真阅读一下你部署成功的这个Web程序的整体结构，理解每一个模块是什么作用，应用到了什么知识，之间如何关联。

**深入学习部署**

按照上述步骤，对于Docker后续的学习计划应该按照Docker文档上的Docker三剑客：Compose、Machine以及Swarm mode、数据卷、网络配置这块知识了。当然这个过程也只是知识概念识别。一般来讲，你的需求还没有到应用这些高级知识的阶段。此时应该跟入门阶段一样，保持对这些新鲜知识的铭感度和熟悉度。
后续的过程就是大量的看各种文档和教学实例，这个需求也是从实际出发。类似于本篇文章的出发点就可以是：Docker、Django/Flask/、Nginx等。这个时候又是知识学习循环的一个过程了，对于Docker、Django不可避免的会涉及到多容器之间协作、数据库、数据备份等问题。此时对于Compose的应用与理解就更上一层楼了。其中Docker-machine知识：前面我们的实验环境中只有一个Docker host，所有的容器都是运行在这一个host上的。但在真正的环境中会有多个host，容器在这些host中启动、运行、停止和销毁，相关容器会通过网络相互通信，无论它们是否位于相同的host。对于这样一个multi-host 环境，我们将如何高效地进行管理呢？我们面临的第一个问题是：为所有的host安装和配置Docker。Docker中解决该问题的策略是利用Docker-Machine。这只是其中的一个小的实例知识点。另外在这个时候，我通过在网上查阅资料，发现了一份更好的教程[《每天5分钟玩转Docker》](http://www.cnblogs.com/CloudMan6/tag/Docker/default.html?page=1)，看其博客已经出版成书了，但是让人惊讶的是该系列的博客文章竟然还在更新，最近一次更新在2017年12月18号,真的是非常值得一看的系列文章：

![](http://ww1.sinaimg.cn/large/b10d1ea5ly1fmn2tei9m9j216g0e0wpb.jpg)


**深入实战学习**

毕竟自己的工作内容不是Docker，学习之初的想法就是来认识一下Docker这个优秀的工具，以及能够实现自己的基本应用。后续的学习过程就是跟着自己的需求来，部署Django Web App,这一方面也不用自己瞎捉摸，GitHub上有很多非常好的学习资料，Readme也写得非常好。我首先在GitHub的搜索栏，输入：Docker Django.然后按照star数目结合Readme的完整程度，选择了[dockerizing-django](https://github.com/realpython/dockerizing-django).面对这样的一个崭新的项目，先要浏览一下其项目结构和Readme文档，发现恰好应用到了Docker中的Compose、Machine还有Nginx、redis等知识，此时就决定将其作为深入学习的项目了，最终的目的还是理清部署流程，弄清楚Compose和Machine解决了什么问题，带来了什么好处。开始尝试在本地或者VPS上部署该应用，使其成功运行起来。这样对于学习后续的项目结构、配置文档则更有动力。首先通读其文档Readme，跟着步骤一步一步在本地运行该Web，具体执行命令过程如下所示：

* 创建一个新的Machine，具体命令如下所示：
``` shell
docker-machine create -d virtualbox dev;
```

* Build出镜像
``` shell
docker-compose build
```

* 启动Docker Compose群组服务
``` shell
docker-compose up -d
```

* 创建迁移
``` shell
docker-compose run web /usr/local/bin/python manage.py migrate
```

* 此步骤获取Machine host的IP地址
``` shell
docker-machine ip dev
```

在这一步骤Docker会为你的虚拟主机分配一个IP。接着在本机上输入对应的IP或者LocalHost，则会出现如下所示界面：

![](http://ww1.sinaimg.cn/large/b10d1ea5ly1fmnifneaduj21jo11egs6.jpg)

通过前面的学习和实战，知道了Docker中的Machine会创建一个新的虚拟host，我们尝试进入该host，之后所有执行的Docker命令都相当于在host主机上执行：

* 进入虚拟host
``` shell
$ eval $(docker-machine env dev)
```

* 显示当前Machine信息：
``` shell
$ docker-machine ls
NAME   ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER        ERRORS
dev    *        virtualbox   Running   tcp://192.168.99.100:2376           v17.09.1-ce
```

接下来分析一下项目的结构，目录结构如下所示，采用tree命令查看：

``` shell
├── README.md
├── docker-compose.yml # compose的配置文件
├── nginx  # Nginx相关
│   ├── Dockerfile 
│   └── sites-enabled
│       └── django_project
├── production.yml # 生产环境中的配置文件
└── web # Python Web
    ├── Dockerfile # Web应用的dockerfile
    ├── docker_django
    │   ├── __init__.py
    │   ├── apps
    │   │   ├── __init__.py
    │   │   └── todo
    │   │       ├── __init__.py
    │   │       ├── admin.py
    │   │       ├── models.py
    │   │       ├── templates
    │   │       │   ├── _base.html
    │   │       │   └── home.html
    │   │       ├── tests.py
    │   │       ├── urls.py
    │   │       └── views.py
    │   ├── settings.py
    │   ├── urls.py
    │   └── wsgi.py
    ├── manage.py
    ├── requirements.txt # Python Web依赖的库
    ├── static
    │   └── main.css
    └── tests
        ├── __init__.py
        └── test_env_settings.py
```

重点查看docker-compose.yml文件,其具体内容如下所示：

``` yml
web:
  restart: always
  build: ./web  # 指定Dockerfile文件所在路径
  expose: # 暴露端口
    - "8000"
  links: # 关联组件
    - postgres:postgres
    - redis:redis
  volumes: # 数据卷
    - /usr/src/app
    - /usr/src/app/static
  env_file: .env
  command: /usr/local/bin/gunicorn docker_django.wsgi:application -w 2 -b :8000

nginx: # Nginx相关
  restart: always
  build: ./nginx/
  ports:
    - "80:80"
  volumes:
    - /www/static
  volumes_from: # 加载其他容器的所有卷
    - web
  links:
    - web:web

postgres:  # 数据库相关
  restart: always
  image: postgres:latest # 指定镜像
  ports:
    - "5432:5432"
  volumes:
    - pgdata:/var/lib/postgresql/data/

redis: # redis相关
  restart: always
  image: redis:latest # 指定镜像
  ports:
    - "6379:6379"
  volumes:
    - redisdata:/data
```

其中expose暴露出来的端口不能映射到宿主主机上，只对links的容器开发。command用于替换默认执行的command命令。restart作用是如果更改了.yml配置文件，会立刻重启生效。

再查看学习一下web目录下的Dockerfile，比较简单：

``` yml
FROM python:3.5-onbuild
```

**总结与推荐资料**

上面详细叙述了自己这几天对于Docker这一块知识的学习过程，对于任何语言或者技术知识，大致的学习过程都跟上面表述的差不多。多了解、多实践，这样总是最好的学习路径。后续如果要深入学习的话，肯定要多看Dockerfile和Docker-Compose.yml写法的最佳实践，根据自己的Web App写出合适的两个配置文件。在学习Docker的几天里发现了除了官方文档之外不错的资料，整理如下，方便他人学习和自己使用，如果你要深入学习Docker的应用或者原理性知识，强烈推荐下面的前两个资料，第一本是出版书，作者是阿里云容器相关的技术人员，网上有相关技术博客导读。第二个是每天5分钟玩转Docker，这是一个博客系列，已经出版成书，但是目前博客文章还在保持高频率更新，非常人性化,巨牛逼。

* 自己动手写Docker

* [每天5分钟玩转Docker](http://www.cnblogs.com/CloudMan6/tag/Docker/default.html?page=1)

* [Docker从入门到实践](https://www.gitbook.com/book/yeasy/docker_practice/details)

**最后，感谢阅读！！！**


