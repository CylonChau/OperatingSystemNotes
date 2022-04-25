[toc]

## Backgound

信号量 `semaphores`，是操作系统中非常重要的技术，通过使用一个简单的整数值来管理并发进程，信号量只是一个在线程之间共享的整数变量。该变量用于解决临界区问题并实现进程同步。 信号量具有两个原子操作：

- `P()`：sem减一，如果sem<0，等待；否则继续
- `V()`：sem加一，如果sem≤0，唤醒一个等待的P；


## Semaphore

### 信号量的使用

型号量的特点：

- **两个类型信号量**

  - **二进制信号量** `Binary Semaphore`：也称为互斥锁。它只能有两个值0和1。它的值被初始化为1。它用于实现多进程临界区问题的解决。

  - **计数信号量** `Counting Semaphore`：值可以跨越一个不受限制的域（可以取任何非负数）。它用于控制对具有多个实例的资源的访问。

- 信号量是被保护的变量

  - 初始化完成后，唯一改变一个信号量的值的办法是通过`P()` 和 `V()`

  - 操作必须是原子

  - `P()` 能够阻塞，`V()` 不会阻塞

- 信号量可以用在2个方面

  - 互斥

  - 条件同步(调度约束  ——  一个线程等待另一个线程的事情发生)

### 信号量实现的互斥

```c
mutex = new Semaphore(1);

mutex->P(); // 临界区前p
... 
    critical section
...    
mutex->V(); // 临界区后v
```

#### 信号量实现调度约束

```c
condition = new Semaphore(0);

// Thread A
...
condition->P(); // 等待线程B某一些指令完成之后再继续运行,在此阻塞
...

// Thread B
...
condition->V(); // 线程b执行某程度后，使用信号量增加唤醒线程A
```

#### 信号量实现有界缓冲

在更复杂的同步场景下，用二进制信号量无法有效的解决问题，此时就需要计数信号量来完成这些功能；例如一个线程等待另一个线程处理事情

- 比如生产东西或消费东西(生产者消费者模式)，互斥(锁机制)是不够的

- 有界缓冲区的生产者-消费者问题
  - 一个或者多个生产者产生数据将数据放在一个缓冲区里
  - 单个消费者每次从缓冲区取出数据
  - 在任何一个时间只有一个生产者或消费者可以访问该缓冲区

在这里需要注意一些问题：

- 正确性要求

  - 在任何一个时间只能有一个线程操作缓冲区(互斥)；可多个

  - 当缓冲区为空时，消费者必须等待生产者(调度，同步约束)

  - 当缓存区满，生产者必须等待消费者(调度，同步约束)

- 每个约束用一个单独的信号量

  - 一个二进制信号量，互斥
  - 两个计数信号量
    - 一般信号量 fullBuffers
    - 一般信号了 emptyBuffers

```c
class BoundedBuffer{
		mutex = new Semaphore(1);
		fullBuffers = new Semaphore(0);   //说明缓冲区初始为空
 		emptyBuffers = new Semaphore(n);  //同时可以有n个生产者来生产
};

// 生产者
BoundedBuffer::Deposit(c){
		emptyBuffers->P(); // emptyBuff 操作 -1，当EB被阻塞时，
		mutex->P();  // 操作buffer时，是互斥操作，需要使用pv
		Add c to the buffer; // 临界区
		mutex->V();	// 操作buffer时，是互斥操作，需要使用pv
		fullBuffers->V(); // FB初始值为0，通过通知机制可以通知消费者可以开始取数据
}

// 消费者
BoundedBuffer::Withdraw(c){
    	// 消费者先执行，此时BF为0 会阻塞，等待先写后读
    	// 生产者先执行，初始FB为0，+1 此时会读取数据
		fullBuffers->P();
		mutex->P(); 	// 操作buffer时，是互斥操作，需要使用pv
		Remove c from buffer; // 临界区
		mutex->V();		// 操作buffer时，是互斥操作，需要使用pv
		emptyBuffers->V();
    	// 消费后会+1，使得EB不满，起到通知生产者继续写数据
}
```



## 管程

管程是一种解决进程间同步问题的程序结构，英文是 `Monitors`；管程通过编程语言级别的支持，实现进程间的互斥。管程包含了一些列共享变量，以及针对变量的共享函数的组合，在设计上管程定义了：

- 锁
  - 用锁来确保在任何情况下监视器中只有一个进程。
  - 对共享数据提供互斥
