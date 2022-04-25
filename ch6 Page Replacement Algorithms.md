[toc]

## Overviews

- 功能与目标
- 实验设置与评价方法
- 局部页面算法
  - 最优页面置换算法
  - 先进先出算法
  - 最近最久未使用算法
  - 时钟页面置换算法
  - 最不常用置换算法
  - Belady现象
  - LRU FIFO Clock对比
- 全局页面置换算法
  - 工作集模型
  - 工作集页面置换算法
  - 缺页率置换算法



## 功能与目标

功能 : 当缺页中断发生，需要调入新的页面而内存已满时，选择内存当中哪个物理页面被置换。

目标 : 尽可能地减少页面的换进换出次数(即缺页中断的次数)。 具体来说，把未来不再使用的或短期内较少使用的页面换出，通常只能在局部性原理指导下依据过去的统计数据来进行预测。

页面锁定 `frame locking`：用于描述必须常驻内存的操作系统的关键部分或时间关键（`time critical`）的应用进程。实现的方式是：在页表中添加锁定标记位(`lock bit`)。

## 实验设置与评价方法

实例：记录一个进程对页访问的一个轨迹

- 举例 : 模拟一个实验环境，记录对应的地址访问序列，虚拟地址跟踪(页号, 偏移)...
  - `(3,0)`  `(1,9)`  `(4,1)`  `(2,1)`  `(5,3)`  `(2,0)` ...
- 而offset可以忽略（页不存在才会产生 `page fault`），生成的页面轨迹
  - 3, 1, 4, 2, 5, 2, 1, ...（替换为，3,1,4,2,5,2,1）

模拟一个页面置换的行为并且记录产生页缺失数的数量

- 更少的缺失，更好的性能

## 局部页面置换算法

### 最优页面置换算法

基本思路：当一个缺页中断发生时，对于保存在内存当中的每一个逻辑页面，计算在它的下一次访问之前，还需等待多长时间，从中选择等待时间最长的那个，作为被置换的页面。

这是一种理想情况, 在实际系统中是无法实现的, 因为操作系统无法知道每一个页面要等待多长时间以后才会再次被访问.

最优页面置换算法（`Optimal Page Replacement`）可用作其他算法的性能评价的依据，(在一个模拟器上运行某个程序, 并记录每一次的页面访问情况，在第二遍运行时即可使用最优算法)

在该算法中，会**替换在未来最长持续时间内不会使用的页面**。如下图所示有 a b c d e五个页，但是只有四个页帧。此时会产生物理页不够，会产生 `Page Fault`。

![image-20220321222007583](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch6 page replacement algorithms/image-20220321222007583.png)

前四次因为a b c d 已经存在物理页帧中，故前四次不会产生缺页中断，第5次请求e不在物理页帧，此时会产生`page fault`，发生页面置换。可以看出目前最久不会被访问的页面为d，故将d替换出。

![image-20220321222225824](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch6 page replacement algorithms/image-20220321222225824.png)

### 先进先出置换算法

先进先出页面置换算法 `First In First Out (FIFO)`，这是最简单的页面替换算法。在这个算法中，操作系统在一个队列中跟踪内存中的所有页面，最旧的页面在队列的前面。当一个页面需要被替换时，**队列前面的页面被移除**，进行置换。

性能较差, 调出的页面有可能是经常要访问的页面。并且有belady现象，FIFO算法很少单独使用.

FIFO算法实现起来非常简单。通过对主存储器中的队列来跟踪所有页面。一旦页面进入，我们就会将其放入队列并继续。这样，最旧的页面将始位于于队列中的第一位。 

图是FIFO的伪代码

![由 QuickLaTeX.com 渲染](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch6 page replacement algorithms/quicklatex.com-dbcb178160c7c5f3cc5dff1ce288a146_l3.svg)

