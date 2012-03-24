<%--
Author: monnand
Key: /home/monnand/.ssh/monnand.pem
Title: 各语言中哈希表实现的性能分析
Slug: hashtable-bench
Language: zhCN
Tags: ["golang", "golang map", "HashMap", "unordered_map", "hash table"]
--%>

上一篇[文章](http://monnand.me/p/golang-map-bench/zhCN)中对go语言的map做出了分析，并且（错误地）推断go中的map实际为树。之后在[golang-nuts](https://groups.google.com/forum/?fromgroups#!topic/golang-nuts/HDz5KiG6oMY)上被人指出，我所观测到的速度降低，是由于缓存缺失（cache miss）导致。不过那个对数变化，其实我说对了（少）一半：go的map是一个hash tree，所以在元素数量变多的时候，的确会表现出一些树的特性（对数变化）。这也部分地解释了为什么我会观察到访问时间成对数增长。

这篇文章，旨在继续对其它语言，或语言标准库，或语言的准标准库中的哈希表实现进行一个性能分析。

#测试方法#

既然之前发现是缓存命中率的原因导致哈希表访问速度不一致了，那么这回就必须要考虑到缓存命中率的问题。既然是公平比较，那么就有两个方法：

- 提高所有语言的缓存命中率，当命中率足够高的时候，缓存缺失（cache miss）带来的代价就可以忽略不计。由此测出的每个访问操作时间将是一个常数，这个常数大约就是哈希表（在缓存命中的情况下）的访问时间。
- 降低所有语言的缓存命中率，当命中率足够低的时候，此时缓存缺失的代价占操作时间的主要部分。测出的访问时间将会随着元素数量逐渐增加。这时候测出访问时间，是哈希表在缓存缺失情况下的访问时间。

提高命中率，可以通过按照哈希表key存储的自然序列访问哈希表。这样相当于顺序访问一个数组，但每次访问key对应的value时，根据key重新计算哈希值并找出value，而不是直接取出value。

降低命中率，可以采用以下两种方法：

- 随机访问哈希表。把哈希表中的所有key存在一个数组里，然后随机打乱数组元素的顺序，最后按照打乱顺序后的数组，访问哈希表。
- 先找出所有key在哈希表中的自然序，然后把这些key重新排列，使得缓存命中率最小。

第一种方法是实现简单，但每次运行缓存命中率未必保持一样。不过只要随机数生成得不是太差，命中率应该会保证在一定水平之上且比较稳定。第二中方式的优点是可以最小化缓存命中率，但缺点是实现困难，而且要根据不同体系结构，缓存策略，设计不同算法。所以我只实现了第一种。至于第二种，有兴趣的朋友可以考虑一下。

这次测试分别使用了go中的map，python中的dict，C++中的[unordered_map](http://www.cplusplus.com/reference/stl/unordered_map/)和java中的HashMap进行测试。由于代码较多且实现简单，我就不在文中贴出了。但在[github](https://github.com/monnand/hashtable-bench)上可以找到全部的完整代码。

#背景知识#

由于涉及到缓存命中问题，有必要先普及一下缓存的一些常识。

##内存平均访问时间##
内存平均访问时间（Average Memory Access Time, AMAT）大约可以通过以下公式计算得出：

AMAT = hit time + cache miss rate \* cache miss penalty

- hit time等于直接从缓存中取出数据所用的时间
- cache miss rate等于缓存缺失次数除以全部内存访问次数。即缺失率
- cache miss penalty等于数据从内存/下一级缓存到当前缓存所需时间

在现在的计算机体系结构下，内存速度相对cache/CPU速度非常慢，导致cache miss penalty >> hit time，所以缓存缺失率就会在很大程度上影响程序运行的效率。

##缓存缺失的几种类型##

根据缓存缺失的原因，缓存缺失可能分别由以下几个类型：

- 强制缺失（Compulsory Miss）：任何数据在未被载入缓存前都会造成缓存缺失，必须先把数据从内存/上一级缓存载入当前缓存，才能继续工作。由此带来的缺失，称为强制缺失
- 容量缺失（Capacity Miss）：由于缓存容量小于内存/上一级缓存，使得数据不能全部载入缓存，由此带来的缺失，叫做容量缺失
- 冲突缺失（Conflict Miss）：缓存中，每个cache line（缓存中数据访问的最小单位）会对应多处内存。之前的内存访问刷新了cache line，由此导致的缺失，成为冲突缺失

关于缓存的详细内容，可以参考[wikipedia](http://en.wikipedia.org/wiki/CPU_cache)。我不多说了。看数据吧。

#顺序访问#

前面说了，这种访问方式的目的，是最大化缓存命中率，这样得到的结果，将会约等于一个常数，并且该常数正比于哈希表的访问时间。

先看看Go的结果：
<pre>
Map Seq. Access. N = 10: 0.4000000000000001 us/op
Map Seq. Access. N = 100: 0.14 us/op
Map Seq. Access. N = 1000: 0.12300000000000001 us/op
Map Seq. Access. N = 10000: 0.119 us/op
Map Seq. Access. N = 100000: 0.15627000000000002 us/op
Map Seq. Access. N = 1000000: 0.17811000000000002 us/op
Map Seq. Access. N = 10000000: 0.2497328 us/op
</pre>
刚刚一开始，由于元素数量少，使得强制缺失带来的访问延迟成为主导因素。所以访问速度比较慢。之后基本稳定在0.1x us/op上。继续增加元素，由于缓存命中率降低，访问速度则逐渐变慢。其他语言的结果也基本都呈现出这样的趋势。

下面是Python的：
<pre>
Dict Seq. Access. N = 10 :  4.6 us/op
Dict Seq. Access. N = 100 :  0.51 us/op
Dict Seq. Access. N = 1000 :  0.143 us/op
Dict Seq. Access. N = 10000 :  0.1143 us/op
Dict Seq. Access. N = 100000 :  0.26562 us/op
Dict Seq. Access. N = 1000000 :  0.331683 us/op
Dict Seq. Access. N = 10000000 :  0.3797068 us/op
</pre>
python的结果比go略差，只在N=10000的时候比同级别的go强出0.005 us/op。

下面是C++的：
<pre>
Unordered Map Seq. Access. N = 10. 0.2 us/op.
Unordered Map Seq. Access. N = 100. 0.14 us/op.
Unordered Map Seq. Access. N = 1000. 0.166 us/op.
Unordered Map Seq. Access. N = 10000. 0.1946 us/op.
Unordered Map Seq. Access. N = 100000. 0.36684 us/op.
Unordered Map Seq. Access. N = 1000000. 0.392766 us/op.
Unordered Map Seq. Access. N = 10000000. 0.471303 us/op.
</pre>
在元素数量少的时候，基本与go持平，但随着元素数量增加，效率则不如go。

最后是Java：
<pre>
HashMap Seq. Access. N = 10: 8.751800 us/op.
HashMap Seq. Access. N = 100: 1.613250 us/op.
HashMap Seq. Access. N = 1000: 0.815325 us/op.
HashMap Seq. Access. N = 10000: 1.555751 us/op.
HashMap Seq. Access. N = 100000: 0.219468 us/op.
HashMap Seq. Access. N = 1000000: 0.073412 us/op.
HashMap Seq. Access. N = 10000000: 0.061496 us/op.
</pre>
与其他语言的结果不同，java的访问时间，在最后增加了一步之后，竟然逐渐下降。目前我还没有一个比较满意的解释，当前我能给出的推断是：随着元素的增多，Java虚拟机有了足够的时间学习到了循环的行为（这毕竟是个非常简单的循环），由此通过乱序执行，提高了缓存命中率，并最终提高了效率。是否通过乱序执行我不知道，但我个人感觉应该是和java虚拟机脱不开干系的。否则也确实解释不了这个现象。之后的随机访问结果，也出现了类似现象。

最后总结一下，在顺序访问中，Go的表现还算令人满意，访问时间还算维持在一个常数上下。参见下图。

![Hash Table Access Time (Sequential Access)](http://monnand.me/media/img/hashtable-seq-access.png)

#随机访问#
还是继续，先看Go的：
<pre>
Map Random Access. N = 10: 0.3 us/op
Map Random Access. N = 100: 0.11000000000000001 us/op
Map Random Access. N = 1000: 0.087 us/op
Map Random Access. N = 10000: 0.1141 us/op
Map Random Access. N = 100000: 0.38335 us/op
Map Random Access. N = 1000000: 0.535501 us/op
Map Random Access. N = 10000000: 0.6896599 us/op
</pre>

然后是Python：
<pre>
Dict Random Access. N = 10 :  4.9 us/op
Dict Random Access. N = 100 :  0.54 us/op
Dict Random Access. N = 1000 :  0.13 us/op
Dict Random Access. N = 10000 :  0.121 us/op
Dict Random Access. N = 100000 :  0.40927 us/op
Dict Random Access. N = 1000000 :  0.497939 us/op
Dict Random Access. N = 10000000 :  0.6353964 us/op
</pre>
这里有必要说一下。Python的dict实现，是在一片连续内存上存储的哈希表。也就是经典的哈希表实现方法。这样，计算出每个key的哈希值后，实际就相当于一个数组访问。但这有一个缺点：当哈希表元素数量不断增加时，需要频繁地释放和再申请整片内存。这样会对性能有影响。在我们的测试中，则没有考虑到这部分。而Go的实现，是采用树结构的哈希表，这样每次增加元素，最多只需要重新申请和复制一部分内存。但这样带来的代价，就是访问速度的增加。一个缓解的方式就是增加树节点的度数，从而降低树的高度。从这个测试中，我们可以看到，当节点数量增加到一定程度，Go的劣势就显现出来了：由于树结构的深度增加，导致每次访问时多次缓存缺失，从而使得访问速度增加。

下面是C++的
<pre>
Unordered Map Random Access. N = 10. 0.4 us/op.
Unordered Map Random Access. N = 100. 0.16 us/op.
Unordered Map Random Access. N = 1000. 0.168 us/op.
Unordered Map Random Access. N = 10000. 0.2055 us/op.
Unordered Map Random Access. N = 100000. 0.54728 us/op.
Unordered Map Random Access. N = 1000000. 0.690345 us/op.
Unordered Map Random Access. N = 10000000. 0.743512 us/op.
</pre>

最后，是Java：
<pre>
HashMap Random Access. N = 10: 0.803400 us/op.
HashMap Random Access. N = 100: 0.307150 us/op.
HashMap Random Access. N = 1000: 0.363706 us/op.
HashMap Random Access. N = 10000: 0.633317 us/op.
HashMap Random Access. N = 100000: 0.298482 us/op.
HashMap Random Access. N = 1000000: 0.160155 us/op.
HashMap Random Access. N = 10000000: 0.178708 us/op.
</pre>
可以看到，继续出现了类似前面的情况，即最后访问速度反而提高了。不过有一点必须提，不是我黑Java，元素增加之后，访问速度的确快了，但是插入速度和CPU使用率实在不敢恭维。如果算上插入速度，可能Java反而是最慢的。

最后，把结果再画在图上：

![Hash Table Access Time (Random Access)](http://monnand.me/media/img/hashtable-rand-access.png)

#总结#
基本说来，关于Go中map是否是性能杀手的问题：如果你能忍受C++的unordered\_map，python中的dict以及元素数在100000以内的Java中的HashMap，那么Go中的map就应该还能接受。

另外还有一点让我意外的，就是Java在元素数庞大时的效率。如果你有很多很多的元素，而且数量不会增加（Java中插入元素的效率比较低），那么采用Java的HashMap应该不错。关于这部分，可以再做一些benchmark。

#后续讨论#
我把结果发到[golang-china](https://groups.google.com/d/topic/golang-china/BGvYJu_vHQI/discussion)上，有些朋友给出了更多的测试结果或建议。

##Java的startup##
lihui指出：

	不能用java的小数量的条目来恒定它的性能，因为在测试环
	境中，小数量的条目不能让它热起来，而在实际环境中，由
	于hashmap会在程序中多处使用，并且会循环调用使用过
	hashmap的历程，因此在实际中hashmap必然是热的，从而对
	小数目的hashmap而言，性能仍然是处于最优化的状态。

我的回复：

	为此，我又对java做了一个测试。测试方法是：在计时开始
	前，先调用一遍循环取得所有元素。这样做对于元素多的哈希
	表，会增加conflict miss和capacity miss。但对于小的哈希
	表，可以完全避免compulsory miss。简单说，这个循环先让
	cache预热起来。

	我没有测试N=1E7的时候，因为N=1E7的时候java实在太耗费时
	间了。我前面说了，可能是我插入部分的代码没有做任何性能
	考虑的缘故，但毕竟这部分不计时，也就没关心。

	以下是Java预热启动的结果：
	HashMap Random Access. N = 10: 0.362500 us/op.
	HashMap Random Access. N = 100: 0.254190 us/op.
	HashMap Random Access. N = 1000: 0.219619 us/op.
	HashMap Random Access. N = 10000: 0.639815 us/op.
	HashMap Random Access. N = 100000: 0.244724 us/op.
	HashMap Random Access. N = 1000000: 0.159392 us/op.

	作为对比，以下是Go冷启动的结果（没有cache预热）：
	Map Random Access. N = 10: 0.3 us/op
	Map Random Access. N = 100: 0.14 us/op
	Map Random Access. N = 1000: 0.08800000000000001 us/op
	Map Random Access. N = 10000: 0.1157 us/op
	Map Random Access. N = 100000: 0.46925 us/op
	Map Random Access. N = 1000000: 0.520184 us/op

	个人认为，Java慢的原因还是和JVM脱不开干系。成也萧何，败
	也萧何。小数据量的时候，JVM本身的overhead占了主流。大数
	据量的时候，JVM的预测，乱序执行和prefetch发挥了作用。

	当然，以上纯粹我的推测，还请了解JVM的朋友们帮忙指正。

##key值对哈希表的影响##

laputa指出：

	测试结果，除了和这两者有关：
	1、key的类型（int、string或其它）
	2、key的遍历顺序（自然序还是随机序）

	还和key序列的特征有关。

	如果把key进行md5，然后取其中9位，Python的测试结果如下。

	Dict Seq. Access. N = 10 :  0.882148742676 us/op
	Dict Seq. Access. N = 100 :  0.340938568115 us/op
	Dict Seq. Access. N = 1000 :  0.312805175781 us/op
	Dict Seq. Access. N = 10000 :  0.339293479919 us/op
	Dict Seq. Access. N = 100000 :  0.623297691345 us/op
	Dict Seq. Access. N = 1000000 :  0.772630929947 us/op
	Dict Seq. Access. N = 10000000 :  0.986476206779 us/op

	这前后差异的原因是python的dict采用open addressing实现。

	Go 的结果如下，变化较小：
	Map Seq. Access. N = 10: 0.8000000000000002 us/op
	Map Seq. Access. N = 100: 0.3 us/op
	Map Seq. Access. N = 1000: 0.239 us/op
	Map Seq. Access. N = 10000: 0.29430000000000006 us/op
	Map Seq. Access. N = 100000: 0.37426 us/op
	Map Seq. Access. N = 1000000: 0.371087 us/op
	Map Seq. Access. N = 10000000: 0.4287918 us/op

	不知道Java的结果跟此有没有关系~

	btw，之前忘了说，测试的环境是Linux
	Linux 2.6.29.3-server-1mnb SMP  i686 AMD Opteron(tm) Processor 246 GNU/Linux

我的回复是：

	个人认为，这取决于哈希函数。取过MD5 
	后，相当于 --- 虽然不能完全等效于 --- 把key做了随机化。这样哈希值相差也可能比 
	较大。而此时Go的tree strucutre优势就体现出来了。


