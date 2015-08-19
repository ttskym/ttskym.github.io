title: 某高校getshell实例
date: 2014-05-24 21:17:41
tags: 
 - 渗透
 - SQL注入
 - 文件上传
---

写篇文章，记录下渗透某高校网站过程及思路，顺便简单科普下SQL注入和文件上传。

目标链接：

http://xxx.xxx.edu.cn/html/Index/article/id/239

用单引号和 and 1=1 测试了下id，发现可以注入，便丢给了SQLmap

`python sqlmpyp.py -u "http://xxx.xxx.edu.cn/html/Index/article/id/239"`

-u 指定目标链接
<!--more-->

果然，如图所示

![1.jpg](http://novsec.qiniudn.com/%E6%9F%90%E9%AB%98%E6%A0%A1getshell%E5%AE%9E%E4%BE%8B/1.png)

接着，顺手又列了库

`python sqlmpyp.py -u "http://xxx.xxx.edu.cn/html/Index/article/id/239" --dbs`

--dbs 列出数据库

![2.jpg](http://novsec.qiniudn.com/%E6%9F%90%E9%AB%98%E6%A0%A1getshell%E5%AE%9E%E4%BE%8B/2.png)

猜想job，zs，zsjyc应该是对应着三个网站的库，输入下域名，验证后三个库对应着三个使用一套模版的网站，都存在SQL注入。

这样想来密码应该不难拿到。但是接下来，用google搜，扫路径，都没有找到后台。只能另寻它法。

仔细翻了下路径遍历的结果，发现了这样一个记录

![3.jpg](http://novsec.qiniudn.com/%E6%9F%90%E9%AB%98%E6%A0%A1getshell%E5%AE%9E%E4%BE%8B/3.png)

phpMyAdmin映入眼帘。

phpMyAdmin的账户密码存存储在mysql库中的user表中，从上面列出的数据库中我们便可以找到mysql库。

dump下myslq库中的user表，顺利拿到phpMyAdmin密码。

`python sqlmpyp.py -u "http://xxx.xxx.edu.cn/html/Index/article/id/239" -D mysql -T user --dump`

-D 指定数据库

-T 指定数据库中某表

--dump 下载表内容

登陆后，仔细翻了一下数据库，发现了后台的路径。

![4.jpg](http://novsec.qiniudn.com/%E6%9F%90%E9%AB%98%E6%A0%A1getshell%E5%AE%9E%E4%BE%8B/4.jpg)

用库中查到的后台密码，顺利登入后台管理。

![5.jpg](http://novsec.qiniudn.com/%E6%9F%90%E9%AB%98%E6%A0%A1getshell%E5%AE%9E%E4%BE%8B/5.jpg)

接下来要上马了。

试了下，明显过滤了后缀，拿出burpsuit，果然只是在前台验证了上传类型，绕过它。这个后台上传文件需要分两步，首先选择本地文件上传，上传成功后，还需要再保存，这个文件才算成功上传到服务器。如下图

![7.png](http://novsec.qiniudn.com/%E6%9F%90%E9%AB%98%E6%A0%A1getshell%E5%AE%9E%E4%BE%8B/7.png)

另外后台对上传文件的主名进行了加密，浏览器不能直接通过加密前文件名访问，只能通过加密后的文件名访问。这里猜想应该是做了一个映射。如下两图，用burpsuit成功绕过。

![8.png](http://novsec.qiniudn.com/%E6%9F%90%E9%AB%98%E6%A0%A1getshell%E5%AE%9E%E4%BE%8B/8.png)

图中标注的这个文件名应该是需要在服务器端存储的文件名。这里已经将.jpg后缀改为了.php后缀。

![9.png](http://novsec.qiniudn.com/%E6%9F%90%E9%AB%98%E6%A0%A1getshell%E5%AE%9E%E4%BE%8B/9.png)

此图中下划线标注的文件名猜测可能是外部访问的文件名，这个文件名英语上幅图中的文件名相同。这里要将其后缀改为.php。

笔者尝试过有一处不改都无法成功访问上传的马。

下图为用中国菜刀成功访问上传好的一句话木马。

![10.jpg](http://novsec.qiniudn.com/%E6%9F%90%E9%AB%98%E6%A0%A1getshell%E5%AE%9E%E4%BE%8B/10.png)

*end*

**声明：Novsec文章均为原创，转载请注明出自novsec.com**
