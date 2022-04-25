[toc]

## Overview

进程间是相互保持独立的，内存管理中，就是保护进程的地址空间不被其他进程访问。而**进程间通信 ( Inter-process Communication IPC)**用于在一个或多个进程间交换数据

进程间合作是那些可以影响或受其他过程影响的过程。例如网站包含 JS、H5、Flash，当有一个相应缓慢时，会发生整个网站的布局或其他功能的展示。

通常情况下进程间合作被允许的原因有：

- 信息共享：多个进程需要访问同一个文件。（如管道）

- 计算加速：将复杂功能拆分为多个子任务（多处理器时效果更佳），可以更快地解决问题
- 模块化：将整体系统架构分为不同功能模块，模块间相互协作
- 便利：单用户可以同时多任务处理，如 编辑、编译、打印等

### Communications model

<center><img src="https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch12 IPC/3_12_CommunicationsModels.jpg" alt="img" style="zoom:100%;" /></center>

<center>(a) 消息队列（间接通信） &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(b) 共享内存（直接通信）</center>

###  Message Passing

IPC背后关键的一点是消息的传递，即一个进程发消息，一个进程接收消息

而为了使进程间通信，就必须在进程间建立连接，连接可以是单/双向。连接可以使用直接通信和间接通信来实现

### Direct Communication

直接通信，必须明确声明发送者或接收者的名称，通常定义为：

- `Send(P, message)`：发送信息到进程 P
- `Receive(Q, message)`：接收来自进程 Q 的信息

在直接通信中，一般连接的属性有以下特征：

- 一个链路与一对进程相关联
- 自动建立链接
- 链接是通用的双向链接

### Indirect Communication

间接通信，为异步通信，通常情况下互通信都需要有**消息队列**；发送者将信息放置消息队列中，接受者从消息队列中取出消息

- `Send(P, message)`：像消息队列发送消息
- `Receive(Q, message)`：接受消息队列中的消息
- 每个进程都有唯一ID
- 共享一个消息队列

在间接通信中，一般连接的属性有以下特征：

- 一对进程共享消息队列时，才会在进程之间建立链接
- 链接可以被许多进程关联
- 链接可以是单向也可以是双向
- 每个进程可以有多个链接

<center><img src="https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch12 IPC/image-20220419222401782.png" alt="img" style="zoom:120%;" /></center>

<center>&nbsp;&nbsp;直接通信&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;间接通信</center>



###  Synchronization

从另一方面来讲，消息传递可以是阻塞 `Blocking` 或非阻塞 `Non-Blocking` 的；同步 `synchronous` 会**阻塞**一个进程，直到发送完成。异步 `asynchronous` 则是是**非阻塞的**，发送操作完成后会立即返回不等待返回结果

### Buffer

消息通过队列传递，队列的容量则具有下列三种配置之一：

- **0容量**：消息不能存储在队列中，因此发送者必须阻塞，直到接收者接受消息
- **有限容量**：队列中有一定的预先约定大小的容量。如果队列已满，发送者必须阻塞，直到队列中有可用空间（反之为空，接受者阻塞），否则可能是阻塞的或非阻塞的
- **无限容量**：具有无限容量的队列，发送者永远不会被迫阻塞

至此整节围绕对消息传递功能的三个方面做了介绍：

- 直接或间接通信
- 同步或异步通信
- 显式缓冲



## Signal

信号是一种有限IPC形式，通常用于Unix、类Unix、POSIX的操作系统。信号是异步的，发送信号后，操作系统会中断进程正常的执行流程来传递信号，如果进程注册过相关信号处理逻辑，则执行对应代码，否则按照POSIX标准执行相关操作。

信号类似于中断，与中断的区别是，中断是由CPU调解，内核处理，信号是由内核调解，用户程序处理

**接收到信号时会发生什么?**

- `catch`:  指定信号处理函数被调用
- `ignore`: 依靠操作系统的默认操作(abort，memory dump，suspend or resume process)
- `mask`:   屏蔽信号因此不会传送(可能是暂时的，当处理同样类型的信号)

**信号的缺点**

- 不能传输要交换的任何数据

**信号的实现**：

- 应用程序注册信号处理函数，作为系统调用发给操作系统
- 产生了信号，操作系统返回为信号处理函数入口
- 通过修改 Stack ，将后续的执行改为信号处理函数的入口

## Pipe

管道是用来数据交换的，是进程之间最简单的通讯方式。通过将一个命令的输出，作为另一个命令的输入来实现的

![img](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch12 IPC/3_22_PipeFileDescriptors.jpg)

## Message Queue

消息队列是按FIFO 无界 队列，克服了管道的一些缺点：

- 无需父进程建立通道
- Pipe是数据流，MQ是一个结构化数据
- MQ和Pipe一样是 Bounded Buffer

![使用消息队列的 IPC](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch12 IPC/ipc-using-message-queues.png)

## Share Memory

共享内存是一种直接通信方式，对于进程来说，每个进程是私有地址空间；在每个地址空间内，明确地设置了共享内存段。

- 优点：共享内存速度快更快，不需要系统调用并且以正常的内存速度进行访问。

- 缺点

  - 设置更复杂
    - 需要保障进程间同步机制

  - 无法在在多台计算机上正常工作

**如何实现内存共享**

如图所示，可以将共享内存映射到两个不同的进程内存段之间

![share memory](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch12 IPC/xshared-memory.png.pagespeed.ic.lPvwNQSSC_.webp)

## Other IPC

Socket被定义为通信中的端点，是指一对进程通过网络使用一对套接字进行通信，套接字由连接的端口号和 IP 地址组成，通常为C/S架构。

`UNIX Domain Socket` 是基于Socket网络通信架构的基础上发展出的一种IPC机制，相比于Socket，UNIX Domain Socket更高效。

`IP Socket` 是可以用在同一台主机的进程间通讯（通过loopback地址127.0.0.1），但 `UNIX Domain Socket` ，不是使用底层网络协议（不需要打包拆包、计算校验和、维护序号和应答等），所有通信都完全发生在操作系统内核中。通过使用文件系统，是两个进程可以同时打开一个UDS进行通信

![img](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch12 IPC/1ekw1o4xE_7ew9kYh6tVkCA.png)

**一些其他的IPC**

- RPC `Remote Procedure Calls`
- 基于Socket的 FTP HTTP等

> Reference
>
> [Processes](https://www.cs.uic.edu/~jbell/CourseNotes/OperatingSystems/3_Processes.html)
>
> [IPC](https://www.cs.unc.edu/~dewan/242/s07/notes/ipc/node4.html)
>
> [UNIX Domain Socket](https://akaedu.github.io/book/ch37s04.html)