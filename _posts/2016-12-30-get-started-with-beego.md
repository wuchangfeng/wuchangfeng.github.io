---
layout: post
title: Go Web 开发之 Beego 框架初探
date: 2016-12-30 16:43:46 +0800
categories: Go
---

又到周末啦。干完活之后，喷了一篇关于学习 Beego 的文章。这应该是 2016 年最后一篇技术文章啦，也是一气呵成，没有什么技术含量。

### 选择 Go 语言

断断续续看了 Go 几个星期了，讲真的真是喜欢的不得了。认真学过之后，你会觉得非常的优雅，写东西很舒服。学习 Go 我觉得很有必要的是，Go 中自带的数据结构很少，类似于 List 或者 Tree 之类的，最好尝试一下如何去设计一些常用的数据结构。话说回来，Go 的出身始终是一门后端语言。我非常后悔用 Flask 或者 Django 来作为我的后端入门框架或者选择。封装的太好了，往往对于一个入门新手来说学习不到什么。

而 Go 就不一样了，它天生被设计是一门后端语言。也就是说，你将会学习到非常多的后端知识。看看下面这一张图，当时我看着就有一种很过瘾的感觉，因为这些知识你都知道，但是作为一个后端开发者你没有去了解过，这是非常大的失误。并不是想去用学习好 Go 去弥补没有学习好 C++ 的遗憾，只是新生事物，多尝试尝试总是极好的，哪怕只是浅尝辄止。Go 作为一门新的语言，其语言设计的特性，背后 Google 爸爸的撑腰以及现在 Docker 技术发展，前景应该还是不错的。所以如果你是编程新手或者是想入门后端的开发者，我强烈建议你选择 Go 语言。

语言学到最后，框架总是少不了的。虽然不能完全依赖框架，但是还是可以学习一下框架的设计思想。对于 Beego 框架的评价总是各种各样，这也要看自己的选择了。之所以选择 Beego 框架来入门，主要是因为其详细的文档以及教程示例非常多。

<img src="http://ww4.sinaimg.cn/large/b10d1ea5jw1fb7yzu3cqoj21kw13ln8t.jpg"/>

### Go Web 初探

先看一下最基本的 Go 中的 web 服务，只用到了 Go 中的 net/http 这个包：

```go
package main

    import (
        "fmt"
        "net/http"
        "strings"
        "log"
    )

    func sayhelloName(w http.ResponseWriter, r *http.Request) {
        r.ParseForm()  //解析参数，默认是不会解析的
        fmt.Println(r.Form)  //这些信息是输出到服务器端的打印信息
        fmt.Println("path", r.URL.Path)
        fmt.Println("scheme", r.URL.Scheme)
        fmt.Println(r.Form["url_long"])
        for k, v := range r.Form {
            fmt.Println("key:", k)
            fmt.Println("val:", strings.Join(v, ""))
        }
        fmt.Fprintf(w, "Hello astaxie!") //这个写入到w的是输出到客户端的
    }

    func main() {
        http.HandleFunc("/", sayhelloName) //设置访问的路由
        err := http.ListenAndServe(":9090", nil) //设置监听的端口
        if err != nil {
            log.Fatal("ListenAndServe: ", err)
        }
    }
```

执行 `go run web.go`根据提示在浏览器地址栏打开 URL ，如下图所示：

<img src="http://ww1.sinaimg.cn/large/b10d1ea5jw1fb8r99mxx7j21kw13ladv.jpg"/>

### 云端IDE Cloud Studio

本示例博客提供两种练习Go语言基础的方式，分别是：传统的本地环境安装和**云端IDE Cloud Studio**。本地环境安装为常规套路，在下一章节提出。由于Go语言的环境配置稍显麻烦，笔者在Mac和Win上都曾经花费不少精力，更有同学告诉我，光配置环境这个过程就耗尽了一部分他们学习Go语言的兴趣。正好近期有朋友向我推荐了一下云端IDE Cloud Studio，仔细体验了一下，发现确实不错。首先对新手可以暂时避免浪费精力在配置环境上，另外一点可以随时Web云端练习，甚至在手机、Pad上都可以编程写代码。并且，Web IDE支持多种编程环境，这一点是令人比较惊喜的，你可以一键切换自己需要的编程环境。如下截图所示基本上支持了大部分编程语言的环境。

