#AWK

## AWK 有三个版本：
- awk 原始的老版本
- nawk AT&T后来更新的版本
- gawk 自由基金组织版本

## 基本结构

pattern {action}

pattern指定什么时候执行action。awk是面向行的。只有pattern为true的时候，action才会被执行，默认的pattern是匹配每一行。有两个特殊的pattern可由两个关键字指定BEGIN 和 END（大写），BEGIN匹配的action在任何行读入之前执行，END则在最后一行读入后执行。示例如下：

BEGIN{print “START”} {print} END{print “STOP”}

$符号指定一行中的第几个域(列)，双引号间的$符号无效。转义字符在双引号间可以执行，如“\t”，和shell不同，在shell中$x指x这个变量，而在awk中，$x指x变量所指的域。如下示例会输出两个域，一个是数字5，另一个是输入行的第五列。

BEGIN{x=5} {print $x}


##变量

示例1：

{% codeblock lang:bash %}
#!/bin/sh
#NOTE - this script does not work!
column=$1
awk '{print $column}'
{% endcodeblock %}

示例2：

{% codeblock lang:bash %}
#!/bin/sh
column=$1
awk '{print $'$column'}'
{% endcodeblock %}

执行命令：ls -l | Column 3 | uniq -c | sort -nr

示例2可以得到预期效果，而示例1则不行，主要原因是shell中对单引号的处理。单引号内的内容直接输出，而双引号内的变量等才是正常引用的。awk '{print $'$column'}'实际执行的语句是awk {print $3}。另外shell中可以通过下面格式设置变量默认值：

{% codeblock lang:bash %}
${variable:-defaultvalue}
column=${1:-1}
{% endcodeblock %}

另外，awk的脚本开始执行前，变量赋值已经完成。

awk '{print $c}' c=${1:-1}


给变量赋空值，可以在输出时删除对应变量编号的那个与值。示例如下：

{% codeblock lang:bash %}
#!/bin/awk -f
{
	$1="";
	$3="";
	print;
} 
{% endcodeblock %}

上述示例不同于下述示例，上述示例多两个域分隔符。

{% codeblock lang:bash %}

#!/bin/awk -f
{
	print $2, $4;
}

{% endcodeblock %}

##内建变量

###FS(域分隔符)
指定域分隔符的方法有两种，一种是命令行形式，一种是使用内建变量的方法。内建变量比命令行形式更灵活，更强大。内奸变量可以指定多个一定顺序字符的分隔符，还可以在执行过程中更改域分隔符，需要注意的是，如果在读取当前行之前更改了分隔符，那么新的分隔符在当前行生效，否则下一行生效，在读取了当前行之后更改分隔符，它不会冲定义变量。最新的版本中更改了分隔符后，需要加上$0=$0。

{% codeblock lang:bash %}
#!/bin/awk -f
{
	if ( $0 ~ /:/ ) {
		FS=":";
		$0=$0
	} else {
		FS=" ";
		$0=$0
	}
	#print the third field, whatever format
	print $3
}
{% endcodeblock %}

