---
layout: post
title: "Android XML 属性参考手册"
date: 2015-04-10 15:36:04 +0800
comments: true
categories: android
keywords: android xml 属性 参考手册
---

## TextView
* android:autoLink：url、email、phone 等自动转换为可点击的 link
* android:autoText：拼写错误检查，错误的会被标出来
* android:digits：过略用户可输入的字符，只有被列出的字符可被输入
* android:drawableTop：文字内容上方显示的 drawable
* android:drawableBottom：文字内容下方显示的 drawable
* android:drawableLeft：文字内容左方显示的 drawable
* android:drawableRight：文字内容右方显示的 drawable
* android:drawableStart：文字内容开始位置显示的 drawable，和 drawableLeft 区别在于 RTL 情况
* android:drawableEnd：文字内容结束结位置显示的 drawable，和 drawableRight 区别在于 RTL 情况
* android:drawablePadding：文字内容和 drawable 之间的 padding
* android:ellipsize：当文字内容超出 TextView 的宽度时会加上...位置可以设为 start, end, middle, marquee
* android:gravity：当 text 的 size 小于 TextView 的 size时，gravity 决定 text 在 TextView 内的对齐方式<!--more-->
* android:height：TextView 的高度
* android:width：TextView 的宽度
* android:maxHeight：TextView 的最大高度
* android:maxWidth：TextView 的最大宽度
* android:minHeight：TextView 的最小高度
* android:minWidth：TextView 的最小宽度
* android:lines：text 正好显示的行数
* android:maxLines：text 的最大显示行数
* android:hint：当 text 为空时显示的提示内容
* android:linksClickable：设置 link 是否可点击，如果为 false 即使 autoLink 为 true 也不可点击
* android:marqueeRepeatLimit：marquee 动画的持续次数
* android:editable：设置 TextView 可以有输入法
* android:numeric：设置 TextView 的输入为数字输入法
* android:phoneNumber：设置 TextView 的输入为电话号码输入法
* android:password：text 内容显示为密码形式的小点，而非内容本身
* android:scrollHorizontally：设置 TextView 是否可以水平滚动
* android:selectAllOnFocus：当 TextView 处于 Focus 状态，Text 是否被全部选中
* android:shadowColor：阴影颜色
* android:shadowDx：阴影 X 方向偏移
* android:shadowDy：阴影 Y 方向偏移
* android:shadowRadius：阴影模糊半径
* android:singleLine：强制 Text 单行显示
* android:text：Text 的内容
* android:textAllCaps：将 Text 以大写的方式显示，当 TextView 同时包含 editable 或者 selectable 属性时，该属性被忽略
* android:textColor：Text 的颜色
* android:textColorHighlight：Text 被选中的高亮颜色
* android:textColorHint：提示 Text 的颜色
* android:textColorLink：Link 的文字颜色
* android:textIsSelectable：Text  内容是否可以被选中
* android:textScaleX：水平拉伸比例
* android:textSize：Text 的 size
* android:textStyle：Text 的 style (bold, italic, bolditalic) 
* android:typeface：Text 的字体 (normal, sans, serif, monospace) 
* android:width：设置 TextView 的宽度，目前没发现和 layout_width 有什么明显区别 ？

转载请注明出处：<http://glgjing.github.io/>  
待更新...



