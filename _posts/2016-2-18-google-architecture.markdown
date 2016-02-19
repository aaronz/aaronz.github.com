---
layout: post
title:  "谷歌架构简介"
date:   2016-2-18 00:00:00
categories: architecture
---

Google的扩展性堪称世界之最。我们都知道Google的搜索准确，迅速而且复杂，但是这并不是Google唯一的专长。他们一直致力于构建高度可扩展的应用的框架和平台，他们是如何做到这一点的？

![Google](/assets/images/posts/google-logo.jpg)

<!--more-->

# 平台  

- Linux
- Python, Java, C++

# 目前状态  

- 2006年约有45w台常规服务器  
- 2005年索引了80亿页面  
- 2008年约有200个GFS集群。每个集群有1000只5000台主机。上千台主机组成的资源池从GFS集群读取可达5PB的存储。集群间聚集读写速度可达40GB/s。  
- 2008年有约6000个MapReduce应用，并且以每月几百个新应用的速度在增加。  
- BigTable存储数十亿的URL,数百TB的图片资源以及数亿用户的喜好信息。    

# 产品架构  

谷歌将自己平台总结为三层结构，确保可以以低成本快速部署，以产品角度观察性能数据，花钱买硬件存储日志信息，不丢失数据。
- 产品层：搜索，广告，邮件，地图，视频，聊天，博客等
- 分布式系统平台： GFS，MapReduce, BigTable  
- 计算平台： 一系列的数据中心  

# 可靠地存储机制 - GFS(谷歌文件系统)

- 可靠、可扩展的存储机制是任何应用的核心需求。GFS是谷歌的核心存储系统。  
- GFS是大型分布式结构化文件存储系统，可以用来存储超大规模数据。
- 谷歌自己写GFS而不是用现成的系统原因在于，
	- 提供跨数据中心的高可靠性  
	- 数千个网络节点的可扩展性  
	- 满足海量数据读写的需求  
	- 支持GB级数据块 
	- 有效的分发操作减少节点之间性能瓶颈  
- 系统包含主从服务器  
	- 主服务器保存元数据。数据存储为64MB的数据块。客户端与主节点通讯来进行文件元数据操作来定位包含数据的从服务器。  
	- 从服务器存储真正的数据。每个数据块在三个数据服务器之间作冗余备份。一旦主服务器指向，客户端则直接与从服务器之间进行通信。  

- 新应用上线可以使用已有的GFS集群，也可以创建自己的集群。  
- GFS可以在应用级别调整以符合应用的需求。

# 通过MapReduce操作数据

- 谷歌如何实现在海量数据上面的数据操作？例如我们有很多TB的数据存储在1000台服务器上。数据库系统是无法有效的支持这种级别的数据操作的。这就是MapReduce被引入的原因。  
- MapReduce是一种用来处理和生成大数据集的编程模型以及相关的实现。用户指定一个map方法用以处理键值对来生成一组中间键值对，指定一个reduce方法来合并具有相同中间键的结果。现实世界中许多问题都可以依赖于这个模型得到解决。这样以函数形式的程序被许多常规主机并行处理。管理系统负责对输入数据进行分区，调度程序在不同主机上执行，处理主机故障，管理主机间通讯。这让没有并行计算经验的程序员也可以很容易的利用大型分布式系统。  
- 为什么要用MapReduce  
	- 是一个不错的在大量主机间调度任务的方式  
	- 可以处理主机故障  
	- 可以支持不同的应用，例如搜索，广告。几乎所有的应用都有MapReduce类型的操作。例如可以用它来预处理有效数据，计算统计单词，TB级数据排序等。  
	- 可以调整计算资源与IO资源之间的距离。 
- MapReduce系统包含三种类型的服务器  
	- 主服务器分发任务到map和reduce服务器，同时追踪任务状态。  
	- Map服务器接收用户输入并进行调用map操作。将结果分发到中间文件。  
	- Reduce服务器接收中间文件并调用Reduce方法。  
	
- 例如我们想统计所有页面中的单词数量。我们将GFS中所有的页面作为输入给MapReduce。这项工作将会交由1000台主机同时完成，其中所有调度，错误处理，数据传输都会自动完成。  
	- 具体的步骤是GFS -> Map -> Shuffle -> Reduction -> GFS存储结果。  
	- 在MapReduce中Map方法将一个视图的数据映射成为一个中间视图，生成一个键值对，例如统计单词的单词和计数的对应。  
	- Shuffling按照键类型进行聚合。  
	- Reduce方法将所有的键值对进行拼接算出总数生成最终结果。  

- Google索引管道上有大概20种不同的MapReduce过程。这些MapReduce在一个管道上依次得到执行。  
- MapReduce的方法可能非常小，甚至小到20-50行代码。  
- 一个常见的问题是掉队，例如一个节点由于调度或者主机忙等原因造成其他人物的等待。解决这种问题的方法一般可以在多个节点同时执行，然后一旦有完成的节点则结束其他节点的重复工作。  
- 数据在Map和Reduce服务器之间是压缩传输的。原因在于这些服务器并不是严重消耗CPU资源的服务器，通过CPU的压缩来解决IO问题是一个不错的选择。  

