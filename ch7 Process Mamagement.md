[toc]

## Overview

- 进程的描述
- 进程的状态 State
- 线程 Thread
- 进程间通信 Inter-Process Communication
- 进程互斥与同步
- 死锁 DeadLock



## 进程的描述

在操作系统中，通常来说进程 `Process` 是当前正在执行的东西。因此，一个具有一定独立功能的程序在一个数据集合上的一次动态执行过程，可以称之为进程。

- 程序是静态的文件

- 进程是程序动态执行的过程

### 进程的组成

进程包括 :

- 程序的代码
- 程序处理的数据
- 程序计数器 (PC) 中的值, 指示下一条将运行的指令
- 一组通用的寄存器的当前值, 堆 `Heap` , 栈 `Stack`
- 一组系统资源(如打开的文件、内存、网络)

而进程的主要构成如下，

1. Stack Section
2. Heap Section
3. Data Section
4. Text Section

<img src="https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch7%20Process/image-20220323215907450.png" alt="image-20220323215907450" style="zoom: 67%;" />

#### Stack

Stack部分包含：

- 局部变量
- 函数和返回地址
- main函数

如上图所示，`Stack`和 `heap` 以相反的方向增长，如果两者都以相同的方向增长，那么其两者可能会重叠，因此如果它们以相反的方向增长则很好。

示例：如，调用下列函数时，将存储在Stack部分，一旦函数返回，该函数堆栈部分的值将被删除。

Stack上有一个堆栈帧，其中包含main函数以及局部变量`a, b sum` 。使用 `printf()`，创建的帧以及局部变量只能在内存中访问，帧的持续时间在从函数 `return 0`  后释放。

```c
int main(void) {
    int a, b, sum;
    a = 2;
    b = 3;
    sum = a + b;
    printf("%d\n", sum);
    return 0
}
```

Stack是一种后进先出 (LIFO) 数据结构，最后一个被推到Stack上的内容就是从顶部弹出的第一个内容。不允许从Stack的中间插入或移除。因此Stack必须至少支持两种操作：`push ` 和  `pop` ；其他操作也是可以，但不是必需的。

 ![stack2](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch7%20Process/stack2.gif)

在Linux中，`ulimit -a` 是可以获取和设置用户限制的函数

![image-20220323223534060](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch7%20Process/image-20220323223534060.png)

#### Heap

当程序在运行时需要内存时，此部分用于提供动态内存。它是从Heap提供的。

如在C语言中，`malloc()`和`calloc()`用于此目的，例如：

- `malloc(4)` 将**返回Heap区域中 4 BYTE 块的起始地址**。
- `alloca()`：从Stack申请内存，因此无需释放.

>需要注意的是，在 C 语言中，动态内存需要处理，即在不需要时释放，否则一段时间后堆会变满。
>
>因此，在 C 程序中使用了`free()`函数来执行此操作。

相比于Stack，Heap更为灵活，在Heap中，程序可以在的任何位置分配或释放内存。这种情况下就意味着Heap中间可能有一个“hole”，即未分配的内存被分配的内存包围着

由图可以看出 ，当程序释放或释放两个相邻的内存块时，Heap区域会将其合并成一个大块。这样做可以让heap更好地满足未来对大块内存的需求。交叉阴影（彩色块大小的两倍）块说明了对大块内存的请求。

![heap-Demonstration](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch7%20Process/heap-animation.gif)

#### data

data部分包含全局变量和静态局部变量。例如：

```c
#include<stdio.h>
int glbal _var ; // 全局变量将存储到data区
int main()
{
	static int var ;  // 静态变量存储到data区
	// code statement
	return 0;
}
```

在保存全局变量和静态变量的内存通常在程序启动时分配。

#### text

text部分包含可执行指令、常量和宏，它是只读位置并且是可共享的，因此也可以被其他进程使用。

通常情况下，Text区域是可共享的，因此对于频繁执行的程序，只需要在内存中保存一个副本。此外，文本段通常是只读的，以防止程序意外修改其指令。



