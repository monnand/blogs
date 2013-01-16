<%--
Author: monnand
Key: /home/monnand/.ssh/monnand.pem
Title: Ready? Go! 下篇：多核并起
Slug: ready-go-2
Language: zhCN
Tags: ["golang"]
--%>

*本文分两部分连载于2012年5月和6月的《程序员》杂志。当时[Go语言](http://golang.org)刚刚推出第一个稳定版：Go 1。刊载时略有删改。[上篇](http://monnand.me/p/ready-go-1/zhCN/)*

Google于2009年11月发布了Go编程语言，旨在同时具备C语言的效率和Python的简便。今年3月，Go开发组正式发布了Go语言的第一个稳定发行版：Go version 1，简称Go 1。这意味着Go语言本身和它的标准库已经稳定下来，开发者现在可以将其作为一个稳定的开发平台，构建自己的应用。我们用两篇文章介绍Go语言的特性和应用，本文是其中的第二篇。

[上一期](http://monnand.me/p/ready-go-1/zhCN/)介绍了Go语言的部分语法和类型系统。本篇重点介绍Go语言对于并行的处理和go工具链。本文中的完整代码可以在[github](http://github.com/monnand/goexamples)上找到。

# 并行和goroutine

> *然而，处理器技术的发展指出，比起[掩盖了各种并行结构的]单处理器，由多个类似的处理器（各自包含自己的存储单元）组成的多处理器计算机也许会更加强大，可靠和经济。*
> --- C.A.R. Hoare，图灵奖获得者，CSP作者，于1978年

20世纪六七十年代，为了弥补处理器的处理能力，并行计算曾一度成为研究热点。期间不乏优秀的想法，如信号量（Semaphore），管程（Monitor），锁（mutex）以及基于消息传递的同步机制。但八十年代起，随着单核处理器性能飞速提高，学术界迎来了并行计算的黑暗时期。六七十年代的研究成果中，只有早期的一些思想被大规模使用在实际开发中。而七十年代后期的很多成果甚至还没被大规模应用，就伴随着并行计算黑暗期的到来，或不温不火，或被收藏入库。[CSP]（Communicating Sequential Processes）便是其中之一。但它优雅简洁的处理方式却依然在一些小众语言中流传了下来。如今，由于能耗和散热问题，处理器的发展转而以多核的方式提高处理器性能。我们再次迎来了曾经面对过的并行计算。这时候，[CSP]模型逐渐展露头脚。

[CSP]的基本思路是基于消息机制的同步和数据共享。与传统的锁同步不同，消息机制简化了程序设计，并且可以有效地减少潜在bug。基于CSP模型的语言主要有三个分支：忠于原始CSP设计，以[Occam]为代表的一支；强调网络和模式，以[Erlang]为代表的一支；再一个就是强调传递消息的信道（channel），以[Squeak]，[Newsqueak]，[Alef]，[Limbo]和[Go]为代表的一支。值得一提的是，第三支的语言中，大部分都是有Rob Pike主持或参与开发的，其中自然也包括Go。

既然说起Go的这一分支是以强调信道（channel）为特色，那么就先从Go的信道说起。Go的信道是一种数据类型，goroutine可以使用它来传递数据。至于goroutine是什么，之后会详细讨论。此处仅需把它理解为与线程类似的运行时结构即可。

定义一个信道，需要指定这个信道上传递的数据类型。可以是int，float32，float64等基本数据类型，也可以是用户自定义的结构体，接口，甚至可以是信道本身。

    :::go
    ch := make(chan int)

这样，就定义了一个传递整数类型的信道。如果要从这个信道中读取一个值，则可以使用``<-``操作。类似的，写入则使用``->``操作符：

    :::go
    // 从ch中读取一个值存入i中
    i := <- ch
    
    // 向ch中写入j的值
    ch <- j
    
信道的操作是同步的，一个读操作只有在真正读到内容之后，才继续执行下面的语句；而写操作则只有在写入数据被信道另一端读到，才执行之后的语句。（Go中信道也可以加入缓存队列，在此不多讨论）

同时，对于信道，还允许使用for循环依次处理来自信道的内容：

    :::go
    func handle(queue chan *Request) {
        for r := range queue {
            process(r)
        }
    }
    
这个函数的任务就是不断地从信道中读取Request结构体的指针，然后调用process函数进行处理。

除此以外，还可以使用select对多个信道进行读写操作：

    :::go
    func Serve(queue chan *Request,
               quit chan bool) {
        for {
            select {
            case req := <- queue:
                process(r)
            case <- quit:
                return
            }
        }
    }

这个函数接受两个信道作为参数。第一个信道queue用来传递各种请求。第二个信道quit则用来发布一条信令，告诉该函数返回。

接下来要说的，就是goroutine。它是一种比线程还要轻量的并行结构。在Go程序运行时，一般会并行运行几个线程，然后把goroutine分配到各个线程中。当一个goroutine结束或者被阻塞的时候，另外一个goroutine将被调度到被阻塞或结束的goroutine所在的线程中。这样的调度保证了每个线程可以有较高的使用率，不必一直处于阻塞状态。由此省去了很多操作系统调度线程而导致上下文切换。按照Go官方的说法，一个Go程序同时运行几万到几十万个goroutine是非常正常的。

使用一个goroutine也非常简单，只要在函数调用前面加入``go``就可以了：

    :::go
    go process(r)
    
这样，process这个函数就单独运行在一个goroutine中了。

由此带来的结果，就是极度地简化了服务器端对并发连接的处理。众所周知，如果让一个线程只处理一个用户连接，那么开发起来会非常简单，但是效率不高；而如果一个线程处理多个用户连接，又无端增加了开发难度。而配合信道使用goroutine则在不增加开发难度的同时，也提高了效率。

考虑这样一个应用场景：服务器从网络接收客户端请求，做一些处理，再把结果返回给客户。

对于不同的用户连接，用不同的goroutine处理。定义名为UserConn的结构体来表示一个用户连接。同时，这个结构体定义了一个叫做ReadRequest的方法，用于从网络读取用户的请求；还有一个叫做WriteResponse的方法，用于从网络给用户传递结果。作为一个想象的例子，具体的实现细节在此不详述。

那么，对于每个连接，要做的事情大约如此：

    :::go
    func ServeClient(conn *UserConn) {
        ch := make(chan *Response)

        // 创建一个goroutine，
        // 专门用于向用户发送结果
        go writeRes(conn, ch)

        for {
            // 读取一个请求，
            //  判断类型
            // 如果用户请求关闭，
            //  则函数返回
            req := conn.ReadRequest()
            switch req.Type {
            case NORMAL_REQUEST:
                go process(req, ch)
            case EXIT:
                return
            }
        }
    }

writeRes和process的基本结构大约如下：

    :::go
    func writeRes(conn *UserConn,
                 ch chan *Response) {
        for r := range ch {
            conn.WriteResponse(r)
        }
    }
    
    func process(req *Request,
                ch chan *Response) {
        res := calculate(req)
        ch <-res
    }

信道本身很符合人们对于通信工具的直觉定义，开发者可以很自然地使用信道在goroutine之间建立各种关系。使用信道和goroutine，每个函数要完成的任务都被单一化，减少了发生错误的可能。代码中，通过传递指针的方式来共享内存空间，在每次共享之前，都是以消息进行同步。这又是一条Go的原则：用传递消息来共享内存；而不是用共享内存来传递消息。由此简化了并行程序的开发。

作为一个实用的编程语言，Go并没有按照CSP原始论文中说的，仅仅提供信道的方式来进行同步。Go在标准库中也提供了基于锁，信号量等传统同步机制的工具。在以上代码中，其实存在着一个潜在bug：ServeClient函数不是在所有运行process的goroutine执行结束后再退出，而是在一收到来自客户端的退出命令后直接退出的。更合理的操作应该在所有处理该连接的goroutine都退出后再返回。在标准库中，有一个``WaitGroup``结构体就可以专门解决等待多个goroutine的问题。在此不详述。

接下来，就是为每个用户连接开启一个goroutine，执行``ServeClient``函数。前面已经说过，由于goroutine是一种比线程还轻量的调度单位，如此数目的goroutine并不会带来严重的性能下降。

由于goroutine和消息机制简化了开发，并且Go也鼓励这样的设计，开发者会自觉地选择基于多个goroutine的设计。由此带来的另一个好处，就是程序在多核系统上的扩展性。随着处理器核数量的增加，如何发掘程序内在的并行结构成了当前开发人员面临的很大挑战。而使用Go编写，基于多个goroutine的设计，往往会天生具备着足够的并行结构来扩展到多核处理器之上。每个goroutine实际都是可以放在一个独立的处理器上，与其他goroutine并行执行。也就是说，今天为四核处理器写的代码，也许不必修改，就可以运行在未来128核的CPU上，并且同时使用所有的核。

# 无需配置，直接编译

> *如果Go需要一个配置文件，描述如何编译和构建Go写的程序，那就是Go的失败。*
> --- Go官方文档

对于make，autoconf，automake等用于指定编译顺序和依赖关系的工具，Go的态度是：开发者在写代码的时候，就留下了关于依赖的足够信息，不该要求开发者再单独写一份配置文件，去指明依赖关系和编译顺序。为此，开发者只需要在安装go工具链之后，按照官方文档，配置好一个目录结构和一个环境变量即可。以后任何安装Go程序，编译任何Go程序/库都只需要几条简单的命令就可以了。

对于一个自包含（不依赖任何第三方库）的程序，只需要在当前目录下运行``go build``就会编译好整个程序。

如果我的程序依赖第三方库，又该如何呢？很简单，在代码中的import语句里，写入第三方库的在网络中的位置即可。这里的import和Java/Python中的import的概念一样，都是引入一个包。

    :::go
    import (
        "fmt"
        "github.com/monnand/goredis"
    )
    
import中引入的第一个包，是fmt，这是标准库中的包，提供Printf一类的格式化输入和输出。第二个引入的包则是位于github上的代码库。它会引入github上，用户monnand下，goredis这个项目定义的包。

接下来，再调用go命令安装这个库：

    go get github.com/monnand/goredis
    
这样，go程序就会自动下载，编译和安装这个库（包括它的依赖）。接下来再使用``go build``编译依赖goredis的程序。

除此以外，如果依赖goredis的程序也在github（或其他go支持的版本控制库）中，那么只用一条``go get``命令指明该程序所在的远程地址就足够了，go会自己下载安装各种依赖。除了github，go还支持google code，BitBucket，Launchpad，或者是任何位于其他服务器上，使用svn，git，Bazzar，Mercurial做版本控制的Go程序/库。这一切都极大地简化了开发人员和最终用户的操作。

# 再谈运行效率

> - *Matt: 使用Pat/Go后，比起（原来的）Sinatra/Ruby方案，JSON API节点效率提升了多少？给个估计就可以。*
> - *Blake: 大约10,000倍*
> - *Matt: 漂亮！我能引述你的话吗？*
> - *Blake: 我再查查，我觉得好像低估了。*
>
> --- Matt Aimonetti与Blake Mizerany在推特上的对话。

Go程序的运行效率一直是人们关注的焦点。一方面，Go的语法，类型系统都非常简单，为编译器的开发和优化提供了很大空间。另一方面，Go作为静态编译型语言，代码直接编译为机器码，无需中间解释。

不过倘若在网上搜索一下，就会发现关于Go程序的运行效率，存在着严重的两极分化。一部分测试显示，Go的程序运行效率非常高，甚至一些方面超过了C++写的同等程序。另一部分测试则现实，某些方面，Go甚至不如Stackless Python写的脚本。

Go编译器本身虽然还存在很大优化空间，但产生的机器码效率已经比较高。而标准库 -- 其中包括各种运行时代码，比如垃圾回收，哈希表等 -- 则还没有怎么优化，甚至有些还处于很初级的阶段。这是网络上的测试结果存在着严重差异的原因之一。另外，作为一个新的语言，开发人员由于对它不熟悉，写出的代码可能存在性能瓶颈，也加大了评测结果的差异。

Go语言的开发者之一，Russ Cox曾在Go的官方博客上发表了[一篇文章](http://blog.golang.org/2011/06/profiling-go-programs.html)。其中使用了某基准测试程序（Benchmark）的代码，分别优化了其中的C++测试和Go测试部分。优化后的Go程序运行时间，甚至仅仅是优化后的C++程序运行时间的65.8%！这也从一个侧面反应出了Go的潜力。

当前Go语言中，还存在不少缺陷：垃圾回收还处于比较初级的阶段，而且对于32位系统的支持还不太完善，一些标准库的代码还有待优化。按照Go官方的说法，未来将会使用完全并行的垃圾回收器，这对于性能来说将会有很大的提高。而随着Go 1的发布，Go开发组也会将精力从语法和标准库的规范，转移到对编译器和标准库的优化上。Go程序的运行效率，目标将会是逼近C++，超越Java。

# 总结

> *现在来说，我觉得在系统级开发方面，它（Go）比C++要好上许多。使用它开发更高效，并能使用比C++更简单的方式解决很多问题。*---- Bruce Eckel, 《C++编程思想》《Java编程思想》作者

Unix创始人Ken Thonpson；UNIX/Plan 9开发者Rob Pike，Russ Cox；memcached作者Brad Fitzpatrick；Java Hotspot编译器作者之一，Chrome V8引擎作者之一Robert Griesemer；Gold连接器作者，GCC社区活跃开发人员Ian LanceTaylor……当这样一群人凑在一起，无论开发什么，这团队本身也许已经足以吸引众人眼球了。而Go作为这样一个团队开发出的语言，目前为止还是给不少人带来了惊喜。

已经有很多公司使用Go开发生产级程序。Rob Pike曾透露过Google内部正逐渐开始使用Go。YouTube则使用Go编写核心部件，并且将部分代码组织成了开源项目vitess。国内包括豆瓣，QBox等公司也已经率先踏入Go语言这个领域。

随着Go 1的推出，一个稳定的Go语言平台和开源社区已经形成。对于喜欢尝试新鲜语言的开发者，Go不失为一个选择。

[CSP]: http://en.wikipedia.org/wiki/Communicating_sequential_processes
[Occam]: http://en.wikipedia.org/wiki/Occam_%28programming_language%29
[Erlang]: http://erlang.org
[Newsqueak]: http://en.wikipedia.org/wiki/Newsqueak
[Squeak]: http://doc.cat-v.org/bell_labs/squeak/
[Alef]: http://en.wikipedia.org/wiki/Alef_%28programming_language%29
[Limbo]: http://en.wikipedia.org/wiki/Limbo_%28programming_language%29
[Go]: http://golang.org

