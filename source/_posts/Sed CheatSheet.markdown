title: Sed CheatSheet
date: 2016-09-18 11:21:38
tags:
  - sed
---
1. `sed "s/my/lion's/g" exp.txt`

	单引号无法用\'转义，而双引号可以使用\"进行转义
	
2. `sed -i "s/my/lion's/g" exp.txt`

	-i 原地修改
<!-- more -->
