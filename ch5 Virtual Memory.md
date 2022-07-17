[toc]

## Objective

- 覆盖技术
- 交换技术
- 虚拟内存
  - 目标
  - 程序局部性原理
  - 基本概念
  - 基本特征
  - 虚拟页式内存管理

##  覆盖技术 overlay

在固定分区中的主要遇到的问题是进程的大小受到分区的最大大小的限制，这将意味着一个进程将不能跨越另一个进程。为了解决这个问题，早期使用了称为覆盖(`overlay`) 的解决方案，覆盖技术是为了在较小的可用内存中运行较大的程序。常用于多道程序系统，与**分区存储管理**配合使用。这样并非所有模块都需要同时存在于内存中，实现了运行大于物理内存大小的程序的技术。

### 覆盖技术的原理：

- 将程序按照执行逻辑拆分为多个功能上相对独立的部分（`overlays`）, 那些不会同时执行的模块共享同一块内存区域, 按时间先后来运行（分时）。
  - 必要部分，常驻内存的代码和数据，负责管理，在某个时间片将相应的程序和数据导入或导出内存。
  - 可选部分，在其他程序模块中实现, 平时存放在外存中, 在需要用到时才装入内存;
  - 不存在调用关系的模块不必同时装入到内存, 从而可以相互覆盖, 即这些模块共用一个分区.

### 覆盖技术实例

> 覆盖技术说明：
>
> 有一个程序，分位A B C D E F G 六个模块，每一个模块占用了一定空间，程序的覆盖树如图所示。
>
> 问：当满足加载（和运行）该程序所需物理内存中的大小是多少？

使用覆盖技术，实际上不需要将整个程序放在主内存中。只需要在对应时间片时所需要的部分即可，其逻辑调用关系树可以分为：`Root-A-D`或者 `Root-A-E` ;  `Root-B-F ` 或  `Root-C-G ` 部分。

Root是常驻内存，因为其需要调用A B C D E F G 六个模块，占用2KB



![img](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch5%20Virtual%20Memory/os_overlaymemory-164128439798517.png)

如图：加载与运行改程序所需的物理内存大小是多少？

​	(a) 12 KB

​	(b) 14 KB 

​	(c) 10 KB 

​	(d) 8 KB

答：由公式可得，最大运行层所需的物理内存为14KB，即拥有 14KB 大小的分区就可以运行上图任意一个分区

```
Root+A+D = 2KB + 4KB + 6KB = 12KB
Root+A+E = 2KB + 4KB + 8KB = 14KB
Root+B+F = 2KB + 6KB + 2KB = 10KB
Root+C+G = 2KB + 8KB + 4KB = 14KB
```

 

![Concepts of overlays](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch5%20Virtual%20Memory/Concepts-of-overlays-164128440033218.png)

如图所示，`Overlay Driver` ,也就是root区，是一个由用户负责如何覆盖的代码段，并不是操作系统提供的功能。这就意味着Overlay模式下需要用户自行去管理内存，这就是所谓的`Overlay Driver`，通俗来讲，`Overlay Driver` 是帮助整个程序如何换入换出各个部分的代码。

### 覆盖技术的优缺点

**优势 **

- 减少内存需求
- 减少时间要求

**坏处 **

- 覆盖的关系必须由开发者去指定
- 开发者需要知道整个程序所需的内存
- 覆盖的模块必须完全没有交集



## 交换技术



交换技术(`swapping`)，是操作系统实现的一种内存交换机制，与覆盖技术最大的不同是，覆盖技术是由开发者在程序内实现的内存交换机制，而swapping则是操作系统实现的内存交换机制。

实现原理：可将暂时不能运行的程序送到外存，从而获得空闲内存空间。操作系统把一个进程的整个地址空间的内容保存到外存中(换出 swap out)，而将外存中的某个进程的地址空间读入到内存中(换入 swap in)。换入换出内容的大小为整个程序的地址空间。

![交换](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch5%20Virtual%20Memory/Swapping.jpg)

### 交换技术存在的问题

- 交换时机的确定：何时需要发生交换? 只当内存空间不够或有不够的危险时换出;
- 交换区的大小：必须足够大以存放所有用户进程的所有内存映像的拷贝，必须能够对这些内存映像进行直接存取
- 程序换入时的重定位：换出后再换入的内存位置一定要在原来的位置上嘛?(可能出现寻址问题) 最好采用动态地址映射的方法

### 交换技术与覆盖技术的比较

覆盖技术是一种编程方法（换入换出单位为程序内的一个模块），用来解决程序大于计算机主内存的限制，在嵌入式系统中通常会考虑该技术。

交换技术是操作系统中内存交换的机制，换入换出单位是在内存中一个程序为单位，无需开发者给出各个模块之间的逻辑覆盖结构。

