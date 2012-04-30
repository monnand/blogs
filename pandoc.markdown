<%--
Author: monnand
Key: /home/monnand/.ssh/monnand.pem
Title: Pandoc：全能文档转换
Slug: pandoc
Language: zhCN
Tags: ["pandoc", "tools"]
--%>

本人会不定期地推荐一些小工具，都会放在[tools](http://monnand.me/tag/18/)这个标签下。上次推荐了[pelican](http://monnand.me/p/pelican/zhCN/)，这次推荐[pandoc](http://johnmacfarlane.net/pandoc/index.html)：一个由[haskell](http://www.haskell.org)编写的文档转换的软件。如果你有一份文档，是用[markdown](http://daringfireball.net/projects/markdown/)写的，想把它转换成[reStructuredText](http://docutils.sourceforge.net/rst.html)，或者[texinfo](http://www.gnu.org/software/texinfo/)；再或者你用[markdown](http://daringfireball.net/projects/markdown/)写了一本书，想转换成epub……总之，很多文档格式之间的转换，都可以用[pandoc](http://johnmacfarlane.net/pandoc/index.html)完成。

# 为什么转来转去？ #

虽然这个问题也许没什么意义，但是实际中，确实会经常遇到各种需要转换格式的场合。以我个人为例：[uniqush.org](http://uniqush.org)是用[trac](http://trac.edgewall.org/)做的，它上面的文档全部是[wiki](http://meta.wikimedia.org/wiki/Wiki_syntax)格式。而[uniqush的博客](http://blog.uniqush.org)则是用[pelican](http://monnand.me/p/pelican/zhCN/)搭的，上面的文档是[markdown](http://daringfireball.net/projects/markdown/)格式。uniqush在[github](http://github.com/monnand/uniqush)上放着代码，里面的文档显然也是[markdown](http://daringfireball.net/projects/markdown/)格式，本人博客也是[markdown](http://daringfireball.net/projects/markdown/)格式……总之，各种格式之间需要来回转换。这个时候，[pandoc](http://johnmacfarlane.net/pandoc/index.html)就管用了。

# 到底支持哪些格式 #

基本说来，常用的格式[pandoc](http://johnmacfarlane.net/pandoc/index.html)都会支持。如果想具体看看能不能从某个格式转换为另外一个格式，[这张图](http://johnmacfarlane.net/pandoc/diagram.png)详细说明了支持哪些格式之间的转换。


