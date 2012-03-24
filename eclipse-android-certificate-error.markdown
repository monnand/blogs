<%--
Author: monnand
Key: /home/monnand/.ssh/monnand.pem
Title: Eclipse运行Android程序报错
Slug: eclipse-android-certificate-error
Language: zhCN
Tags: ["android", "uniqush", "eclipse"]
--%>

这个标题太纠结了。我想了好久也没相出怎么描述。是的，报错了：Your project contains error(s), please fix them before running your application。但是只有在项目左上角一个小红叉子，文件中没有任何错误。google了一下，找到了[解决方案](http://stackoverflow.com/questions/4954316/your-project-contains-errors-please-fix-it-before-running-it)。

简单说来，是因为调试用的证书的问题。删除~/.android/debug.keystore就可以了。
