> # 操作内存管理：连续内存分配



## 一 计算机体系结构及内存分层体系

 1.计算机硬件体系结构大致分为

- CPU，完成程序的执行控制
- 主存 （`main memory`），放置程序代码和数据
- I/O（外）设备，配合程序工作。
   ![image-20220425192206925](images\ch3%20Contiguous memory allocation\image-20220425192206925.png)
   2.内存分层体系（金字塔结构)

什么是内存结构：CPU所访问的指令和数据在什么地方。

第一类：位于CPU内部，操作操作系统无法直接进行管理的，寄存器，cache；特点，速度快，容量小

第二类：主存或物理内存，主要用来放置操作系统本身及要运行的代码；其特点是，容量比cache要大很多，单速度交于cache要慢一些。

第三类：磁盘，永久保存的数据及代码，当对于主存要慢，容量比主存大很多。

操作系统的作用可以将数据访问的速度（cache与内存）与存储的大小（硬盘）很好的融合在一起

![image-20220425192237300](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch3%20Contiguous memory allocation/image-20220425192237300.png)
 3.OS管理内存时需要完成的目标

① 抽象：逻辑地址空间（将物理内存，外设等抽象成逻辑地址空间，只需要访问对应地址空间)

② 保护（独立）：操作系统完成隔离机制实现，独立地址空间（每段程序执行时，不受其他程序的影响）

③ 共享：进程间安全，可靠，有效，进行数据传递，访问相同的内存。

④ 虚拟化：更多的地址空间（利用磁盘的空间)
![image-20220425192251895](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch3%20Contiguous memory allocation/image-20220425192251895.png)



4.需要完成在操作系统中管理内存的不同方法

- 操作系统层面

- 程序重定位

- 分段

- 分页

- 虚拟内存

- 按需分页虚拟内存

- 硬件层面

  - 必须知道内存架构
  - MMU(内存管理单元)：处理CPU的内存访问请求
  

## 二 地址空间&地址生成

- 地址空间的定义
- 地址生成器
- 地址安全检查

   ### 1.内存地址的定义

![image-20220425192335531](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch3%20Contiguous memory allocation/image-20220425192335531.png)

   ① 物理内存地址：硬件支持的地址空间，如主存（内存）和硬盘，由硬件完成管理和控制
   ② 逻辑内存地址：一个程序运行时所需要的内存范围。

---

两者间的关系：逻辑地址空间最终是一个存在的物理地址空间，两者间的映射关系是由操作系统来管理的

---

### 2.逻辑地址生成过程（把代码转化为计算机能理解语言）

一段代码运行→编译→汇编语言→机器语言→产生链接文件→将硬盘中程序载入到内存当中运行（完成逻辑地址的分配）

![image-20220425192353658](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch3%20Contiguous memory allocation/image-20220425192353658.png)

 如C中变量的名字，函数的位置，为逻辑地址。

### 3.物理地址生成（逻辑地址对应的物理地址的过程）

- CPU方面 MMU表示映射关系
   - ① CPU ALU `Arithmetic logic unit` 发出请求，为逻辑地址
   - ② CPU MMU `Memory management unit` 查找逻辑地址映射表，不存在会去内存中找
   - ③ 控制器提出内存请求（需要的内容，内容即指令）
- 内存方面
   - ④ 内存通过bus发送物理内存地址的内容给CPU

- OS方面
   建立逻辑地址和物理地址之间的映射关系（需在前四部前将映射管理建立好）

![image-20220425194557713](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch3%20Contiguous memory allocation/image-20220425194557713.png)

### 4. 内存安全监测：检查运行的内存是否在对应内存空间范围内

![image-20220425194611872](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch3%20Contiguous memory allocation/image-20220425194611872.png)

操作系统确保程序的有效访问的地址空间，==起始地址==与==长度==（基址寄存器和界限寄存器），也是操作系统所建立和维护的对应的表。

## 三、连续式内存分配：内存碎片与分区的动态分配

- 内存碎片问题
- 分区动态分配
  - 第一适配
  - 最佳适配
  - 最差适配