在内存不够用的情形下, 可以采用覆盖技术和交换技术, 但是 :

- 覆盖技术：需要程序要自己把整个程序划分为若干个小的功能模块, 并确定各个模块之间的覆盖关系, 增加了程序员的负担.
- 交换技术 : 以进程作为交换的单位，需要把进程的整个地址空间都换入换出, 增加了处理器的开销.

## 虚拟内存技术

### Objective

- 像覆盖技术那样，不用把程序的所有内容都放在内存中，因而能够运行比当前的空闲内存空间还要大的程序。但做的更好，由操作系统自动来完成，无需程序员的干涉。

  像交换技术那样，能够实现进程在内存与外存之间的交换，因而获得更多的空闲内存空间。但做的更好，只对进程的部分内容在内存和外存之间进行交换。

![虚拟内存](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch5%20Virtual%20Memory/virtual_memory.jpg)

### 程序局部性原理

程序的局部性 (`principle of locality`)，是虚拟内存中重要组成的概念，表明了，程序在操作系统中运行期间在任意的时间内只需要访问整个内存中的一小部分。有个这个概念就可以对操作系统的内存进行优化，从而获得更好的整体性能。局部性又可分为：

- **时间局部性**：时间局部性(`Temporal locality`)是指，访问过的内存地址很快又会被再次访问，例如：在循环中，循环变量每次迭代期间都会被访问到。
- **空间局部性**：空间局部性(`Spatial locality`)是指，在特定时间访问过的内存位置，则很可能很快就会引用其附近位置的内存地址，例如：在数组中，常规情况下，访问完数组的第一个元素会直接访问数组的下一个元素。
- **顺序局部性**：所谓顺序局部性(`Sequential locality`)，是指内存位置按升序或降序顺序方法被访问。
- **分支局部性**：分支局部性(` Branch locality`)，在计算机中，大多数指令是顺序执行的，这种情况通常发生在分支情况下，当在简单结构或分支中，所需要访问的内存地址仅限于在一个小范围内。
- **等距局部性**：等距局部性(`Equidistant locality`)位于空间和分支之间，是指如果某个位置被访问，那和它相邻等距离的连续地址极有可能会被访问到。

