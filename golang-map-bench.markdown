<%--
Author: monnand
Key: /home/monnand/.ssh/monnand.pem
Title: Go语言中的map的性能分析
Slug: golang-map-bench
Language: zhCN
Tags: ["golang", "golang map"]
--%>

[Go语言](http://golang.org)提供了[map类型](http://weekly.golang.org/ref/spec#Map_types)。该类型与python中的dict，perl中的hash类似，都是可以以“任意”（但需要对这些类型有一定限制）类型作为索引键值，来取得对应数据。这样可以方便地存储Key-Value对。虽然C++的STL，java的标准库，也都提供类似的类型，但作为一个编译型静态的imperative语言，在语法层面上支持该类型，至少对我来说还是第一次见到。但map被很多人认为是性能杀手，这可能一部分归咎于Go官方博客上的[一篇文章](http://blog.golang.org/2011/06/profiling-go-programs.html)。本文重点是希望利用几个自己写的简单的benchmark，来比较go语言的map和其他语言中的对应实现。

*注意*：本文内容已被[另外一篇](http://monnand.me/p/hashtable-bench/zhCN/)更新，请务必读过[该文](http://monnand.me/p/hashtable-bench/zhCN/)后再做判断。

#Go中使用map#

由于是在语法层面的支持，所以go的map很好使用。我就用我的benchmark代码，来大概说说go中map的使用：

	:::go
	package main

	import (
		"fmt"
		"runtime"
		"time"
	)

	func SeqGetMapBenchmark(N int, gc bool) {
		if gc {
			runtime.GC()
		}
		// 创建一个map类型，以字符串为键（key），整数为值（value）
		a := make(map[string]int, N)
		// 创建一个slice，用于存储访问顺序
		seq := make([]string, N)

		// Insert numbers
		for i := 0; i < N; i++ {
			key := fmt.Sprintf("%09d", i)
			a[key] = i
			seq[i] = key
		}

		sum := 0

		// 开始计时
		start := time.Now()
		for _, key := range seq {
			sum += a[key]
		}
		duration := time.Since(start)

		fmt.Printf("Map Seq. Access. iterations: %v. %v us/op\n",
			N, duration.Seconds() * 1e6/float64(N))
	}

	func main() {
		gc := true
		SeqGetMapBenchmark(1000, gc)
		SeqGetMapBenchmark(10000, gc)
		SeqGetMapBenchmark(100000, gc)
		SeqGetMapBenchmark(1000000, gc)
		SeqGetMapBenchmark(10000000, gc)
	}

啥也不说了。东西都在注释里面。如果想详细了解Go，网上有很多[文档](http://weekly.golang.org/doc/)。不多说了。

这个benchmark很简单：插入N个数到map中，将该数字的10进制表示的字符串作为key，前面补零以保证没个key长度都为9个字符。按照数字顺序插入map中，每个key对应的value是相应的数字的值。最后，按照插入顺序依次取出每个key和对应值，计算依次取出全部内容所需时间，除以N，得到平均每次取操作所花费的时间。N的值分别去1000， 10000， 100000，1000000和10000000。

#与Python中的dict做对比#

以上等价的python程序如下：

	:::python
    import datetime

    def seq_get_dict_benchmark(n):
        a = {}
        seq = []

        for i in xrange(n):
            key = "%09d" % i
            a[key] = i
            seq.append(key)

        start = datetime.datetime.now()

        s = 0
        for key in seq:
            s += a[key]

        duration = datetime.datetime.now() - start

        print "Dict Seq. Access. iterations: ", \
            n, ". ", duration.total_seconds() * 1e6/n, "us/op"

    if __name__ == "__main__":
        seq_get_dict_benchmark(1000)
        seq_get_dict_benchmark(10000)
        seq_get_dict_benchmark(100000)
        seq_get_dict_benchmark(1000000)
        seq_get_dict_benchmark(10000000)

分别运行go和python的代码，得到以下结果：

- Go的运行结果（编译器使用6g version weekly.2012-03-13）
<pre>
Map Seq. Access. iterations: 1000. 0.08400000000000002 us/op
Map Seq. Access. iterations: 10000. 0.1177 us/op
Map Seq. Access. iterations: 100000. 0.26981 us/op
Map Seq. Access. iterations: 1000000. 0.390944 us/op
Map Seq. Access. iterations: 10000000. 0.4742399 us/op
</pre>

- Python运行结果（解释器版本为2.7.2）
<pre>
Dict Seq. Access. iterations:  1000 .  0.123 us/op
Dict Seq. Access. iterations:  10000 .  0.1137 us/op
Dict Seq. Access. iterations:  100000 .  0.14328 us/op
Dict Seq. Access. iterations:  1000000 .  0.163299 us/op
Dict Seq. Access. iterations:  10000000 .  0.1822816 us/op
</pre>

从这里我们可以看到，在map中有1000个元素的时候，go的性能是比python强的。而当增加到10000个的时候，go就与python持平。当继续增加元素个数的时候，python的优势则非常明显了。

好了，我们现在的结论是：Go的map不如python的dict。如果带着仇恨说这句话，就是：Go作为一个编译型语言，它的map的效率甚至还不如python这样的脚本语言！完毕！

等等！这就完了？显然没有！

#Hash?Map?#

数感强的同学一定发现了，go的map访问时间是随着map中元素个数而以log(N)的速度增长的。可哈希表访问时间应该是O(1)啊，怎么成了log(N)了？我们从python的运行结果就能看出来，无论元素怎么增长，平均访问时间数量级几乎没变化。而为什么go的访问时间会有log(N)呢？

作为一个计算机系的学生，看到运行时间是log(N)，必须要有一个第一直觉反应：*树*。

没错，树这个数据结构常常被用于搜索，因为树的搜索复杂度与树的深度成正比，而平衡的树深度则是log(N)。

关于这部分，我还没有十足的把握，因为从官方的资料上看，我似乎得到了两个结论：

- 官方的[语言规范](http://weekly.golang.org/ref/spec)中，对于[map](http://weekly.golang.org/ref/spec#Map_types)有这样一段说法：
<pre>
The comparison operators == and != ... must be fully
defined for operands of the key type
</pre>
这一点似乎印证了map的实现是树的说法。因为如果是哈希表，那么能作为key的类型则必须在其上定义哈希函数。这在python中显然是[明确说明](http://wiki.python.org/moin/DictionaryKeys)的。倘若仅仅定义等于和不等于的操作，那么我能想到的有效搜索结构，就只剩下树了。

- 官方的[源代码](http://code.google.com/p/go/source/browse/)中，在src/pkg/runtime下，的确有[hashmap.c](http://code.google.com/p/go/source/browse/src/pkg/runtime/hashmap.c)。如果是树结构，为什么又叫hash呢？我个人觉得这应该纯粹是命名方式问题，因为运行结果和语言规范同时暗示着我这不可能是个哈希表，所以我也没读代码。

由此，我只能假设，map实际并非是一个哈希表，而是一个树结构。倘若如此，那么将一个树结构实现与一个哈希表实现做访问时间上的对比，就显得不公平了。

#STL中的map#

得到这样的结论之后，我开始着手写第三个benchmark。既然假设它是树了，那么第三个benchmark显然不能使用哈希表的结构。C++的STL中同时提供了哈希表和树的搜索结构，对应的树搜索结构是map。由此，使用C++的STL中的map，得到如下基准测试程序：


	:::cpp
	#include <map>
	#include <list>
	#include <iostream>
	#include <string>
	#include <cstdio>
	#include <cstring>
	#include <sys/time.h>
	#include <unistd.h>

	using namespace std;

	void seq_get_map_benchmark(int N)
	{
		map<string, int> a;
		list<string> seq;
		list<string>::iterator it;
		int i;
		int sum;
		char buf[10];
		struct timeval start, end;
		long seconds, useconds;
		float mtime;

		for (i = 0; i < N; i++) {
			memset(buf, 0, sizeof(buf));
			snprintf(buf, sizeof(buf), "%09d", i);
			a[buf] = i;
			seq.push_back(buf);
		}


		sum = 0;
		gettimeofday(&start, NULL);

		for (it = seq.begin(); it != seq.end(); it++) {
			sum += a[*it];
		}
		gettimeofday(&end, NULL);

		seconds  = end.tv_sec  - start.tv_sec;
		useconds = end.tv_usec - start.tv_usec;

		mtime = seconds * 1e6 + useconds;
		cout << "Map Seq. Access. iterations: " 
			<< N << ". " << mtime/N << "us/op" << endl;
	}

	int main()
	{
		seq_get_map_benchmark(1000);
		seq_get_map_benchmark(10000);
		seq_get_map_benchmark(100000);
		seq_get_map_benchmark(1000000);
		seq_get_map_benchmark(10000000);
	}

以下是C++的运行结果（使用g++ 4.6.2，-O3编译）：
<pre>
Map Seq. Access. iterations: 1000. 0.143us/op
Map Seq. Access. iterations: 10000. 0.2117us/op
Map Seq. Access. iterations: 100000. 0.33286us/op
Map Seq. Access. iterations: 1000000. 0.559903us/op
Map Seq. Access. iterations: 10000000. 0.731576us/op
</pre>

为了方便对比，以下再次拿出go的测试结果：
<pre>
Map Seq. Access. iterations: 1000. 0.08400000000000002 us/op
Map Seq. Access. iterations: 10000. 0.1177 us/op
Map Seq. Access. iterations: 100000. 0.26981 us/op
Map Seq. Access. iterations: 1000000. 0.390944 us/op
Map Seq. Access. iterations: 10000000. 0.4742399 us/op
</pre>

这样看基本一致。单纯从结果看，go似乎比C++还要强点。不过这还要考虑到STL中的模版，泛型等带来的额外消耗，以及由此带来的难以优化的代码。

#阶段性总结#

我想现在得出任何结论还为时尚早。但可以初步给出的结论是：go的map实现似乎不是一个哈希表结构，而是一个树结构。

我当初一直以为go中的map是一个哈希表，因为在官方博客的[这篇文章](http://blog.golang.org/2011/06/profiling-go-programs.html)中，的确明明白白地看到了hash\_loopup这个函数。于是推断性能之所以慢，应该是哈希表的时间复杂度常数部分太大的缘故。从当前的测试结果看，我想应该不太可能是哈希表了。

为此，如果有朋友想写java的基准测试，那么最好使用[TreeMap](http://docs.oracle.com/javase/6/docs/api/java/util/TreeMap.html)，而非[HashMap](http://docs.oracle.com/javase/6/docs/api/java/util/HashMap.html)或者[HashTable](http://docs.oracle.com/javase/6/docs/api/java/util/Hashtable.html)。

最后，由于运行结果和语言规范让我非常倾向于树结构这个结论，所以我也没读go的代码。这使得这个结论还缺乏一定说服力。欢迎各位读一下go的代码，看看map究竟是怎么实现的。可能的话把结论告诉我。

#为什么是树?#

这一节大概说说使用树结构相对哈希表的优点吧：

- 哈希表的最坏访问时间是O(N)。而倘若哈希表中不引入足够的随机因素，很容易引来[攻击](http://www.kb.cert.org/vuls/id/903934)。攻击的方式很简单：不断产生会生成同一个哈希值的key，使得整个哈希表永远（或大多数时间）工作在最坏情况下。相比而言，树结构则不必引入随机函数，就可以在很多情况下保证log(N)这一搜索时间复杂度。而这个复杂度，对于大多数应用也基本足够了。
- 哈希表需要保证一定的负载系数，简单说，就是总要留下一些空位。由于每次在负载过多的时候都需要重新申请一片连续内存存储以前和新来的数据，由此带来的内存申请和释放，可能会给垃圾回收工作带来额外工作。而树结构只需要为插入的数据申请节点。这就简化了垃圾回收的任务。
- 哈希表要求必须为作为key的类型定义哈希函数。这样的要求就比树更严格了。最后，哪怕所有哈希表访问都是O(1)的复杂度，即常数时间复杂度，但如果这个常数很大，那么也只有在元素数目多过一定数值后，才能体现出优势。实际上，对于少量元素的存储，哈希表未必是最优解（因为可能存在一个树结构，在N较小时，访问时间比哈希表快）。

必须提醒一句：任何一个数据结构都有其各自的优缺点。使用何种数据结构与实际的使用场合有极大联系。以上的分析并非是为go开脱，只是陈述使用树结构相对哈希表而言有哪些优点。我同样也可以说哈希表实现相对树结构的优点。比如平均访问时间是常数等等。

从我当前测试的结果上看，把map中元素的个数限制在十万以内，访问时间应该在大家可接受的范围。

#后记#

好吧，我快崩溃了，扫了一眼[hashmap.c](http://code.google.com/p/go/source/browse/src/pkg/runtime/hashmap.c)的代码，那俨然就是一个哈希表啊。

已经把问题发到[golang-nuts](https://groups.google.com/forum/?fromgroups#!topic/golang-nuts/HDz5KiG6oMY)了。期待有人解答。

*注意*：本文内容已被[另外一篇](http://monnand.me/p/hashtable-bench/zhCN/)更新，请务必读过[该文](http://monnand.me/p/hashtable-bench/zhCN/)后再做判断。