- 0或者多个条件变量，用于管理对共享数据的并发访问
  - 通过使进程进入睡眠状态的同时释放它们的锁，使线程在临界区内进入睡眠状态。

如下图所示：monitor是一种数据结构，用于将所有控制信息、时间信息和共享数据封装为一个整体。这个整体是对信号量的抽象，可以在其中定义互斥的控制语句。

- 进入管程需要有队列（entry queue），是互斥的，首先要获得lock

- 进入临界区后，执行函数对共享变量进行操作，这是在条件变量中

- lock

  - `Lock::Acquire()` 等待直到锁可用,然后抢占锁
  - `Lock::Release()` 释放锁,唤醒等待者如果有
- Condition Variable
  - 允许等待状态进入临界区
    - 允许处于等待(睡眠)的线程进入临界区
    - 某个时刻原子释放锁进入睡眠
  - `Wait()` operation
    - 暂停对任何条件变量执行等待操作的进程。挂起的进程被放置在该条件变量的块队列中。
  - `Signal()` operation(or broadcast() operation)
    - 唤醒等待的进程，需要进程存在

![monitor](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch10%20Semaphore and Pipe/1f9Az-JtUUhn_BrAQYag6VA.jpeg)



```c
class Condition{
		int numWaiting = 0;
		WaitQueue q;
};


Condition::Wait(lock){
		numWaiting++;
		Add this thread t to q;
		release(lock);
		schedule(); // 选择下一个进程执行，选择就绪进程执行
		require(lock);
}

Condition::Signal(){
		if(numWaiting > 0){
				Remove a thread t from q;
				wakeup(t); // 唤醒进程，将睡眠进程置为就绪状态
				numWaiting--;
		}
}
```



使用monitor，抽象出一个管程，并用word满足管程的锁和条件变量，word将计时器和控制信息聚合了，只有初始化时的结构才能获得锁：

- **Wait() ：**如果定义了成员变量，则等待函数获取互斥锁
- **Signal() ：**释放获取的锁，以便其他线程可以获取它。

```go
package main

import (
	"fmt"
	"math/rand"
	"strconv"
	"sync"
	"time"
)

// 一个管程
type Monitor interface {
	Wait()
	Signal()
	GetData() []string // 返回整个数组，
	PutData(string)    //原子操作
}

// 管程的实现
type Words struct {
	mutex         *sync.Mutex
	wordsArray    []string
	isInitialized bool
}

func (m *Words) Wait(p int) {

	if m.isInitialized {
		fmt.Printf("process %d got a lock\n", p)
		m.mutex.Lock()
	}
	fmt.Printf("process %d not get a lock\n", p)
}
func (m *Words) Signal(p int) {
	if m.isInitialized {
		fmt.Printf("process %d release a lock\n", p)
		m.mutex.Unlock()
	}
}
func (m *Words) GetData() []string { return m.wordsArray }

func (m *Words) PutData(word string, pn int) {
	m.Wait(pn)
	fmt.Printf("start process %d \n", pn)
	// critical section
	m.wordsArray = append(m.wordsArray, word)
	time.Sleep(time.Millisecond * time.Duration(rand.Intn(800)))
	// critical section done
	fmt.Printf("process %d done.\n", pn)
	m.Signal(pn)
}

func (m *Words) Init() {
	m.mutex = &sync.Mutex{}
	m.wordsArray = []string{}
	m.isInitialized = true
}

func main() {
	m := &Words{}
	m.Init()
	wg := &sync.WaitGroup{}
	wg.Add(10)
	for n := 0; n < 10; n++ {
		go func(i int) {
			defer wg.Done()
			m.PutData("Angad"+strconv.Itoa(rand.Intn(100000)), i)
		}(n)
	}
	wg.Wait()
	fmt.Println(m.GetData())
}
```

![image-20220414184847152](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch10%20Semaphore and Pipe/image-20220414184847152.png)

