---
layout: post
title: "Windows 文件关联浅析"
date: 2014-11-29 23:43:14 +0800
comments: true
categories: Windows
keywords: Windows 文件关联 注册表 打开方式
description: 对windows下各种文件关联方式进行总结。
---
<h2>引言</h2>
所谓的文件关联就是指系统把指定扩展名的文件自动关联到相应的应用程序，例如.doc默认打开方式是Microsoft Word，当用户双击时就会启动Word打开该文件。windows平台下的文件默认打开方式一直是各大互联网公司的必争之地，如：各种播放器抢媒体文件的默认打开方式，说白了就是为了争入口你抢我的我抢你的，写这篇文章并不是想比谁更流氓，只是简单的总结下windows下的文件关联方式。转载请注明出处：<code>http://glgjing.github.io/</code>。<!-- more -->
<h2>一 文件关联相关的注册表</h2>
Windows用注册表来保存当前系统的所有文件关联设置，如果想更改文件关联设置，就要在注册表上做文章。首先简单介绍下注册表的各个目录(<a href="http://msdn.microsoft.com/zh-cn/library/cc776231%28v=ws.10%29.aspx">引用自msdn</a>)。
<ul>
	<li><code>HKEY_LOCAL_MACHINE</code>：包含关于本地计算机系统的信息，包括硬件和操作系统数据，如总线类型、系统内存、设备驱动程序和启动控制数据。</li>
	<li><code>HKEY_CURRENT_USER</code>：包含当前以交互方式（与远程方式相反）登录的用户的用户配置文件，包括环境变量、桌面设置、网络连接、打印机和程序首选项。该子树是 HKEY_USERS 子树的别名，它指向 HKEY_USERS\当前用户的安全 ID。</li>
	<li><code>HKEY_CLASSES_ROOT</code>：包含用于各种 OLE 技术和文件类关联数据的信息。如果HKEY_LOCAL_MACHINE\SOFTWARE\Classes 或HKEY_CURRENT_USER\SOFTWARE\Classes 中存在相应的项或值，则在 HKEY_CLASSES_ROOT中会存在某个特定的项或值。如果两处均存在项或值，则 HKEY_CURRENT_USER 版本将是出现在HKEY_CLASSES_ROOT 中的那一个。</li>
	<li><code>HKEY_USERS</code>：包含关于动态加载的用户配置文件和默认配置文件的信息。它包含同时出现在 HKEY_CURRENT_USER中的信息。正在远程访问服务器的用户在服务器上的该项下没有配置文件；他们的配置文件将加载到自己计算机的注册表中。</li>
	<li><code>HKEY_CURRENT_CONFIG</code>：包含在启动时由本地计算机系统使用的硬件配置文件的相关信息。该信息用于配置一些设置，如要加载的设备驱动程序、显示时要使用的分辨率。该子树属于 HKEY_LOCAL_MACHINE 子树，它指向HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Hardware Profiles\Current。</li>
</ul>
与文件关联相关的注册表项主要有以下几项：
{% codeblock %}
HKEY_CURRENT_USER\Software\Classe
HKEY_LOCAL_MACHINE\Software\Classe
HKEY_CLASS_ROOT
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\FileExts\
{% endcodeblock %}
以下<code>HKEY_LOCAL_MACHINE</code>简称为<code>HKLM</code>，<code>HKEY_CURRENT_USER</code>简称为<code>HKCU</code>，<code>HKEY_CLASSES_ROOT</code>简称为<code>HKCR</code>。
其中<code>HKCU</code>保存了当前用户的文件关联设置，<code>HKLM</code>保存了本机上所有用户的设置，<code>HKCR</code>是上面两个位置下的键值合并，是为了访问方便而建立的视图，<code>...\FileExts\</code>保存了右键选择“打开方式”改变默认的关联程序。

<h2>二 各项注册表的关联优先级及权限</h2>
<h3>1 关联优先级</h3>
<code>...\FileExts\</code>高于<code>HKCU</code>高于<code>HKLM</code>。（由于<code>HKCR</code>只是为了访问方便而建立的视图，最好只用于读取，若要更改则可以更改<code>HKCU</code>和<code>HKLM</code>下对应的内容即可）
<h3>2 修改权限</h3>
vista以上的windows版本在开启UAC的情况下，更改<code>HKLM</code>本机设置需要管理员权限（提权），更改<code>HKCU</code>当前用户设置不需要提权。
<h3>3 用户双击文件时查找顺序</h3>
首先检查<code>...\FileExts\</code>，找不到时查找<code>HKCU</code>，最后才是<code>HKLM</code>。因此检查一个文件是否与某个程序关联可以按照这个顺序检查。

<h2>三 如何关联文件</h2>
以apk文件为例，需要在上面所说几个位置下有 .apk 注册表项，该项的默认值对应ProgId，例如命名为ApkFile）。在.apk的相同层级下写入ApkFile注册表项，包含subkey：DefaultIcon和shell\open\command。其中DefaultIcon子项的值为显示的图标，shell\open\command的值为关联的程序。整体结构如下：
{% codeblock %}
HKEY_CURRENT_USER\Software\Classes
	.apk
		(Default) = ApkFile
	ApkFile
		DefaultIcon
			(Default) = xxx
	shell
		open
			command
				(Default) = xxx.exe %1
{% endcodeblock %}
在更改了文件关联以后，需要调用<code>SHChangeNotify</code>参数为<code>SHCNE_ASSOCCHANGED</code>通知Windows文件关联设置已经改变，否则要下次登录才能看到变化。

<h2>总结</h2>
文件关联实际上就是更改相应的注册表项，更改<code>HKLM</code>下的注册表项对计算机上所有用户都会有影响，但在开启UAC的计算机上需要提权，而且优先级不如<code>HKCU</code>只对当前用户起作用，且不需要提权，而且优先级高于<code>HKLM</code>，所以一般的桌面软件建议更改这一项，至于以什么方式更改（常驻进程定时查询，暴力抢占等等）就看产品了。<code>...\FileExts\</code>下的UserChoice是用户选择的默认关联方式，由用户指定，不建议更改。