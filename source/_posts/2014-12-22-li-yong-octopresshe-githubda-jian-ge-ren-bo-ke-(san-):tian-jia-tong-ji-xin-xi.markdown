---
layout: post
title: "利用Octopress和Github搭建个人博客（三）：添加统计信息"
date: 2014-12-22 16:46:14 +0800
comments: true
categories: Octopress
keywords: Octopress github 搭建博客 添加统计 SEO 录入搜索引擎
description: 为Octopress博客录入搜索引擎，并添加流量统计信息，进行SEO。
---
{% img center /images/1-4.png %}
<h2>引言</h2>
搭建好基本环境（如果没有请戳<a href="http://glgjing.github.io/blog/2014/12/06/li-yong-octopresshe-githubda-jian-ge-ren-bo-ke-(%5B%3F%5D-):ji-chu-huan-jing-da-jian/">这里</a>），折腾完主题（如果想折腾请戳<a href="http://glgjing.github.io/blog/2014/12/12/li-yong-octopresshe-githubda-jian-ge-ren-bo-ke-(er-):zi-ding-yi-zhu-ti/">这里</a>），就可以安安静静的写博客了。但博客毕竟不是日记，还是需要和其他人分享交流的，尤其是技术博客。那么首先你需要别人能够通过搜索引擎找到你的博客，其次你需要别人能够更方便准确的找到你的博客，这就需要为你的博客录入搜索引擎，添加统计信息，以便进行SEO。转载请注明出处：<code>http://glgjing.github.io/</code>。<!-- more -->
<h2>目录</h2>
<ul>
	<li>一 录入搜索引擎</li>
	<li>二 添加统计信息</li>
	<li>三 总结</li>
</ul>
<h2>一 录入搜索引擎</h2>
在自建博客网站上写的博客，一开始并不会在搜索引擎中搜得到，不像博客平台一样，因为你还没有向搜索引擎提交你的网站URL，让它去抓取你博客网站的内容。所以赶紧提交吧，提交地址：<code>http://www.gongju.com/addurl/</code>，也可以自行google：搜索引擎提交入口。提交完成之后，就可以在相应的搜索引擎中找到你的博客了。现在你还需要为你的网站和博文添加描述信息以及关键字，让其他人可以在搜索引擎中更方便准确的找到你的博客。为每篇博文添加关键字和描述的方法是，打开<code>source/_posts/</code>目录下对应的博文文件，在文件头部添加类似如下内容：
{% codeblock %}
---
layout: post
title: "利用Octopress和Github搭建个人博客（三）：添加统计信息"
date: 2014-12-22 16:46:14 +0800
comments: true
categories: Octopress
keywords: Octopress github 搭建博客 添加统计 SEO 录入搜索引擎
description: 为Octopress博客录入搜索引擎，并添加流量统计信息，进行SEO。
---
{% endcodeblock %}
如果没有为博文添加描述信息，Octopress会自动以文章的前150字作为博文的描述信息。为网站首页添加关键字和描述的方法是，打开<code>source/index.html</code>文件，在文件头部添加类似如下内容：
{% codeblock %}
---
layout: default
tags: [windows c++ octopress Github DirectUI GDI]
keywords: windows c++ octopress Github
description: 博客网站的描述信息在这里
---
{% endcodeblock %}
通过以上操作，搜索引擎就可以根据用户输入的关键字对你的博文进行更准确的索引。如果想让自己的博客出现在搜索结果中靠前的位置，就需要网站流量统计工具进行进一步的SEO了。
<h2>二 添加统计信息</h2>
常用的网站统计工具有：Google Analytics，Yahoo网站流量统计，百度统计，CNZZ，51.la等等。Octopress自带Google Analytics工具，所以添加Google统计比较方便：
<ol>
	<li>注册<a href="http://www.google.com/analytics/">Google Analytics</a>账号</li>
	<li>获得一个google_analytics_tracking_id，添加到_config.yml中google_analytics_tracking_id:后面</li>
	<li>按照Google Analytics提示，对博客网站进行验证</li>
</ol>
完成以上步骤，就可以利用Google Analytics进行流量统计了，你还可以注册<a href="https://www.google.com/webmasters/tools/home?hl=zh-CN">Google站长工具</a>，进行更详细的分析和SEO。
除了自带的Google Analytics你还可以添加其他统计工具，具体步骤都比较类似，这里以CNZZ为例：
<ol>
	<li>注册<a href="http://zhanzhang.cnzz.com/">CNZZ账号</a></li>
	<li>按照提示获取统计代码</li>
	<li>将统计代码添加到需要统计的页面，一般可以添加到<code>source/_includes/custom/footer.html</code>中</li>
</ol>
添加到<code>source/_includes/custom/footer.html</code>中的统计代码类似如下：
{% codeblock lang:html %}
<p>
  Copyright &copy; {{ site.time | date: "%Y" }} - {{ site.author }}
  <script type="text/javascript">var cnzz_protocol = (("https:" == document.location.protocol) ? " https://" : " http://");document.write(unescape("%3Cspan id='cnzz_stat_icon_1253899728'%3E%3C/span%3E%3Cscript src='" + cnzz_protocol + "s4.cnzz.com/z_stat.php%3Fid%3D1253899728' type='text/javascript'%3E%3C/script%3E"));</script>
</p>
{% endcodeblock %}
<h2>三 总结</h2>
这里主要介绍了如何将博客网站添加到搜索引擎，并添加网站流量统计工具，各种流量统计工具的优劣可以自行google，后面需要做的就是SEO了，而SEO是一件长期持续的工作。