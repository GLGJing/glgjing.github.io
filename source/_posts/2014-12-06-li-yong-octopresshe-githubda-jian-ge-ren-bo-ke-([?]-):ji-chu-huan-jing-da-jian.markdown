---
layout: post
title: "利用Octopress和Github搭建个人博客（一）：基础环境搭建"
date: 2014-12-06 15:30:04 +0800
comments: true
categories: Octopress
keywords: Octopress github 博客 搭建博客
description: 利用Octopress和Github Pages搭建个人博客。
---
{% img center /images/1-1.png %}
<h2>引言</h2>
如果你只想安安静静的写博客而不被各种博客网站的条条框框限制，如果你想自建博客网站而又不想买域名租服务器等繁琐流程，如果你恰好懂些shell command和git，那么Octopress + Gitbub搭建个人博客是个不错的选择。<a href="http://octopress.org/">Octopress</a>是一个基于<a href="https://github.com/jekyll/jekyll"><code>Jekyll</code></a>博客引擎开发的博客框架，可以很方便的生成静态页面用于在<a href="https://pages.github.com/">Github Pages</a>上展现。官方说法：<code>A blogging framework for hackers</code>。
本文主要介绍在mac下如何利用Octopress和Github搭建个人博客的基础环境。转载请注明出处：<code>http://glgjing.github.io/</code>。<!-- more -->
<h2>目录</h2>
<ul>
  <li>一 安装基本环境</li>
  <li>二 通过Octopress将博客部署到Github</li>
  <li>三 发布个人博客</li>
  <li>四 其他配置</li>
</ul>
<h2>一 安装基本环境</h2>
<h3>1 安装Ruby</h3>
Octopress运行需要Ruby环境，所以首先需要安装Ruby（<a href=“https://ruby-china.org/wiki/install_ruby_guide”>详细教程</a>）。
首先安装RVM，RVM是用来安装和管理Ruby环境的。
{% codeblock %}
curl -L https://get.rvm.io | bash -s stable
{% endcodeblock %}
然后，载入 RVM 环境，如果新开 Terminal 就不用这么做了，会自动重新载入的。
{% codeblock %}
source ~/.rvm/scripts/rvm
{% endcodeblock %}
检查一下是否安装正确
{% codeblock %}
rvm -v
rvm 1.22.17 (stable) by Wayne E. Seguin <wayneeseguin@gmail.com>, Michal Papis <mpapis@gmail.com> [https://rvm.io/]
{% endcodeblock %}
通过RVM安装Ruby，后面参数为版本号，可以根据版本进行调整。
{% codeblock %}
rvm install 2.0.0
{% endcodeblock %}
完成以后，Ruby就安装好了。RVM 装好以后，需要执行下面的命令将指定版本的 Ruby 设置为系统默认版本。
{% codeblock %}
rvm 2.0.0 --default
{% endcodeblock %}
测试是否设置正确
{% codeblock %}
ruby -v
ruby 2.0.0p247 (2013-06-27 revision 41674) [x86_64-darwin13.0.0]
{% endcodeblock %}

<h3>2 安装Octopress</h3>
安装Octopress，需要git。所以先确保你的电脑里已经安装了git，在终端输入<code>git —version</code>，如果看到版本号类似<code>git version x.x.x.x...</code>，证明git环境已经OK，否则请先<a href="http://git-scm.com/">安装git</a>。
git安装完成之后，就可以把Octopress从Github上clone到本地。切换到你想存储Octopress的目录运行如下命令：
{% codeblock %}
git clone git://github.com/imathis/octopress.git octopress
{% endcodeblock %}
接下来安装依赖项：
{% codeblock %}
cd octopress
gem install bundler
rbenv rehash # If you use rbenv, rehash to be able to run the bundle command
bundle install
{% endcodeblock %}
然后安装默认主题：
{% codeblock %}
rake install
{% endcodeblock %}

<h2>二 通过Octopress将博客部署到Github</h2>
首先需要在Github上建立一个仓库，Github号称是程序员的Facebook，如果还没有Github账号那就赶紧注册一个吧，仓库名称格式为：<code>username.github.io</code>。该名称就是博客以后的访问地址：<code>http://username.github.io</code>。仓库创建完成后，需要运行下面命令来将该仓库和Octopress关联起来：
{% codeblock %}
rake setup_github_pages
{% endcodeblock %}
运行该命令期间会要求你输入仓库url，按照提示格式进行输入即可。该命令的具体功能详细参考<a href="http://octopress.org/docs/deploying/github/"><code>Deploying to Github Pages</code></a>。主要就是设置Github仓库的URL，在本地创建_deploy，该目录存储的就是后续要部署到Github Pages的文件。详细内容会在后面的Octopress目录结构介绍博文中说明。
接下来就可以将博客相关文件部署到Github Pages上了，运行如下命令：
{% codeblock %}
rake generate
rake deploy
{% endcodeblock %}
上述命令主要是根据<code>source</code>目录下的文件，生成博客文件到<code>public</code>目录下，然后将<code>public</code>目录下的文件拷贝到<code>deploy</code>目录，并将<code>deploy</code>目录下的文件commit和push到Github上的<code>username.github.io</code>仓库里的<code>master</code>分支，执行完上述操作就可以访问<code>http://username.github.io</code>了。有时会需要过几分钟才能打开，是正常现象。
到此为止我们就完成了，博客的基本部署，但一般习惯性的我们会把生成博客文件的原始文件提交到另外一个<code>source</code>分之上，执行如下命令：
{% codeblock %}
git add .
git commit -m 'Initial source commit'
git push origin source
{% endcodeblock %}

<h2>三 发布个人博客</h2>
想要新建一个博文，运行如下命令：
{% codeblock %}
rake new_post[“title"]
{% endcodeblock %}
其中title为博客名，该命令会在<code>source/_posts/</code>目录下创建一个名称类似<code>2014-12-5-title.markdown</code>的文件，当然你也可以不用上述命令，手动在该目录下添加该文件，命名需要遵守<code>year-month-day-title.markdown</code>的规则。这个文件就是后续我们写博客的地方，通过<code>rake new_post</code>命令生成的文件会默认带如下内容：
{% codeblock %}
---
layout: post
title: “title"
date: 2013-08-03 16:36
comments: true
categories:
---
{% endcodeblock %}
到目前为止我们就可以在这里写自己的博客了。博客完成之后，通过如下步骤部署到Github上：
{% codeblock %}
rake generate
rake deploy
{% endcodeblock %}
在部署到Github之前可以运行如下命令，在浏览器中输入<code>http://localhost:4000/</code> 进行本地预览：
{% codeblock %}
rake preview
{% endcodeblock %}
总结一下发布博文的完成流程：
{% codeblock %}
rake new_post[’title’]     # 新建博文文件
rake generate              # 将编辑好的博文生成网页
rake preview               # 提交前可以进行本地预览
rake deploy                # 将博文部署到Github上
git commit -a              # 提交本地更改的文件
git push origin source     # 将源文件push到Github的source分支
{% endcodeblock %}
<h2>四 其他配置</h2>
更多的<code>Octopress</code>配置如：侧边栏定制，添加评论插件，字体高亮等等，会在后续博文中<code>持续更新</code>。