>Reference
>
>[fifo-page-replacement](https://www.baeldung.com/cs/fifo-page-replacement)
>
>[page replacement algorithms](https://slideplayer.com/slide/17170897/)

实例，0时刻物理页中存放了 a b c d虚拟页，

![image-20220321225453455](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch6 page replacement algorithms/image-20220321225453455.png)

当时刻为5时，此时e不在物理页帧中，触发 page fault进行页面置换，假设 0 时刻时 入栈顺序为 a-b-c-d，此算法将会把a置换出，把e置换入。后续的换入换出也是按照进入队列顺序进行替换

![image-20220321225600170](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch6 page replacement algorithms/image-20220321225600170-16478745692234.png)

### 最近最久未使用页面置换算法

最近最久未使用，`Least Recently Used`。基本思路是LRU会在一段时间内跟踪页面的使用情况，**当发生缺页时，将最长时间未使用的页面替换为新请求的页面**。

LRU是与OPT近似的一个算法，该算法基于程序的局部性原理，即在最近时间内, 被频繁地访问页面,  再将来的一小段时间内，还可能会再一次被频繁地访问。

- LRU 根据历史推测未来
- OPT 根据未来推测未来

实例，0时刻物理页中存放了 a b c d虚拟页，

![image-20220321230813035](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch6 page replacement algorithms/image-20220321230813035.png)

当访问5时刻时，此时该替换的应该为最久没有被访问的页面，此时c上次访问时间为1，c为最久没有被访问的页面。

![image-20220321230831419](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch6 page replacement algorithms/image-20220321230831419.png)

### 时钟页面置换算法

时钟页面置换算法 `The Clock Algorithm`，是类似于LRU的一种算法，对FIFO的一种改进

基本思路 :

- 需要用到页表项的访问位，当一个页面被装入内存时，把该位初始化为0。 然后如果这个页面被访问，则把该位置设为1;
- 把各个页面组织成环形链表(类似钟表面)，把指针指向最老的页面(最先进来的)；
- 当发生一个缺页中断时，考察指针所指向的最老页面，若它的访问位为0，立即淘汰；若访问位为0，然后指针往下移动一格。如此下去，直到找到被淘汰的页面，然后把指针移动到下一格。



实例：维护一个驻留在内存中的链表，指针指向上一次访问的位置

- 使用时钟（或use/reference bit）位来记录页面的访问频率
- 每当引用页面（被访问）时，都会设置`reference bit` 为 1

时钟指针扫过页面，寻找 `reference bit` = 0 的页面，替换时钟扫过一圈未被引用的页面

![image-20220321234327704](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch6 page replacement algorithms/image-20220321234327704.png)

实例：0时刻物理页中存放了 a b c d虚拟页，在 1 2 3 4时刻请求时，此时会命中，并且在访问时，将`reference bit` 设置为1，由下图可见

![image-20220321235251072](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch6 page replacement algorithms/image-20220321235251072.png)

当时刻为5时，触发置换条件，此时时钟所有`reference bit` 都为 1，此时会转到第二圈，由于第一圈全将`reference bit` 设置为0，故，替换的页为 `a`，同时指针指向下个位置

![image-20220321235223407](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch6 page replacement algorithms/image-20220321235223407.png)

### 二次机会算法

二次机会置换算法 `Second Chance`，是对时钟算法的一个改进，具体表现为如下几个方面：

- 为每个帧添加一个 `drity bit`。
- 当每次引用时，将 该  `drity bit` 设置为`1` ； 这样就为该页面提供了二次机会。
- 当需要找到被置换出的页面时，请在时钟（维护的帧列表）中循环查找
  - 如  `drity bit`=1，则将其重置（设置为零）并继续。
  - 如 `drity bit`=0，则置换出该物理帧中的页面。

![image-20220322161003485](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch6 page replacement algorithms/image-20220322161003485.png)

增加了 `drity bit` 如 drity bit=0，此时仅为读操作，在置换时无需做写入操作。这样也被称作，增强时钟算法 `Enhance Clock`。



实例：0时刻物理页中存放了 a b c d虚拟页，在 1 2 3 4时刻请求时，此时会命中，并且在访问时，将`reference bit` 设置为1，并且，区分了读写操作，基于这种方式可以清楚的了解那页可以被置换出。

![image-20220322162430894](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch6 page replacement algorithms/image-20220322162430894.png)

因为做了写操作，当时刻为4时，a b 的`dirty bit` 都为1。在经过两轮后，将00位的页替换出，同时指针指向下一位，则替换出C，并将指针指向下一位

![image-20220322162834999](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch6 page replacement algorithms/image-20220322162834999.png)

> Reference
>
> [Second Chance Page Replacement Policy](http://www.mathcs.emory.edu/~cheung/Courses/355/Syllabus/9-virtual-mem/SC-replace.html)

### 最不常用置换算法



最不常用置换算法 `Least Frequently Used`，并不是说算法本身不常用，而是说在该算法中，系统会跟踪内存页的引用次数。当发生 `Page Fault`时，会置换出使用频率最低的页。

设计思路， LFU是在每个页表项都有增加一个计数器，对于每次内存引用，MMU 都会递增该计数器。当发生缺页中断时，操作系统选择计数器最小的页作为置换。

实例，有如下页面，此时物理页帧仅有三个，执行图如下：

> 0 1 2 3 0 1 2 3 0 1 2 3 4 5 6 7

![LFU](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch6 page replacement algorithms/LFU.png)

Page Fault = $12 \div 16 = 75%$



### Belady现象

Belady现象也可以称作Belady异常 `beladys anomaly`，是在操作系统中，随着增加物理页帧数量会导致``Page fault`数量增加的现象。

如 : FIFO算法的置换特征与进程访问内存的动态特征是矛盾的，与置换算法的目标是不一致的（即替换较少使用的页面），因此，被他置换出去的页面不一定是进程不会访问的。

如：

`f` 标记位为 缺页中断。

| Page Requests | 3    | 2    | 1    | 0    | 3    | 2    | 4    | 3    | 2    | 1    | 0    | 4    |
| ------------- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| Newest Page   | 3f   | 2f   | 1f   | 0f   | 3f   | 2f   | 4f   | 4    | 4    | 1f   | 0f   | 0    |
|               |      | 3    | 2    | 1    | 0    | 3    | 2    | 2    | 2    | 4    | 1    | 1    |
| Oldest Page   |      |      | 3    | 2    | 1    | 0    | 3    | 3    | 3    | 2    | 4    | 4    |

示例1：当存在3个物理页面时 Page Fault = 9。

| Page Requests   | 3    | 2    | 1    | 0    | 3    | 2    | 4    | 3    | 2    | 1    | 0    | 4    |
| --------------- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| **Newest Page** | 3f   | 2f   | 1f   | 0f   | 0    | 0    | 4f   | 3f   | 2f   | 1f   | 0f   | 4f   |
|                 |      | 3    | 2    | 1    | 1    | 1    | 0    | 4    | 3    | 2    | 1    | 0    |
|                 |      |      | 3    | 2    | 2    | 2    | 1    | 0    | 4    | 3    | 2    | 1    |
| **Oldest Page** |      |      |      | 3    | 3    | 3    | 2    | 1    | 0    | 4    | 3    | 2    |

示例2：当存在4个物理页面时 Page Fault = 10。

如何避免：使用stack 算法



### 什么是stack算法

> 堆栈算法是指，大小为N的集合始终是大小为N+1集合的子集。
>
> 例如：一个大小为N页的集合保存在内存中的页面始终是大小为 N + 1 的帧保存的页面的子集。

如  具有3 帧的内存中的 `{0,1,2}` 不是具有 4 帧内存中 `{0,1,4,5}` 的子集  ，这种情况下基于堆栈的算法的。

从上面belady现象可以看出，从 4  3  2  1  0  4  开始是违反基于堆栈的算法的属性

| Page Requests | 4    | 3    | 2    | 1    | 0    | 4    |
| ------------- | ---- | ---- | ---- | ---- | ---- | ---- |
| Newest Page   | 4f   | 3f   | 2f   | 1f   | 0f   | 4f   |
|               | 0    | 4    | 3    | 2    | 1    | 0    |
|               | 1    | 0    | 4    | 3    | 2    | 1    |
| Oldest Page   | 2    | 1    | 0    | 4    | 3    | 2    |



| Page Requests | 4    | 3    | 2    | 1    | 0    | 4    |
| ------------- | ---- | ---- | ---- | ---- | ---- | ---- |
| Newest Page   | 4f   | 4    | 4    | 1f   | 0f   | 0    |
|               | 2    | 2    | 2    | 4    | 1    | 1    |
| Oldest Page   | 3    | 3    | 3    | 2    | 4    | 4    |

>Reference
>
>[why stack-based cache algorithms avoidbeladys-anomaly](https://cs.stackexchange.com/questions/59355/how-do-stack-based-cache-algorithms-avoid-beladys-anomaly)
>
>[page replacement algorithms](https://www.geeksforgeeks.org/beladys-anomaly-in-page-replacement-algorithms/)

#### 为什么stack-based算法不会发生belady现象

基于堆栈的算法不会产生Belady 现象，这是因为这些类型的算法会为页面（用于替换）分配一个优先级，该优先级与页帧的数量没有管理。如 `Optimal`、`LRU` 和 `LFU`。此外，此类算法还具有良好的模拟特性，即通过一次引用可以计算出任意数量的页帧的命中率缺页率。

| Page Requests | 1    | 2    | 3    | 4    | 1    | 2    | 5    | 1    | 2    | 3    | 4    | 5    |
| ------------- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| Newest Page   | 1    | 2    | 3    | 4    | 1    | 2    | 5    | 1    | 2    | 3    | 4    | 5    |
|               |      | 1    | 2    | 3    | 4    | 1    | 2    | 5    | 1    | 2    | 3    | 4    |
| Oldest Page   |      |      | 1    | 2    | 3    | 4    | 1    | 2    | 5    | 1    | 2    | 3    |
| Page Fault    | F    | F    | F    | F    | F    | F    | F    |      |      | F    | F    | F    |



| Page Requests | 1    | 2    | 3    | 4    | 1    | 2    | 5    | 1    | 2    | 3    | 4    | 5    |
| ------------- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| Newest Page   | 1    | 2    | 3    | 4    | 1    | 2    | 5    | 1    | 2    | 3    | 4    | 5    |
|               |      | 1    | 2    | 3    | 4    | 1    | 2    | 5    | 1    | 2    | 3    | 4    |
|               |      |      | 1    | 2    | 3    | 4    | 1    | 2    | 5    | 1    | 2    | 3    |
| Oldest Page   |      |      |      | 1    | 2    | 3    | 4    | 4    | 4    | 5    | 1    | 2    |
| Page Fault    | F    | F    | F    | F    |      |      | F    |      |      | F    | F    | F    |

由上述两表可以看出，在LRU算法中，每次引用一个页面时，它都会移动到堆栈的顶部，因此堆栈的顶部的n个页面是最近使用的n个页面*。*即使帧数增加到 `n+1`，堆栈顶部也会有 `n+1` 个最近使用的页面。 构成基于堆栈的算法。

### LRU / FIFO 和 Clock 的比较

LRU和FIFO都是先进先出的思路，只不过LRU是针对页面最近访问时间来进行排序，所以需要在每一次页面访问的时候动态地调整各个页面之间的先后顺序(有一个页面的最近访问时间变了)。而FIFO是针对页面进入内存的时间来进行排序，这个时间是固定不变的, 所以各个页面之间的先后顺序是固定的。如果一个页面在进入内存后没有被访问，那么它的最近访问时间就是它进入内存的时间。 换句话说，如果内存当中的所有页面都未曾访问过，那么LRU算法就退化为了FIFO算法。

例如 : 给进程分配3个物理页面, 逻辑页面的访问顺序是 : 1,2,3,4,5,6,1,2,3 ...

## 全局页面置换算法

局部页面置换算法是基于单一程序来说明的，但对于操作系统来讲，执行的程序有很多，如果每个程序使用固定的页面置换算法会产生一定的问题，所以全局页面置换算法就是解决这种问题的。

存在问题，随着物理页帧的增加，通常情况下会大大减少缺页的次数，而为每个程序分配固定的物理页帧则会大大限制了程序执行的性能；程序是一个动态变化的过程，对内存的需求是可变的。

### 工作集模型

- 如果局部性原理不成立，那么各种页面置换算法就没有说明分别，也没有什么意义。例如：假设进程对逻辑页面的访问顺序是`1,2,3,4,5,6,6,7,8,9...`，即单调递增，那么在物理页有限的前提下,  不管采用何种置换算法，每次的页面访问都必然导致缺页中断。
- 如果局部性原理是成立的，那么如何来证明它的存在，如何来对它进行定量地分析? 这就是工作集模型.

**什么是工作集**：进程当前正在使用的页面集称之为工作集 `Working Set`，可以用一个二元函数来表示：$W(t, \tau )$；在区间$[t-\tau+1..t] $ 内的页数。

- t是当前执行的时刻
- $\tau$ 是工作集窗口 `Working-set window`，一个定长页面访问的时间窗口
- $W(t, \tau )$ 是工作集的大小，即逻辑页的数量.

>![image-20220322192606257](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch6 page replacement algorithms/image-20220322192606257.png)
>
>如果 Example ($\tau = 10$ ):
>
>t1 → WS =` {1,2,5,6,7}`
>
>t2 → WS = `{3,4}`

由此可知，

- 工作集是工作集窗口中的页面集
  - 工作集是窗口是一个移动的窗口，表现形式为对每个内存的引用。
  - 当一个新的引用出现在窗口中，对应的页将被标记位该工作集中的成员
  - 最旧一端的引用将从工作窗口中弹出，相应的页就不会再被标记位工作集中的成员了。

例如：如图所示：一个请求序列，假设 $\tau = 10$ ，则应该如何计算工作集？

![image-20220322193820089](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch6 page replacement algorithms/image-20220322193820089.png)



工作集大小的变化 : 进程开始执行后，随着访问新页面逐步建立较稳定的工作集。当内存访问的局部性区域的位置大致稳定时，工作集大小也大致稳定；局部性区域的位置改变时，工作集快速扩张和收缩过渡到下一个稳定值。

| ti   | WS              |
| ---- | --------------- |
| t1   | {1,2,5,6,7}     |
| t2   | {1,5,6,7}       |
| t3   | {1,2,5,6,7}     |
| t4   | {1,2,3,5,6,7}   |
| t5   | {1,2,3,4,5,6,7} |



> Reference
>
> [Computing the working set](https://www.cs.cornell.edu/courses/cs4410/2016su/slides/lecture13.pdf)

### 常驻集

常驻集是指在当前时刻, 进程实际驻留在内存当中的页面集合.

- 工作集是进程在运行过程中固有的性质，而常驻集取决于系统分配给进程的物理页面数目，以及所采用的页面置换算法;
- 如果一个进程的整个工作集都在内存当中，即常驻集 包含 工作集, 那么进程将很顺利地运行，而不会造成太多的缺页中断(直到工作集发生剧烈变动, 从而过渡到另一个状态);
- 当进程常驻集的大小达到某个数目之后，再给它分配更多的物理页面，缺页率也不会明显下降。

### 工作集页面置换算法

如图所示，跟踪最后一个 $\tau$ 参考（不包括断层参考）

- 最后一次$\tau$ 内存访问期间引用的页面是工作集
- $\tau$ 被称为窗口大小
- 例如，工作集大小为$\tau=4$  

0时刻，被引用的页面为 a d e 此时 工作集窗口为 `{-2, -1, 0}`

![image-20220322215333300](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch6 page replacement algorithms/image-20220322215333300.png)

时刻1，工作集窗口为 `{-2, -1, 0, 1} `  工作集为`{a c d e}` 

时刻2，此时工作集窗口 {-1, 0, 1,2 } 而 工作集为 {a c d} 因为 e已经不在工作集窗口内了。

时刻4，产生缺页中断，此时将a换出，因为a已经不在工作集窗口内了

时刻6，产生缺页，将e换入工作集 此时工作集为 {b c e d}

时刻7，因d不在工作集窗口内，则将d换出，此时工作集为 {b c d}

![image-20220322215753155](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch6 page replacement algorithms/image-20220322215753155.png)

### 缺页率页面置换算法

计算工作集的另一种方法：

- 尝试最小化页面错误
  - 当缺页率较高时，增加工作集
  - 当缺页率较低时，减少工作集

缺页率页面置换算法 `Page-Fault-Frequency Page Replacment`，即可变分配策略：**常驻集大小可变**。 例如 : 每个进程在刚开始运行的时候, 先根据程序大小给它分配一定数目的物理页面, 然后在进程运行过程中, 再动态地调整常驻集的大小.

- 可采用全局页面置换的方式，当发生一个缺页中断时，被置换的页面可以是在其他进程当中，各个并发进程竞争地使用物理页面。
- 优缺点：性能较好，但增加了系统开销。
- 具体实现：可以使用缺页率算法来动态调整常驻集的大小.

缺页率： $缺页次数 \div 内存访问次数$；

影响因素 : 

- 页面置换算法
- 分配给进程的物理页面数目
- 页面本身的大小
- 程序的编写方法

缺页集算法实现：保持跟踪缺页发生的概率，当出现缺页异常时，计算并记录从上一次缺页异常起到现在的时间。上次最后一次缺页异常的时间 **t<sub>last</sub>**

如果两次缺页异常间隔时间 “很大”，则减少工作集

- 如果 $t_{current} - t_{last} > \tau $，则从内存中移除 [$t_{current}$ , $t_{last}$]时间内没有被引用的页。

如果两次缺页异常间隔时间 “很小”，则增加工作集

- 如果 $t_{current} - t_{last} < \tau $，则将缺失的页增加到工作集中。

**示例：**假设窗口大小为2  $\tau = 2$

如果当 $t_{current} - t_{last} > 2$，则移除 [$t_{current}$ , $t_{last}$]时间内没有被引用的页。

如果当  $t_{current} - t_{last} < 2$，则将缺失的页增加到工作集中

![image-20220322224045359](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch6 page replacement algorithms/image-20220322224045359.png)

时刻1：产生缺页异常

时刻4：产生缺页异常，此时 $t_{current} - t_{last} = 4-3 > 2$，此时工作集窗口为{1,2,3,4}，工作集为 {a,c,d,e}则在工作集窗口内没有被访问到的 a,e 则被从工作集中清除。

时刻6：产生缺页异常，此时 $t_{current} - t_{last} = 6-4 \leq 2$，此时工作集窗口为{6,5,4}，工作集为 {b,c,d}；此时增加工作集，将e增加到工作集中

![image-20220322224322218](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch6 page replacement algorithms/image-20220322224322218.png)

> Reference
>
> [Page Fault Frequency](http://www.cs.cornell.edu/courses/cs4410/2013su/slides/lecture14.pdf)

### 抖动问题

抖动问题是对工作集与常驻集问题的深入

- 工作集：程序在执行过程中对内存访问的固有属性
- 常驻集：当前程序要访问那些页面放到内存中来

如果分配给一个进程的物理页面太少，不能包含整个的工作集, 即常驻集 属于工作集，那么进程将会造成很多的缺页中断，需要频繁的在内存与外存之间替换页面，从而使进程的运行速度变得很慢，我们把这种状态称为 "抖动" `Thrashing` 。

产生抖动的原因：随着驻留内存的进程数目增加, 分配给每个进程的物理页面数不断就减小, 缺页率不断上升. 所以OS要选择一个适当的进程数目和进程需要的帧数, 以便在并发水平和缺页率之间达到一个平衡.

更好的负载控制标准：调整MPL，以便：

平均缺页间隔时间（ `means time between page faults` MTBF）= 缺页服务时间（ `page fault service time` PFST）

$\sum WS_i = Size of memory$ 

![image-20220322231054093](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch6 page replacement algorithms/image-20220322231054093.png)