> Reference
>
> [ layout of a process](https://www.includehelp.com/operating-systems/memory-layout-of-a-process.aspx)
>
> [memory](https://icarus.cs.weber.edu/~dab/cs1410/textbook/4.Pointers/memory.html)
>
> [stack vs heap](https://nickolasteixeira.medium.com/stack-vs-heap-whats-the-difference-and-why-should-i-care-5abc78da1a88)

### 程序和进程的关系

进程和程序之间的联系

- 程序是产生进程的基础
- 程序的每次运行构成不同的进程
- 进程是程序功能的体现
- 通过多次执行，一个程序可以对应多个进程，通过调用关系，一个进程可包括多个程序。

进程和程序的区别 :

- 进程是动态的，程序是静态的：程序是有序代码的集合。进程是程序的执行，进程有核心态/用户态.
- 进程是暂时的，程序是永久的：进程是一个状态变化的过程，程序可以长久保存。
- 进程和程序的组成不同：进程的组成包括程序，数据和进程控制块(进程状态信息)

### 进程的特点

- **动态性** : 可动态地创建，结果进程;
- **并发性**：进程可以被独立调度并占用处理机运行；(并发:一段, 并行:一时刻)
- **独立性**：不同进程的工作不相互影响；(页表是保障措施之一)
- **制约性** ：因访问共享数据, 资源或进程间同步而产生制约。

### 进程控制结构

#### 进程控制块

在操作系统中会同时运行多个进程。每个进程都有一些数据和执行指令。这些指令可以是代码执行或在进程执行期间将用于交互的设备列表（如打印机）。因此，需要一种可以存储进程的每个进程运行时的信息的数据结构，这个数据结构称为**进程控制块 (  Process Control block PCB)**。

PCB是进程存在的唯一标准	

- **进程的创建**： 为该进程生成一个PCB
- **进程的终止：** 回收它的PCB
- **进程的组织管理：** 通过对PCB的组织管理来实现

### PCB的组成

**PCB有以下三大类信息 :**

- 进程标志信息
  - 进程标志信息：如本进程的标志, 本进程的产生者标志(父进程标志). 用户标志
  - 进程号 (PID)：每个进程的唯一标识号

- 处理器状态信息保存区：保存进程的运行现场信息 :
  - 进程结构 `Process structuring`：进程的子id，或以某种功能方式与当前进程相关的其他进程的id，可以表示为队列 `queue` 、环 `ring` 或其他类型的数据结构
  - 程序计数器 ( `Program Counter` PC)：指向该进程要执行的下一条指令地址
  - CPU 寄存器 ：存储进程以执行运行状态
  - CPU调度信息：调度CPU的时间

- 进程控制信息
  - 进程调度状态 `Process Sheduling  State` ，指定了进程的状态，如 “ready”、“waiting ”等和一些其他信息，例如优先级、自进程获得 CPU 控制权或自进程获得 CPU 控制权以来经过的时间向量。此外，在暂停进程的情况下，必须为进程正在等待的事件记录事件标识数据。
  - 进程状态 `Process State`：`new`,  `ready`,  `running`,  `waiting`,  `dead`
  - 内存管理信息：页表、内存限制、段表
  - I/O 状态信息：分配给进程的 I/O 设备列表。
  - 进程权限 `Privileges`：是否允许访问系统资源
  - 进程间通信 `IPC`：独立进程之间的通信相关的标志、信号和消息



## 进程的状态

### 进程生命期管理

#### 进程的生命周期

当一个进程执行时，它会经历不同的状态。这些阶段在不同的操作系统中可能会有所不同，并且这些状态的名称也没有标准化。但一般来将，一个进程一次可以有以下五种状态之一。而进程在执行过程中改变其状态被称为进程的声明周期 `process life cycle`

| State            | Descriptio                                                   |
| ---------------- | :----------------------------------------------------------- |
| New&Start        | 进程被首次启动/创建时的初始状态                              |
| Ready            | 表示进程正在等待操作系统分配CPU资源，以便其可以运行。        |
| Running          | 一旦操作系统将CPU资源分配给进程，进程状态就会设置为Running，并且执行其指令。 |
| Waiting          | 进程需要等待资源的完成，如：等待用户输入或等待文件变为可用，则进程进入等待状态。 |
| Terminated  Exit | 当进程完成其执行，或者它被操作系统终止，其状态就会变为Exit状态，等待从主存中删除 |

#### 进程的创建

引起进程创建的3个主要事件：

- 系统初始化
- 用户请求创建一个新进程
- 正在运行的进程执行了创建进程的系统调用

#### 进程运行

创建完进程时，不一定会被执行，还需要操作系统内核选择一个可以执行的进程，这种进程称之为就绪进程 `Ready`，让它占用CPU并执行  (为何选择？如何选择？)

#### 进程等待(阻塞)

在执行过程中，可能会发生等待，无法完成的事件会发生进程等待(阻塞)，通常下面情况下会触发

- 请求并等待系统服务, 无法马上完成（如进程从辅存将数据读入主存，这个时间对于CPU会很慢，此时会发生等待）

- 启动某种操作, 无法马上完成（如多进程协同，其他进程没有完成也会等待）

- 需要的数据没有到达

进程等待事件发起只能进程自己阻塞自己，因为只有进程自身才能知道何时需要等待某种事件的发生。

#### 进程唤醒

进程唤醒是将进程的等待状态转换为就绪态，意味着该进程可以被CPU去调度执行。唤醒进程的原因 :

- 被阻塞进程需要的资源可被满足
- 被阻塞进程等待的事件到达
- 将该进程的PCB插入到就绪队列

进程只能被别的进程或操作系统唤醒

#### 进程结束

在以下四种情况下, 进程结束 :

- 正常退出（自愿）
- 错误退出（自愿）
- 致命错误（强制性）
- 被其他进程杀死（强制性）

### 进程的变化模型

####  两态模型

两态模型 ` Two state` 是指，进程主要有两个状态：

- `Not-running`：非运行状态是指，进程正在等待执行
- `Running`：运行状态是指当前正在运行的状态

![两态过程模型图](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch7%20Process/2state.png)

如图可以看出，当操作系统创建进程时，会为该进程初始化PCB；当进程被事件打断，操作系统会将该进程从`Running`状态转换到到`Not-running`状态。

####  三态模型

三态模型是两态模型的改善，在两态模型中，存在一个主要的缺点。当调度器将一个新进程从非运行态转换为运行态时，该进程可能仍在等待某个事件或 I/O请求。因此，调度器必须遍历队列并从中找到准备好执行但还未运行进程。这样的话效率很低。而且两态模型总体来说不能算满足进程的生命周期。

为了克服这个问题，三态模型将 `Not-running` 状态分为两种情况：

- 就绪 `Ready`：一个进程获得了除CPU之外的一切所需资源，等待CPU分配时间片
- 等待 `Waiting`（阻塞 `Blocked`）：进程在等待某一事件而暂停运行；如等待某资源，等待输入/输出完成。



![三态过程模型图](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch7%20Process/3statemodel.png)



**操作系统会为Ready**和**Waiting**维护一个单独的队列。一旦进程的等待的事件完成，进程就会从阻塞态进入就绪态。

#### 五态模型

五态模型，是在三态模型的基础上，加入了进程的两个新状态：**New** 和 **Terminated**。

**为什么会有五态**

在之前三态模型中，进程有状态代表的，在主存储器中加载所有进程。很显然这是不可能实现的。所以当创建一个新进程时，程序并不会立即加载到主存中。操作系统仅在主存中存储有关进程的一些信息。当内存有足够可用空间时，长期调度器 `long term scheduler` 会将程序移动到主存。这样的过程就是**New**。

##### 五态模型下的状态

- **Running**：当前正在执行的进程；也可以理解为当前占用CPU时间的进程。
- **Waiting/Blocked**： 等待某些事件的进程，如 I/O，等待其他进程，同步信号等。
- **Ready**：等待执行的进程；也可以理解为等待分配CPU时间的进程。
- **New**：刚创建的新进程。PCB已经初始化完成，但程序尚未加载到主内存中。程序保持在New状态，直到长期调度器 `long term scheduler` 将进程转换到Ready状态（已在主内存中）。
- **Terminated/Exit**：完成的进程或中止的进程。

五态模型下的状态变化过程

- **NULL -> New**：新进程的创建过程

- **New -> Ready**：当有足够的可用资源时，长期调度程序从辅存中选择一个New状态的进程并将其加载到主存中。该进程现在处于Ready状态，等待分配CPU时间。

- **Ready -> Running** ：短期调度器`Short Term Scheduler` 或调度器将一个进程从Ready调度到Running，标记着该进程正在被执行。

- **Running -> Exit**：当进程完成执行或中止，操作系统将进程从Running移动到Exit。

- **Running -> Ready**： 当一个进程运行了一定时间而没有任何中断时，可能会发生这种变化；如，轮训调度算法。另一个常见的情况是，如果处于就绪状态的进程的优先级高于当前正在运行的进程的优先级，操作系统会抢占正在运行的进程并将其改变为就绪状态。

- **Running -> Waiting**：进程必须等待某个事件，则将其置于Waitting状态。如：进程可能会请求一些可能不可用的资源或内存事，该进程可能正在等待 I/O 操作，或者该进程正在等待其他进程完成，然后才能继续执行。
- **Waiting -> Ready**：进程完成了等待的事件，将从Waitting状态进入Ready。

### 进程挂起模型

进程挂起 `process suspension`，是为了合理且充分地利用系统资源。进程在挂起状态时, 意味着进程没有占用内存空间，处在挂起 `Suspended` 状态的进程把进程放到磁盘上。当一个挂起进程准备好时运行时，它会进入 Ready-Suspend 队列中。因此，挂起状态也分为两个状态，即 阻塞挂起 `Blocked Suspend` 和 就绪挂起`Ready Suspend`。

#### 六态模型

六态模型通常情况下是具有挂起状态的五态模型。在六态模型存在一个缺陷，众所周知处理器比 I/O 设备要快很多。因此，会出现CPU执行速度过快导致所有进程都处于阻塞态而没有进程处于就绪态的情况。此时CPU 处于空闲状态，直到至少有一个进程完成 I/O 操作。这种情况下会导致 CPU 利用率低。

为了防止这种情况发生，即如果主存中的所有进程都处于阻塞态，操作系统会挂起( `Suspended` ) 处于阻塞态的进程并将其移动到辅存中。这种过程称为交换。所有处于挂起状态的进程都保存在一个队列中，内存被释放。此时，CPU 可以将一些其他进程带入主存。起到更好的利用CPU资源

![具有挂起状态的五态进程模型图](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch7%20Process/5state1suspend.png)

**六态模型的状态转换**

在六态模型中，除了五态模型的转换外，还具有下述的几个状态间装换：

- **Waiting -> Suspend**：如果主存中的所有进程都处于等待状态，操作系统将进程从Waitting转换为到`Suspended` 
- **Suspend -> Ready**：当有足够的内存可用时，操作系统将处于Suspended状态的进程移回主内中执行。
- **Suspend -> Waiting**：操作系统从辅存换入到主存的进程仍在等待某个事件的完成。

#### 七态模型

七态模型是将六态模型的阻塞状态又严格的分为 **阻塞挂起（Blocked Suspend)** 和 **就绪挂起 (Ready Suspend )**

- **阻塞挂起（Blocked Suspend)** ：进程在辅存中，尚未Ready。

- **就绪挂起 (Ready Suspend )**：:进程在辅存中, 但只要进入内存，即可运行。

**七态模型的状态转换**

- **在内存中会出现的转换情况**
  - **Blocked -> Blocked-Suspend**：如果主存中的所有进程都处于Waiting状态，则处理器将至少一个等待进程换回辅存以释放内存使得CPU资源被有效的利用。
  - **Ready ->Ready-Suspend**：当具有高优先级发生阻塞等待，操作系统会认为该进程很快会被就绪，此时，操作系统会选择挂起低优先级就绪进程，从Ready移动到Ready-Suspend，以释放主内存用于更高优先级的进程
  - **Running ->Ready-Suspend**：对抢先式分时系统，当有高优先级阻塞挂起进程因事件出现而进入就绪挂起时, 系统可能会把运行进程转换为就绪挂起状态。
  - **New -> Ready-Suspend：**如果主存使用空间不足，操作系统可能会将新进程移动为`Ready-Suspended` 状态。

- **在外存会出现的转换情况**
  - **Blocked-Suspend -> Ready-Suspend**： 当有阻塞挂起因相关事件得到满足时，操作系统会把`Blocked-Suspend `进程转换为`Ready-Suspend`。
- **与挂起相关的状态转换**（将进程从外存转入内存）
  - **Ready-Suspend -> Ready**：当处于Ready-Suspend的高优先级进程高于Ready的进程，则操作系统会将其与主存中的较低优先级Ready进程交换。
  - **Blocked-Suspend -> Blocked：**当主存空间足够时, 系统会把一个高优先级Blocked-Suspend进程(系统认为会很快出现所等待的事件)进程转换为Blocked状态

![具有两个挂起状态的五状态进程模型图](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch7%20Process/2state2suspended2.png)



> Reference
>
> [process state models](https://slaystudy.com/process-state-models-in-operating-system/)
>
> [process-scheduling](https://www.guru99.com/process-scheduling.html#long-term-scheduler)
>
> [process suspension](https://www.tutorialspoint.com/what-is-process-suspension-and-process-switching)


## 进程队列

进程队列 (`Process Queues`) 是操作系统为管理进程的各种状态维护不同类型的队列，PCB会被存放在相同状态的队列中。如果进程状态发送改变，那么PCB也会进行转换到新的状态队列中。

- 工作队列 `Job queue`：保存操作系统中所有的进程，存储在辅存中
- 就绪队列 `Ready queue`： 保存在主存中，短期调度器负责分配CPU时间
- 等待队列 `Waiting Queue`  或  `Device queues`：当进程发生阻塞事件，即需要I/O，此时进程会从 Ready转换为 Waitting 状态，进程的上下文 `Context`（PCB），都将存储在在等待队列中。

![Process Queue](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch7%20Process/Process-Queue-2.png)

> Reference
>
> [Process scheduling](https://srinix.org/lecture_notes/Materials_2020_21/CSE/5TH%20SEM/OS(MOD-2).pdf)

## 线程

线程 `Thread` 是进程中的一条执行流程，线程是CPU独立运行的基本单位，由程序计数器（PC），Stack，和一组寄存器组成。Code、File和Data段等可以在不同的线程共享。

![img](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch7%20Process/4_01_ThreadDiagram.jpg)

由上图可以看出，线程的组成比进程要少一些，进程的主要构成是：

1. Stack Section
2. Heap Section
3. Data Section
4. Text Section

而线程主要构成部分是：

- 独立的Stack
- Heap 与进程共享
- Data 与进程共享
- Text 与进程共享

### 线程的控制结构

线程控制块 TCB，是操作系统中的数据结构，每个线程都会维护一个TCB，TCB则是操作系统中线程的表现方式。

### 线程的特点

#### 线程的优点

- 线程减少了上下文切换时间，这有助于管理任务的时间；
- 各个线程之间可以并发地执行；
- 各个线程之间可以共享地址空间和文件等资源；

#### 线程的缺点

- 整个进程过分依赖线程，如单个线程中断，则整个进程中断并阻塞。
- 安全可靠性无保障：因为共享data（全局变量在线程间共享），这会产生安全问题。

**多线程优点大概分为以下四类：**

- **响应能力**：

交互式应用程序的多线程可以允许程序继续运行，即使它的一部分被阻塞或正在执行冗长的操作，从而提高对用户的响应能力。例如，多线程网络浏览器仍然可以允许用户交互

在另一个线程中加载图像时在一个线程中。

- **资源共享**：线程共享其所属进程的内存和资源。

- **经济**：因为线程共享所属进程的资源，线程的创建和上下文切换线程更

- **更高效的利用多处理器**：每个线程可以在不同的处理器上并行运行，而一个单线程进程只能在一个 CPU 上运行，多线程增加了并发性。

#### 线程与进程的比较

- 进程是最小的资源分配单位，线程是最小的CPU调度单位
- 进程拥有一个完整的资源平台，而线程只独享必不可少的资源，如寄存器和栈；
- 线程同样具有就绪、阻塞、执行三种基本状态，同样具有状态之间的转换关系；
- 线程能减少并发执行的时间和空间开销；
  - 创建时间
  - 终止时间
  - 切换（如页表切换、CPU上下文切换）
    - 不同的进程CPU分别执行，不同的线程视为同一个任务
  - 资源的使用（IPC和资源共享）

### 线程的实现

现代系统中实现了两种类型的线程 用户线程 (`User-Level Threads`)和内核线程(`Kernel-Level Threads`)。

- 内核线程是操作系统内核支持的线程，由操作系统直接管理。所有现代操作系统都支持内核级线程。
  - 内核知道并管理所有线程。
  - 每个进程一个进程控制块 (PCB)。
  - 系统中每个线程一个线程控制块 (TCB)。
  - 提供系统调用：可以从用户空间创建和管理线程。
- 用户线程是存与用户空间，并且在没有内核支持的情况下进行管理；内核不知道用户线程。从内核的角度来看，进程是一个不透明的黑匣子。
  - 线程完全由运行时系统 ` run-time system` （用户级线程库）管理。
  - 理想情况下，线程操作应该与函数调用一样快。
  - 内核不知道用户线程的存在，管理用户线程与管理单进程一样。

### 用户线程模型

通常，可以使用一下四种模型之一来实现用户线程。

- 多对一
- 一对一
- 多对多
- 两级

以上所有模型都是将用户级线程映射到内核级线程。而**内核线程**又类似于非线程（单线程）系统中的进程。内核线程是内核调度在CPU上执行单元。

#### 多对一

在多对一模型是将所有用户线程都映射到同一个内核线程上执行。该进程一次只能运行一个用户线程。

![img](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch7%20Process/many-to-one.png)

从图可以看出，一对一模型的特点如下：

- 多个用户线程都映射到单个内核线程上。
- 线程对内核不透明，线程的管理是由用户空间的线程库处理，效率很高。
- 如果发生阻塞的系统调用，那么整个进程都会阻塞，即使其他用户线程能够继续执行。
- 因为单个内核线程只能在单个 CPU 上运行，所以多对一模型不允许将单个进程拆分到多个CPU上。

#### 一对一

在一对一模型中，内核必须提供一个系统调用来创建一个新的内核线程。

![img](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch7%20Process/many-to-many.png)

一对一模型的特点：

- 在一对一模型中，一个单独的内核线程用户来处理一个用户线程。
- 一对一模型有效的克服了一对多模型中涉及阻塞式系统调用和跨CPU拆分进程的问题。
- 管理一对一模型的开销更大，大的开销将减慢系统的调用速度。
- 该模型中，的大多数实现都对可创建的线程数量进行了限制。

#### 多对多

在多对多模型中，进程将分配了 m 个内核级线程来执行 n 个用户级线程。

![img](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch7%20Process/many-to-many.png)

多对多模型的特点：

- 可将任意数量的用户程多路复用到数量相等或更少的内核线程上。
- 用户对创建的线程数没有限制。
- 阻塞内核级系统调用不会阻塞整个进程。
- 进程可以跨多个处理器拆分。

#### 两级

两级模型（`Two-Level`）是严格意义上的多对多模型，可以为单个用户线程专门一对一绑定内核线程的能力的模型

![img](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch7%20Process/two-level.png)

### 用户线程缺点

- 用户级线程与操作系统的集成度不高；如用空闲线程调度进程，阻塞其线程发起 I/O 的进程，即使该进程有其他线程可以运行，以及有锁的线程取消调度进程。

- 用户级线程需要非阻塞系统调用，否则，当一个线程阻塞，即使进程中还有可运行的线程，整个进程也会在内核中阻塞。例如，如果一个线程导致页面错误，则进程阻塞。
- 用户线程和操作系统内核之间缺乏协调性；无论进程有 1 个线程还是 1000 个线程，都仅能获得一个CPU时间片。由每个线程主动将控制权交给其他线程。
- 由于进程时资源分配的最小单位，多线程情况下，每个线程得到的时间片较少，执行会较慢。

#### 内核级线程优缺点

**优点：**

- 线程对操作系统内核透明，调度程序可以决定给拥有大量线程的进程比拥有少量线程的进程更多的时间。
- 内核级线程适用于经常阻塞的应用程序。

**缺点**：

- 内核级线程缓慢且效率低下；如，操作内核级线程比用户级线程慢。
- 由于内核管理和调度线程与进程。每个线程又需要一个完整的TCB来维护有关线程的信息。因此，存在大量开销并增加了内核复杂性。

> Reference
>
> [User-level Thread Library](https://www-users.cselabs.umn.edu/classes/Fall-2019/csci5103/proj_writeup/PROJ1.pdf)
>
> [implementing threads](http://www.it.uu.se/education/course/homepage/os/vt18/module-4/implementing-threads/)
>
> [Threads](https://www.cs.uic.edu/~jbell/CourseNotes/OperatingSystems/4_Threads.html)

## 进程控制

### 上下文切换

#### Introduction

上下文（`Context`）是指，进程的CPU占用被移除时，需要存储进程的相关信息，以便稍后获得处理器时，可以从相同的状态继续运行。这个状态数据被称之为上下文。可以理解为为上下文对进程的就如同书签对一本书。

上下文切换（`Context Switch`）是将CPU占用的时间片从一个进程或切换到另一个的过程。在这种现象中，处于运行状态的进程的执行会被挂起，而另一个处于就绪态的进程则会占用CPU时间。

**在操作系统中，什么情况下会发生上下文切换？**

- 当进程终止时
- 当CPU计时器超时，应该切换到其他进程时
- 当前进程挂起
- 当前进程需要一些时间等待I/O事件
- 当来自计时器之外的中断产生，如优先级高的进程抢占

**上下文切换时需要存储什么上下文?**

当进程从一种状态转换到另一种状态时，操作系统必须更新进程PCB信息。PCB是一种数据结构，在操作系统中用于将所有与数据相关的信息存储到进程和上下文中。PCB包含进程，即寄存器、时间片、优先级等。

进程的上下文包含：

- 地址空间、Stack空间、虚拟地址空间
- 寄存器集，如进程计数器（PC）、Stack指针（SP）、指令寄存器（IR）、程序状态寄存器（PSW）

所有信息都会保存在PCB中

**什么时候会触发上下文切换**

上下文切换只要发生在三种情况下：

- **多任务**：一个进程从CPU 中换出，下一个可运行进程换入。在抢占式系统只不过，进程可能会被调度器换出。
- **中断处理**：当中断发生时，硬件切换部分上下文。这会自动发生。
- **用户空间到内核空间**：操作系统在用户空间和内核空间之间进行转换时，会发生上下文切换。

#### 上下文切换的过程

![img](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch7%20Process/what-is-context-switching-in-operating-system-context-switching-flow.png)

又图可以看出P1占用CPU，P2处于就绪状态。如果发生中断或进程发生 I/O 事件，则 P1将会被换出；在更改 P1 的状态前，会将P1 的上下文保存在寄存器中，并将PC保存到 PCB1。之后，将加载PCB2上下文信息，此时PC从就绪状态转变为运行状态。

类似地，P2发生中断，恢复P1恢复执行。P1 从 PCB1 重新加载到运行状态以在上次停止时间重新执行任务。如信息丢失，并且CPU再次执行该进程时，会从其初始状态开始执行。

> Reference
>
> [context-switching in os](https://iq.opengenus.org/context-switching-in-os/)



### 进程创建

进程创建是操作系统给用户使用的一个系统调用，用来完成新进程的创建工作。在Unix中进程的创建采用 `fork()/exec()` 系统调用来完成新进程的创建

- `fork()` 的功能是创建一个与调用它的进程的几乎完全相同的副本
  - 使用父进程的资源（例如，打开的文件）初始化新进程
  - PC、SP与父级相同
  - 使用父进程的地址空间的内容作为副本初始化一个新地址空间
  - `fork()` 的系统调用会返回两次
    - 父进程会返回子进程的PID
    - 子进程返回0，失败返回-1
- `exec()` 的功能是用新的进程替换当前进程，exec会将程序加载到当前的地址空间内并从头开始执行。
  - 进程的Text、Data、Stack段会被替换成新的
  - exec()系统调用是使用了新程序替换了当前进程，因此PID没有发生改变
  - exec()的调用时由调用的进程发起的，被执行的是一个新程序，并不是一个进程，没有创建新进程
  - exec()系统调用不会返回给调用程序在执行，除非exec() 执行出错。

一个fork()示例：

```c
int main(void) {
  printf(“Parent (PID = %d)\n”, getpid());
  fork();
  printf(“My PID is %d\n”, getpid() );
  return 0;
}
```

一个exec()示例

```c
int main(void) {
  printf(“before execl\n”);
  execl(“/bin/ls”, “/bin/ls”, NULL);
  printf(“after execl\n”);
  return 0;
}
```

**fork()是如何工作的**

- 在内核空间内，进程被排列成一个双向链表，称为**任务列表**。
  - 父进程1234调用fork()
  - PCB会在内核空间内被复制，同时用户空间的代码也会被复制
  - 子进程返回0，父进程返回子进程的PID

<img src="https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch7%20Process/image-20220404233414592.png" alt="image-20220404233414592" style="zoom: 80%;" />

<img src="https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch7%20Process/image-20220404233344415.png" alt="image-20220404233344415" style="zoom:80%;" />



> Reference
>
> [process management](https://levelup.gitconnected.com/operating-system-process-management-26c73901166)

### 进程加载

进程加载是指，用户程序通过exec()系统调用进行加载。

exec()系列函数用新的用户程序替换当前的进程的地址空间

- 通过exec()更改子进程正在执行的程序代码来转换子进程，切换后的程序从main()开始执行
- 允许程序加载时指定启动参数；`execvp(argv[1], &argv[1])`
- 当调用成功时
  - 两个为相同的进程，仅仅做了替换，不会生成新的进程
  - 运行的是不同的程序

**exec()是如何工作的**

- 清除局部变量和动态分配的内存
- 全局的程序（代码）和常量会被替换为新的程序（代码）和常量
- 全局变量重置为基于新代码的

![image-20220404233011879](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch7%20Process/image-20220404233011879.png)

### 进程等待与退出

#### 进程的等待

等待和退出实际上是父子进程间的交互，完成子进程的资源回收

- `wait()`系统调用用于父进程等待子进程的结束
  - 子进程被创建并被执行后，父进程会被挂起，父进程通过wait()系统调用等待子进程返回值并再次获得控制权
    - 子进程调用exit()唤醒父进程，将exit()返回值作为父进程中wait()的返回值
    - 有僵尸子进程等待时，wait()立即返回其中一个值
    - 无子进程存活，wait()立即返回

#### 进程的有序退出

进程在执行结束时调用`exit()`，完成进程资源回收。`exit()` 是一个系统调用，有如下功能：

- 接受退出状态作为参数传给父进程。
- 资源回收
  - 关闭所有文件和套接字
  - 释放内存
  - 释放创建出进程相关的数据结构
    - 检查子进程是否存活，保留结果值直到父进程需要，进入zombie状态
    - 否则释放所有数据结构，进程结束

### 其他进程控制系统调用

上述是对进程模型流的一些进程控制的系统调用，但操作系统还必须包含对进程的特殊控制：

- 优先级操作
  - `nice()`，指定进程的初始优先级进
  - 在UNIX中，进程优先级会随着进程对CPU的消耗而减少
- 调试的支持：`ptrace()`，允许一个进程来控制另一个进程，如设置断点、查看寄存器。
- 定时：`Sleep()`可以将进程置于定时器等待队列中，等待数秒

> Reference
>
> [Process Management](https://www.cs.utexas.edu/~lorenzo/corsi/cs372/03F/notes/9-9.pdf)