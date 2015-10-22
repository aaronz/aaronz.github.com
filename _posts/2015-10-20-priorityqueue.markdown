---
layout: post
title:  "优先队列"
date:   2015-10-20 00:00:00
categories: Data-Structures
---

优先队列中的每个元素都有各自的优先级，优先级最高的元素最先得到服务；优先级相同的元素按照其在优先队列中的顺序得到服务。优先队列往往用堆来实现。

* 若堆的元素个数为 N，则高度不超过 [log N] + 1。
* 优先队列可以用数组表示。在这种情况下，节点 N 的父节点在 N/2 处，左子节点在 N*2 处，右子节点在 N*2+1 处。

![priority queue](http://algs4.cs.princeton.edu/24pq/images/heap-ops.png)

<!--more-->

#基础操作

优先队列起码支持下述操作：

*插入带优先级的元素（insert_with_priority）
*取出具有最高优先级的元素（pull_highest_priority_element）
*查看最高优先级的元素（peek）：O(1)时间复杂度。

其它可选的操作：

*检查优先级高的一批元素
*清空优先队列
*批插入一批元素
*合并多个优先队列
*调整一个元素的优先级


#操作实现

下面是用C实现的插入和删除操作。

{% highlight c %}

void priq_push(pri_queue q, void *data, int pri)
{
  q_elem_t *b;
  int n, m;
 
  if (q->n >= q->alloc) {
    q->alloc *= 2;
    b = q->buf = realloc(q->buf, sizeof(q_elem_t) * q->alloc);
  } else
    b = q->buf;
 
  n = q->n++;
  /* append at end, then up heap */
  while ((m = n / 2) && pri < b[m].pri) {
    b[n] = b[m];
    n = m;
  }
  b[n].data = data;
  b[n].pri = pri;
}
 
/* remove top item. returns 0 if empty. *pri can be null. */
void * priq_pop(pri_queue q, int *pri)
{
  void *out;
  if (q->n == 1) return 0;
 
  q_elem_t *b = q->buf;
 
  out = b[1].data;
  if (pri) *pri = b[1].pri;
 
  /* pull last item to top, then down heap. */
  --q->n;
 
  int n = 1, m;
  while ((m = n * 2) < q->n) {
    if (m + 1 < q->n && b[m].pri > b[m + 1].pri) m++;
 
    if (b[q->n].pri <= b[m].pri) break;
    b[n] = b[m];
    n = m;
  }
 
  b[n] = b[q->n];
  if (q->n < q->alloc / 2 && q->n >= 16)
    q->buf = realloc(q->buf, (q->alloc /= 2) * sizeof(b[0]));
 
  return out;
}

{% endhighlight %}

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