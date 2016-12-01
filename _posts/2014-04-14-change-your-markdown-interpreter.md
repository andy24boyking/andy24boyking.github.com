---
layout: post
title: 更换博客的Markdown解析程序
category: news
tags: markdown
---
前几天 push 新文章的时候 github 上给出了警告，大意是说我博客使用的 Markdown 解析程序已经过时了，最好更换为新的。了解了一下，Github Pages 上使用的 Jekyll 引擎默认的 Markdown 解析程序是 [Maruku](https://github.com/bhollis/maruku/)。而从2012年10月起，Maruku 的维护者就声称他们即将关闭这个项目。因此 Maruku 现在已经过时了，继续使用的话有可能会在处理非法的 Markdown 或者 HTML 语法时出现不可预期的错误。

目前 Github Pages 还没有采用新的默认 Markdown 解析器，因此我们可以自己先切换到新的 Markdown 解析器，例如 [Kramdown](https://github.com/gettalong/kramdown)。只需要在博客项目的`_config.yml`配置文件里加上如下指定即可：

    markdown: kramdown

这条语句会为 Github Pages 指定相应的 Markdown 解析器。除了 Kramdown 还有一些其他的解析器，例如 [Redcarpet](https://github.com/vmg/redcarpet)。不过据 github 上面的文章说，一些已经更换了 Markdown 解析器的人发现 Kramdown 更适合。

Kramdown 和 Maruku 对于 Markdown 的语法实现基本上是一致的。如果你是初次接触 Kramdown 或者 想熟悉一下 Markdown 的语法。你可以查看 Kramdown 官网上的[快速指南](http://kramdown.gettalong.org/quickref.html)或者[完整文档](http://kramdown.gettalong.org/documentation.html)。Kramdown 的实现中也存在一些和标准 Markdown 语法不一致的地方，在它官网的[语法指南](http://kramdown.gettalong.org/syntax.html)里用黄色标出来了，有兴趣的同学可以关注一下。