>Reference
>
>[explame of locality](https://stackoverflow.com/questions/9784407/locality-of-reference-english-explanation-for-equidistant-locality)
>
>[locality of reference](https://www.quora.com/What-is-the-locality-of-reference-or-the-principle-of-locality)

**实例**：为什么下列代码有问题?

页面大小为4k，但会产生4M大小。分配给每个进程的物理页面是1

在一个进程中, 定义了如下的二维数组 `int A[1024][1024]`. 该数组按行存放在内存, 每一行放在一个页面中。考虑一下程序的编写方法对缺页率的影响?

```
# Option #1
for (j = 0; j < 20; j++)
	for (i = 0; i < 200; i++)
		x[i][j] = x[i][j] + 1;

## Option #2
for (i = 0; i < 200; i++)
	for (j = 0; j < 20; j++)
		x[i][j] = x[i][j] + 1;
```

option1，按照行访问

|        |        |        |      |           |
| ------ | ------ | ------ | ---- | --------- |
| a(0,0) | a(1,0) | a(2,0) | .... | a(1023,0) |
| a(0,1) | a(1,1) | a(2,1) | .... | a(1023,1) |

option #2 按照列访问 


|        |        |        |      |           |
| ------ | ------ | ------ | ---- | --------- |
| a(0,0) | a(0,1) | a(0,2) | .... | a(0,1023) |
| a(1,0) | a(1,1) | a(1,2) | .... | a(1,1023) |

option #1 每个数据值占用1页，总共$1024X 1024$页，会发生$1024X 1024$ 次`page fault`

option #2 总共会发生1024次 `page fault`





基本特征

- 大的用户空间：通过把物理内存和外存相结合, 提供给用户的虚拟内存空间通常大于实际的物理内存，即实现了这两者的分离。如32位的虚拟地址理论上可以访问4GB，而可能计算机上仅有256M的物理内存，但硬盘容量大于4GB。
- 部分交换：与交换技术相比较，虚拟存储的调入和调出是对部分虚拟地址空间进行的;
- 不连续性： 物理内存分配的不连续性，虚拟地址空间使用的不连续性。

## 虚拟页式内存管理



### 请求调页 Demand Paging

当用户程序要调入内存运行时，不是将该程序的所有页面都装入内存，而是只装入部分的页面，就可启动程序运行。

在运行的过程中，如果发现要运行的程序或要访问的数据不再内存，则向系统发出缺页的中断请求，系统在处理这个中断时，将外存中相应的页面调入内存，使得该程序能够继续运行。

为了能够实现请求调页和页面置换，需要在页表项中增加一些位(`bit`)，来辅助完成该功能。

![image-20220320174819945](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch5%20Virtual%20Memory/image-20220320174819945.png)

访问位或引用位（`Reference bit`）：如果该页被访问过(包括读写操作)，则设置此位，用于页面置换算法。

修改位（`Modified bit (or “dirty bit”)`）：表示此页在内存中是否被修改（写）过；当系统回收该物理页面时，根据此位来决定是否把它的内容写回辅存

保护位 `Protection bits (RWX)`：能否访问该页,, 如只读, 可读写, 可执行等

驻留位或存在位 `present/absent bit` or `resident bit `：表示该页是在内存中还是在外存；逻辑页号与物理页帧相对应。

> Reference
>
> [Page State](https://www.cs.umd.edu/~hollings/cs412/s02/lectures/lect13/lect13.pdf)
>
> [Mapping Pages to Page Frames](https://sites.cs.ucsb.edu/~chris/teaching/cs170/doc/cs170-08.pdf)

如图所示：

- Virtual memory: 64KB 

- Physical memory: 32KB 

- Page size: 4KB

- Virtual memory pages: 16

- Physical memory pages: 8 

![image-20220320175528794](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch5%20Virtual%20Memory/image-20220320175528794.png)

当虚拟内存页第一项为2，那么代表物理页帧为2的项，公式则为 $vmItme \times page size= 2\times4069=8192$

### 页式内存管理的处理流程

#### 什么是缺页中断

在操作系统中，进程和内核都会通过页表项(`PTE`)，来访问一个物理页面的，访问一个目前并未被加载在物理内存中的一个页面时，由MMU引发异常，触发缺页中断 `Page Fault`。通常情况下，缺页中断不能被准确说为是一种异常错误，而是操作系统内存管理的一种机制，通过这一机制可以实现增加程序可用的内存空间。

#### 内存管理中的处理流程

![image-20220320180502614](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch5%20Virtual%20Memory/image-20220320180502614.png)

由图可知：

- 第一部，CPU Load内存地址，如果有对页面的引用，首先对该页面的引用将追溯到操作系统（第二步），否则产生 `Page Fault` 到第四步
  - 操作系统查看页表，请求无效则终止，如不在内存中就进行加载到内存中
- 第二步，找到空闲帧
- 第三步，使用页面置换算法，从辅存中调度到物理页帧中，换出前修改标记位
- 第四步，重置页表项标记位为1
- 第五步，重启导致 `Page Fault` 的指令

> Reference
>
> [Page Fault](https://www.bbau.ac.in/dept/dit/TM/Virtual%20Memory%20and%20Demand%20Paging.pdf)

### 后备存储

后备存储（有时称为辅助存储）是所有其他存储数据的设备的统称：如硬盘

- 一个虚拟地址空间的页面可以被映射到一个文件(在二级存储中)的某个位置
- 代码段 : 映射到可执行二进制文件
- 动态加载的共享库程序段 : 映射到动态调用的库文件
- 其他段 : 可能被映射到交换文件(swap file)

### 虚拟内存性能计算

`effective memory access time` ETA 是指有效的内存访问时间，计算EAT的公式如下：

P：页表命中率

1-P：缺页率

$EAT= P \times hit \quad memory \quad time + (1-P) \times miss \quad memory \quad time. $​

例如：在 TLB 找到页的百分比称为命中率。 80% 的命中率意味着在 80% 的时间在 TLB 中找到所需的页码。 如果搜索 TLB 需要 20 纳秒，访问内存需要 100 纳秒，那么当页码在 TLB 中时，映射内存的访问需要 120 纳秒。

如果在 TLB 中找不到页号（20 纳秒），那么必须首先访问内存中的页表和帧号（100 纳秒），然后访问内存中所需的字节（100 纳秒），总共 220 纳秒。 为了找到有效的内存访问时间，则需要权衡有效的内存访问的概率：

$EAT = 0.80 \times 120 + 0.20 \times 220 = 140 \quad nano seconds$​



例2：已知内存访问时间是10millisecond，缓存访问时间为10microseconds。设，TLB命中率15%，则有效内存访问时间是多少

$EAT = 0.15\times(10+0.001)+(1-0.15)\times(10\times2 + 0.001)$​

例3：

已知TLB命中率为 70%，TLB 访问时间为 30ns，访问主存时间为 90ns，则有效内存访问时间是多少

$0.7 \times (30+90) + (1-0.7) \times (30+90\times2)$​

$0.7\times 120 + 0.3\times210 = 84+63=170$​

> Reference
>
> [ETA](https://www.cukashmir.ac.in/cukashmir/User_Files/imagefile/DIT/StudyMaterial/OperatingSystemBTech/BTechCSE_3_Rizwana_BTCS304_unit3_B.pdf)
>
> [calculate the effective access time](https://stackoverflow.com/questions/18550370/calculate-the-effective-access-time)

