<%--
Author: monnand
Key: /home/monnand/.ssh/monnand.pem
Title: Blog Bug Fix：博文无子标题排版不正常
Slug: blog-bug-fix-no-title-in-post
Language: zhCN
Tags: ["My Blog"]
--%>

刚刚修正了一个本[博客系统](http://github.com/monnand/myblog)的Bug：如果博客文章没有子标题，则排版不正常。

正常行为是：提取第一个子标题之前的部分作为博客摘要，在首页显示，每篇文章的其他部分需要点击Read More，进入单独页面才能看到。但是当时忘了考虑文章没有子标题的情况，这才出现了这么个不大不小的bug。特此公告一下。（虽然似乎也没人用我这个博客系统）


