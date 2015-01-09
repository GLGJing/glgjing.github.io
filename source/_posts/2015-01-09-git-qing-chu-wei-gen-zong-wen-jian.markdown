---
layout: post
title: "Git 清除未跟踪文件"
date: 2015-01-09 16:34:45 +0800
comments: true
categories: git
keywords: git 未跟踪 unstaged 删除 清理 clean
description: 本文介绍了利用git clean命令清理未跟踪文件。
---
在利用<code>git</code>工作时，工程目录下经常会出现一些未跟踪文件，虽然<code>git</code>支持通过<code>.gitingore</code>文件添加一些忽略文件类型和文件目录。但有时需要清理一些临时文件和自动生成的文件，手动删除显得太麻烦，这时你可以利用<code>git clean</code>命令来帮你完成这项操作。转载请注明出处：<code>http://glgjing.github.io/</code>。<code>git clean</code>命令支持以下参数：
{% codeblock %}
git clean [-d] [-f] [-i] [-n] [-q] [-e <pattern>] [-x | -X] [--] <path>...
{% endcodeblock %}
其中几个主要参数用法如下：
{% codeblock %}
-d   # 删除未跟踪目录以及目录下的文件，如果目录下包含其他git仓库文件，并不会删除（-dff可以删除）。
-f   # 如果 git cofig 下的 clean.requireForce 为true，那么clean操作需要-f(--force)来强制执行。
-i   # 进入交互模式
-n   # 查看将要被删除的文件，并不实际删除文件
{% endcodeblock %}
通过以上几根参数组合，基本上可以满足删除未跟踪文件的需求了。例如在删除前先查看有哪些文件将被删除运行：
{% codeblock %}
git clean -n
{% endcodeblock %}
想删除当前工作目录下的未跟踪文件，但不删除文件夹运行（如果<code>clean.requireForce</code>为<code>false</code>可以不加<code>-f</code>选项）：
{% codeblock %}
git clean -f
{% endcodeblock %}
想删除当前工作目录下的未跟踪文件以及文件夹运行：
{% codeblock %}
git clean -df
{% endcodeblock %}