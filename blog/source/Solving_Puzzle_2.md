title: "使用BFS算法来解决Puzzle（二）"
date: 2016-12-28 13:40:00 +0800
update: 2016-12-28 13:40:00 +0800
author: me
cover: "-/images/bfs2.png"
tags:
    - 算法
    - 多线程
preview: 系列文章第二篇，谈谈怎么实现BFS

---

## 一个简单的BFS实现

![BFS路径图](-/images/bfs3.png)

**以下为伪代码:**

**Queue:** 用来储存需要遍历的结点，FIFO先进先出。
``` text
假设我们从`A`点出发
queue.enqueue(A) // 把A点加入到队列，现在队列为 [ A ]
queue.dequeue() // 取出队列里第一个结点A，现在队列为 [ ]，(A被标记为已遍历，以下同)
A的关联结点为B和C
queue.enqueue(B)
queue.enqueue(C) // 把B和C加入到队列， 现在队列为 [ B, C ]
queue.dequeue() // 取前队列里第一个结点B，现在队列为 [ C ]
B的关联结点为D和E
queue.enqueue(D)
queue.enqueue(E) // 把D和E加入到队列， 现在队列为 [ C, D, E ]
queue.dequeue() // 取前队列里第一个结点C，现在队列为 [ D, E ]
B的关联结点为F和G
queue.enqueue(F)
queue.enqueue(G) // 把F和G加入到队列， 现在队列为 [ D, E, F, G ]
queue.dequeue() // 取前队列里第一个结点D，现在队列为 [ E, F, G ]
D没有关联结点
queue.dequeue() // 取前队列里第一个结点E，现在队列为 [ F, G ]
E的关联结点为H
queue.enqueue(H) // 把H加入到队列， 现在队列为 [ F, G, H ]
queue.dequeue() // 取前队列里第一个结点F，现在队列为 [ G, H ]
F没有关联结点
queue.dequeue() // 取前队列里第一个结点G，现在队列为 [ H ]
G没有关联结点
queue.dequeue() // 取前队列里第一个结点H，现在队列为 [  ]
H没有关联结点，至此遍历结束
```
我们遍历的顺序为: `A` `B` `C` `D` `E` `F` `G` `H` 如下图：
![BFS遍历图](-/images/bfs4.png)

## 尝试遍历Puzzle
![Puzzle遍历图](-/images/bfs5.png)

## 施工中...