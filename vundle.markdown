<%--
Author: monnand
Key: /home/monnand/.ssh/monnand.pem
Title: Vundle：轻松管理vim插件
Slug: vundle
Language: zhCN
Tags: ["tools", "vim", "vundle"]
--%>

每次重新装完系统，vim的各种插件都要重新安装一遍。安装倒也罢了，关键经常会忘掉哪个插件。一个解决方案是把所有用到的插件都写到某个地方，下次安装的时候，一个一个安装。但是更好的方法，是使用一个插件管理器，把需要用的插件写好，让管理器自动从网上下载安装这些插件。显然，[vundle]就非常适合做这种事情。

# 所有信息放在vimrc里

你再也不用把整个.vim目录都备份，也不用记得去把系统安装的vim插件一个一个都写下来。你需要做的，就是在你自己的vimrc里添加几行信息，再把需要安装的插件地址写到vimrc里，所有任务就都完成了。[vundle]可以自动识别github和vim-script上的vim插件，你需要执行`:BundleInstall`命令，它就会从网上下载并安装好所有插件。

# 我的vimrc

还是一样，放到了[github](https://github.com/monnand/vimrc)上。里面用到了一些常见插件，也用到了我自己修改的插件。

[vundle]: https://github.com/gmarik/vundle


