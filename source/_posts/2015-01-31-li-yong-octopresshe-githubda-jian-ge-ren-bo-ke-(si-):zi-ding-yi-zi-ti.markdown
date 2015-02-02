---
layout: post
title: "利用Octopress和Github搭建个人博客（四）：自定义字体"
date: 2015-01-31 17:00:48 +0800
comments: true
categories: Octopress
keywords: Octopress github 博客 搭建博客 自定义字体 字体配置
description: 本文介绍了如何配置Octopress的字体。 
---
<h2>修改博客字体</h2>
Octopress字体的设置文件是：sass/custom/_fonts.scss，内容类似如下：
{% codeblock %}
$sans: "Open Sans","Helvetica Neue", Arial, sans-serif !default;
$serif: "PT Serif", Georgia, Times, "Times New Roman", serif !default;
$mono: Menlo, Monaco, "Andale Mono", "lucida console", "Courier New", monospace !default;
$heading-font-family: "Fjalla One", "Georgia", "Helvetica Neue", Arial, sans-serif !default;
$header-title-font-family: $heading-font-family !default;
$header-subtitle-font-family: $serif !default;
{% endcodeblock %}
<ul>
	<li>$sans：定义的是无衬线正文的字体</li>
	<li>$serif：定义的是衬线正文的字体</li>
	<li>$mono：定义的是代码的字体</li>
	<li>$heading-font-family：定义的是文章标题的字体</li>
	<li>$header-title-font-family：定义的是博客标题的字体</li>
	<li>$header-subtitle-font-family：定义的是博客子标题的字体</li>
</ul>
如果需要更改字体，在这里更改对应的值即可。需要注意的是在跨平台的情况下，一些字体在很多机上不一定会有，所以尽量定义一些常用字体作为备选方案。例如Adobe的开源字体Source Code Pro作为代码字体，个人感觉很不错，但很多机器上不一定有装，所以这里最好补上其他字体作为备选，Windows上比较适合的编程字体Consolas，Mac上比较适合的编程字体Monaco，以及Ubuntu上比较适合的编程字体Ubuntu Mono。浏览器在渲染字体的时候会依次进行寻找，如果本机装有Source Code Pro就用该字体进行渲染，如果没有则寻找下一个也就是Consolas，依此类推直到找到一个可用的字体进行渲染，上诉三款字体都是三个平台默认安装的，这样可以最大程度地保证代码阅读质量。转载请注明出处：<code>http://glgjing.github.io/</code>。