- 压缩式碎片整理
- 交换式碎片整理
## 1.内存碎片问题

*什么是碎片？* 为程序分配空间后有一些无法被利用的空闲空间，这就是内存碎片。

*碎片的种类：*

- 外部碎片：在分配单元间未使用的内存（没分配给程序的那块）
- 内部碎片：在分配单元中未使用的内存（分配给程序之后，程序无法使用）

![image-20211216172522934](../Library/Application%20Support/typora-user-images/image-20211216172522934.png)

### 2.分区的动态分配

操作系统为了管理空闲与非空闲空间，有对应的内存分配算法：

- 首次适配 `First-Fit`
- 最优适配 `Worst Fit` 
- 最佳适配 `Best Fit`

| sort         | First-Fit                                                    | Worst Fit                                                    | Best Fit                                                     |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **overview** | 空闲分区链首开始查找，直至找到一个能满足其大小要求的空闲分区为止。然后再按    照作业的大小，从该分区中划出一块内存分配给请求者，余下的空闲分区仍留在空闲分区链中。 | 最坏适应算法与最佳适应算法的排序正好相反，它的队列指针总是指向最大的空闲区，在进行分配时，总是从最大的空闲    区开始查寻；`Worst fit` 会按大小递减的顺序形成空闲区链，分配时直接从空闲区链的第一个空闲区中分配 | 寻找整个空间中最适合的的空间块（比分配请求的大小要大，但其差值是最小的） |
|              |                                                              | <img src="../Library/Application%20Support/typora-user-images/image-20211216182201177.png" alt="image-20211216182201177" style="zoom: 50%;" /> | ![image-20211216174516469](../Library/Application%20Support/typora-user-images/image-20211216174516469.png) |
| **benefit**  | 该算法倾向于使用内存中低地址部分的空闲区，在高地址部分的空闲区很少被利用，从而保留了高地址部分的大空闲     区。显然为以后到达的大作业分配大的内存空间创造了条件。 | 给文件分配分区后剩下的空闲区不至于太小，产生碎片的几率最小，对中小型文件分配分区操作有利 | 每次分配给文件的都是最合适该文件大小的分区，避免吧大空间块拆散 |
| **defect**   | 不确定性<br/>   容易产生外碎片，（低地址部分不断被划分，留下许多难以利用、很小的空闲区，而每次查找又都从低地址部分开始，会增加查找的开销。) | 重新分配慢<br>产生很多外部碎片<br>易于破坏大的空闲块以致大分区无法被分配 | **缺点**：内存中留下许多难以利用的小的空闲区。               |
| **require**  | 按地址排序<br/>   分配需要找到合适的分区<br/>   重新分配需要检查，看是否自由分区能合并于相邻的空间分区。<br/> | 按尺寸排列的空闲块列表<br>分配很快（获得最大的分区）<br>重新分配需要合并相邻的空闲分区，然后调整空闲块列表 | 需要按尺寸排列好空闲块列表<br/>分配需要寻找适合的分区<br/>重新分配（空闲块利用)需要搜索及合并相邻的空闲分区 |

## 四、连续式内存分配 (contiguous memory allocation)：压缩式与交换式碎片化整理

无论采用那种算法，都会产生内碎片`internal fragmentation` 和外碎片 `External fragmentation ` ，所以需要对这些碎片进行处理使得碎片减少甚至消失。

- 紧致算法 `compaction`：能够调整内存中运行的程序的位置
  - 什么时候做（重定位）
  - 开销
  - ![image-20211216191220714](../Library/Application%20Support/typora-user-images/image-20211216191220714.png)      ![image-20211216191258888](../Library/Application%20Support/typora-user-images/image-20211216191258888.png)
- 换入换出 `swapping`：`swapping mechanism`，是进程可以临时从主存中交换（或移动）到辅存储（磁盘），并使该内存可供其他进程使用。稍后，系统将进程从辅存交换回主存。
  - ![image-20211216194523202](../Library/Application%20Support/typora-user-images/image-20211216194523202.png)

> English Version: https://www.geeksforgeeks.org/memory-management-in-operating-system/