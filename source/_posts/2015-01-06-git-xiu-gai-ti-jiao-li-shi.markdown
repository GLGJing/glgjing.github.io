---
layout: post
title: "git 修改提交历史"
date: 2015-01-06 11:14:40 +0800
comments: true
categories: git
keywords: git commit 历史 修改 合并 拆分 重排 删除 提交
description: 本文介绍了如何修改git的提交历史，包括修改最近一次提交，合并、重排、拆分多次提交等情况。
---
<h2>引言</h2>
在使用git进行工作时，经常会碰到想要修改本地提交记录（还没有push到远程仓库）的情况。例如这样一个场景：你接到一个任务，这个任务可能需要一个星期的开发时间，于是你从<code>master</code>分之上运行<code>git checkout -b new_branch</code>命令，<code>checkout</code>到<code>new_branch</code>分之上进行该任务的开发，在开发的过程中向<code>new_branch</code>分之提交了多个<code>commit</code>，然后发现有些代码不是很合理，于是又更改代码提交新的<code>commit</code>，反反复复终于完成了该任务的开发，是时候提交review并合并到<code>master</code>分之上了，但是你发现提交记录非常混乱，并不利于同事review，而且你的同事也不关心你的具体修改过程，他们只想看到最终的实现，此时问题来了，如果你把所有的<code>commit</code>合并成一个大<code>commit</code>，虽然你的同事可以直接看到最终代码，但你的各个模块都混在一起提交，很难按模块单独进行review并发现问题；如果你直接提交review，你的同事就不得不把你的所有提交历史都review一遍，包括一些实际上已经在后面提交中被你修改掉的代码，脾气好的估计内心一万只草泥马奔过，脾气不好的保证不打死你！当然<code>git</code>是不可能考虑不到这种情况的，扯了这么多，下面就来看一下如何修改<code>git</code>的提交历史。转载请注明出处：<code>http://glgjing.github.io/</code>。<!-- more -->
<h2>一 修改最近一次提交记录</h2>
<h3>1 只修改提交说明</h3>
修改最近一次提交是非常常见的需求，如果只想更改最近一次的提交说明是非常方便的，只需输入如下命令：
{% codeblock %}
git commit —amend
{% endcodeblock %}
然后你就会进入文本编辑器，输入你想要的内容，保存并退出即可。
<h3>2 添加新的更改</h3>
如果你完成提交后又想修改被提交的快照，增加或者修改其中的文件，过程基本和上面一样。先运<code>git add</code>命令，将修改的文件添加到缓存区,然后运行<code>git commit —amend</code>命令，该命令会获取你当前的暂存区的内容一并提交到最后一次<code>commit</code>。例如：新加了一个文件new_file.cpp，想要合并到最后一次提交，过程如下：
{% codeblock %}
git add new_file.cpp
git commit —amend
{% endcodeblock %}
也可以直接运行下面的命令，不过要小心，不要提交了多余的文件。
{% codeblock %}
git commit -a —amend
{% endcodeblock %}
<h3>3 将文件从本次提交中移除</h3>
如果想把已经<code>commit</code>的文件从这次<code>commit</code>移除的话，可以运行如下命令：
{% codeblock %}
git reset [—soft] HEAD~1 # —soft可加可不加，默认就是soft选项
git checkout —filename # 要从本次提交移除的文件名
git commit -m “new commit"
{% endcodeblock %}
<h2>二 修改多个提交</h2>
要修改历史中更早的提交，你必须采用更复杂的工具。<code>git</code>没有一个修改历史的工具，但是你可以使用<code>rebase</code>工具来衍合一系列的提交到它们原来所在的<code>HEAD</code>上。依靠这个交互式的<code>rebase</code>工具，你就可以停留在每一次提交后，如果你想修改或改变说明、增加文件或任何其他事情。你可以通过给<code>git rebase -i</code>命令以交互方式进行<code>rebase</code>。例如，你想修改最近三次的提交说明，或者其中任意一次，你必须给<code>git rebase -i</code>提供一个参数，指明你想要修改的提交的父提交。例如<code>HEAD~3</code>是指从<code>HEAD</code>指针到<code>HEAD+3</code>的位置，也就是最近第4次提交。所以想修改最近3次提交，你需要指明第3次提交的父提交（第4次提交）即<code>HEAD~3</code>。运行命令
{% codeblock %}
git rebase -i HEAD~3
{% endcodeblock %}
再次提醒这是一个衍合命令，也就是<code>HEAD~3</code>到<code>HEAD</code>范围内的每一次提交都会被重写，不管你是否修改提交说明<code>SHA-1</code>的值都会发生变化。<font color="red">所以千万不要涵盖你已经推送到中心服务器的提交。</font>这么做会使其他开发者产生混乱，因为你提供了同样变更的不同版本。运行该命令后进入交互界面，类似：
{% codeblock %}
pick fecb551 Init the view model
pick bb199a0 Update the version
pick bc5cd9d Add new method

