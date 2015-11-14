---
layout: post
title: "利用 Android Studio 阅读 Android 源码"
date: 2015-11-14 17:50:48 +0800
comments: true
categories: android, 源码
---
Android 源码中有一个目录 `development/tools/idegen`, 该目录下的 idegen.sh 脚本可以生成 intellij IDE 的 project 文件，当然也可用于 Android Studio, 这样就可以利用 Android Studio 来阅读 Android 源码了，具体步骤如下：

1. **生成 idegen.jar**: 执行 idegen.sh 脚本需要 idegen.jar 文件，生成该文件需要编译源码，如果不想编译源码，也可以下载一个编译好的 idegen.jar，附上[下载链接](/assets/idegen.jar)，并将该文件 copy 到 `/out/host/linux-x86/framework/idegen.jar`。
2. **运行 idegen.sh**: 切换到 Android 源码根目录，运行`development/tools/idegen/idegen.sh`，过几秒钟后会在根目录下生成 android.ipr 和 android.iml 两个文件。
3. **导入到 Android Studio**: 打开 Android studio，点击 File > Open，选择刚刚生成的 android.ipr 打开，等待加载好了就可以了。

不得不说函数跳转、查找等...体验太爽了！