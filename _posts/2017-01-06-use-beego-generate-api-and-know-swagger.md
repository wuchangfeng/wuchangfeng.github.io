---
layout: post
title: 体验 Beego API 开发和自动化文档
date: 2017-01-06 19:57:12 +0800
categories: Go
---

学习 Go 的基础语法时候，我就看到一些相关博客，说用 Go 来开发 API 非常简单友好。随手 Google 了一下，当时就看到可以用 Beego 框架来开发 API 并且还能结合 Swagger 自动化生成文档。当时也没有时间或者没有响应基础知识体验一下，近来时间宽裕了不少。话不多说，跟随官方文档来体验一番。不过官方文档对于基础的数据库知识介绍的一笔带过啦。

### 数据导入

肯定要在本地安装、配置好数据库，写入数据信息。我们选择 MySql 数据库，毕竟这个用的还是挺多的 。具体的 Mac 平台下的安装配置自己 Goolge 一下就好了。还要推荐一个 Mac 平台下的数据库可视化工具—Sequel Pro。我个人也是刚接触这个可视化工具，还有许多不熟悉，边用边摸索吧。具体关于这个数据管理软件的使用可以参见[这里](https://segmentfault.com/a/1190000006255923)。

![](http://ww4.sinaimg.cn/large/b10d1ea5jw1fbg1io6q8nj21go0z4wne.jpg)

接下来数据库的建表(建表名称为 app)语句如下所示：

```mysql
CREATE TABLE `app` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `create_date` datetime NOT NULL,
  `app_code` varchar(255) COLLATE utf8_unicode_ci NOT NULL,
  `app_name` varchar(255) COLLATE utf8_unicode_ci NOT NULL,
  `publish_date` date DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `app_code` (`app_code`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;
```

向 app 表插入两条数据：

```mysql
INSERT INTO `app` VALUES ('1', NOW(), '100000', '神庙逃亡', '2015-08-06');
INSERT INTO `app` VALUES ('2', NOW(), '100001', '愤怒的小鸟', '2015-08-06');
```

对应的终端截图如下所示：

![](https://ws2.sinaimg.cn/mw690/b10d1ea5jw1fbfs4d0u7mj211w0z4tkz.jpg)

### 创建 API 项目

通过上述创建好的数据库表，继续结合 Beego 框架，我们执行如下命名生成 API 项目：

```shell
bee api go-api -conn="root:123456@tcp(127.0.0.1:3306)/api"
```

其中项目名称叫做 go-api，数据库名称叫做 api。Mysql 用户名为 root，密码为 123456。命令行的截图如下所示：![](https://ws2.sinaimg.cn/large/b10d1ea5jw1fbfzqo10atj211w0z4h1v.jpg)

我们打开指定目录，查看是否自动生成 API 项目，指定目录下生成如下所示项目，成功生成：

![](http://ww4.sinaimg.cn/large/b10d1ea5jw1fbg0qzycprj21gu1a4k2h.jpg)

### 集成Swagger UI

首先我们开启本地调试：

``` shell
bee run -downdoc=true
```

swagger ui 是一个 API 在线生成文档的生成和测试的最好的工具，评价也是非常好的。API 设计人员完成项目之后，就可以远离痛苦的写文档的活了。

我们到[这里](https://github.com/beego/swagger/releases)下载 swagger 压缩包，解压。赋值其中的内容到我们刚才生成的项目中的 swagger 文件夹中。最后访问http://127.0.0.1:8080/swagger. 即可进入如下界面，怎么样，第一感觉是不是很炫酷？接下来，我们在来体验一下功能上的全面吧。

![](https://ws4.sinaimg.cn/large/b10d1ea5jw1fbfzwrpwjej21kw17hgwe.jpg)

小试牛刀，试一下 Get 操作，获取全部的内容。如下所示显示了数据库表中的两条数据：

![](https://ws2.sinaimg.cn/large/b10d1ea5jw1fbg0erwh4vj21kw17hdrt.jpg)

接着试一下，每次请求取限量数据的操作，也就是 Limit 操作。如下图所示限制每次请求只取一条数据：

![](http://ww1.sinaimg.cn/large/b10d1ea5jw1fbg0jrd2ruj21kw17htky.jpg)

哈哈，是不是很简单。测试和开发人员都很方便。还有其他的一些常见的网络请求操作，就不做过多说明了，自己体验一下就能感觉得到方便啦。

### 代码解读

找到自动生成的项目路径，在 Main 文件中代码如下所示，跟一般的 Beego 项目没有什么区别，引入了 Swagger 文件。

```go
package main

import (
	_ "beego-api/routers"

	"github.com/astaxie/beego"
	"github.com/astaxie/beego/orm"
	_ "github.com/go-sql-driver/mysql"
)

func init() {
	orm.RegisterDataBase("default", "mysql", "root:123456@tcp(127.0.0.1:3306)/api")
}

func main() {
	if beego.BConfig.RunMode == "dev" {
		beego.BConfig.WebConfig.DirectoryIndex = true
		beego.BConfig.WebConfig.StaticDir["/swagger"] = "swagger"
	}
	beego.Run()
}
```

在 router.go 中代码如下所示：

```go
// @APIVersion 1.0.0
// @Title beego Test API
// @Description beego has a very cool tools to autogenerate documents for your API
// @Contact astaxie@gmail.com
// @TermsOfServiceUrl http://beego.me/
// @License Apache 2.0
// @LicenseUrl http://www.apache.org/licenses/LICENSE-2.0.html
package routers

import (
	"beego-api/controllers"

	"github.com/astaxie/beego"
)

func init() {
  	// 注释一处
	ns := beego.NewNamespace("/v1",
		// 注释二处
		beego.NSNamespace("/app",
			beego.NSInclude(
				&controllers.AppController{},
			),
		),
	)
	beego.AddNamespace(ns)
}
```

对于上面注释的两处代码，引入官方文档的一些说明：

> 我们看一下路由代码，这是一个层级嵌套的函数，第一个参数是`/v1`，即为`/v1`开头的路由树，第二个参数是`beego.NSNamespace`，第三个参数也是`beego.NSNamespace`，也就是路由树嵌套了路由树，而我们的`beego.NSNamespace`里面也是和`NewNamespace`一样的参数，第一个参数是路由前缀，第二个参数是slice参数。这里我们调用了`beego.NSInclude`来进行注解路由的引入，这个函数是专门为注解路由设计的，我们可以看到这个设计里面我们没有任何的路由信息，只是设置了前缀，那么这个的路由是在哪里设置的呢？我们接下来分析什么是注解路由。

接下来看一下 controllers 中的 app.go 文件。

```go
// GetOne ...
// @Title Get One
// @Description get App by id
// @Param	id		path 	string	true		"The key for staticblock"
// @Success 200 {object} models.App
// @Failure 403 :id is empty
// @router /:id [get]
func (c *AppController) GetOne() {
	idStr := c.Ctx.Input.Param(":id")
	id, _ := strconv.Atoi(idStr)
	v, err := models.GetAppById(id)
	if err != nil {
		c.Data["json"] = err.Error()
	} else {
		c.Data["json"] = v
	}
	c.ServeJSON()
}
```

官方的解释依然是最好的。注意为了精简代码篇幅，将其中的 `post`操作换成了 `Get`操作：

> 我们看到我们的每一个函数上面有大段的注释，注解路由其实主要关注最后一行`// @router / [post]`，这一行的注释就是表示这个函数是注册到路由`/`，支持方法是`post`。
>
> 和我们平常的时候使用`beego.Router("/", &ObjectController{},"post:Post")`的效果是一模一样的，只是这一次框架帮你自动注册了这样的路由，框架是如何来自动注册的呢？在应用启动的时候，会判断是否有调用`NSInclude`，在调用的时候，判断RunMode是否是`dev`模式，是的话就会判断是否之前有分析过，并且分析对象目录有更新，就使用Go的AST进行源码分析(当然只分析`NSInclude`调用的`controller`)，然后生成文件`routers/commentsRouter.go`，在该文件中会自动注册我们需要的路由信息。这样就完成了整个的注解路由注册。
>
> 注解路由是使用`// @router `开头来申明的，而且必须放在你要注册的函数的上方，和其他注释`@Title @Description`的顺序无关，你可以放在第一行，也可以最后一行。有两个参数，第一个是需要注册的路由，第二个是支持的方法。
>
> 路由可以支持beego支持的任意规则，例如`/object/:key`这样的参数路由，也可以固定路由`/object`，也可以是正则路由`/cms_:id([0-9]+).html`

其他的比较重要的就是 model 文件夹中的文件了，就是实体内容，其实也是根据数据库中的 app 这张表生成的。



好了，以上就是关于利用 Beego 框架生成 API 项目并且自动化构建文档的实例内容。要探索的还有许多，继续加油。

以上，谢谢阅读。