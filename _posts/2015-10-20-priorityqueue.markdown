---
layout: post
title:  "优先队列"
date:   2015-10-20 00:00:00
categories: Data-Structures
---

优先队列中的每个元素都有各自的优先级，优先级最高的元素最先得到服务；优先级相同的元素按照其在优先队列中的顺序得到服务。优先队列往往用堆来实现。

堆的实现通过构造二叉堆（binary heap），实为二叉树的一种；由于其应用的普遍性，当不加限定时，均指该数据结构的这种实现。这种数据结构具有以下性质。
* 任意节点小于（或大于）它的所有后裔，最小元（或最大元）在堆的根上（堆序性）。
* 堆总是一棵完全树。即除了最底层，其他层的节点都被元素填满，且最底层尽可能地从左到右填入。


![priority queue](http://algs4.cs.princeton.edu/24pq/images/heap.png)

<!--more-->

堆实际上是一棵完全二叉树，它可以按照层次关系存放在数组中，其在数组中的表示如下，
![priority queue array representation](http://algs4.cs.princeton.edu/24pq/images/heap-representations.png)

#基础操作

优先队列起码支持下述操作：

* 插入带优先级的元素（insert_with_priority） 
* 取出具有最高优先级的元素（pull_highest_priority_element） 
* 查看最高优先级的元素（peek）：O(1)时间复杂度。 

其它可选的操作：

* 检查优先级高的一批元素
* 清空优先队列
* 批插入一批元素
* 合并多个优先队列
* 调整一个元素的优先级


#操作实现

* 将元素X插入堆中，找到空闲位置，建立一个空穴，若满足堆序性（英文：heap order），则插入完成；否则将父节点元素装入空穴，删除该父节点元素，完成空穴上移。直至满足堆序性。这种策略叫做上滤（percolate up）。

![swim](http://algs4.cs.princeton.edu/24pq/images/swim.png)

{% highlight c %}

private void swim(int k) {
   while (k > 1 && less(k/2, k)) {
      exch(k, k/2);
      k = k/2;
   }
}

{% endhighlight %}

* 删除最小元，即二叉树的根或父节点。删除该节点元素后，队列最后一个元素必须移动到堆得某个位置，使得堆仍然满足堆序性质。这种向下替换元素的过程叫作下滤。

![sink](http://algs4.cs.princeton.edu/24pq/images/sink.png)

{% highlight c %}

private void sink(int k) {
   while (2*k <= N) {
      int j = 2*k;
      if (j < N && less(j, j+1)) j++;
      if (!less(k, j)) break;
      exch(k, j);
      k = j;
   }
}

{% endhighlight %}

* Insert. We add the new item at the end of the array, increment the size of the heap, and then swim up through the heap with that item to restore the heap condition. 

* Remove the maximum. We take the largest item off the top, put the item from the end of the heap at the top, decrement the size of the heap, and then sink down through the heap with that item to restore the heap condition. 

![insert & remove](http://algs4.cs.princeton.edu/24pq/images/heap-ops.png)


#应用

* Bandwidth management
* [Discrete event simulation](https://en.wikipedia.org/wiki/Discrete_event_simulation)
* [Dijkstra's algorithm](https://en.wikipedia.org/wiki/Dijkstra%27s_algorithm)
* [Huffman coding](https://en.wikipedia.org/wiki/Huffman_coding)
* [Best-first search algorithm](https://en.wikipedia.org/wiki/Best-first_search)
* [ROAM triangulation algorithm](https://en.wikipedia.org/wiki/ROAM)
* [Prim's algorithms for minimum spanning tree](https://en.wikipedia.org/wiki/Prim%27s_algorithm)

#参考

* [Wikipedia priority queue](https://en.wikipedia.org/wiki/Priority_queue)
* [Princeton priority queue](http://algs4.cs.princeton.edu/24pq/)
* [Rosettacode priority queue](http://rosettacode.org/wiki/Priority_queue)