> Reference
>
> [monitor types](https://pages.mtu.edu/~shene/NSF-3/e-Book/MONITOR/monitor-types.html)
>
> [monitor implement](https://medium.com/@angadsharma1016/process-synchronization-monitors-in-go-d31f4c42fce7)
>
> https://www.cs.utexas.edu/users/lorenzo/corsi/cs372h/07S/notes/Lecture12.pdf
>
> https://medium.com/algorithm-solving/os-synchronization-2-semaphore-and-classical-problems-of-synchronization-fdbbcb027b79

## question of synchronize 

### Readers-Writers问题

`Readers-Writers Problem` 是允许多个进程读临界区，多个写者修改临界区；该问题的约束:

- 允许同一时间有多个读者，但在任何时候只有一个写者
- 没有写者时，读者才能访问数据
- 没有读者和写者时，写者才能访问数据
- 在任何时候只能有一个线程可以操作共享变量

读进程

```
wait(mutex); // 修改计数器，因为保证计数器互斥，需要加锁
	rc++;
	if (rc == 1)
		wait(wrt);  // 保证不会有写进入
signal(mutex);

// critical section

// critical section END
wait(mutex); // release rc 
	rc--;
	if (rc == 0) // 计数器为0则代表已经无读进程，
	signal (wrt); // 释放读写锁
signal(mutex);
```

上面代码 `mutex` 和 `wrt` 是信号量，`rc` 是读进程计数器，初始化时为0

写进程

```
wait(wrt);
// critical section
signal(wrt);
```

**基于管程的实现**

```
AR = 0; // # 活跃的读者进程
AW = 0; // # 活跃的写者进程
WR = 0; // # 等待的读者进程
WW = 0; // # 等待的写者进程

Condition okToRead;
Condition okToWrite;
Lock lock;

//writer
Public Database::Write(){
		//Wait until no readers/writers;
		StartWrite();
		write database;
		//check out - wake up waiting readers/writers;
		DoneWrite();
}

Private Database::StartWrite(){
		lock.Acquire();
		// 写优先，存在任意读 写进程都将被阻塞直到满足条件
		while((AW + AR) > 0){
				WW++;
				okToWrite.wait(&lock);
				WW--;		
		}
		AW++;
		lock.Release();
}

Private Database::DoneWrite(){
		lock.Acquire();
		AW--;
		if(WW > 0){
				okToWrite.signal();  // signal是唤醒一个
		}
		else if(WR > 0){
				okToRead.broadcast(); // broadcase是唤醒所有
		}
		lock.Release();
}

//reader
Public Database::Read(){
		//Wait until no writers;
		StartRead();
		read database;
		//check out - wake up waiting writers;
		DoneRead();
}

Private Database::StartRead(){
		lock.Acquire();
		while(AW + WW > 0){    //关注等待的writer,体现出写者优先
				WR++;
				okToRead.wait(&lock);
				WR--;
		}
		AR++;
		lock.Release();
}

private Database::DoneRead(){
		lock.Acquire();
		AR--;
		if(AR == 0 && WW > 0){  //只有读者全部没有了,才需要唤醒
				okToWrite.signal();
		}
		lock.Release();
}
```

**其他实现方式**

通常情况下为了保证读写问题，一般会通过互斥或信号量来实现。然而，go中提供了**读写锁** `sync.RWMutex` 是为了解决这个问题的一种数据结构。

```go
package main

import (
	"fmt"
	"math/rand"
	"sync"
	"time"
)

var mutex = new(sync.RWMutex)
var cs = []string{}

func writer(data string) {
	mutex.Lock()
	defer mutex.Unlock()
	cs = append(cs, fmt.Sprintf("updated with %s", data))

	// Write to data.
}

func reader(data string) {

	fmt.Printf("%s start execute.\n", data)
	mutex.RLock()
	defer mutex.RUnlock()
	fmt.Println(cs)
	time.Sleep(time.Millisecond * time.Duration(rand.Intn(800)))
	// Read from data.
}

func main() {
	wg := &sync.WaitGroup{}
	wg.Add(12)
	for i := 0; i < 2; i++ {
		go func(i int) {
			go writer(fmt.Sprintf("writer thread %d", i))
			wg.Done()
			time.Sleep(time.Millisecond * time.Duration(rand.Intn(800)))
		}(i)
	}
	for i := 0; i < 10; i++ {
		go func(i int) {
			reader(fmt.Sprintf("reader thread %d", i))
			wg.Done()
		}(i)
	}

	wg.Wait()
}
```

### 哲学家就餐问题

哲学家就餐问题 `dining philosophers problem`；有五位哲学家，餐厅中间是一张圆桌，但只有五根筷子，每次吃饭需要两根筷子；当哲学家饿了就会拿起离自己最近的两根筷子；如果可以同时拿起离自己最近的两根筷子吃饭，吃完饭后，放下筷子思考。

<center><img src="https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch10%20Semaphore and Pipe/DIAGRAM-philosopher.jpg" alt="img" style="zoom:100%;" /></center>

#### 如何设计

![img](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch10%20Semaphore and Pipe/DIAGRAM-philosopher-cycle.jpg)

如图所示首先，哲学家们处于的状态 思考------拿筷子------吃饭------放下筷子------思考的状态中变化。

吃就是对临界区的访问，而如何拿筷子才是重点问题，而筷子则是 整个问题中的 **Race Condition**；可以将每根筷子互斥锁保护的共享对象，而在吃饭前，对其左右筷子进行加锁，两把锁都加成功，视为可以吃饭（访问临界区），吃完饭解锁筷子，进行思考

![img](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch10%20Semaphore and Pipe/DIAGRAM-philosopher-flow.jpg)

共享数据有：

- Bowl of rice(data set)
- Semaphone chopsticks [5] initialized to 1

步骤：

- `think()`:
- `pickUpChopsticks()`：
- `eating()`
- `PutDownChopsticks()`

```cpp
#define N 5 // 哲学家数量

void philosopher(int i) { // 哲学家编号
    while(TRUE) {
        think();					// 思考
        PickUpChopsticks(i);       // 拿起左边的筷子
        PickUpChopsticks((i+1)%N); // 拿起右边的筷子，筷子保证不大于哲学家数量
        eat();		// 吃饭
        PutDownChopsti(i);		// 放下左边的筷子
        PutDownChopsti((i+1)%N); // 放下右边的筷子
    }
}
```

这样在哪左边筷子完成时，再拿右边筷子时发现都被占用拿不到，而又不满足吃饭条件，导致死锁。为了防止死锁问题需要进一步的判断

```cpp
#define N 5 // 哲学家数量

void philosopher(int i) { // 哲学家编号
    while(TRUE) {
        think();					// 思考
        PickUpChopsticks(i);       // 拿起左边的筷子
        if (chopsticks((i+1)%N)){
            PickUpChopsticks((i+1)%N); // 拿起右边的筷子，筷子保证不大于哲学家数量
            break;
        } else { 
            // 不存在则放下左边筷子，并阻塞
            PutDownChopsti(i);		// 放下左边的筷子
            wait()
        }
        
        eat();		// 吃饭
        PutDownChopsti(i);		// 放下左边的筷子
        PutDownChopsti((i+1)%N); // 放下右边的筷子
    }
}
```

互斥访问，可以解决但是同时只能一个哲学家就餐；这里将就餐看为临界区，而不是筷子，会造成筷子资源的浪费

```cpp
#define N 5 // 哲学家数量

void philosopher(int i) { // 哲学家编号
    while(TRUE) {
        p(mutex)
            think();					// 思考
            PickUpChopsticks(i);       // 拿起左边的筷子
            PickUpChopsticks((i+1)%N); // 拿起右边的筷子，筷子保证不大于哲学家数量
            eat();		// 吃饭
            PutDownChopsticks(i);		// 放下左边的筷子
            PutDownChopsticks((i+1)%N); // 放下右边的筷子
        v(mutex)
    }
}
```

为了防止死锁的发生，可以设置两个条件：

- 必须同时拿起左右两根筷子；
- 只有在两个邻居都没有进餐的情况下才允许进餐。
- 这种就诞生了一个原则：要么不拿，要么拿两个：
  - step1：thinking
  - step2：Hungry
  - step3：左右邻居正在就餐则等待，否则下一步
  - step4：拿起两个筷子
  - step5：eating
  - step6：方向左边筷子
  - step7：方下右边筷子
  - step8：to step1

```cpp
#define N 5  // 哲学家数量
#define LEFT (i + N - 1) % N // 左邻居
#define RIGHT (i + 1) % N    // 右邻居
#define THINKING 0
#define HUNGRY   1
#define EATING   2
typedef int semaphore;
int state[N];                // 跟踪每个哲学家的状态
semaphore mutex = 1;         // 临界区的互斥，临界区是 state 数组，对其修改需要互斥
semaphore s[N];              // 同步信号量，每个哲学家一个信号量

void philosopher(int i) {
    while(TRUE) {
        think(i);   // step1
        take_two(i); // step2~4
        eat(i);	// step5
        put_two(i); // step6~7
    }
}

void take_two(int i) {
    P(&mutex);
    state[i] = HUNGRY; // 饿了
    checkChopsticks(i);  // 拿筷子
    V(&mutex);
    // 离开临界区
    P(&state[i]); // 
}

void put_two(i) {
    P(&mutex);
    state[i] = THINKING;
    // 尝试通知左右邻居，自己吃完了，你们可以开始吃了
    checkChopsticks(LEFT); 
    checkChopsticks(RIGHT);
    V(&mutex);
}

void eat(int i) {
    down(&mutex);
    state[i] = EATING;
    up(&mutex);
}

// 检查两个邻居是否都没有用餐，如果是的话，就 up(&s[i])，使得 down(&s[i]) 能够得到通知并继续执行
void checkChopsticks(i) {   
    // 第一个，当前哲学家饿了
    // 左边和右边都没有在吃饭
    if(state[i] == HUNGRY && state[LEFT] != EATING && state[RIGHT] !=EATING) {
        state[i] = EATING;
        V(&s[i]);
    }
}
```

具体的实现：

```go
package main

import (
	"log"
	"math/rand"
	"sync"
	"time"
)

const (
	THINKING = iota
	HUNGRY
	EATING
)

type chopstick struct {
	sync.Mutex
}

type Philosopher struct {
	Id    int
	Left  *chopstick
	Right *chopstick
	State int
}

func init() {
	log.SetFlags(log.Ldate | log.Lmicroseconds | log.Lshortfile)
}

// 哲学家
var ph = []string{"Mark", "Russell", "Rocky", "Haris", "Root"}

// 同时可以吃饭的哲学家数量
var host = make(chan int, int(len(ph)/2))
var wg sync.WaitGroup

func (p *Philosopher) think() {
	log.Printf("Philosopher %s start thinging.\n", ph[p.Id])
	time.Sleep(time.Millisecond * time.Duration(rand.Intn(1000)))
}

func (p *Philosopher) hungry() {
	log.Printf("Philosopher %s has hungry.\n", ph[p.Id])
	time.Sleep(time.Millisecond * time.Duration(rand.Intn(500)))
}

func (p *Philosopher) pickUpChopsticks() {
	// 两个筷子被锁，则阻塞
	p.Left.Lock()
	p.Right.Lock()
	log.Printf("Philosopher %s pick up two chopsticks.\n", ph[p.Id])
}

func (p *Philosopher) eating() {
	// 有两个哲学家在吃，阻塞
	host <- p.Id
	p.State = EATING
	p.pickUpChopsticks()
	log.Printf("Philosopher %s begin eating.\n", ph[p.Id])
	time.Sleep(time.Millisecond * time.Duration(rand.Intn(10000)))
	p.putDonwChopsticks()
	<-host
}

func (p *Philosopher) putDonwChopsticks() {
	p.Left.Unlock()
	p.Right.Unlock()
	log.Printf("Philosopher %s put down two chopsticks.\n", ph[p.Id])
}

func (p *Philosopher) seat() {
	for n := 0; n < 3; n++ {
		p.think()
		p.hungry()
		p.eating()
	}
	wg.Done()
}

func main() {
	// 创建五根筷子
	ChopSticks := make([]*chopstick, 5)
	for i := 0; i < 5; i++ {
		ChopSticks[i] = new(chopstick)
	}

	// 创建五个哲学家
	philosophers := make([]*Philosopher, len(ph))

	for n := 0; n < len(ph); n++ {
		philosophers[n] = &Philosopher{
			Id:    n,
			Left:  ChopSticks[n],
			Right: ChopSticks[(n+1)%len(ph)],
			State: THINKING,
		}
	}
	// 哲学家就坐
	for i := 0; i < 5; i++ {
		wg.Add(1)
		go philosophers[i].seat()
	}

	wg.Wait()
}
```

可以从执行结果看到，同时满足左右筷子都可以拿起的哲学家才可以进程吃

![image-20220416192756283](https://raw.githubusercontent.com/CylonChau/OperatingSystemNotes/main/images/ch10%20Semaphore and monitors/image-20220416192756283.png)

> Reference
>
> [read-write problom](https://medium.com/algorithm-solving/os-synchronization-2-semaphore-and-classical-problems-of-synchronization-fdbbcb027b79)
>
> [ Dining Philosophers Problem](https://pages.mtu.edu/~shene/NSF-3/e-Book/index.html)

