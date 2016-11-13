# Blog

脚本 Rakefile 简化 Jeklly 发布博客的流程：

* 保存 Rakefile 到 Jekyll 创建的站点根目录下
* 在根目录下执行 rake new 
* 输入 URL，比如要创建 2016-11-12-go-learn.md 文件，我们只需要输入 -go-learn，当然，你也可以输入中文，但通常不建议
* 输入文章标题，即 .md 文件中 front matter 里的 title 的值
* 输入文章的分类，如果有多个分类，以空格分隔，比如 Jekyll zfanw 这样
* 最后，Rake 会启动 vi，打开新创建的 md 文件，当然，你也可以配置其他编辑器

博客样式面板来源于 [mthli](https://github.com/mthli/mthli.github.io),分页功能样式定义来自于 [写代码的猴子](http://jaeger.itscoder.com/)，Thanks！！！