title: Sed CheatSheet
date: 2016-09-18 11:21:38
tags:
  - sed
  - linux 
---

1、 单引号无法用\'转义，而双引号可以使用\"进行转义
{% codeblock lang:bash %}
sed "s/my/lion's/g" exp.txt
{% endcodeblock %}


	
2、 -i 原地修改
{% codeblock lang:bash %}
sed -i "s/my/lion's/g" exp.txt
{% endcodeblock %}

<!-- more -->

3、 ^, $均为正则语法，\< 表示词首， \>表示词尾。如`\<abc`表示以abc为首的词 
{% codeblock lang:bash %}
sed 's/^/#/g' exp.txt
sed 's/$/--/g' exp.txt
{% endcodeblock %}
	
正则举例：
	
{% codeblock lang:bash %}
sed 's/<.*>//g' html.txt (x)
sed 's/<[^>]*>//g html.txt (v)
{% endcodeblock %}

4、 指定行替换
{% codeblock lang:bash %}
sed '3s/my/your/g' exp.txt
sed '3,6s/my/your/g' exp.txt
{% endcodeblock %}
   
   
5、 指定行内位置替换
{% codeblock lang:bash %}
sed 's/my/your/1' exp.txt
sed 's/my/your/2' exp.txt
sed 's/my/your/3g' exp.txt
{% endcodeblock %}

6、 多个匹配模式，下述两个命令执行效果相同
{% codeblock lang:bash %}
sed '1,3s/my/your/g; 3,$s/this/that/g' exp.txt
sed '1,3s/my/your/g' -e '3,$s/this/that/g' exp.txt
{% endcodeblock %}

7、 &可以作为被匹配的指代变量
{% codeblock lang:bash %}
sed 's/my/[&]/g' exp.txt
{% endcodeblock %}

8、 圆括号匹配，\1,\2分别指代第一个圆括号匹配的内容和第二个圆括号匹配的内容
{% codeblock lang:bash %}
sed 's/This is my \([^,]*\), .* is \(.*\)/\1:\2/g' exp.txt
{% endcodeblock %}

9、 N命令式将下一行纳入缓冲区当做一行进行匹配
{% codeblock lang:bash %}
sed 'N;s/\n/,/g' exp.txt
{% endcodeblock %}

10、 前查i， 后插a
{% codeblock lang:bash %}
sed '1 i Insert in front of the line' exp.txt
sed '$ a Insert behind the line' exp.txt
sed '/fish/a Insert behind the pattern' exp.txt
{% endcodeblock %}

11、 c命令替换命令
{% codeblock lang:bash %}
sed '2 c example line' exp.txt
sed '/fish/ c pattern line' exp.txt
{% endcodeblock %}

12、 删除命令
{% codeblock lang:bash %}
sed '/fish/d' exp.txt
sed '2 d' exp.txt
sed '2,$' exp.txt
{% endcodeblock %}

13、 grep效果
{% codeblock lang:bash %}
sed -n '/fish/p' exp.txt
{% endcodeblock %}
   
   
[Ref]
[sed 简明教程](http://coolshell.cn/articles/9104.html)
[学习awk和sed](http://yikun.github.io/2015/05/24/%E5%AD%A6%E4%B9%A0awk%E5%92%8Csed/)


