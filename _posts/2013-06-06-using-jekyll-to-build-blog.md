---
layout: post
title: 利用Jekyll构建个人静态博客
category: technoledgy
tags: [jekyll, liquid, markdown]
---
一直想建立一个自己的博客来记录学习历程，与大家分享个人的一些思考。可是我既不想折腾服务器、申请域名，也不愿使用各大网站提供的博客系统，这个想法就一直耽搁着。
直到前不久无意中了解到了 [Jekyll](http://jekyllrb.com/) + [GitHub Pages](http://pages.github.com/) 来构建博客的方法，这种方法很合我的胃口，
于是着手学习并搭建了这个博客。在此我简单的介绍下利用Jekyll构建博客的方法，相关的知识，以及我的一些经验来作为此博客的第一篇文章。

#Jekyll 简介

{:.quote}
> Jekyll is a simple, blog aware, static site generator.

这是来自 Jekyll 官网的一段说明。Jekyll 是一个简单的，适用于博客平台的，静态网站生成器。
Jekyll 提供了一套模板目录来作为构建网站的基础框架，以及快速构建所需页面的Rakefile，
并在此基础上支持 [Markdown](http://daringfireball.net/projects/markdown/) 、
[Textile](http://www.textism.com/tools/textile/) 、
[Liquid](https://github.com/Shopify/liquid) 标记语言的转换，最终生成一个完整的静态网站。你可以把生成的静态网站放在任意web服务器上。由于 GitHub Pages 支持 Jekyll 引擎，你也可以把用于生成静态站点的整个 Jekyll 项目上传到 GitHub 上，由后台的 Jekyll 引擎来生成一个具有username.github.io域名的静态站点。

<!-- excerpt -->

首先从安装 Jekyll 开始。由于 Jekyll 是 Ruby 写的，首先需要 [下载安装 Ruby](http://www.ruby-lang.org/zh_cn/downloads/) 。
如果你和我一样使用的是 Ubuntu 或者其他基于 Debian 的 Linux 发行版，可以使用

    $ sudo apt-get install ruby1.9.1

来安装 Ruby 。然后使用 RubyGems 来安装 Jekyll：

    $ gem install jekyll

Jekyll 依赖以下的gems模块：liquid、fast-stemmer、classifier、directory_watcher、syntax、maruku、kramdown、posix-spawn 和 albino 。
它们会在执行上述安装命令的时候被gem install命令自动安装。

一个典型的 Jekyll 项目的结构看起来像下面这个样子：

    .
    |-- _config.yml
    |-- _includes/
    |-- _layouts/
    |   |-- default.html
    |   |-- post.html
    |-- _posts/
    |   |-- 2013-06-05-jekyll-test.md
    |   |-- 2013-06-04-hello-world.md
    |-- _site/
    |-- index.html
    |-- assets/
        |-- css/
            |-- style.css
        |-- javascripts/

简单地介绍一下每个部分的功能：

- **\_config.yml**  
  存放 Jekyll 所需的配置文件，例如我把一些常用的路径作为变量存放于此，用 liquid 标记语言写页面生成代码的时候可以方便调用。

- **\_includes**  
  可将一些能与 \_layouts 和 \_posts 中的文件混合、匹配使用，并且具有重用价值的文件存放于此(按照我的经验，多为liquid代码。使用的时候，
  在文件中嵌入liquid标签 {% capture liquidcode %}||.% include path\filename %.||{% endcapture %}{% include AH/print_code %} 来调用 \_includes\path\filename 文件。

- **\_layouts**  
  “模板文件”存放于此。所谓的模板文件常用来进行网页布局，模板文件之间也可以嵌套调用。

- **\_posts**  
  所有博客文章的文本文件存放于此。每篇博文均必须以 "2013-06-06-using-jekyll-to-build-blog" 这样的格式命名，文件格式可以是 .html、.md、.textile，你可以选择自己熟悉的标记语言或者直接使用html来书写博文。还是以本文为例，如果在文档中没有指定 title 属性值，则 title 默认为 "using jekyll to build blog" 。

- **\_site**  
  该路径下存放的是 Jekyll 最终生成的静态站点的所有文件，每次运行 Jekyll 引擎的时候自动生成。
所以假如你和我一样使用 GitHub 托管博客，可以把 \_site 加入到 .gitignore 列表中。

- **index.html**  
  提供静态站点的首页，同样，也可以是 index.md 或者 index.textile。这也适用于任何其他自建的文件。

- **assets(其他目录或文件)**  
  我们还可以根据自己的网站需要创建任意的目录或文件，例如我创建了一个 assets/ 目录，将站点所用到的 css、js 文件以及图片均存放于此进行管理。

在 Jekyll 项目中，任何以下划线开头的内容都是用于生成网站的，不会出现在最终的静态网站目录下，而任何不以下划线开头的内容都会被复制到 \_site 里。

在 Jekyll 项目的主文件夹下运行如下命令

    $ jekyll --server

(注：从 Jekyll1.0 起启动本地web服务器的命令变为了 `jekyll serve` ，另外可使用命令 `jekyll new` 来生成一个默认的 Jekyll 项目结构。)

这样可以在本地启动一个临时的web服务器，在浏览器中输入 `http://localhost:4000` 便可以访问生成的静态站点。这样可以方便在本地调试站点。注意，在有些系统里，可能还需要手动添加 Jekyll 的环境变量。

#开始建立博客

接下来我以一个完全不加任何 css 样式简单的例子来说明。

##在 \_layouts 下建立模板

首先在 \_layouts 下创建模板文件 mydefault.html ，内容如下：

{% assign lang = 'html' %}
{% capture codeblock %}<!DOCTYPE html>
<html>
  <head>
    <meta http-equiv="content-type" content="text/html; charset=utf-8" />
    <title>
      ||.% if page.title %.|| ||.{ page.title }.|| | ||.% endif %.|| Andy24boyking
    </title>
  </head>
  <body>
    <header>
      <h1>点滴积累 Andy24boyking's Blog</h1>
      <ul>
        <li>
          <a href="||.{ site.baseurl }.||"> Home </a>
        </li>
        <li>
          <a href="about"> About </a>
        </li>
      </ul>
    </header>
      ||.{ content }.||
    <footer>
      <p>
        @copy; Andy24boyking 2013 | All Rights Reserved.
      </p>
    </footer>
  </body>
</html>
{% endcapture %}
{% include AH/print_code %}

模板中诸如 {% capture liquidcode %}||.{ ... }.|| 、 ||.% ... %.||{% endcapture %}{% include AH/print_code %} 的内容即为 liquid 标记语言。
其中前者引用的内容会直接输出，后者则是执行一些判断操作，包括 if/else、for 循环等，更加详细的说明请参阅 [liquid 的 wiki 页](https://github.com/shopify/liquid/wiki/liquid-for-designers) 。
模板中的 `page.title` `content` 都是 Jekyll 提供的变量，前者的值为当前页面的 title ，后者的值即为使用该模板的文件中的所有内容，假设我在 index.html 中使用模板mydefault，
则最后生成的网页即为将 index.html 中的内容全部嵌入到 `content` 所在位置后呈现，Jekyll 还提供了许多实用的变量，可以参阅 [Jekyll 的变量说明页](http://jekyllrb.com/docs/variables/)。而 `site.baseurl` 则是我自己在 \_config.yml 中自定义的变量，即为网站的根目录。

    baseurl : /

##填写 \_config.yml

关于 \_config.yml ，这里暂时不说太细，大家知道可以自定义变量 variable ，并且在 liquid 里能以 site.variable 的形式调用即可。Jekyll 官网的 [configuration 说明页](http://jekyllrb.com/docs/configuration/)的最下方提供了一个可行的 \_config.yml 文件模板，大家可以暂时用这个。
在[后面的章节](#_jekyll_)里大家还会接触到很多别人的 \_config.yml 文件，
看得多试得多了就知道自己该怎么灵活定义了。

##建立index.html以及发一篇博文

*index.html* 文件内容如下：

{% assign lang = 'rb' %}
{% capture codeblock %}---
layout: mydefault
---
<p> 最新文章 </p>
<ul>
  ||.% for post in site.posts %.||
    <li>||.{ post.date | date_to_string }.||
      <a href="||.{ post.url }.||"> ||.{ post.title }.|| </a>
    </li>
  ||.% endfor %.||
</ul>
{% endcapture %}
{% include AH/print_code %}

在 \_posts 里新建一篇文档 *2013-06-06-hello-jekyll.md* ，内容如下：

{% assign lang = 'rb' %}
{% capture codeblock %}---
layout: mydefault
title: 你好，jekyll
---
#||.{ page.title }.||
##||.{ page.date | date_to_string }.||

第一篇博文。你好，jekyll 。
{% endcapture %}
{% include AH/print_code %}

简单讲解一下。大家注意到这两个文档的开头均有以三个短杠开始和结束的部分，这是 **YAML front matter** 块，该块必须位于文档最开头，并且里的内容会被 Jekyll 优先处理。
上面两个文档中均有的 `layout: mydefault` 即为该文档以 mydefault 为模板解析。

而在文档  *2013-06-06-hello-jekyll.md* 里(大家注意命名格式，前面已经提到过)还有一条 `title: 你好，jekyll` 则是设置该文档的 title ，如果没有设置，则 Jekyll 会从标题中解析，得到 title 为 "hello jekyll"。
有关 YAML front matter 的介绍可以参看官网的 [Front-matter 说明页](http://jekyllrb.com/docs/frontmatter/) 。
这篇文档的内容即为打印文章标题，文章创建日期，文章内容。
至于 # 和 ## 则分别表示该行内容为一级标题和二级标题，这是 Markdown 的语法。

至此，一个最简洁的博客系统已经成型了，为了演示效果，我又类似的建立了两篇博文文档。使用 `jekyll --server` 命令运行后效果如下：

<p class="post-image"><img src="{{BASE_PATH}}/assets/posts/images/2013-06-06-index.png"><br/>博客首页</p>
<p class="post-image"><img src="{{BASE_PATH}}/assets/posts/images/2013-06-06-post.png"><br/>文章页</p>


#在 GitHub 上托管博客

如果需要把网站托管在 GitHub 上，只需要把 Jekyll 项目添加到 git 仓库，并且同步到 GitHub 上即可。项目名称要起为 `username.github.com` 的形式，这样就会被解析为 GitHub Page ，生成对应域名的网站。托管在 GitHub 上可以充分享受 git 版本控制带来的方便，网站可以随时退回到某个版本状态，在本地写好了新的文章，只需 commit 即可。如果你对 git 和 GitHub 还不熟悉，在网上可以查到很多相关资料，我这里就不再赘述，推荐一个简单明了的 [入门介绍](http://rogerdudler.github.io/git-guide/index.zh.html) 。

#使用 Jekyll 主题模板快速建立博客

有了上面的认识后，想必大家都已经摩拳擦掌开始规划自己的博客了。一开始我也是如此，开始自己设计版面，写 css 样式，制作模板。
但是我很快就意识到，这对于不熟悉网页前端开发的我是个极其巨大的任务，而且更重要的是，我费尽心思设计出来的版面总是显得太简陋、不好看 :( 。所以对于大多数和我一样不擅长做网页设计的程序员来说，最好的方法是在现有模板的基础上进行修改，来创建一个看得过去的个人博客。

Jekyll 提供了一个[站点推荐列表](https://github.com/mojombo/jekyll/wiki/Sites)，大家可以浏览一下，找到自己喜欢的风格，然后 clone 他博客的 GitHub 项目，仔细研究，改造出一个适合自己的博客网站。
大家在浏览的过程中可能会发现，不少列表中的博客都是在一个叫做 [Jekyll-Bootstrap](http://jekyllbootstrap.com/) 的模板主题的基础上做出来的。
顾名思义，这个模板主题是基于当前已经滥用到审美疲劳的 [Twitter Bootstrap](http://twitter.github.io/bootstrap/) 前端模板的基础上开发出来的。
话虽如此，我还是选择在 JB(不是一个很友好的缩写...) 的基础上创建的我的博客系统，并且在仔细学习研究后几乎重写了原作者的框架，似乎现在可以称之为 andy-Bootstrap 了 :) 。当然这是玩笑话，没有 Jekyll-Bootstrap 的帮助我的博客是建立不起来的。JB 的 git 地址是 `https://github.com/plusjade/jekyll-bootstrap.git` 。

#总结

如何使用 Jekyll 建立个人静态博客的相关事项已经基本介绍完毕，下面简单总结一下需要使用的工具和需要参考的资料。

可以使用的编辑器(对 Markdown 和 Textile 有较好的语法高亮支持(其中有的是插件支持))

- **Vim**
- **GNU Emacs**
- **TextMate**
- **gedit**
- **Kate**
- **Sublime Text 2**

我个人使用的是 Sublime Text 2，但是在 linux 系统中的中文输入支持有点问题，详情请自行了解。

可能会用到的标记语言和模板引擎

- **[Markdown](http://daringfireball.net/projects/markdown/)**
- **[Textile](http://www.textism.com/tools/textile/)**
- **[Liquid](https://github.com/Shopify/liquid)**

一个好用的模板主题

- **[Jekyll-Bootstrap](http://jekyllbootstrap.com/)**

<p style="margin-top:60px;">参考资料：</p>

【1】 [Jekyll 文档](http://jekyllrb.com/docs/home/)

【2】 [用Jekyll构建静态网站【译文】](http://yanping.me/cn/blog/2011/12/15/building-static-sites-with-jekyll/)

【3】 [像黑客一样写博客——Jekyll入门](http://www.soimort.org/posts/101/)

【4】 [liquid 文档](https://github.com/shopify/liquid/wiki/liquid-for-designers)
