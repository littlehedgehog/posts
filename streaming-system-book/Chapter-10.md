# 大规模数据处理的演化历程

终于熬到本书最后一章节了，可喜可贺! 关于流式系统学习过程即将结束。在本书最后一章，我打算和你一起浏览下大数据的发展史，我们从最开始MapReduce计算模型开始，一路下来看大数据这十五年关键发展变化，同时也顺便会讲解流式处理这个领域是如何发展到今天的这幅模样。这一章节阅读起来显然较之前章节略显轻松，这其中我也会加入一些我对一些业界知名大数据处理系统(可能里面有些也不那么出名)的观察和评论，同时考虑到我很有可能简化、低估甚至于忽略了很多重要的大数据处理系统，我也会附带一些参考材料帮助大家学习更多更详细的知识。

另外一方面，在阅读本章内容时请务必注意，我们仅仅讨论了大数据处理中偏MapReduce/Hadoop系统及其派系分支的大数据处理。我没有讨论任何SQL引擎[1]，我们同样也没有讨论HPC或者超级计算机。尽管我这章的标题听上去领域覆盖非常广泛，但实际上我仅仅会讨论一个相对比较垂直的大数据领域。

[1] 这意味着，我会跳过大量有关流处理的学术文献，因为这部分内容浩如烟海实在太多。如果你确实对这批学术论文感兴趣，你可以从[《The DataFlow Model》](http://www.vldb.org/pvldb/vol8/p1792-Akidau.pdf)那篇论文开始学习。相信你应该很容易能够找到你顺手的学习方式。

同样需要提醒的一件事情是，我在本文里面或多或少会提到一些Google的技术，不用说这块是因为与我在谷歌工作了十多年的经历有关。 但还有另外两个原因：1）大数据对谷歌来说一直很重要，因此在那里创造了许多有价值的东西值得详细讨论，2）我的经验一直是 谷歌以外的人似乎更喜欢学习Google所做的事情，因为Google公司在这方面一直有点守口如瓶。 所以，当我过分关注我们一直在"闭门造车"的东西时，姑且容忍下我吧。

![10-1](https://github.com/littlehedgehog/posts/blob/master/streaming-system-book/images/stsy_1001.png)

为了使我们这一次大数据旅行显得更加具体有条理，我们设计了图10-1的时间表，这张时间表概括地展示了不同系统的诞生日期。

在每一个系统介绍过程中，我会尽可能说明清楚该系统的简要历史，并且我会尝试从流式处理系统的演化角度来阐释该系统对演化过程的贡献。最后，我们将回顾以上系统所有的贡献，从而全面了解上述系统如何演化并构建出现代流式处理系统的。

# MapReduce

我们从MapReduce开始我们的旅程。

![10-2](https://github.com/littlehedgehog/posts/blob/master/streaming-system-book/images/stsy_1002.png)

我认为我们可以很确定地说，今天我们讨论的大规模数据处理系统都源自于2003年MapReduce。当时，谷歌的工程师正在构建各种定制化系统，以解决互联网时代下大数据处理难题。当他们这样尝试去解决这些问题时候，发现有三个难以逾越的坎儿：

* 数据处理很难
  只要是数据科学家或者工程师都很清楚。如果你能够精通于从原始数据挖掘出对企业有价值的信息，那这个技能能够保你这辈子吃喝不愁。

* 可伸缩性很难
  本来数据处理已经够难了，要从大规模数据集中挖掘出有价值的数据更加困难。

* 容错很难
  要从大规模数据集挖掘数据已经很难了，如果还要想办法在一批廉价机器构建的分布式集群上可容错地、准确地方式挖掘数据价值，那真是难于上青天了。

在多种应用场景中都尝试解决了上述三个问题之后，Google的工程师们开始注意到各自构建的定制化系统之间颇有相似之处。最终，Google工程师悟出来一个道理: 如果他们能够构建一个可以解决上述问题二和问题三的框架，那么工程师就将可以完全放下问题二和三，从而集中精力解决每个业务都需要解决的问题一。于是，MapReduce框架诞生了。

MapReduce的基本思想是提供一套非常简洁的数据处理API，这套API来自于函数式编程领域的两个非常易于理解的操作：map和reduce（图10-3）。使用该API构建的底层数据流将在这套分布式系统框架上执行，框架负责处理所有繁琐的可扩展性和容错性问题。可扩展性和容错性问题对于分布式底层工程师来说无疑是非常有挑战的课题，但对于我们普通工程师而言，无益于是灾难。

![10-3](https://github.com/littlehedgehog/posts/blob/master/streaming-system-book/images/stsy_1003.png)

我们已经在第6章详细讨论了MapReduce的语义，所以我们在此不再赘述。仅仅简单地回想一下，我们将处理过程分解为六个离散阶段（MapRead，Map，MapWrite，ReduceRead，Reduce，ReduceWrite）作为对于流或者表进行分析的几个步骤。我们可以看到，整体上Map和Reduce阶段之间差异其实也不大; 更高层次来看，他们都做了以下事情：
* 从表中读取数据，并转换为数据流 (译者注: 即 MapRead、ReduceRead)
* 针对上述数据流，将用户编写业务处理代码应用于上述数据流，转换并形成新的一个数据流。 (译者注: 即Map、Reduce)
* 将上述转换后的流根据某些规则分组，并写出到表中。 (译者注: 即MapWrite、ReduceWrite)

随后，Google内部将MapReduce投入生产使用并得到了非常广泛的业务应用，Google认为应该和公司外的同行分享我们的研究成果，最终我们将MapReduce论文发表于OSDI 2004（见图10-4）。

![10-4](https://github.com/littlehedgehog/posts/blob/master/streaming-system-book/images/stsy_1004.png)


论文中，Google详细描述了MapReduce项目的历史，API的设计和实现，以及有关使用了MapReduce框架的许多不同生产案例的详细信息。当然，Google没有提供任何实际的源代码，以至于最终Google以外的人都认为：“是的，这套系统确实牛啊！”，然后立马回头去模仿MapReduce去构建他们的定制化系统。

在随后这十年的过程中，MapReduce继续在谷歌内部进行大量开发，投入大量时间将这套系统规模推进到前所未有的水平。如果读者朋友希望了解一些更加深入更加详细的MapReduce说明，我推荐由我们的MapReduce团队中负责扩展性、性能优化的大牛Marián Dvorský撰写的文章[《History of massive-scale sorting experiments at Google》](https://cloud.google.com/blog/products/gcp/history-of-massive-scale-sorting-experiments-at-google)（图10-5）

![10-5](https://github.com/littlehedgehog/posts/blob/master/streaming-system-book/images/stsy_1005.png)

我这里希望强调的是，这么多年来看，其他任何的分布式架构最终都没有达到MapReduce的集群规模，甚至在Google内部也没有。从MapReduce诞生起到现在已经跨越十载之久，都未能看到真正能够超越MapReduce系统规模的另外一套系统，足见MapReduce系统之成功。14年的光阴看似不长，对于互联网行业已然永久。

从流式处理系统来看，我想为读者朋友强调的是MapReduce的简单性和可扩展性。 MapReduce给我们的启发是：MapReduce系统的设计非常勇于创新，它提供一套简便且直接的API，用于构建业务复杂但可靠健壮的底层分布式数据Pipeline，并足够将这套分布式数据Pipeline运行在廉价普通的商用服务器集群之上。


# Hadoop

我们大数据旅程的下一站是Hadoop（图10-6）。需要着重说明的是：我为了保证我们讨论的重心不至于偏离太多，而压缩简化讨论Hadoop的内容。但必须承认的是，Hadoop对我们的行业甚至整个世界的影响不容小觑，它带来的影响远远超出了我在此书讨论的范围。

![10-6](https://github.com/littlehedgehog/posts/blob/master/streaming-system-book/images/stsy_1006.png)

Hadoop于2005年问世，当时Doug Cutting和Mike Cafarella认为MapReduce论文中的想法太棒了，他们在构建Nutch webcrawler的分布式版本正好需要这套分布式理论基础。在这之前，他们已经实现了自己版本的Google分布式文件系统（最初称为Nutch分布式文件系统的NDFS，后来改名为HDFS或Hadoop分布式文件系统）。因此下一步，自然而然的，基于HDFS之上添加MapReduce计算层。他们称MapReduce这一层为Hadoop。

Hadoop和MapReduce之间的主要区别在于Cutting和Cafarella通过开源（以及HDFS的源代码）确保Hadoop的源代码与世界各地可以共享，最终成为Apache Hadoop项目的一部分。雅虎聘请Cutting来帮助将雅虎网络爬虫项目升级为全部基于Hadoop架构，这个项目使得Hadoop有效提升了生产可用性以及工程效率。自那以后，整个开源生态的大数据处理工具生态系统得到了蓬勃发展。与MapReduce一样，相信其他人已经能够比我更好地讲述了Hadoop的历史。我推荐一个特别好的讲解是Marko Bonaci的[《The history of Hadoop》](https://medium.com/@markobonaci/the-history-of-hadoop-68984a11704)，它本身也是一本已经出版的纸质书籍（图10-7）。

![10-7](https://github.com/littlehedgehog/posts/blob/master/streaming-system-book/images/stsy_1007.png)


在Hadoop这部分，我是期望读者朋友能够了解到是围绕Hadoop的开源生态系统对整个行业产生的巨大影响。通过创建一个开放的社区，工程师可以从早期的GFS和MapReduce论文中改进和扩展这些想法，这直接促进生态系统的蓬勃发展，并基于此之上产生了许多有用的工具，如Pig，Hive，HBase，Crunch等等。这种开放性是导致我们整个行业现有思想多样性的关键，同时Hadoop开放性生态亦是直接促进流计算系统发展。


# Flume

我们现在再回到Google，讨论Google公司中MapReduce的官方继承者：Flume（[图10-8]，有时也称为FlumeJava，这个名字起源于最初Flume的Java版本。需要注意的是，这里的Flume不要与Apache Flume混淆，这部分是面向不同领域的东西，只是恰好有同样的名字）。

![10-8](https://github.com/littlehedgehog/posts/blob/master/streaming-system-book/images/stsy_1008.png)

Flume项目由Craig Chambers在2007年谷歌西雅图办事处成立时发起。Flume最初打算是希望解决MapReduce的一些固有缺点，这些缺点即使在MapReduce最初大红大紫的阶段已经非常明显。其中许多缺点都与MapReduce完全限定的Map→Shuffle→Reduce编程模型相关; 这个编程模型虽然简单，但它带来了一些缺点：

* 由于单个MapReduce作业并不能完成大量实际上的业务案例，因此许多定制的编排系统开始在Google公司内部出现，这些编排系统主要用于协调MapReduce作业的顺序。这些系统基本上都在解决同一类问题，即将多个MapReduce作业粘合在一起，创建一个解决复杂问题的数据管道。然而，这些编排系统都是Google各自团队独立开发的，相互之间也完全不兼容，是一类典型的重复造轮子案例。

* 更糟糕的是，由于MapReduce设计的API遵循严格结构，在很多情况下严格遵循MapReduce编程模型会导致作业运行效率低下。例如，一个团队可能会编写一个简单地过滤掉一些元素的MapReduce，即，仅有Map阶段没有Reduce阶段的作业。这个作业下游紧接着另一个团队同样仅有Map阶段的作业，进行一些字段扩展和丰富(仍然带一个空的Reduce阶段作业）。第二个作业的输出最终可能会被第三个团队的MapReduce作业作为输入，第三个作业将对数据执行某些分组聚合。这个Pipeline，实际上由一个合并Map阶段(译者注: 前面两个Map合并为一个Map)，外加一个Reduce阶段即可完成业务逻辑，但实际上却需要编排三个完全独立的作业，每个作业通过Shuffle和Output两个步骤链接在一起。假设你希望保持代码的逻辑性和清洁性，于是你考虑将部分代码进行合并，但这个最终导致第三个问题。

* 为了优化MapReduce作业中的这些低效代码，工程师们开始引入手动优化，但不幸的是，这些优化会混淆Pipeline的简单逻辑，进而增加维护和调试成本。

Flume通过提供可组合的高级API来描述数据处理流水线，从而解决了这些问题。这套设计理念同样也是Beam主要的抽象模型，即PCollection和PTransform概念，如图10-9所示。

![10-9](https://github.com/littlehedgehog/posts/blob/master/streaming-system-book/images/stsy_1009.png)

图10-9: Flume的高层抽象模型（图片来源：Frances Perry）

这些数据处理Pipeline在作业启动时将通过优化器生成，优化器将以最佳效率生成MapReduce作业，然后交由框架编排执行。整个编译执行原理图可以在图10-10中看到。

![10-10](https://github.com/littlehedgehog/posts/blob/master/streaming-system-book/images/stsy_1010.png)

图10-10。从逻辑管道到物理执行计划的优化

也许Flume在自动优化方面最重要的案例就是是合并（Reuven在第5章中讨论了这个主题），其中两个逻辑上独立的阶段可以在同一个作业中顺序地（消费者-生产者融合）执行或者并行执行（兄弟融合），如图10-11所示。

![10-11](https://github.com/littlehedgehog/posts/blob/master/streaming-system-book/images/stsy_1011.png)


图10-11。合并优化将顺序或并行操作(算子)组合在一起，到同一个操作(算子)。

将两个阶段融合在一起消除了序列化/反序列化和网络开销，这在处理大量数据的底层Pipeline中非常重要。

另一种类型的自动优化是combiner lifting（见图10-12），当我们讨论增量合并时，我们已经在第7章中讨论了这些机制。combiner lifting只是我们在该章讨论的多级组合逻辑的编译器自动优化：以求和操作为例，求和的合并逻辑本来应该运算在分组(译者注: 即Group-By)操作后，由于优化的原因，被提前到在group-by-key之前做局部求和（根据group-by-key的语义，经过group-by-key操作需要跨网络进行大量数据Shuffle）。在出现数据热点情况下，将这个操作提前可以大大减少通过网络Shuffle的数据量，并且还可以在多台机器上分散掉最终聚合的机器负载。

![10-12](https://github.com/littlehedgehog/posts/blob/master/streaming-system-book/images/stsy_1012.png)


图10-12: combiner lifting在数据上游直接进行局部聚合后再发送给下游端进行二次聚合。

由于其更清晰的API定义和自动优化机制，在2009年初Google内部推出后FlumeJava立即受到巨大欢迎。之后，该团队发表了题为[《Flume Java: Easy, Efficient Data-Parallel Pipelines》](https://storage.googleapis.com/pub-tools-public-publication-data/pdf/35650.pdf)的论文（参见图10-13），这篇论文本身就是一个很好的学习FlumeJava的资料。

![10-13](https://github.com/littlehedgehog/posts/blob/master/streaming-system-book/images/stsy_1013.png)

Flume C++版本很快于2011年发布。之后2012年初，Flume被引入为Google的所有新工程师提供的Noogler6培训内容。MapReduce框架于是最终被走向被替换的命运。

从那时起，Flume已经迁移到不再使用MapReduce作为执行引擎;相反，Flume底层基于一个名为Dax的内置自定义执行引擎。
工作本身。不仅让Flume更加灵活选择执行计划而不必拘泥于Map→Shuffle→Reduce MapReduce的模型，Dax还启用了新的优化，例如Eugene Kirpi-chov和Malo Denielou的[《No shard left behind》](https://cloud.google.com/blog/products/gcp/no-shard-left-behind-dynamic-work-rebalancing-in-google-cloud-dataflow)博客文章中描述的动态负载均衡（图10-14）。

![10-14](https://github.com/littlehedgehog/posts/blob/master/streaming-system-book/images/stsy_1014.png)

尽管那篇博客主要是基于Google DataFlow框架下讨论问题，但动态负载均衡（或液态分片，Google内部更习惯这样叫）可以让部分已经完成工作的Worker能够从另外一些繁忙的Worker手中分配一些额外的工作。在Job运行过程中，通过不断的动态调整负载分配可以将系统运行效率趋近最优，这种算法将比传统方法下有经验工程师手工设置的初始参数性能更好。Flume甚至为Worker池变化进行了适配，一个拖慢整个作业进度的Worker会将其任务转移到其他更加高效的Worker上面进行执行。Flume的这些优化手段，在Google内部为公司节省了大量资源。

最后一点，Flume后来也被扩展为支持流语义。除Dax作为一个批处理系统引擎外，Flume还扩展为能够在MillWheel流处理系统上执行作业（稍后讨论）。在Google内部，之前本书中讨论过的大多数高级流处理语义概念首先被整合到Flume中，然后才进入Cloud Dataflow并最终进入Apache Beam。

总而言之，本节我们主要强调的是Flume产品给人引入高级管道概念，这使得能够让用户编写清晰易懂且自动优化的分布式大数据处理逻辑，从而让创建更大型更复杂的分布式大数据任务成为了可能，Flume让我们业务代码在保持代码清晰逻辑干净的同时，自动具备编译器优化能力。


# Storm

接下来是Apache Storm（图10-15），这是我们研究的第一个真正的流式系统。 Storm肯定不是业界使用最早的流式处理系统，但我认为这是整个行业真正广泛采用的第一个流式处理系统，因此我们在这里需要仔细研究一下。

![10-15](https://github.com/littlehedgehog/posts/blob/master/streaming-system-book/images/stsy_1015.png)

Storm是Nathan Marz的心血结晶，Nathan Marz后来在一篇题为[《History of Apache Storm and lessons learned》](http://nathanmarz.com/blog/history-of-apache-storm-and-lessons-learned.html)的博客文章中记录了其创作历史（图10-16）。 这篇冗长的博客讲述了BackType这家创业公司一直在自己通过消息队列和自定义代码去处理Twitter信息流。Nathan和十几年前Google里面设计MapReduce相关工程师有相同的认识：实际的业务处理的代码仅仅是系统代码很小一部分，如果有个统一的流式实时处理框架负责处理各类分布式系统底层问题，那么基于之上构建我们的实时大数据处理将会轻松得多。基于此，Nathan团队完成了Storm的设计和开发。值得一提的是，Storm的设计原则和其他系统大相径庭，Storm更多考虑到实时流计算的处理时延而非数据的一致性保证。后者是其他大数据系统必备基础产品特征之一。Storm针对每条流式数据进行计算处理，并提供至多一次或者至少一次的语义保证；同时不提供任何状态存储能力。相比于Batch批处理系统能够提供一致性语义保证，Storm系统能够提供更低的数据处理延迟。对于某些数据处理业务场景来说，这确实也是一个非常合理的取舍。

![10-16](https://github.com/littlehedgehog/posts/blob/master/streaming-system-book/images/stsy_1016.png)

不幸的是，人们很快就清楚地知道他们想要什么样的流式处理系统。他们不仅希望快速得到业务结果，同时希望系统具有低延迟和准确性，但仅凭Storm架构实际上不可能做到这一点。针对这个情况，Nathan后面又提出了Lambda架构。

鉴于Storm的局限性，聪明的工程师结合弱一致语义的Storm流处理以及强一致语义的Hadoop批处理。前者产生了低延迟，但不精确的结果，而后者产生了高延迟，但精确的结果，双剑合璧，整合两套系统整体提供的低延迟但最终一致的输出结果。我们在第1章中了解到，Lambda架构是Marz的另一个创意，详见他的文章[《“如何击败CAP定理”》](http://nathanmarz.com/blog/how-to-beat-the-cap-theorem.html)（图10-17）。

![10-17](https://github.com/littlehedgehog/posts/blob/master/streaming-system-book/images/stsy_1017.png)


我已经花了相当多的时间来分析Lambda架构的缺点，以至于我不会在这里啰嗦这些问题。但我要重申一下：尽管它带来了大量成本问题，Lambda架构当前还是非常受欢迎，仅仅是因为它满足了许多企业一个关键需求：系统提供低延迟但不准确的数据，后续通过批处理系统纠正之前数据，最终给出一致性的结果。从流处理系统演变的角度来看，Storm确实为普罗大众带来低延迟的流式实时数据处理能力。然而，它是以牺牲数据强一致性为代价的，这反过来又带来了Lambda架构的兴起，导致接下来多年基于两套系统架构之上的数据处理带来无尽的麻烦和成本。

撇开其他问题先不说，Storm是行业首次大规模尝试低延迟数据处理的系统，其影响反映在当前线上大量部署和应用各类流式处理系统。在我们要放下Storm开始聊其他系统之前，我觉得还是很有必要去说说Heron这个系统。在2015年，Twitter作为Storm项目孵化公司以及世界上已知最大的Storm用户，突然宣布放弃Storm引擎，宣称正在研发另外一套称之为Heron的流式处理框架。Heron旨在解决困扰Storm的一系列性能和维护问题，同时向Storm保持API兼容，详见题为[《Twitter Heron：Stream Processing at scale》](https://www.semanticscholar.org/paper/Twitter-Heron%3A-Stream-Processing-at-Scale-Kulkarni-Bhagat/e847c3ec130da57328db79a7fea794b07dbccdd9)的论文（图10-18）。

![10-18](https://github.com/littlehedgehog/posts/blob/master/streaming-system-book/images/stsy_1018.png)


Heron本身也是开源产品（但开源不在Apache项目中）。鉴于Storm仍然在社区中持续发展，现在又冒出一套和Storm竞争的软件，最终两边系统鹿死谁手，我们只能拭目以待了。

# Spark

继续走起，我们现在来到Apache Spark（图10-19）。再次，我又将大量简化Spark系统对行业的总体影响探讨，仅仅关注我们的流处理领域部分。

![10-19](https://github.com/littlehedgehog/posts/blob/master/streaming-system-book/images/stsy_1019.png)


Spark在2009年左右诞生于加州大学伯克利分校的著名AMPLab。最初推动Spark成名的原因是它能够经常在内存执行大量的计算工作，直到作业的最后一步才写入磁盘。工程师通过弹性分布式数据集（RDD）理念实现了这一目标，在底层Pipeline中能够获取每个阶段数据结果的所有派生关系，并且允许在机器故障时根据需要重新计算中间结果，当然，这些都基于一些假设 a）输入是总是可重放的，b）计算是确定性的。对于许多案例来说，这些先决条件是真实的，或者看上去足够真实，至少用户确实在Spark享受到了巨大的性能提升。从那时起，Spark逐渐建立起其作为Hadoop事实上的继任产品定位。

在Spark创建几年后，当时AMPLab的研究生Tathagata Das开始意识到：嘿，我们有这个快速的批处理引擎，如果我们将多个批次的任务串接起来，用它能否来处理流数据？于是乎，Spark Streaming诞生了。

关于Spark Streaming的真正精彩之处在于：强大的批处理引擎解决了太多底层麻烦的问题，如果基于此构建流式处理引擎则整个流处理系统将简单很多，于是世界又多一个流处理引擎，而且是可以独自提供一致性语义保障的流式处理系统。换句话说，给定正确的用例，你可以不用Lambda架构系统直接使用Spark Streaming即可满足数据一致性需求。为Spark Streaming手工点赞！

这里的一个主要问题是“正确的用例”部分。早期版本的Spark Streaming（1.x版本）的一大缺点是它仅支持特定的流处理语义：即，处理时间窗口。因此，任何需要使用事件时间，需要处理延迟数据等等案例都无法让用户使用Spark开箱即用解决业务。这意味着Spark Streaming最适合于有序数据或事件时间无关的计算。而且，正如我在本书中重申的那样，在处理当今常见的大规模、以用户为中心的数据集时，这些先决条件看上去并不是那么常见。

围绕Spark Streaming的另一个有趣的争议是“microbatch和true streaming”争论。由于Spark Streaming建立在批处理引擎的重复运行的基础之上，因此批评者声称Spark Streaming不是真正的流式引擎，因为整个系统的处理基于全局的数据切分规则。这个或多或少是实情。尽管流处理引擎几乎总是为了吞吐量而使用某种批处理或者类似的加大吞吐的系统策略，但它们可以灵活地在更精细的级别上进行处理，一直可以细化到某个key。但基于微批处理模型的系统在基于全局切分方式处理数据包，这意味着同时具备低延迟和高吞吐是不可能的。确实我们看到许多基准测试表明这说法或多或少有点正确。当然，作业能够做到几分钟或几秒钟的延迟已经相当不错了，实际上生产中很少有用例需要严格数据正确性和低延迟保证。所以从某种意义上说，Spark瞄准最初目标客户群体打法是非常到位的，因为大多数业务场景均属于这一类。但这并未阻止其竞争对手将此作为该平台的巨大劣势。就个人而言，在大多数情况下，我认为这只是一个很小问题。

撇开缺点不说，Spark Streaming是流处理的分水岭：第一个广泛使用的大规模流处理引擎，它也可以提供批处理系统的正确性保证。 当然，正如前面提到的，流式系统只是Spark整体成功故事的一小部分，Spark在迭代处理和机器学习领域做出了重要贡献，其原生SQL集成以及上述快如闪电般的内存计算，都是非常值得大书特书的产品特性。

如果您想了解有关原始Spark 1.x架构细节的更多信息，我强烈推荐Matei Zaharia关于该主题的论文[《 “An Architecture for Fast and General Data Processing on Large Clusters》](https://www2.eecs.berkeley.edu/Pubs/TechRpts/2014/EECS-2014-12.pdf)（图10-20）。 这是113页的Spark核心讲解论文，非常值得一读。

![10-20](https://github.com/littlehedgehog/posts/blob/master/streaming-system-book/images/stsy_1020.png)


时至今日，Spark的2.x版本极大地扩展了Spark Streaming的语义功能，其中已经包含了本书中描述流式处理模型的许多部分，同时试图简化一些更复杂的设计。 Spark甚至推出了一种全新的、真正面向流式处理的架构，用以规避掉微批架构的种种问题。但是曾经，当Spark第一次出现时，它带来的重要贡献是它是第一个公开可用的流处理引擎，具有数据处理的强一致性语义，尽管这个特性只能用在有序数据或使用处理时间计算的场景。


# MillWheel

接下来我们讨论MillWheel，这是我在2008年加入Google后的花20％时间兼职参与的项目，后来在2010年全职加入该团队（图10-21）。

![10-21](https://github.com/littlehedgehog/posts/blob/master/streaming-system-book/images/stsy_1021.png)


MillWheel是Google最早的通用流处理架构，该项目由Paul Nordstrom在Google西雅图办事处开业时发起。 MillWheel在Google内的成功与长期以来一直致力于为无序数据提供低延迟，强一致的处理能力不无关系。在本书的讲解中，我们已经多次分别讨论了促使MillWheel成功产品特点的方方面面。

* 第五章，Reuven详细讨论过数据精准一次的语义保证。精准一次的语义保证对于正确性至关重要。
* 第七章，我们研究了状态持久化，这为在不那么靠谱的普通硬件上执行的长时间数据处理业务并且需要保证正确性奠定了基础。
* 第三章，Slava讨论了Watermark。Watermark为处理无序数据提供了基础。
* 第七章，我们研究了持久性计时器，它们提供了Watermark与业务逻辑之间的某些关联特性。

有点令人惊讶的是，MillWheel项目最开始并未关注数据正确性。保罗最初的想法更接近于Storm的设计理论：具有弱一致性的低延迟数据处理。这是最初的MillWheel客户，一个关于基于用户搜索数据构建会话和另一个对搜索查询执行异常检测（来自MillWheel论文的Zeitgeist示例），这两家客户迫使项目走向了正确的方向。两者都非常需要强一致的数据结果：会话用于推断用户行为，异常检测用于推断搜索查询的趋势; 如果他们提供的数据不靠谱，两者效果都会显着下降。最终，幸运的是，MillWheel的设计被客户需求导向追求数据强一致性的结果。

支持乱序数据处理，这是现代流式处理系统的另一个核心功能。这个核心功能通常也被认为是被MillWheel引入到流式处理领域，和数据准确性一样，这个功能也是被客户需求推动最终加入到我们系统。 Zeitgeist项目的大数据处理过程，通常被我们拿来用作一个真正的流式处理案例来讨论。Zeitgeist项目希望检测识别搜索查询流量中的异常，并且需要捕获异常流量。对于这个大数据项目数据消费者来说，流计算将所有计算结果产出并让用户轮询所有key用来识别异常显然不太现实，数据用户要求系统直接计算某个key出现异常的数据结果，而不需要上层再来轮询。对于异常峰值（即查询流量的增加），这还相对来说比较简单好解决：当给定查询的计数超过查询的预期值时，系统发出异常信号。但是对于异常下降（即查询流量减少），问题有点棘手。仅仅看到给定搜索词的查询数量减少是不够的，因为在任何时间段内，计算结果总是从零开始。在这些情况下你必须确保你的数据输入真的能够代表当前这段时间真实业务流量，然后才将计算结果和预设模型进行比较。


> 真正的流式处理

> “真正的流式处理用例”需要一些额外解释。流式系统的一个新的演化趋势是，舍弃掉部分产品需求以简化编程模型，从而使整个系统简单易用。例如，在撰写本文时，Spark Structured Streaming和Apache Kafka Streams都将系统提供的功能限制在第8章中称为“物化视图语义”范围内，本质上对最终一致性的输出表不停做数据更新。当您想要将上述输出表作为结果查询使用时，物化视图语义非常匹配你的需求：任何时候我们只需查找该表中的值并且(译者注: 尽管结果数据一直在不停被更新和改变)以当前查询时间请求到查询结果就是最新的结果。但在一些需要真正流式处理的场景，例如异常检测，上述物化视图并不能够很好地解决这类问题。

> 接下来我们会讨论到，异常检测的某些需求使其不适合纯物化视图语义（即，依次针对单条记录处理），特别当需要完整的数据集才能够识别业务异常，而这些异常恰好是由于数据的缺失或者不完整导致的。另外，不停轮询结果表以查看是否有异常其实并不是一个扩展性很好的办法。真正的流式用户场景是推动watermark等功能的原始需求来源。(Watermark所代表的时间有先有后，我们需要最低的Watermark追踪数据的完整性，而最高的Watermark在数据时间发生倾斜时候非常容易导致丢数据的情况发生，类似Spark Structured Streaming的用法)。省略类似Watermark等功能的系统看上去简单不少，但换来代价是功能受限。在很多情况下，这些功能实际上有非常重要的业务价值。但如果这样的系统声称这些简化的功能会带来系统更多的普适性，不要听他们忽悠。试问一句，功能需求大量被砍掉，如何保证系统的普适性呢？


Zeitgeist项目首先尝试通过在计算逻辑之前插入处理时间的延迟数值来解决数据延迟问题。当数据按顺序到达时，这个思路处理逻辑正常。但业务人员随后发现数据有时可能会延迟很大，从而导致数据无序进入流式处理系统。一旦出现这个情况，系统仅仅采用处理时间的延迟是不够的，因为底层数据处理会因为数据乱序原因被错误判断为异常。最终，我们需要一种等待数据到齐的机制。

之后Watermark被设计出来用以解决数据乱序的问题。正如Slava在第3章中所描述的那样，基本思想是跟踪系统输入数据的当前进度，对于每个给定的数据源，构建一个数据输入进度用来表征输入数据的完整性。对于一些简单的数据源，例如一个带分区的Kafka Topic，每个Topic下属的分区被写入的是业务时间持续递增的数据（例如通过Web前端实时记录的日志事件），这种情况下我们可以计算产生一个非常完美的Watermark。但对于一些非常复杂的数据输入，例如动态的输入日志集，一个启发式算法可能是我们能够设计出来最能解决业务问题的Watermark生成算法了。但无论哪种方式，Watermark都是解决输入事件完整性最佳方式。之前我们尝试使用处理时间来解决事件输入完整性，有点驴头不及马嘴的感觉。


# Kafka

我们开始讨论Kafka（图10-23）。 Kafka在本章讨论的系统中是独一无二的，因为它不是数据计算框架，而是数据传输和存储的工具。但是，毫无疑问，Kafka在我们正在讨论的所有系统中扮演了推动流处理的最有影响力的角色之一。

![10-23](https://github.com/littlehedgehog/posts/blob/master/streaming-system-book/images/stsy_1023.png)


如果你不熟悉它，我们可以简单描述为: Kafka本质上是一个持久的流式数据传输和存储工具，底层系统实现为一组带有分区结构的日志型存储。它最初是由Neha Narkhede和Jay Kreps等业界大牛在LinkedIn公司内部开发的，其卓越的特性有:

* 提供一个干净的持久性模型，让大家在流式处理领域里面可以享受到批处理的产品特性，例如持久化、可重放。
* 在生产者和消费者之间提供弹性隔离。
* 我们在第6章中讨论过的流和表之间的关系，揭示了思考数据处理的基本方式，同时还提供了和数据库打通的思路和概念。
* 来自于上述所有方面的影响，不仅让Kafka成为整个行业中大多数流处理系统的基础，而且还促进了流处理数据库和微服务运动。

在这些特性中，有两个对我来说最为突出。第一个是流数据的持久化和可重放性的应用。在Kafka之前，大多数流处理系统使用某种临时、短暂的消息系统，如Rabbit MQ甚至是普通的TCP套接字来发送数据。数据处理的一致性往往通过生产者数据冗余备份来实现（即，如果下游数据消费者出现故障，则上游生产者将数据进行重新发送），但是上游数据的备份通常也是临时保存一下。大多数系统设计完全忽略在开发和测试中需要重新拉取数据重新计算的需求。但Kafka的出现改变了这一切。从数据库持久日志概念得到启发并将其应用于流处理领域，Kafka让我们享受到了如同Batch数据源一样的安全性和可靠性。凭借持久化和可重放的特点，流计算在健壮性和可靠性上面又迈出关键的一步，为后续替代批处理系统打下基础。

作为一个流式系统开发人员，Kafka的持久化和可重放功能对业界产生一个更有意思的变化就是: 当今大量流处理引擎依赖源头数据可重放来提供端到端精确一次的计算保障。可重放的特点是Apex，Flink，Kafka Streams，Spark和Storm的端到端精确一次保证的基础。当以精确一次模式执行时，每个系统都假设/要求输入数据源能够重放之前的部分数据(从最近Checkpoint到故障发生时的数据)。当流式处理系统与不具备重放能力的输入源一起使用时（哪怕是源头数据能够保证可靠的一致性数据投递，但不能提供重放功能），这种情况下无法保证端到端的完全一次语义。这种对可重放（以及持久化等其他特点）的广泛依赖是Kafka在整个行业中产生巨大影响的间接证明。

Kafka系统中第二个值得注意的重点是流和表理论的普及。我们花了整个第6章以及第8章、第9章来讨论流和表，可以说流和表构成了数据处理的基础，无论是MapReduce及其演化系统，SQL数据库系统，还是其他分支的数据处理系统。并不是所有的数据处理方法都直接基于流或者表来进行抽象，但从概念或者理论上说，表和流的理论就是这些系统的运作方式。作为这些系统的用户和开发人员，理解我们所有系统构建的核心基础概念意义重大。我们都非常感谢Kafka社区的开发者，他们帮助我们更广泛更加深入地了解到批流理论。

如果您想了解更多关于Kafka及其理论核心，JackKreps的《I❤Logs》（O'Reilly;图10-24）是一个很好的学习资料。另外，正如第6章中引用的那样，Kreps和Martin Kleppmann有两篇文章（图10-25），我强烈建议您阅读一下关于流和表相关理论。

![10-24](https://github.com/littlehedgehog/posts/blob/master/streaming-system-book/images/stsy_1024.png)

 图10-24 I ❤ Logs

Kafka为流处理领域做出了巨大贡献，可以说比其他任何单一系统都要多。特别是，对输入和输出流的持久性和可重放的设计，帮助将流计算从近似工具的小众领域发展到在大数据领域妇孺皆知的程度起了很大作用。此外，Kafka社区推广的流和表理论对于数据处理引发了我们深入思考。


# DataFlow
Cloud Dataflow（图10-26）是Google完全托管的、基于云架构的数据处理服务。 Dataflow于2015年8月推向全球。DataFlow将MapReduce，Flume和MillWheel的十多年经验融入其中，并将其打包成Serverless的云体验。

![10-26](https://github.com/littlehedgehog/posts/blob/master/streaming-system-book/images/stsy_1026.png)


虽然Google的Dataflow的Serverless特点可能是从系统角度来看最具技术挑战性以及有别于其他云厂商产品的重要因素，但我想在此讨论主要是其批流统一的编程模型。编程模型包括我们在本书的大部分内容中所讨论的转换，窗口，水印，触发器和聚合计算。当然，所有这些讨论都包含了思考问题的what、where、when、how。

DataFlow模型首先诞生于Flume，因为我们希望将MillWheel中强大的无序数据计算能力整合到Flume提供的更高级别的编程模型中。这个方式可以让Google员工在内部使用Flume进行统一的批处理和流处理编程。

关于统一模型的核心关键思考在于，尽管在当时我们也没有深刻意识到，批流处理模型本质上没有区别: 仅仅是在表和流的处理上有些小变化而已。正如我们在第6章中所讨论到的，主要的区别仅仅是在将表上增量的变化转换为流，其他一切在概念上是相同的。通过利用批处理和流处理两者大量的共性需求，可以提供一套引擎，适配于两套不同处理方式，这让流计算系统更加易于使用。

除了利用批处理和流处理之间的系统共性之外，我们还仔细查看了多年来我们在Google中遇到的各种案例，并使用这些案例来研究统一模型下系统各个部分。我们研究主要内容如下：：

* 未对齐的事件时间窗口（如会话窗口），能够简明地表达这类复杂的分析，同时亦能处理乱序数据。
* 自定义窗口支持，系统内置窗口很少适合所有业务场景，需要提供给用户自定义窗口的能力。
* 灵活的触发和统计模式，能够满足正确性，延迟，成本的各项业务需求。
* 使用Watermark来推断输入数据的完整性，这对于异常检测等用例至关重要，其中异常检测逻辑会根据是否缺少数据做出异常判断。
* 底层执行环境的逻辑抽象，无论是批处理，微批处理还是流式处理，都可以在执行引擎中提供灵活的选择，并避免系统级别的参数设置（例如微批量大小）进入逻辑API。

总之，这些平衡了灵活性，正确性，延迟和成本之间的关系，将DataFlow的模型应用于大量用户业务案例之中。

考虑到我们之前整本书都在讨论DataFlow和Beam模型的各类问题，我在此处重新给大家讲述这些概念纯属多此一举。但是，如果你正在寻找稍微更具学术性的内容以及一些应用案例，我推荐你看下2015年发表的[《DataFlow论文..》](http://www.vldb.org/pvldb/vol8/p1792-Akidau.pdf)（图10-27）。

![10-27](https://github.com/littlehedgehog/posts/blob/master/streaming-system-book/images/stsy_1027.png)


DataFlow还有不少可以大书特书的功能特点，但在这章内容构成来看，我认为DataFlow最重要的是构建了一套批流统一的大数据处理模型。DataFlow为我们提供了一套全面的处理无界且无序数据集的能力，同时这套系统很好的平衡了正确性、延迟、成本之间的相互关系。



# Flink

Flink（图10-28）在2015年突然出现在大数据舞台，然后似乎在一夜之间从一个无人所知的系统迅速转变为人人皆知的流式处理引擎。

![10-28](https://github.com/littlehedgehog/posts/blob/master/streaming-system-book/images/stsy_1028.png)


在我看来，Flink崛起有两个主要原因：

* 采用Dataflow/Beam编程模型，使其成为完备语义功能的开源流式处理系统。
* 其高效的快照实现方式（源自Chandy和Lamport的原始论文[《“Distributed Snapshots: Determining Global States of Distributed Systems”》](http://lamport.azurewebsites.net/pubs/chandy.pdf)）的研究，这为其提供了正确性所需的强一致性保证。

Reuven在第5章中简要介绍了Flink的一致性机制，这里在重申一下，其基本思想是在系统中的Worker之间沿着数据传播路径上产生周期性Barrier。这些Barrier充当了在不同Worker之间传输数据时的对齐机制。当一个Worker在其所有上游算子输入来源（即来自其所有上游一层的Worker）上接收到全部Barrier时，Worker会将当前所有key对应的状态写入一个持久化存储。这个过程意味着将这个Barrier之前的所有数据都做了持久化。

![10-29](https://github.com/littlehedgehog/posts/blob/master/streaming-system-book/images/stsy_1029.png)

图10-29 Chandy-Lamport快照
 
通过调整Barrier的生成频率，可以间接调整Checkpoint的执行频率，从而降低时延并最终获取更高的吞吐（其原因是做Checkpoint过程中涉及到对外进行持久化数据，因此会有一定的IO导致延时）。

Flink既能够支持精确一次的语义处理保证，同时又能够提供支持事件时间的处理能力，这让Flink获取的巨大的成功。接着， Jamie Grier发表他的题为“[《Extending the Yahoo! Streaming Benchmark》](https://data-artisans.com/blog/extending-the-yahoo-streaming-benchmark)“（图10-30）的文章，文章中描述了Flink性能具体的测试数据。在那篇文章中，杰米描述了两个令人印象深刻的特点:

1. 构建一个用于测试的Flink数据管道，其拥有比Twitter Storm更高的准确性（归功于Flink的强一次性语义），但成本却降到了1％。

  ![10-30](https://github.com/littlehedgehog/posts/blob/master/streaming-system-book/images/stsy_1030.png)

  图10-30。 Extending the Yahoo! Streaming Benchmark

2. Flink在精确一次的处理语义参数设定下，仍然达到Storm的7.5倍吞吐量（而且，Storm还不具备精确一次的处理语义）。此外，由于网络被打满导致Flink的性能受到限制; 进一步消除网络瓶颈后Flink的吞吐量几乎达到Storm的40倍。

从那时起，许多其他流式处理项目（特别是Storm和Apex）都采用了类似算法的数据处理一致性机制。

![10-31](https://github.com/littlehedgehog/posts/blob/master/streaming-system-book/images/stsy_1031.png)

图10-31。 “Savepoints: Turning Back Time”

通过快照机制，Flink获得了端到端数据一致性。Flink更进了一步，利用其快照的全局特性，提供了从过去的任何一点重启整个管道的能力，这一功能称为SavePoint（在Fabian Hueske和Michael Winters的帖子
[《Savepoints: Turning Back Time》(https://data-artisans.com/blog/turning-back-time-savepoints)]中有所描述，[图10-31]）。Savepoints功能参考了Kafka应用于流式传输层的持久化和可重放特性，并将其扩展应用到整个底层Pipeline。流式处理仍然遗留大量开放性问题有待优化和提升，但Flink的Savepoints功能是朝着正确方向迈出的第一步，也是整个行业非常有特点的一步。
如果您有兴趣了解有关Flink快照和保存点的系统构造的更多信息，请参阅[《State Management in Apache Flink》](http://www.vldb.org/pvldb/vol10/p1718-carbone.pdf)（图10-32），论文详细讨论了相关的实现。

![10-32](https://github.com/littlehedgehog/posts/blob/master/streaming-system-book/images/stsy_1032.png)


除了保存点之外，Flink社区还在不断创新，包括将第一个实用流式SQL API推向大规模分布式流处理引擎的领域，正如我们在第8章中所讨论的那样。
总之，Flink的迅速崛起成为流计算领军人物主要归功于三个特点：
1. 整合行业里面现有的最佳想法（例如，成为第一个开源DataFlow/Beam模型）
2. 创新性在表上做了大量优化，并将状态管理发挥更大价值，例如基于Snapshot的强一致性语义保证，Savepoints以及流式SQL。
3. 迅速且持续地推动上述需求落地。

另外，所有这些改进都是在开源社区中完成的，我们可以看到为什么Flink一直在不断提高整个行业的流计算处理标准。


# Beam

我们今天谈到的最后一个系统是Apache Beam（图10-33）。 Beam与本章中的大多数其他系统的不同之处在于，它主要是编程模型，API设计和可移植层，而不是带有执行引擎的完整系统栈。但这正是我想强调的重点：正如SQL作为声明性数据处理的通用语言一样，Beam的目标是成为程序化数据处理的通用语言。

![10-33](https://github.com/littlehedgehog/posts/blob/master/streaming-system-book/images/stsy_1033.png)


具体而言，Beam由许多组件组成：
* 一个统一的批量加流式编程模型，继承自Google DataFlow产品设计，以及我们在本书的大部分内容中讨论的细节。该模型独立于任何语言实现或runtime系统。您可以将此视为Beam等同于描述关系代数模型的SQL。
* 一组实现该模型的SDK（软件开发工具包），允许底层的Pipeline以不同API语言的惯用方式编排数据处理模型。 Beam目前提供Java，Python和Go的SDK，可以将它们视为Beam的SQL语言本身的程序化等价物。
* 一组基于SDK的DSL（特定于域的语言），提供专门的接口，以独特的方式描述模型在不同领域的接口设计。SDK来描述上述模型处理能力的全集，但DSL描述一些特定领域的处理逻辑。 Beam目前提供了一个名为Scio的Scala DSL和一个SQL DSL，它们都位于现有Java SDK之上。
* 一组可以执行Beam Pipeline的执行引擎。执行引擎采用Beam SDK术语中描述的逻辑Pipeline，并尽可能高效地将它们转换为可以执行的物理计划。目前，针对Apex，Flink，Spark和Google Cloud Dataflow存在对应的Beam引擎适配。在SQL术语中，您可以将这些引擎适配视为Beam在各种SQL数据库的实现，例如Postgres，MySQL，Oracle等。

Beam的核心愿景是实现一套可移植接口层，最引人注目的功能之一是它计划支持完整的跨语言可移植性。尽管最终目标尚未完全完成（但即将面市），让Beam在SDK和引擎适配之间提供足够高效的抽象层，从而实现SDK和引擎适配之间的任意切换。我们畅想的是，用JavaScript SDK编写的数据Pipeline可以在用Haskell编写的引擎适配层上无缝地执行，即使Haskell编写的引擎适配本身没有执行JavaScript代码的能力。

作为一个抽象层，Beam如何定位自己和底层引擎关系，对于确保Beam实际为社区带来价值至关重要，我们也不希望看到Beam引入一个不必要的抽象层。这里的关键点是，Beam的目标永远不仅仅是其所有底层引擎功能的交集（类似最小公分母）或超集（类似厨房水槽）。相反，它旨在为整个社区大数据计算引擎提供最佳的想法指导。这里面有两个创新的角度:

* Beam本身的创新

  Beam将会提出一些API，这些API需要底层runtime改造支持，并非所有底层引擎最初都支持这些功能。这没关系，随着时间的推移，我们希望许多底层引擎将这些功能融入未来版本中; 对于那些需要这些功能的业务案例来说，具备这些功能的引擎通常会被业务方选择。

![10-34](https://github.com/littlehedgehog/posts/blob/master/streaming-system-book/images/stsy_1034.png)

这里举一个Beam里面关于SplittableDoFn的API例子，这个API可以用来实现一个可组合的，可扩展的数据源。（具体参看Eugene Kirpichov在他的文章[《 “Powerful and modular I/O connectors with Splittable DoFn in Apache Beam》](https://beam.apache.org/blog/2017/08/16/splittable-do-fn.html)中描述[图10-34]）。它设计确实很有特点且功能强大，目前我们还没有看到所有底层引擎对动态负载均衡等一些更具创新性功能进行广泛支持。然而，我们预计这些功能将随着时间的推移而持续加入底层引擎支持的范围。

* 底层引擎的创新

  底层引擎适配可能会引入底层引擎所独特的功能，而Beam最初可能并未提供API支持。这没关系，随着时间的推移，已证明其有用性的引擎功能将在Beam API逐步实现。

  这里的一个例子是Flink中的状态快照机制，或者我们之前讨论过的Savepoints。 Flink仍然是唯一一个以这种方式支持快照的公开流处理系统，但是Beam提出了一个围绕快照的API建议，因为我们相信数据Pipeline运行时优雅更新对于整个行业都至关重要。如果我们今天推出这样的API，Flink将是唯一支持它的底层引擎系统。但同样没关系，这里的重点是随着时间的推移，整个行业将开始迎头赶上，因为这些功能的价值会逐步为人所知。这些变化对每个人来说都是一件好事。

通过鼓励Beam本身以及引擎的创新，我们希望推进整个行业快速演化，而不用再接受功能妥协。 通过实现跨执行引擎的可移植性承诺，我们希望将Beam建立为表达程序化数据处理流水线的通用语言，类似于当今SQL作为声明性数据处理的通用处理方式。这是一个雄心勃勃的目标，我们并没有完全实现这个计划，到目前为止我们还有很长的路要走。

# 总结

我们对数据处理技术的十五年发展进行了蜻蜓点水般的回顾，重点关注那些推动流式计算发展的关键系统和关键思想。来，最后，我们再做一次总结：

* MapReduce：可扩展性和简单性
  通过在强大且可扩展的执行引擎之上提供一组简单的数据处理抽象，MapReduce让我们的数据工程师专注于他们的数据处理需求的业务逻辑，而不是去构建能够适应在一大堆普通商用服务器上的大规模分布式处理程序。

* Hadoop：开源生态系统
  通过构建一个关于MapReduce的开源平台，无意中创建了一个蓬勃发展的生态系统，其影响力所及的范围远远超出了其最初Hadoop的范围，每年有大量的创新性想法在Hadoop社区蓬勃发展。

* Flume：管道及优化
  通过将逻辑流水线操作的高级概念与智能优化器相结合，Flume可以编写简洁且可维护的Pipeline，其功能突破了MapReduce的Map→Shuffle→Reduce的限制，而不会牺牲性能。

* Storm：弱一致性，低延迟
  通过牺牲结果的正确性以减少延迟，Storm为大众带来了流计算，并开创了Lambda架构的时代，其中弱一致的流处理引擎与强大一致的批处理系统一起运行，以实现真正的业务目标低延迟，最终一致型的结果。

* Spark: 强一致性
  通过利用强大一致的批处理引擎的重复运行来提供无界数据集的连续处理，Spark Streaming证明至少对于有序数据集的情况，可以同时具有正确性和低延迟结果。

* MillWheel：乱序处理
  通过将强一致性、精确一次处理与用于推测时间的工具（如水印和定时器）相结合，MillWheel做到了无序数据进行准确的流式处理。

* Kafka: 持久化的流式存储，流和表对偶性
  通过将持久化数据日志的概念应用于流传输问题，Kafka支持了流式数据可重放功能。通过对流和表理论的概念进行推广，阐明数据处理的概念基础。

* Cloud Dataflow：统一批流处理引擎
  通过将MillWheel的无序流式处理与高阶抽象、自动优化的Flume相结合，Cloud Dataflow为批流数据处理提供了统一模型，并且灵活地平衡正确性、计算延迟、成本的关系。

* Flink：开源流处理创新者
  通过快速将无序流式数据处理的强大功能带到开源世界，并将其与分布式快照及保存点功能等自身创新相结合，Flink提高了开源流处理的业界标准并引领了当前流式处理创新趋势。

* Beam: 可移植性
  通过提供整合行业最佳创意的强大抽象层，Beam提供了一个可移植API抽象，其定位为与SQL提供的声明性通用语言等效的程序接口，同时也鼓励在整个行业中推进创新。

可以肯定的说，我在这里强调的这10个项目及其成就的说明并没有超出当前大数据的历史发展。但是，它们对我来说是一系列重要且值得注意的大数据发展里程碑，它共同描绘了过去十五年中流处理演变的时间轴。自最早的MapReduce系统开始，尽管沿途有许多起伏波折，但不知不觉我们已经走出来很长一段征程。即便如此，在流式系统领域，未来我们仍然面临着一系列的问题亟待解决。正所谓：路漫漫其修远兮，吾将上下而求索。
