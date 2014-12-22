---
layout: post
title: "利用Octopress和Github搭建个人博客（二）：自定义主题"
date: 2014-12-12 16:19:09 +0800
comments: true
categories: Octopress
keywords: Octopress github 搭建博客 主题
description: 自定义Octopress的主题。
---
{% img center /images/1-2.png %}
<h2>引言</h2>
在为Octopress博客自定义主题之前，你应该已经搭建好了基础环境。如果还没开始搭建可以参考<a href="http://glgjing.github.io/blog/2014/12/06/li-yong-octopresshe-githubda-jian-ge-ren-bo-ke-(%5B%3F%5D-):ji-chu-huan-jing-da-jian/">利用Octopress和Github搭建个人博客（一）：基础环境搭建</a>。在搭建好基础环境之后，如果单纯的写写博文，默认主题也是可以满足需求的。但既然选择了用Octopress和Github Pages来管理自己的博客，不折腾折腾都对不起自己。不要求效果有多炫，但要方便浏览查找。转载请注明出处：<code>http://glgjing.github.io/</code>。<!-- more -->
<h2>目录</h2>
<ul>
	<li>一 Octopress目录结构介绍</li>
	<li>二 安装第三方主题</li>
	<li>三 主题样式修改</li>
	<li>四 总结</li>
</ul>
<h2>一 Octopress目录结构介绍</h2>
在更改主题之前有必要了解一下Octopress目录下的文件结构，后续所有的修改都在这个目录下进行的。Octopress的目录文件如下：
{% img center /images/1-3.png %}
初始的Octopress目录下是没有source、sass、public、_deploy这几个文件夹的，运行<code>rake install['theme_name']</code>命令后会生成这几个文件夹。
<ul>
	<li>_config.yml: 站点的配置文件，也是后续需要修改最多的文件。</li>
	<li>source: 该目录是执行<code>rake install['theme_name']</code>后从.themes\theme_name目录中的source拷贝而来(theme_name是你选择的主题名)，并且添加了一个_posts目录了，当执行了rake new_post[''title'']后会在_post生成博文的markdown文件。包括后面的很多设置页是在该目录中进行；</li>
	<li>sass：也是在执行<code>rake install['theme_name']</code>后从.themes\theme_name目录中的sass拷贝而来。关于sass可以参考<a href="http://sass-lang.com/guide">这里</a></li>
	<li>public: 当执行了<code>rake generate</code>命令后会编译source目录下的内容然后将编译后的内容复制到public目录中。</li>
  <li>_deploy: 在执行<code>rake deploy</code>命令后，会将public的内容拷贝到_deploy目录下然后提交到Github的master分支上，我们最终看到的网站内容就是master分支下的内容。所以public和_deploy中的内容都是自动生成的，不要手动修改，否则在运行<code>rake generate</code>和<code>rake deploy</code>命令后所有的更改都会被覆盖掉。</li>
</ul>
<h2>二 安装第三方主题</h2>
在做任何主题相关的修改之前，最好选定一款自己喜欢的主题，否则后续再更换主题，会覆盖之前的修改。Github上有很多第三方的主题，可以到<a href="https://github.com/imathis/octopress/wiki/3rd-Party-Octopress-Themes">这里</a>下载。运行类似如下命令进行主题安装：
{% codeblock %}
cd octopress
git clone git://github.com/macjasp/cleanpress.git .themes/cleanpress
rake install['cleanpress']
rake generate
{% endcodeblock %}
<h2>三 主题样式修改</h2>
<h3>1 基本配置修改</h3>
在选定好主题之后就可以进行自定义修改了。首先更改基本配置，Octopress的基本配置存在_config.yml文件里，具体内容如下：
{% codeblock %}
url: http://glgjing.github.io  # 网站的url
title: GLGJing's Blog          # 网站的标题
author: GLGJing                # 网站作者，会显示在底部等位置
simple_search: https://www.google.com/search #搜索引擎
description:                   # 网站描述
{% endcodeblock %}
此处列出了主要的博客配置信息，有些配置项大概看名字就知道功能了，例如网站title，email等；有些配置项比较复杂，后面单独介绍，如添加评论插件、侧边栏等。
<h3>2 设置标题栏: Header</h3>
Octopress的很多自定义配置是存储在/source/_includes/custom/目录下的，如果想要更改标题栏，可以修改该文件夹下的header.html文件。
{% codeblock lang:html %}
<hgroup>
  <h1><a href="{{ root_url }}/">{{ site.title }}</a></h1>
  {% raw %}{% if site.subtitle %}
    <h2>{{ site.subtitle }}</h2>
  {% endif %}{% endraw %}
</hgroup>
{% endcodeblock %}
其中的title和subtitle可以直接在_config.yml中修改。
<h3>3 设置导航栏: Navigation</h3>
Navigation的配置方法和Header类似，直接修改/source/_includes/custom/navigation.html文件。
{% codeblock lang:html %}
<ul class="main-navigation">
  <li><a href="{{ root_url }}/">主页</a></li>
  <li><a href="{{ root_url }}/blog/archives">文章</a></li>
  <li><a href="{{ root_url }}/about">关于</a></li>
</ul>
{% endcodeblock %}
如果想添加新的页面，可以运行如下命令：
{% codeblock %}
rake new_page['page_name']
{% endcodeblock %}
该命令会建立source/page_name/index.html文件，然后编辑此文件，添加自己想要展示的内容即可，再在navigation.html里添加正确的路径即可,如:
{% codeblock lang:html %}
<li><a href="/page_name">新标签页</a></li>
{% endcodeblock %}
<h3>4 设置尾栏: Footer</h3>
修改/source/_includes/custom/footer.html文件来设置尾栏。
{% codeblock lang:html %}
<p>
  Copyright &copy; {{ site.time | date: "%Y" }} - {{ site.author }} -
  <span class="credit">
    Powered by<a href="http://octopress.org">Octopress</a>
  </span>
</p>
{% endcodeblock %}
可以将不需要的信息去掉，添加自己想要的信息如：流量统计工具，个人声明等。
<h3>5 设置背景图和LOGO</h3>
想要更改背景图片，可以在sass/custom/_styles.scss中添加如下内容：
{% codeblock lang:html %}
html {
  background: #555555 url("/images/bg3.jpg");
}
body > div {
  background-image: none;
}
body > div > div {
  background-image: none;
}
{% endcodeblock %}
更改LOGO图片可以直接替换/source目录下的favicon.png, 或者将LOGO图片放入source/images中，然后修改source/_includes/head.html，找到favicon.png，修改其路径指向你的图片即可。
<h2>四 总结</h2>
这里介绍了Octopress主题配置的几个主要部分，其他更细节配置如：字体的配置优化、侧边栏的定制、评论插件的配置等，会在后续的博文中更新。