# 在BigTable中存储结构化数据  

- BigTable是一个高容错性、自我管理的大型存储系统，可以支持TB级内存和PB级的外存，可以支持每秒百万读写操作。  
- BigTable在GFS的基础上构建了一个分布式哈希机制，BigTable不是一个关系型数据库，也不支持类SQL语句查询。  
- BigTable提供了根据Key访问结构化数据的机制。GFS存储模糊数据，而各种应用通常需要结构化数据，这个时候BigTable就会派上用场。  
- 商用数据库无法扩展到类似横跨上千台服务器的级别。  
- 谷歌通过控制底层存储系统得到更多的系统控制权，并且可以根据自己的需求对系统进行改进。例如他们需要一些跨数据中心的功能也可以将这些功能集成其中。  
- 系统支持运行时的主机添加和减少。  
- 每个数据都可以存储在一个可以通过Row Key, Column Key或者时间戳访问的单元中。  
- 每个Row都存储在一个多个tablets中，每个tablet由一系列64KB的数据块组成，被称作SSTable。  
- BigTable有三种类型的服务器：  
	- 主服务器分配tablets和tablets服务器，负责追踪tablets的位置并分发任务。  
	- Tablets服务器处理tablets的读写请求，他负责拆分大小超过限制(通常100MB-200MB)的tablets。当一个Tablets服务器失败，则会有100个tablet服务器每个获取一个tablet然后系统就恢复了。  
	- Lock服务器提供了分布式的lock服务。需要锁的操作如写表可以通过该服务请求锁。  
- Tablets尽量被缓存于RAM中。  

# 硬件环境

- 当你有很多主机时如何搭建一个节能的系统？  
- 使用便宜的硬件在其上搭建应用来合理的处理回收。
- 使用低容错性的硬件价格要远低于高容错性硬件的价格(差距可高达33倍)。必须在地容错性的硬件上构建高容错性的系统和应用。  
- 使用Linux，普通的主板，普通的存储配置。
- 需要巨大的电力供应以及冷却系统的支持。  
- 同时使用自己的数据中心和第三方协作的方式。  

# 其他  

- 快速发布变更，不要等待QA。  
- 构建库是构建应用的主要方式。  
- 将一些应用发布成服务，例如爬虫服务。  
- 构建应用版本的基础设施从而支持快速发布而不用担心出错。  

# 未来方向  

- 支持地理分布式集群。  
- 为所有的数据创建全局命名空间。目前数据是按照集群区分。  
- 使数据迁移和计算更高程度的自动化。  
- 解决大规模数据复制和网络分区时的一致性问题。  

# 新技能get  

- 基础架构可以使你具备非凡的竞争力。  
- 跨数据中心应用仍然是一个没有解决的问题。  
- Hadoop值得关注。  
- 初级程序员也可以利用平台搭建分布式应用是一个非常巨大的优势。  
- 协同工作不是空谈。所有的系统都使用统一的组件，这样每个组件的提高都会带来整个组织效率的提升。  
- 构建自我管理的系统, 不能依赖于关闭系统来进行升级或添加删除节点。  
- 创建Darwinian的基础架构，并行执行耗时操作然后选择最快完成的。  
- 不要忽略学术。其中很多蕴含着巨大生产力的技术。  
- 考虑压缩平衡计算和IO。  


# 参考文章  

- [Google Architecture](http://highscalability.com/google-architecture)  
- [Video: Building Large Systems at Google](http://video.google.com/videoplay?docid=-5699448884004201579)  
- [Google Lab: The Google File System](http://labs.google.com/papers/gfs.html)  
- [Google Lab: MapReduce: Simplified Data Processing on Large Clusters](http://labs.google.com/papers/mapreduce.html)  
- [Google Lab: BigTable.](http://labs.google.com/papers/bigtable.html)  
- [Video: BigTable: A Distributed Structured Storage System.](http://video.google.com/videoplay?docid=7278544055668715642)  
- [Google Lab: The Chubby Lock Service for Loosely-Coupled Distributed Systems.](http://labs.google.com/papers/chubby.html)  
- [How Google Works by David Carr in Baseline Magazine.](http://www.baselinemag.com/article2/0,1540,1985514,00.asp)  
- [Google Lab: Interpreting the Data: Parallel Analysis with Sawzall.](http://labs.google.com/papers/sawzall.html)  
- [Dare Obasonjo's Notes on the scalability conference.](http://www.25hoursaday.com/weblog/2007/06/25/GoogleScalabilityConferenceTripReportMapReduceBigTableAndOtherDistributedSystemAbstractionsForHandlingLargeDatasets.aspx)  