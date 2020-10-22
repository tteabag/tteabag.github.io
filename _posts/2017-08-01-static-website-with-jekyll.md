---
layout: post
title: "从WordPress到静态网站"
subtitle: "我是subtitle，看看是怎么样的"
date: 2017-08-01 12:00:00
categories: coding4fun
tags: [github, jekyll, bootstrap]
---

作为一个极简主义者，越来越受不了WordPress的臃肿。主要自己平时都是用Markdown来做记录，而WordPress支持的不是很好，另外有时写的md长文通过三方软件导出的html也不造放在wp的什么地方才合适，管理起来还不如几个静态页面来得方便。于是有了某天来个大整顿的想法。

对前端只停留在css的我，花了整三天时间，从Hexo到Jekyll再到Bootstrap，以及各种markdown2html解析器...最后基本算是从头写了一个基于Jekyll的website theme 😓  在此记录了各种前端坑，自认为还算详细吧。<!-- more -->


# Hexo

主流的静态页面生成器有俩，一个Jekyll，一个Hexo，都支持Markdown。 前者用的是Ruby，后者为Node.js。

Hexo会比Jekyll搭建起来更方便一些，而且各种配置项的设计也很便捷，网站themes还很丰富，所以我一开始弄的就是这个。

* [Hexo官方文档](https://hexo.io/zh-cn/docs/)
* [next](https://github.com/iissnan/hexo-theme-next) : 一款优雅并且功能强大的主题. [[doc]](http://theme-next.iissnan.com/getting-started.html) [[demo]]([http://notes.iissnan.com](http://notes.iissnan.com/))

搭建方法上面这两个链接写的很详细。

这里主要记录一下让Hexo支持LaTex的注意地方。Hexo用MathJax时会有一些[转义问题](http://2wildkids.com/2016/10/06/%E5%A6%82%E4%BD%95%E5%A4%84%E7%90%86Hexo%E5%92%8CMathJax%E7%9A%84%E5%85%BC%E5%AE%B9%E9%97%AE%E9%A2%98/#)，经我自己测试，最好的方式是用`hexo-renderer-pandoc`渲染器代替原有的`hexo-renderer-marked`。而网上说的用`hexo-renderer-kramed`，对矩阵的情况显示的不好。

~~~
$ npm uninstall hexo-renderer-marked --save
$ npm install hexo-renderer-kramed --save
~~~


# Jekyll

其实用Hexo/Jekyll+现成的主题，很方便就能搭出一个静态网站出来。只是除了代码高亮、LaTex这些功能之外，我还想能区分出code block 和普通无需highlighting的`<pre>`部分，还有其他一些改动。因为markdown解析的原因，基本所有themes出的效果都是将两者统一处理了。强迫症（其实就是zuo），主题挑来挑去都没找到个合心意的，比如next这种，太复杂，让我一个前端白痴改，哈，那还是自己从头写一个比较符合程序员造轮子的习惯吧。。。（捂脸

（至于为啥从Hexo转到了Jekyll，原因未明）

## Get Started

参考：[Jekyll Doc](https://jekyllrb.com/docs/home/) ([中文版](http://jekyll.com.cn/docs/home/)，内容比较旧)

### Installation

本来在本地用Homebrew安装一路顺畅，但放到服务器就各种问题。

首先直接用apt-get下载的Ruby版本比较旧。

~~~
# Install Ruby & RubyGems
$ sudo apt-add-repository ppa:brightbox/ruby-ng
$ sudo apt-get update
$ sudo apt-get install ruby2.4

# Install Jekyll and Bundler gems
$ sudo gem install jekyll bundler
~~~
然后在`gem install jekyll`时会报错：

```
error: could not find a valid gem (>= 0) in any repository
```

参考[github issues](https://github.com/jekyll/jekyll/issues/1409)，F*k GWF，需要将`https://rubygems.org/`换成其他可访问的镜像，如[Ruby China 镜像](http://gems.ruby-china.org/)。

```
$ gem sources --remove https://rubygems.org/
$ gem sources -a https://gems.ruby-china.org/
$ gem sources -l
*** CURRENT SOURCES ***
https://gems.ruby-china.org/   
# 确保只有 gems.ruby-china.org
```

重新尝试 install jekyll，还是报错：

```
current directory: /var/lib/gems/2.4.0/gems/ffi-1.9.18/ext/ffi_c
/usr/bin/ruby2.4 -r ./siteconf20170801-1836-1n75umz.rb extconf.rb
mkmf.rb can't find header files for ruby at /usr/lib/ruby/include/ruby.h
```

解决方式：

```
$ sudo apt-get install ruby2.4-dev
```

之后就可以正常安装jekyll了。

另外，根据[Ruby China 镜像](http://gems.ruby-china.org/)，你可以用 Bundler 的 [Gem 源代码镜像命令](http://bundler.io/v1.5/bundle_config.html#gem-source-mirrors)：

```
$ bundle config mirror.https://rubygems.org https://gems.ruby-china.org
```
这样就不用改你的 Gemfile 中的 source了。

```
source "https://rubygems.org"
...
```

### Basic Command

用jekyll新建一个site:

```
$ jekyll new myblog
$ cd myblog
$ bundle exec jekyll serve
```

打开http://localhost:4000测试。

默认使用的是`Minima theme`. 目前基于jekyll 3.x的themes还比较少。
因为我是自己新建的theme，就不用这样方式了。

---

启动服务：

```
# 启动服务
$ jekyll serve

# 启动服务(gem-based theme)
$ bundle exec jekyll serve

# 脱离终端在后台运行
# 如果你想关闭服务器，可以使用`kill -9 1234`命令，"1234" 是进程号（PID）。
# 如果你找不到进程号，那么就用`ps aux | grep jekyll`命令来查看，然后关闭服务器。
$ jekyll serve --detach

# 和jekyll serve相同，但是会查看变更并且自动再生成。
$ jekyll serve --watch
```

生成静态页面（位于`_site`目录）：

```
$ jekyll build
```

或者用：(watched for changes, and regenerated automatically.)

```
$ jekyll build --watch
```

# Set up Your Site on VPS

网上清一色都是通过github pages来发布的站点，然后通过修改`CNAME`文件来达到绑定域名的目的。

我这里是放到VPS上，需要自己搭建web server环境。

> Jekyll is a static site generator, not a webserver. You may generate the static files and serve with webserver like `nginx`, which provides such abilities.

参考：

* [How To Get Started with Jekyll on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-get-started-with-jekyll-on-an-ubuntu-vps)
* [Set up a Jekyll site on a vps with Ubuntu, Nginx and Letsencrypt](https://thomasroest.com/2016/11/05/set-up-a-jekyll-site-on-a-vps-with-ubuntu-nginx-and-letsencrypt.html)

因为是静态网站，所以Jekyll其实安装在本地就可以，静态页面也放在本地。然后将Jekyll生成的静态HTML文件（`_site`目录下）通过类似FTP的方式上传到VPS就行。

将本地数据上传到远程服务用`scp`就行，后来了解到`rsync`这个命令。`rsync`只传送两个文件的不同部分，而不是每次都整份传送，因此速度很快。

先来理清关系：

- [Jekyll](http://jekyllrb.com/) for write our content
- [nginx](http://nginx.org/en/) to serve our content
- [Capistrano](http://www.capistranorb.com/) to deploy

## Nginx

```
sudo apt-get install nginx
sudo service nginx start
```

然后输入vps地址就能看到nginx的欢迎页面。

查看运行状态：

```
sudo service nginx status
# 显示：
- nginx is running
```

修改 nginx 配置：sudo vim /etc/nginx/sites-enabled/default

将 `root /usr/share/nginx/html;` 注释掉，改为 `root /home/deploy/your_blog_name/_site`。

将 `server_name localhost;` 注释掉，改为 `server_name your_domain.com;`。

重新启动nginx，就能看到建的blog内容了。

## Capistrano

Capistrano 是一个 Ruby 程序，参考第一个文章链接，它可以通过Git复制代码到服务器等操作。

```
sudo gem install capistrano 
```

这里先存留吧，暂时还没用到Capistrano。

目前是按照[利用Apache进行多站点配置](http://blog.lszero.com/coding4fun/multisite-setting.html)设置的。

# Create New Theme From Scratch

既然是from scratch，就先把jekyll的目录结构理清，然后借助bootstrap框架创建主题，最后就是各种features了。

## Basic

### Directory Structure

用`jekyll new-theme xxx`命令，就可以创建出一个theme需要的基本目录项。

```
.
├── _config.yml
├── _drafts
|   ├── begin-with-the-crazy-ideas.textile
|   └── on-simplicity-in-technology.markdown
├── _includes
|   ├── footer.html
|   └── header.html
├── _layouts
|   ├── default.html
|   ├── post.html
├── _posts
|   ├── 2007-10-29-why-every-programmer-should-play-nethack.textile
|   └── 2009-04-26-barcamp-boston-4-roundup.textile
├── _data
|   └── members.yml
├── _site
└── index.html
```

说明：

`_config.yml` 是配置文件。默认配置参见：[https://jekyllrb.com/docs/configuration/#default-configuration](https://jekyllrb.com/docs/configuration/#default-configuration)

`_includes` 里的文件为了布局重用。

`_layouts` 站点布局模板。布局可以在 [YAML 头信息](http://jekyll.com.cn/docs/frontmatter/)中根据不同文章进行选择。 

`_posts` 将md文件以`2017-08-01-welcome-to-jekyll.md`的格式放到该目录中。

`_drafts` 中的md文件直接以`title.md`命名，并不会发布出来。当运行`jekyll serve`或者`jekyll build --drafts`时，草稿文章会被加上日期值并发布出来。

`_site` 存放jekyll转化完成的html文件。有时修改效果不生效，可以尝试删除该目录。

### Variables

自带的变量详见：[Variables](http://jekyllrb.com/docs/variables/)。

或者通过`_config.yml`添加。

### Need to Know

若在`index.html`的YAML头信息中加入：

```
---
layout: default
...
---
```

则在`default.html`中调用{% raw %}`{{ content }}`{% endraw %}时，就会把`index.html`中的所有内容放到调用处。

## Bootstrap

Bootstrap前端框架真是我这种前端白痴的福音。

下载[Bootstrp](https://github.com/twbs/bootstrap/releases/download/v3.3.7/bootstrap-3.3.7-dist.zip)，我这里将解压后的文件放到了`assets/bootstrap-3.3.7-dist` 目录下。

使用Bootstrap：

参考Bootstrap官网里给的[基本HTML模板](http://getbootstrap.com/getting-started/#template)，

（1）将下面这行代码改成自己的相应路径后，放到`head.html` 的`<head></head>` 标签中。

```  html
<!-- Bootstrap -->
<link rel="stylesheet" href="/assets/bootstrap-3.3.7-dist/css/bootstrap.min.css">
```

（2）将下两行改成自己的相应路径后，放到`default.html` 的`<body></body>` 标签中的最下方，为了最后加载。

```html
<!-- jQuery (necessary for Bootstrap's JavaScript plugins) -->
<script src="https://ajax.googleapis.com/ajax/libs/jquery/1.12.4/jquery.min.js"></script>
<!-- Include all compiled plugins (below), or include individual files as needed -->
<script src="/assets/bootstrap-3.3.7-dist/js/bootstrap.min.js"></script>
```

若没效果，删除 `_site` 目录再试试，或者用命令重新 build。

### NavBar

参考Bootstrap官网给的[示例代码](http://getbootstrap.com/components/#navbar)，写入`header.html`文件中。

然后在`default.html`的`<body></body>`标签中引入{% raw %}`{% include header.html %}`{% endraw %}。

注意的点是，

若使用了`.navbar-fixed-top` 类，这个固定的导航条会遮住页面上的其它内容，除非你给 `<body>` 元素底部设置了 `padding`，例如：

```
body { padding-top: 70px; }
```

## Features

### Post Excerpt

在`index.html` 中显示文章摘要。参见[https://jekyllrb.com/docs/posts/#post-excerpts](https://jekyllrb.com/docs/posts/#post-excerpts)。

（1）自带的{% raw %}`{{ post.excerpt }}`{% endraw %} 会自动取第一段的内容作为摘要。

若想自定义，

（2）在`_config.yml` 文件中指定摘要的分隔符：

```
excerpt_separator: <!-- more -->
```

这样会覆盖自带的{% raw %}`{{ post.excerpt }}`{% endraw %}功能，然后在文章内容需要分隔的摘要后面加上`<!-- more -->`。而`<!-- more -->` 本身作为注释，不会影响 markdown 的显示。

（3）或者采用这样的方式：

```
{% raw %}{{ post.content | strip_html | truncatewords:75 }}{% endraw %}
```

### Pagination

用于`index.html` 里对文章的分页显示功能。

（1）首先安装`jekyll-paginate`：

```
$ sudo gem install jekyll-paginate
```

在配置文件`_config.yml`中添加：

```
plugins:
  - jekyll-paginate
paginate: 5
paginate_path: "/pages:num/"
```

其中，`paginate: 5` 设置的是分页数；
`paginate_path: "/pages:num/"` 设置的是URL的显示格式，如 http://localhost:4000/pages2/。

（2）然后参考[https://jekyllrb.com/docs/pagination/](https://jekyllrb.com/docs/pagination/)在`index.html`添加分页功能，并结合Bootstrap里的分页样式修改：[http://getbootstrap.com/components/#pagination](http://getbootstrap.com/components/#pagination)。

### SideBar

这里利用了 bootstrap 的栅格布局，栅格布局将一个页面分割成12个等宽的列。(详见 [example](http://getbootstrap.com/css/#grid-example-basic))

```html
<div class="row-fluid">
  <div class="col-md-9">
    <h2>Content</h2>
  </div>
  <div class="col-md-3">
    <h2>Sidebar</h2>  
  </div>
</div>
```

让侧边栏固定住，不随页面滚动而滚动：加入`class="affix"`。

---

**说明：**

我这里设置两种sidebar：

* 一种是`index.html`里的，用于显示author信息，写在`main_sidebar.html`中；
* 另一种是`post`对应的，用于显示目录列表，写在`post_sidebar.html`中。

---

### Social Icons

icons用的是 [fontawesome](http://fontawesome.io/)。我这里将解压后的 `fonts`和`css`文件夹复制到了`assets`目录下。

在`head.html`中添加：

```html
<link rel="stylesheet" href="/assets/css/font-awesome.min.css">
```

要使用哪个图标，直接去[官网](http://fontawesome.io/icons/)搜代码就行。

修改图标颜色，因为其本身就是一种字体，故使用`color `修改即可。

修改图标大小也是，用`font-size`即可，或者：

```html
<i class="fa fa-camera-retro"></i>
<i class="fa fa-camera-retro fa-2x"></i>
<i class="fa fa-camera-retro fa-3x"></i>
```

对官网搜不到的社交网站图标怎么办，如“知乎”的：

```html
<a href="{{ site.social_media.zhihu_url }}" class="btn" title="Zhihu"><span class="fa-stack fa-lg"><i class="fa fa-circle fa-stack-2x"></i><i class="fa fa-stack-1x fa-inverse">知</i></span></a>
```

### Table of Content

#### post中显示目录 

kramdown自带了解析目录的功能。详见：[https://kramdown.gettalong.org/converter/html.html#toc](https://kramdown.gettalong.org/converter/html.html#toc)

在md文件中加入：

```
# 无序列表
* TOC
{:toc}
{: .this-is-my-class}
```

或者：

```
# 有序列表
1. TOC
{:toc}
{: .this-is-my-class}
```

并且可在`_config.yml`中设置要显示的标题级别：

```
kramdown: 
  toc_levels: "2,3" 
```
#### sidebar中显示目录

侧边栏目录随浏览内容动态滚动功能，利用Bootstrap的[Scrollspy](http://getbootstrap.com/javascript/#scrollspy)来实现。

* 参考 [application.js](http://www.thomaszhao.cn/2015/01/08/how-do-i-build-this-jekyll-blog/#toctable-of-content-) 获取目录结构，和标题滚动效果。
* 参考 [JQUERY实现侧边栏随滚动条滚动并固定位置](https://www.mywpku.com/jquery-pin-elements.html) 来固定sidebar的位置，并且效果是：当滚动到指定模块时才置顶该模块。

最后写了个 [TOP.js](https://github.com/lszero/lszero.github.io/blob/master/assets/js/toc/TOC.js) 终于实现得还算满意了。（当然，还要配合 css 使用）

另外，其它不错的实现：

- [bootstrap-toc](https://afeld.github.io/bootstrap-toc/) ([github](https://github.com/afeld/bootstrap-toc/tree/gh-pages/dist))
- [Tocify plugin](http://gregfranko.com/jquery.tocify.js/)
- [TOC plugin](http://projects.jga.me/toc/)

#### Hide sidebar for mobile display

手机显示时，隐藏侧边栏，否则会显示混乱。

```css
@media (max-width: 800px) {
    #sidebar {
        display:none !important;
    }
}
```

### Search

参考：[Button addons](http://getbootstrap.com/components/#input-groups-buttons) & [Forms](http://getbootstrap.com/components/#navbar-forms)

将下面代码加入`header.html`中：

```html
<form class="navbar-form navbar-left" role="search">
  <div class="input-group">
    <span class="input-group-btn">
      <button class="btn btn-default" type="button">Go!</button>
    </span>
    <input type="text" class="form-control" placeholder="Search">
  </div>
</form>
```

对于search逻辑，目前还没找到一个很好的方式去实现。。。（如果知道的，麻烦告诉我一声！）

### Back to Top

该功能我放在了`post_sidebar.html`中：

```html
<a href="#top" class="back-to-top">^</a>
```

其实就是个link，最后再用css设置样式。

---

更新：

为了不让footer覆盖掉 back-to-top，只能把back-to-top写在footer后面。故改在了`default.html`里。

### Comments

#### Gitment

[Gitment](https://imsun.net/posts/gitment-introduction/) 挺不错的的。按照官网的说明引入js后，在其中修改自己的OAuth Application信息。

#### isso

这是自建评论系统的一种方式。[官网](https://posativ.org/isso/)需要fq.

我之后用的也是这种方法，详见：[搭建Isso评论服务](http://blog.lszero.com/coding4fun/comments-with-isso.html)

### Archives

{% raw %}`{% for post in paginator.posts %}`{% endraw %} 并不能使用，因为paginator只会对`index.html`有效。

`site.posts | size` 获取总的文章数。

因为jekyll不支持按日期获取，只能暂时用{% raw %}`{% for post in site.posts %}`{% endraw %}。

这部分待补充吧。

### Category

参考 [使用Category分类](https://segmentfault.com/a/1190000000406017#articleHeader2)。

首先在每个文章的YAML头信息中定义类别，如：

```
---
layout: post
title:  "your title"
categories: Algorithm
---
```

**获取所有类别：**

```html
{% raw %}{% for category in site.categories %}
  <h2>{{ category[0] }} ({{ category[1].size }})</h2>
  <ul>
  {% for post in category[1] %}
    <li>{{ post.date | date:"%d/%m/%Y"}}<a href="{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
  </ul>
{% endfor %}{% endraw %}
```

使用：

* `site.categories.size` 获取总的类别数。
* `category[0]` 获取分类名称。
* `category[1].size` 获取该分类下文章的数目。
* `site.categories.CATEGORY` 可得到某个特定类别的所有文章。

更多变量可参见：[http://jekyllrb.com/docs/variables/#site-variables](http://jekyllrb.com/docs/variables/#site-variables).

输出单个分类下的所有文章：

`site.categories.CATEGORY` 需要手动指定，并不是很方便。

[使用Category分类](https://segmentfault.com/a/1190000000406017#articleHeader2)这篇文章里使用js来实现。

我这里是先生成好`categories.html`，里面列出了所有的文章。然后利用锚点`categories.html#xxx`进行跳转到指定的类别。

### Tags

参考[使用文章标签索引文章](https://segmentfault.com/a/1190000000406017#articleHeader5)。

首先在每个文章的YAML头信息中定义tags，如：

```
---
layout: post
title:  "your title"
tags: [github, jekyll]
---
```

和[category](#category)的设置方法差不多，就不赘述了。

### Latex

参考：[https://jekyllrb.com/docs/extras/#math-support](https://jekyllrb.com/docs/extras/#math-support)

我这里为了避免所有pages都引入mathjax相关js代码，采用了不同的设置：

在配置文件`_config.yml`中添加：

```
mathjax: false
```

在`head.html`中添加：

```html
{% raw %}{% if page.mathjax %}
  <script src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.0/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
{% endif %}{% endraw %}
```
然后在需要引入mathjax的md文件头中加入`mathjax: true`，如：

```
---
layout: post
...
mathjax: true
---
```

####  Support $

默认只支持双`$`符，修改：

```js
<script type="text/x-mathjax-config">
  MathJax.Hub.Config({
    TeX: {
      equationNumbers: {
        autoNumber: "AMS"
      }
    },
    tex2jax: {
      inlineMath: [ ['$','$'] ],
      displayMath: [ ['$$','$$'] ],
      processEscapes: true,
    }
  });
</script>
```

#### Mathjax Rendering Problem

若latex公式中包含`|`符，如`|a|`，会被markdown解析器当成`<table></table>`来处理。

又如，定义矩阵时，其中的换行`\\`也会被解析出问题，把第一个`\`当成了转义符。

解决方式：

对第一种情况，根据 [Syntax \| Kramdown](https://kramdown.gettalong.org/syntax.html#math-blocks) 上的说明，将`|`用`\vert`来代替。

对第二种情况，用双`$$`符解决转义问题。

### Code Highlighting

Jekyll 3.x 自带了rouge代码高亮的功能，但是不造为啥一直not working（难道又被看脸了。。

于是这里用的是[highlightjs](https://highlightjs.org/)。在`head.html`里添加：

```html
<link rel="stylesheet" href="/lib/highlight/styles/hybrid.css">
<script src="/lib/highlight/highlight.pack.js"></script>
<script>hljs.initHighlightingOnLoad();</script>
```

我这里把下载的`highlightjs`的相关文件放在了`lib`目录下，样式选的是`hybrid.css`。（monokai-sublime 和 hybrid 配色也不错。）

`highlightjs`会解析并高亮`<pre><code>...</code></pre>`中的内容。如

```html
<pre><code class="html">...</code></pre>
```

若不需要高亮则设为：

```html
<pre><code class="nohighlight">...</code></pre>
```

这里有个麻烦的地方，若md里没有指明语言，`highlightjs`也会自动检测是什么语言。如果不想对某个code block进行高亮，有两种解决方式：（当然了，我比较折腾，一般人也不需要这个。。（捂脸）

（1）每次手动加入`nohighlight`。

```
​```nohighlight
xxx
​```
```

（2）修改markdown解析器。

第一种方式比较麻烦，但`highlightjs`又没有提供关闭`highlightAuto`的接口。

### Custom Markdown Parser

在`_config.yml`中可以看到，默认使用的是`kramdown`解析器。

```
markdown: kramdown
```

关于[Custom Markdown Processors](https://jekyllrb.com/docs/configuration/#custom-markdown-processors)官网有一些说明，好吧，没看懂，毕竟不会Ruby。kramdown也不能以overwrited的方式改写相应的convert方法，然后又写了段js代码试图修改kramdown解析出的html，但还是没成功。真是艰辛，网上找了好久都无果，最后还不如直接看代码改来得快。。。

好在`highlightjs`提供了这个：

```js
// ignore languages
<script>hljs.configure({ ignore: ['text'] });</script>
```

然后看Jekyll源码，找到了 `jekyll/lib/jekyll/converters/markdown/kramdown_parser.rb` (see [code](https://github.com/jekyll/jekyll/blob/57fd5f887da1189a16bdfbb982d75f725c38d725/lib/jekyll/converters/markdown/kramdown_parser.rb)) 

```ruby
def convert(content)
  Kramdown::Document.new(content, @config).to_html
end
```

接着看kramdown源码，找到了`kramdown/lib/kramdown/converter/html.rb` (see [code](https://github.com/gettalong/kramdown/blob/f4cdae257159ce0addf542739fed262e4a2cf401/lib/kramdown/converter/html.rb)) 

在我的本地电脑中，相关文件位于`/usr/local/lib/ruby/gems/2.4.0/gems/kramdown-1.14.0/lib/kramdown/converter/html.rb`。

找到`def convert_codeblock(el, indent)`函数进行修改：

在代码`code_attr['class'] = "language-#{lang}" if lang` 前添加：

```ruby
if lang.nil?
  lang = "text"
end
```

并将代码`if highlighted_code` 改为 `if highlighted_code && lang`。

（其中，`nil`是判空的意思。）

这样在不指明语言的时候，不高亮显示。

---

上面是mac环境下，可以直接改代码。但是在linux环境下，只有一个二进制执行文件。

于是，先删除原有的：

```
$ sudo gem uninstall kramdown
```

然后在官网[下载源码](https://github.com/gettalong/kramdown/releases)重新安装：

```
$ ruby setup.rb config
$ ruby setup.rb setup
$ ruby setup.rb install
```

好吧，暂时没弄好。。。

## Else

### Permalinks

在`_config.yml`中设置，如：

```
permalink: /:categories/:year/:month/:day/:title.html
```
### footer

当页面内容长度不够时，footer显示位置可能有问题。

让footer内容永远处于最末端：

```html
$height-footer: 40px;

html {
    height: 100%;
}

body {
    min-height: 100%;
    position: relative;
    padding-bottom: $height-footer;
}

.footer {
    bottom: 0;
    width: 100%;
    position: absolute;
    height: $height-footer;
}
```

子元素的百分比高度也可以基于父元素的百分比高度，前提是父元素的父元素必须有一个明确的高度。要使`min-height`的百分比值生效，其父元素的`height`值必须为一个固定的高度或者是一个有效的百分比高度。

## Problems

{% raw %}`{{ xxx }}`{% endraw %} 或者 {% raw %}`{% xxx %}`{% endraw %} 显示不出来。

解决方式：

（1）

```
\{\{ xxx \}\}
```

（2）参考：[Escaping double curly braces inside a markdown code block in Jekyll](https://stackoverflow.com/questions/24102498/escaping-double-curly-braces-inside-a-markdown-code-block-in-jekyll)

```
{% assign openTag = '{%' %}{{ openTag }} raw %}    
This is a test: {% raw %}{{ xxx }}{% endraw %}
{{ openTag }} endraw %}
```
（3）那么问题又来了，如何显示上面的 raw 和 endraw 呢？参考：[jekyll 如何转义字符](http://www.cnblogs.com/OceanHeaven/p/6959669.html)

```
server {
    listen [::]:80;
    server_name comments.lszero.com;
    
    location /isso {
        proxy_pass http://localhost:8090;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Script-Name /isso;
    }
}
```


