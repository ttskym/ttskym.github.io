title: Kali 2.0 Parallels Tools 安装
date: 2015-09-19 20:00:00
tags:
 - Kali
 - Parallels Tools
---

很高兴8月份发布了Kali 2.0。新版本的Kali给我们带来了漂亮的界面，用户界面更为友好，相信这会让我们的渗透工作更加有趣。

我在Parallels Desktop 11 中安装了Kali 2.0的64位版本，安装过程中没有设置SWAP分区(机器性能充足，哈哈)，很顺利的安装完毕。

但是在安装Parallels Tools过程中遇到了问题。其实这是一个非常小的问题，甚至我觉得根本不需要在这里专门用一篇文章来记录。但是我想可能有初学者会碰到同样的问题，希望这篇文章能为他们节省时间，时间是宝贵的嘛~~

<!-- more -->

安装Parallels Tools中，会提示你需要安装kparts，linux-headers，dkms等。但当你用apt安装的时候却提示你找不到包。这时候就需要检查更新源了。

目标文件：/etc/apt/source.list

将其内容替换为

{% codeblock lang:bash %}

deb http://http.kali.org/kali sana main non-free contrib
deb http://security.kali.org/kali-security sana/updates main contrib non-free
deb-src http://http.kali.org/kali sana main non-free contrib
deb-src http://security.kali.org/kali-security sana/updates main contrib non-free

{% endcodeblock %}

这里稍微解释下：

每条源包含4部分，每部分用空格分离。第一部分，deb指二进制包，deb-src指源码包。第二部分，是指更新的URL。第三部分是发行版名称，最后是软件包的类型。

具体的source.list解释请参考The Debian Administrator's Handbook中的相关章节：

[Filling in the sources.list File](https://debian-handbook.info/browse/stable/apt.html#sect.apt-sources.list)

此外说明，source.list的内容应以官方的具体内容为准，也可以使用国内的镜像。官方的地址如下：

[Kali sources.list Repositories](http://docs.kali.org/general-use/kali-linux-sources-list-repositories)


Finally, enjoy it!

{% image fancybox nocaption http://7xl89b.com1.z0.glb.clouddn.com/Kali%202.0%20parallels%20Tools%20安装/kali2.0.png "kali" %}
