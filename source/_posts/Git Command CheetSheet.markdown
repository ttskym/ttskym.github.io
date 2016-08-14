title: Git Command CheetSheet
date: 2016-06-11 18:21:38
tags:
  - github
---
本文作为自己对于github用法的一个长期积累总结，记录在这里，随时参考、备忘。
<!-- more -->
{% codeblock lang:zsh%}
git checkout origin/master master   #建立本地branc，并与remote branch连接

git rm -r -n --cached  [file]   #-n：加上这个参数，执行命令时，是不会删除任何文件，而是展示此命令要删除的文件列表预览
git rm -r --cached  [file]  	#最终执行命令
git commit -m "comment"  
git push origin master 

git merge [branch]              #合并branch到当前分支
{% endcodeblock %}

Ref:
[Git 建立 Remote Branch 的相關指令操作](https://blog.longwin.com.tw/2013/11/git-create-remote-branch-2013/)
[git删除远程文件夹或文件的方法](http://www.cnblogs.com/xusir/p/4111723.html)

[git 简易指南](http://rogerdudler.github.io/git-guide/index.zh.html)
[阳志平 git入门资料](http://www.yangzhiping.com/tech/git.html)
[阳志平 如何高效利用github](http://www.yangzhiping.com/tech/github.html)
[git 参考使用手册](http://gitref.org/zh/creating/)
[git 中国](http://www.gitchina.org)
[git](http://git-scm.com/)
[Generating SSH Keys](https://help.github.com/articles/generating-ssh-keys#platform-linux)
