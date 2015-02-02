---
layout: post
title: "为 git 添加多个公秘钥"
date: 2015-02-02 16:46:55 +0800
comments: true
categories: git
keywords: git 公钥 多个公钥 rsa gitcafe
description: 本文介绍了git如何同时使用多个公钥。 
---
<h2>小引</h2>
最近将博客从 github 上迁移到了国内的 gitcafe，主要原因是在国内的访问速度你懂的。迁移之后发现问题来了，每次连接时 SSH 客户端发送本地私钥到服务端验证，而这个私钥默认是 ~/.ssh/id_rsa 文件，如果只有一个账户，连接的服务器上保存的公钥和发送的私钥自然是配对的。但是我已经有了两个账户了，当然 username 和 email 是完全不同的，如果还是都读取默认的 ~/.ssh/id_rsa 文件，那么 gitcafe 账户的公钥和私钥显然是不匹配的。所以需要为 gitcafe 账户再添加一对公秘钥，并且可以根据账户自动寻找匹配的公秘钥。转载请注明出处<code>http://glgjing.github.io/</code><!-- more -->
<h2>生成新的 rsa key</h2>
在命令行输入如下命令，将命令中的 YOUR_EMAIL@YOUREMAIL.COM 改为你的 Email 地址。
{% codeblock %}
ssh-keygen -t rsa -C "YOUR_EMAIL@YOUREMAIL.COM" -f ~/.ssh/gitcafe_rsa
{% endcodeblock %}
生成过程中会出现以下信息，按屏幕提示操作输入 passphrase 口令，也可以直接回车设为空。
{% codeblock %}
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /Users/username/.ssh/gitcafe_rsa.
Your public key has been saved in /Users/username/.ssh/gitcafe_rsa.pub.
The key fingerprint is:
15:81:d2:7a:c6:6c:0f:ec:b0:b6:d4:18:b8:d1:41:48 YOUR_EMAIL@YOUREMAIL.COM
{% endcodeblock %}
此时在 ~/.ssh/ 目录下会生成 gitcafe_rsa 和 gitcafe_rsa.pub两个文件。
<h2>配置新的 rsa key</h2>
在 SSH 用户配置文件 ~/.ssh/config 中指定对应服务所使用的公秘钥名称，如果没有 config 文件的话就新建一个，并输入以下内容：
{% codeblock %}
Host gitcafe.com www.gitcafe.com
  IdentityFile ~/.ssh/gitcafe
{% endcodeblock %}
然后将 gitcafe_rsa.pub 文件中的内容复制到 gitcafe 的 SSH 公钥中，具体参见<a href="https://gitcafe.com/GitCafe/Help/wiki/%E5%A6%82%E4%BD%95%E5%AE%89%E8%A3%85%E5%92%8C%E8%AE%BE%E7%BD%AE%20Git">这里</a>，这样就完成了新的公秘钥配置。可以运行下面的命令进行测试
{% codeblock %}
Host gitcafe.com www.gitcafe.com
ssh -T git@gitcafe.com
{% endcodeblock %}
如果连接成功的话，会出现以下信息。
{% codeblock %}
Host gitcafe.com www.gitcafe.com
Hi USERNAME! You've successfully authenticated, but GitCafe does not provide shell access.
{% endcodeblock %}
<h2>总结</h2>
这里以 gitcafe 为例讲述了如何为git添加多个公秘钥，其他的公秘钥添加方法基本类似。参考：<a href="https://gitcafe.com/GitCafe/Help/wiki/%E5%A6%82%E4%BD%95%E5%90%8C%E6%97%B6%E4%BD%BF%E7%94%A8%E5%A4%9A%E4%B8%AA%E5%85%AC%E7%A7%98%E9%92%A5">如何同时使用多个公秘钥</a>。