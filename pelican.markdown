<%--
Author: monnand
Key: /home/monnand/.ssh/monnand.pem
Title: Pelican：静态博客生成程序
Slug: pelican
Language: zhCN
Tags: ["pelican", "uniqush", "tools"]
--%>

刚刚做好这个博客的时候，收到了[CNBorn](http://cnborn.net)发来的邮件，一方面是祝贺新博客落成，另一方面，主要是推荐一款静态博客生成程序:
[pelican](http://pelican.readthedocs.org)。当时这个博客已经写得差不多了，所以也懒得折腾。前几天打算为[Uniqush](http://uniqush.org)建立一个[博客](http://blog.uniqush.org)，于是想起了[pelican](http://pelican.readthedocs.org)。用了之后感觉非常不错，真是有心把自己的博客也转过去了。

# 静态HTML生成 #

Pelican的基本功能，是把一大堆以[markdown](http://daringfireball.net/projects/markdown/)（或者其它[pelican](http://pelican.readthedocs.org)支持的格式）格式编写的文章，按照指定配置（用什么css，什么javascript，什么侧边栏等等）转换成一堆HTML的静态页面。也就是说，只要服务器支持静态HTML，就可以架设一个博客。详细的使用方法在它网站上有。

为什么用静态HTML呢？比起动态生成的HTML，静态HTML有哪些好处呢？

- 使用成本低。你不用费心去管后台用什么数据库，用什么架构，Django还是web.py。只要服务器支持静态HTML，就可以了。可以说是省时省力
- 安全。是的，动态生成就得需要写程序，写程序就难免出bug，出bug就难免有安全漏洞。所以，最简单的方法，就是只放静态页面，把安全问题交给别人的成熟的程序。
- 维护容易。你不用考虑究竟是Fast CGI还是别的。
- 速度快。没有了数据库查询，不用Parse模版，速度自然快了。

当然，静态HTML也有它的缺点，但对于博客这种应用来说，这些缺点基本是可以克服或者忽略的：

- 空间成本。一个页面就是也个静态的HTML文件，这样的话会很占空间。不过对于博客来说，再勤快的作者也写不出1G的博客文章（只考虑文字）。
- 无法评论。因为是静态页面，所以不能动态地添加评论。不过好在有[DISQUS](http://disqus.com)这样的服务，这个问题可以很容易解决。而且[pelican](http://pelican.readthedocs.org)可以很轻松地支持DIQUS。

# Uniqush Blog #

如果不知道[Uniqush](http://uniqush.org)是什么，可以到它的[官方网站](http://uniqush.org)看看。简单说来，如果你想要给手机端App推送消息，那么你可能就需要用到[Uniqush](http://uniqush.org)。前几天为这个项目建立了一个[博客](http://blog.uniqush.org)。就是使用的[pelican](http://pelican.readthedocs.org)。感觉非常好！感谢[CNBorn](http://cnborn.net)！

# 关于这个博客 #

简单说：不折腾了。长点：短期内不打算换到[pelican](http://pelican.readthedocs.org)。原因如下：

- 不想折腾了。
- 我写了这么个[博客系统](http://github.com/monnand/myblog)也怪不容易的。说换就换有点不忍心。
- 我每个月花$3.99从[ramhost](http://ramhost.us)买的VPS，到头来就用来架个静态页面，我真觉得特别亏。

不过说实话，真想过还到pelican下。确实好用。所以在这里给那些想自己架博客的朋友们一点建议：用pelican吧，找个免费放静态页面的地方，真省心啊。

