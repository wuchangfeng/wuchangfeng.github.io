---
title: 在 Ubuntu 下创建虚拟独立 Python 环境
date: 2015-10-29 11:02:57
tags: virtualenv
layout: post

---

本篇文章讲述如何在 Linux 以及 Ubuntu 中创建 Python 虚拟环境。

<!--more-->

**虚拟环境**是程序执行时的独立执行环境，在同一台服务器中可以创建不同的虚拟环境供不同的系统使用，项目之间的运行**环境保持独立性而相互不受影响**。例如项目可以在基于 Python2.7 的环境中运行，而项目 B 可以在基于Python3.x 的环境中运行。在 Python 中通过 [virtualenv](http://pypi.python.org/pypi/virtualenv) 工具管理虚拟环境。

另外在 win 或者 mac 上也是极力推荐安装虚拟环境来管理你的 Python 环境，虚拟环境能为你带来不少好处，比如在 Mac 上，自带的 Python 环境为 2.7 。而我们 Django 开发最合适的就是 3.4+。如此一来，你就要去 Google 如何卸载或者转至 Python3.4 的环境，还是比较麻烦。一旦我们有了虚拟环境之后，我们都可以在独立的环境中去安装我们需要的模块或者包的不同的版本，这样会带来很大方便。

**Install**

在 Linux 系统中执行如下命令安装：

```python 
$ sudo pip install virtualenv
```

在 Ubuntu 中以及其衍生系统中执行如下命令安装即可：

```python 
$ sudo apt-get install python-virtualenv
```

**Create**

安装成功之后，执行如下命令创建名称为 myvenv 的虚拟环境：

```python
$ virtualenv myvenv
```

提示如下：

```python
allen@ubuntu:~$ virtualenv myvenv
Running virtualenv with interpreter /usr/bin/python2
New python executable in myvenv/bin/python2
Also creating executable in myvenv/bin/python
Installing setuptools, pip...done.
```

**Activate**

```python
source kvenv/bin/activate
```

具体过程如下，可以看到我们在当前环境下查看 Python 的版本，显示是在虚拟环境 myvenv 下的：

```python
allen@ubuntu:~$ source myvenv/bin/activate
(myvenv)allen@ubuntu:~$ which python
/home/allen/myvenv/bin/python
```

当然退出当前虚拟环境如下命令即可：

```python
deactivate
```

**Pip**

在激活了虚拟环境之后，你可以在这个环境中任意的Pip 啦:

```python
pip install Pillow
```

**Virtualenvwrapper**

其为虚拟环境扩展包，用于管理虚拟环境，如列表所有虚拟环境，删除等等。目前笔者还没需要用到，等用到时候在研究吧。