# Rebase f77f585..fecb551 onto f77f585
#
# Commands:
#  p, pick = use commit
#  r, reword = use commit, but edit the commit message
#  e, edit = use commit, but stop for amending
#  s, squash = use commit, but meld into previous commit
#  f, fixup = like "squash", but discard this commit's log message
#  x, exec = run command (the rest of the line) using shell
#
# These lines can be re-ordered; they are executed from top to bottom.
# If you remove a line here THAT COMMIT WILL BE LOST.
# However, if you remove everything, the rebase will be aborted.
# Note that empty commits are commented out
{% endcodeblock %}
根据命令提示，就可以进行历史更改了。很重要的一点是你得注意这些提交的顺序与你通常通过<code>log</code>命令看到的是相反的。如果你运行<code>log</code>，你会看到下面这样的结果：
{% codeblock %}
pick bc5cd9d Add new method
pick bb199a0 Update the version
pick fecb551 Init the view model
{% endcodeblock %}
<h3>1 修改指定提交</h3>
例如：只修改最近第3次提交说明可以进行如下更改：
{% codeblock %}
reword fecb551 Init the view model
pick bb199a0 Update the version
pick bc5cd9d Add new method
...
{% endcodeblock %}
保存并退出编辑器，<code>rebase</code>命令在衍合到第3次提交时会进入提交说明编辑页面，在此进行编辑新的提交说明，保存并退出即可，<code>rebase</code>命令继续进行直至完成全部衍合操作。如果你不仅想要修改提交说明，还要更改提交，可以进行如下更改：
{% codeblock %}
edit fecb551 Init the view model
pick bb199a0 Update the version
pick bc5cd9d Add new method
...
{% endcodeblock %}
保存并退出编辑器，<code>rebase</code>命令在衍合到第三次提交时会等待你提交新的更改，并提示你修改完成后运行<code>git commit --amend</code>命令，然后运行<code>git rebase --continue</code>继续进行<code>rebase</code>直至完成全部衍合。
<h3>2 重排提交</h3>
你也可以使用<code>git rebase -i</code>命令对提交历史彻底重排或删除提交。例如你想删除"Update the version"这个提交，并且修改其他两次提交的顺序，可以将
{% codeblock %}
pick fecb551 Init the view model
pick bb199a0 Update the version
pick bc5cd9d Add new method
...
{% endcodeblock %}
改为：
{% codeblock %}
pick bc5cd9d Add new method
pick fecb551 Init the view model
{% endcodeblock %}
然后保存并退出编辑器，此时<code>rebase</code>命令会先应用bc5cd9d (Add new method)，然后应用 fecb551 (Init the view model)bb199a0 Update the version这次提交。
然后保存并退出编辑器，此时<code>rebase</code>命令会先应用bc5cd9d (Add new method)，然后应用 fecb551 (Init the view model)，接着停止。执行完上诉操作，你已经修改了这些提交的顺序，并且删除了bb199a0 (Update the version)这次提交。
<h3>3 合并提交</h3>
<code>git rebase -i</code>命令还可以将一系列提交合并成一个提交。从上面的脚本提示中可以看到<code>s, squash = use commit, but meld into previous commit</code>提示。如果用<code>squash</code>修饰提交就可以进行提交之间的合并，例如可以将脚本修改成这样：
{% codeblock %}
pick fecb551 Init the view model
squash bb199a0 Update the version
squash bc5cd9d Add new method
{% endcodeblock %}
保存并退出编辑器，<code>rebase</code>命令会应用全部三次变更然后进入编辑器来归并三次提交说明。当你保存之后，你就拥有了一个包含前三次提交的全部变更的单一提交。
<h3>4 拆分提交</h3>
拆分提交实际上就是撤销一次提交，然后分多次进行重新提交。例如你想将三次提交中的中间一次拆分。将"Update the version"拆分成两次提交："Update the version1"和"Update the version2"，可以进行如下修改：
{% codeblock %}
pick fecb551 Init the view model
edit bb199a0 Update the version
pick bc5cd9d Add new method
{% endcodeblock %}
当<code>rebase</code>到bb199a0时，会进入等待你提交新<code>commit</code>的状态，这时看可以运行<code>git reset HEAD^</code>对当前提交进行重置，然后分别运行<code>git add</code>命令添加想要提交的文件，分别进行<code>git commit</code>，最后运行<code>git rebase --continue</code>完成所有衍合。整体过程如下：
{% codeblock %}
git reset HEAD^
git add file1
git commit -m 'Update the version1'
git add file2
git commit -m 'Update the version2'
git rebase --continue
{% endcodeblock %}
执行完上诉操作，提交历史看起来就像这样了：
{% codeblock %}
1c002dd Add new method
9b29157 Update the version2
35cfb2b Update the version1
f3cc40e Init the view model
{% endcodeblock %}
<font color="red">再次提醒，这会修改你列表中的提交的<code>SHA</code>值，所以请确保这个列表里不包含你已经推送到共享仓库的提交。</font>
