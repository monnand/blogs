<%--
Author: monnand
Key: /home/monnand/.ssh/monnand.pem
Title: 建站细节
Slug: blog-config
Language: zhCN
Tags: ["Django", "My Blog", "lighttpd"]
--%>

生命不息，折腾不止。伴随着北美冬季的寒风，俺又开始了写博客的折腾人生。而且这次更加体现了凡事都要自己来的宅男个性。从代码到建站，基本都是自己搞定的。本文详细叙述一下具体的建站配置。至于代码，都在[github](http://github.com/monnand/myblog/)上可以看到，我就不多说了。

#网站代码#
好吧，我确实说不多说了，但也不能不说。全站都是用[Django](http://djangoproject.com/)写的，没有用[admin](https://docs.djangoproject.com/en/dev/ref/contrib/admin/)，因为统共就一个作者，不值当的。当然，代码本身是支持多作者的。不过这块的开发还有待完善。

##发文章##
发文章是通过HTTP Post来完成---好吧，这有点废话了。不过鉴于本人买不起SSL的证书，所以只好在http协议上做了。考虑到安全性，发文章只能通过命令行的一个脚本来完成。每个作者在创建的时候，都会上传一个自己的解密密钥。然后该作者在发文章的时候，先随机生成一个密钥，利用这个密钥对文章内容加密，再用作者本人的加密密钥对这个密钥进行加密，接着把这些加了密的玩意用Post传给web服务器。简单说来，就是在没有HTTPS下的土法炼钢。基本来说还算好用

##验证码##
我知道验证码很烦人，但是上次在 [GAE 上的博客](http://monnand.appspot.com/) 上，我加入了无验证码评论的功能，结果没几天就被各种垃圾广告回复占领了。最后又被迫不得不回到必须登陆才能回复的状态。

这次的验证码，使用的是[captchas.net](http://captchas.net/)的服务，很简单，具体就看看代码吧。

#VPS#
在[ramhost.us](http://ramhost.us)上买的VPS，非常便宜，每月才$3.99。基本上用来架个博客是没啥问题的。

#lighttpd#
我快彻底被[apache](http://apache.org/)搞崩溃了。我承认，我这VPS庙太小，请不起Apache这样的大神。动不动就跟我说没内存了，占了一大堆内存不说，还把整个服务器给拖的特别慢。点开个页面要好几秒。最后，在尝试了各种可能的配置之后，我放弃它了。也许有更大的内存还会好点。不过我目前是怎么用过Apache了。手头的两个VPS分别用的是[nginx](http://nginx.org/)和[lighttpd](http://www.lighttpd.net/)。

这个博客用的是[lighttpd](http://www.lighttpd.net/) + Fast CGI + [PostgreSQL](http://www.postgresql.org/)。据说用apache的``mod_wsgi``会更加稳定，但实在供不起Apache。

#Github#
原谅我吧，[github](http://github.com/)！除了代码，我把文章也[放上去](http://github.com/monnand/blogs/)了。
