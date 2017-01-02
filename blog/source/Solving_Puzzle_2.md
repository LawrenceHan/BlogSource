title: "使用BFS算法来解决Puzzle（二）"
date: 2016-12-29 14:40:00 +0800
update: 2016-12-29 14:40:00 +0800
author: me
cover: "-/images/bfs2.png"
tags:
    - 算法
    - 多线程
preview: 系列文章第二篇，谈谈怎么实现BFS

---

#### 配合音乐看，效果更好 ^_^
<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=33922453&auto=1&height=66"></iframe>

## 一个简单的BFS实现

![BFS路径图](-/images/bfs3.png)

**以下为伪代码:**

**Queue:** 用来储存需要遍历的结点，FIFO先进先出。
``` text
假设我们从A点出发

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
根据上图我们得到第一步的结点: `R1`和`R2`，但是第二步白格换位置了，于是我们得到
`R2` `R3` `R4`三个结点。当称动到第三步时我们又得到`R1` `R3` `B3` `R5`四个结点。
如果按照BFS的逻辑进行遍历的话，会有很多无效的step(每走的一步)。很且很容易走进死循环(想一想为什么)。
所以我们需要在BFS的基础上进行一些`自定义的操作`。

## 一个自定义的BFS
首先我们先看以下两种情况:
![重复的 step](-/images/bfs6.png)
![死循环 step](-/images/bfs7.png)

**第一种情况:**

第二步和第四步的步数不同分别为step2和step4(废话，但是必须说)，但是图形完全一样。
在这种情况下，我们需要把`曾经出现过的图形`保存起来，用来在走每一步时判断是否为重复的图形(因为同样的图形可能是由不同的step来产生的)。如果为重复的图形，那么这一步就弃掉不再加入到queue里进行遍历。

**第二种情况:**

以step2为原点，step1为前一步，step3为下一步。step1 == step3。当下一步等于前一步时，弃掉下一步。
这样就可以避免死循环了。

## 把图形抽像成string

根据以上结论我们需要创建这样一个对象:
``` swift
private class Route: CustomStringConvertible {
    
    var previousStep: Int = 0
    var nextStep: Int = 0
    var frame: String
    var stepsList: String
    
    init(previousStep: Int, nextStep: Int, frame: String) {
        self.previousStep = previousStep;
        self.nextStep = nextStep
        self.frame = frame
        self.stepsList = ""
    }
    
    open var description: String {
        return "previous step: \(previousStep), current step: \(nextStep), frame: \(frame), steps: \(stepsList)"
    }
}
```
`steplist`代表走过的每一步的方向。`e.g: 下，右 = Down, Right = DR`。
这里的`Frame`要特别说明一下，在第一种情况下我们需要保存`曾经出现过的图形`，把这个概念抽像成`帧`，每走一步产生的图形即代表`一帧`。
这里用`String`是因为取hash值验证比较方便。所以接下来我们就要定义如何用`string`来表示图形。

白格子用`w(white)`表示，红格子用`r(red)`表示，蓝格子用`b(blue)`表示。那么我们可以得到以下string:
``` text
W R B B
R R B B
R R B B
R R B B
```
简化后为:`WRBB RRBB RRBB RRBB` = `WRBBRRBBRRBBRRBB` = `wrbbrrbbrrbbrrbb`
这样我们就可以推理出起始帧和结束帧:

Begin Frame = `wrbbrrbbrrbbrrbb`
End Frame = `wbrbbrbrrbrbbrbr`

## 验证
有了以上基础，我们就可以开始进行遍历了，但是每走一步我们都要进行如下验证：

首先检查有没有越界，因为我们用的是"wrbbrrbbrrbbrrbb"这样的string
第一行wrbb 第二行rrbb 第三行rrbb 第四行rrbb
上下左右移动就相当于在strings这个字符串数组里换位置，比如上是当前index - 4,
下是当前index + 4, 左是index - 1, 右是index + 1。

因为起始在左上角，所以向上和向左都越界了
我们先演示向下移动
``` text
let index = 0
var nextIndex = index + 4

wrbb rrbb rrbb rrbb
*    ^

rrbb wrbb rrbb rrbb
^    *
```
其次，每一步的四个方向都遍历完后，验证是否已经有解了，这一步只要判断

`rrbb wrbb rrbb rrbb` == `wbrb brbr rbrb brbr` 即
`route.frame.hash` == `endFrame.hash`

下面用伪代码来演示
``` swift
let queue = Queue()
let beginRoute = Route()
queue.enqueue(beginRoute)
let route = queue.dequeue()
let currentStep = route.nextStep
let previousStep = route.previousStep

var nextStep = currentStep - 4 // upward
if nextStep >= 0 && nextStep != previousStep {
    self.moveUp()
}

nextStep = currentStep + 4 // downward
if nextStep < self.totalBlockCount && nextStep != previousStep {
    self.moveDown()
}

nextStep = currentStep - 1 // leftward
if currentStep % self.columnCount - 1 >= 0 && nextStep != previousStep {
    self.moveLeft()
}

nextStep = currentStep + 1 // rightward
if currentStep % self.columnCount + 1 < self.columnCount && nextStep != previousStep {
    self.moveRight()
}

// 因为向下和向右两步都成功了，产生了两个新的route
let newRoute1 = route.moveDown() 
queue.enqueue(newRoute1) // 现在queue为 [ newRoute1 ]
let newRoute2 = route.moveRight()
queue.enqueue(newRoute2) // 现在queue为 [ newRoute1, newRoute2 ]

// 然后继续遍历直到找到解为止
let route = queue.dequeue()
etc...

```

## References:
[BFS by Swift Algorithm Club](https://github.com/raywenderlich/swift-algorithm-club/tree/master/Breadth-First%20Search)

[Puzzle-iOS by Lawrence](https://github.com/LawrenceHan/Puzzle-iOS/blob/master/Puzzle_iOS/Puzzle_iOS/PuzzleSwift.swift)

## 下一篇会讲一下算法的优化