![](http://ww1.sinaimg.cn/mw690/b10d1ea5ly1frer16y9j5j21iw1327fu.jpg)

好了，来说说如何在 Cloud Studio 编写Go语言程序。在下图左边栏目Home目录层级建立go项目文件夹，然后在该级别下新建hello.go文件，接着在文件中编程程序代码即可。完成后，在控制台机底部黑色区域，进入到hello.go的文件夹 输入：go run hello.go 即可调试编译代码，十分方便。如果需要安装什么Go模块，也可以在终端进行。基本上是与本地代码编写环境无差别的。

![](http://ww1.sinaimg.cn/mw690/b10d1ea5ly1frer1tcycqj21iw1327dc.jpg)

另外，该云端IDE还非常好支持了Git版本协作功能，开发者可以很轻松的从代码管理平台上拉取最新的代码、以及将云端的代码提交到代码管理平台等操作。

### 安装 Go 以及 Beego

基本的你得有个 Go 语言的环境，安装什么的就不讲了。只是最后配置其环境变量许多人都容易弄错，包括我自己也是。其实多次也能够配置好，只是每次重新启动就提示上次的配置无效，也不知道是怎么回事。讲一下安装 Beego 框架

```shell
export GOBIN="/usr/local/go/bin"
export GOPATH="/Users/allenwu/GoProjects"
export PATH="$PATH:$GOBIN:$GOPATH/bin"
```

在终端输入如上所示代码，其中 allenwu 替换成你自己的 username，并且在根目录下创建 GoProjects 文件夹，作为下一步工作目录。配置好之后，输入如下命令确保保存成功：

```shell
source ~/.zshrc
```

最后当然要测试一下环境变量是否配置成功，在 shell 中输入 `go env`若如下所示即表明成功(着重注意一下的就是 gopath 是不是为空)：

<img src="http://ww2.sinaimg.cn/large/b10d1ea5jw1fb7zocpf2uj211w0vc48x.jpg"/>

配置环境没问题之后，就是安装 go 和 bee 工具了：

```shell
$ go get github.com/astaxie/beego
$ go get github.com/beego/bee
```

检查安装是否成功，启动一个 `Hello world`级别的 App：

```shell
$ cd $GOPATH/src
$ bee new hello
$ cd hello
$ bee run hello
```

会提示你在浏览器输入地址，然后就能知道是否安装成功啦。

### 体验 Beego 框架

如下图所示即为 Beego 官网所提供的 Beego 框架概览，一眼就能明白其 MVC 模式的构造，结构也是非常清晰的。

<img src="http://ww1.sinaimg.cn/large/b10d1ea5jw1fb7zxt089pj218w0hen00.jpg"/>

安装好 Beego 框架之后，官方给了三个 samples，我们选择其中一个来进行入门体验一下。如下实例选择的是 todo App。我们将 clone 下来的 todo App 放置到指定目录下，用 sublimeText3 打开这个示例项目，强烈建议你打开侧边栏设置选项：

<img src="http://ww1.sinaimg.cn/large/b10d1ea5jw1fb7yuuevvlj21fa18kwol.jpg"/>

了解完基本的结构之后，我们启动这个 App。我们采用 Beego 提供的工具 bee 来启动。进入到最终的指定文件夹 todo 之后，执行 `bee run` 命令：

<img src="http://ww1.sinaimg.cn/large/b10d1ea5jw1fb7yu1xwvnj211w0vcai4.jpg"/>

可以看到打印出一个 Bee 的 Logo，表示启动成功了，稍等一下就会继续提示你在浏览器中输入指定 IP 地址和端口号，也就是如下所示：

<img src="http://ww1.sinaimg.cn/large/b10d1ea5jw1fb7yvdy21nj21kw141aei.jpg"/>

官方讲这个小 App 结合了 Angular ，体验还是挺不错的。接下来我们来简单分析一下示例 App 的代码结构。首先入口程序是 Main.go，这是想都不用想的，一个程序员的直觉：

```go
package main

import (
	"github.com/astaxie/beego"
  	// 注释一
	"samples/todo/controllers"
)

func main() {
  	// 注释二
	beego.Router("/", &controllers.MainController{})
	beego.Router("/task/", &controllers.TaskController{}, "get:ListTasks;post:NewTask")
	beego.Router("/task/:id:int", &controllers.TaskController{}, "get:GetTask;put:UpdateTask")
  	// 注释三
	beego.Run()
}
```

我们第一感觉还是看到 Main() 函数，看到了 Router() 函数，有一些 web 开发或者开发经验的应该都知道这是路由机制。对应的是 url 与 Controller。在 MVC 框架中 Controller 是一个很重要的概念了。我们自然下一步骤就是去往 Controller 中看看：

```go
package controllers

import (
	"github.com/astaxie/beego"
)
// 注释一
type MainController struct {
	beego.Controller
}
// 注释二
func (this *MainController) Get() {
  // 注释三
	this.TplName = "index.html"
	this.Render()
}
```

在看完 Go 的基本语法之后，看到注释一应该也能明白一个一二三四了，我们声明了一个控制器 `MainController`，这个控制器里面内嵌了 `beego.Controller`，这就是 Go 的嵌入方式，也就是 `MainController` 自动拥有了所有 `beego.Controller` 的方法。而 `beego.Controller` 拥有很多方法，其中包括 `Init`、`Prepare`、`Post`、`Get`、`Delete`、`Head`等 方法。我们可以通过重写的方式来实现这些方法，而我们上面的代码就是重写了 `Get` 方法。

在注释三处，我们看到了 `index.html`,应该明白了 Get 方法去获取对应名称的 HTML 文件，并进行渲染。到这里我们很简单的讲述了一下 MVC 中的 V 和 C，发现没，Model 竟然不知道从哪去讲。还请回头看看 Main.go 中的注释二处：

```go
beego.Router("/task/", &controllers.TaskController{}, "get:ListTasks;post:NewTask")
```

上述路由引导我们进入了 TaskController 这个控制器来了，我们分析一下下面这个文件：

```go
package controllers

import (
	"encoding/json"
	"strconv"

	"github.com/astaxie/beego"
	"samples/todo/models"
)

type TaskController struct {
	beego.Controller
}

// Example:
//
//   req: GET /task/
//   res: 200 {"Tasks": [
//          {"ID": 1, "Title": "Learn Go", "Done": false},
//          {"ID": 2, "Title": "Buy bread", "Done": true}
//        ]}
func (this *TaskController) ListTasks() {
	res := struct{ Tasks []*models.Task }{models.DefaultTaskList.All()}
	this.Data["json"] = res
	this.ServeJSON()
}
```

很明显我们看到了 models 关键字,并且调用了其中的 Task ，我们选择进入 Task.go 文件看看：

```go
package models

import (
	"fmt"
)

var DefaultTaskList *TaskManager

// Task Model
type Task struct {
	ID    int64  // Unique identifier
	Title string // Description
	Done  bool   // Is this task done?
}

// NewTask creates a new task given a title, that can't be empty.
func NewTask(title string) (*Task, error) {
	if title == "" {
		return nil, fmt.Errorf("empty title")
	}
	return &Task{0, title, false}, nil
}

// TaskManager manages a list of tasks in memory.
// 注释一
type TaskManager struct {
	tasks  []*Task
	lastID int64
}

// NewTaskManager returns an empty TaskManager.
func NewTaskManager() *TaskManager {
	return &TaskManager{}
}

// Save saves the given Task in the TaskManager.
func (m *TaskManager) Save(task *Task) error {
	if task.ID == 0 {
		m.lastID++
		task.ID = m.lastID
		m.tasks = append(m.tasks, cloneTask(task))
		return nil
	}

	for i, t := range m.tasks {
		if t.ID == task.ID {
			m.tasks[i] = cloneTask(task)
			return nil
		}
	}
	return fmt.Errorf("unknown task")
}

// cloneTask creates and returns a deep copy of the given Task.
func cloneTask(t *Task) *Task {
	c := *t
	return &c
}

// All returns the list of all the Tasks in the TaskManager.
func (m *TaskManager) All() []*Task {
	return m.tasks
}

// Find returns the Task with the given id in the TaskManager and a boolean
// indicating if the id was found.
func (m *TaskManager) Find(ID int64) (*Task, bool) {
	for _, t := range m.tasks {
		if t.ID == ID {
			return t, true
		}
	}
	return nil, false
}

func init() {
	DefaultTaskList = NewTaskManager()
}
```

如上所示的 task Model 主要就是定义了 task 这个实体该有的成员变量。以及一个 taskManager 来管理这些 task，其整体结构在理解了 Go 语言的一些基本的机制之后还是比较简单的。

在之前的整个实例结构中，我们还看到了如下所示的静态文件，它们的作用就很明显啦：

```shell
├── static
    │   ├── css
    │   ├── img
    │   └── js
```

好了，以上就是 Go 的一个框架 Beego 的入门实例了，其实很简单。我也只是简单的写一下入门的东西。后续研究一下 Go 的自动化 API 构建。往后继续学习 Go 和 Docker 的结合应用吧。

以上，谢谢阅读！！！