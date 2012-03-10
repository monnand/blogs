<%--
Author: monnand
Key: /home/monnand/.ssh/monnand.pem
Title: LaTeX符号分类器
Slug: latex-symbol-classifier
Language: zhCN
Tags: ["LaTeX"]
--%>

每次写LaTeX文档的时候，总会忘记某个符号究竟该怎么写。且不说各种乱七八糟的数学符号，单就背全希腊字母，就得有段时间。于是没当此时，就要google一下Latex cookbook，然后在[cookbook](http://www.ce.rit.edu/~cockburn/latex/TeX%20cookbook.pdf)里查半天。肯定不止我一个人想过：要是有个程序，识别我的手写输入，然后自动地找出对应的LaTeX符号该怎么写，那该多好。前几天居然真的找到了这么一个[在线的分类器](http://shapecatcher.com/)。用法也很简单，鼠标画个符号，它就能自动找出可能的几个符号。整个分类程序的源代码也是[公开](https://github.com/kirel/sketch-a-char)的。另外，这个分类器和还有一个[旧版本](http://detexify.kirelabs.org/classify.html)。这个旧版本是用[haskell](http://www.haskell.org)写的，源代码也是[公开](https://github.com/kirel/detexify-hs-backend)的。以后要是再想知道某个符号怎么在LaTeX里表示，就方便许多了。


