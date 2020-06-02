# lec15: 调度算法概念 spoc 思考题


- 有"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的ucore_code和os_exercises的git repo上。


## 视频相关思考题

### 15.1 处理机调度概念

1. 处理机调度的功能是什么？

 > 选线程：从就绪队列中挑选下一个占用CPU运行的线程

 > 选CPU：从多个可用CPU中挑选就绪线程可使用的CPU资源

2. 在什么时候可以进行处理机调度？

 > 调度时机的所有可能：

 > 占用CPU运行的线程的主动放弃（退出、等待）

 > 当前线程被抢先（时间片用完、高优先级线程就绪）

3. 当操作系统的处理机调度导致线程切换时，暂停线程的当前指令指针可能在什么位置？用户态代码或内核代码？给出理由。

 > 系统调用返回时可能出现线程切换

### 15.2 调度准则

1. 处理机的使用模式有什么特征？

 > CPU与I/O交替使用；

 > 每次占用CPU执行指令的时间长度分布多数在10ms以内；

2. 处理机调度的目标是什么？

 > CPU利用率（CPU使用率、吞吐率、周转时间、等待时间）

 > 用户感受（周转时间、等待时间、响应时间、响应时间的方差）

 > 公平性（CPU时间分配公平性）

3. 尝试在ucore上写一个外排序程序，然后分析它的执行时间分布统计（每次切换后开始执行时间和放弃CPU的时间、当前用户和内核栈信息）。

4. 在Linux上有一个应用程序time，可以统计应用程序的执行时间信息。请分析它是如何统计进程执行时间信息的。如可能，请在ucore上实现相同功能的应用程序。下面是可能的参考。
   - [Linux用户态程序计时方式详解](http://www.cnblogs.com/clover-toeic/p/3845210.html)
   - [Get Source Code for any Linux Command](http://www.thegeekstuff.com/2010/02/get-source-code-for-any-linux-command/)
   - [How does time command work](http://unix.stackexchange.com/questions/29800/how-does-time-command-work)
   - https://github.com/illumos/illumos-gate/blob/master/usr/src/cmd/time/time.c

5. 尝试获取一个操作系统的调度算法的性能统计数据（CPU使用率、进程执行）。

### 15.3 先来先服务、短进程优先和最高响应比优先调度算法

1. 尝试描述下列处理机调度算法的工作原理和算法优缺点。
   - 先来先服务算法（FCFC、FIFO）
   - 短进程优先算法（SPN、SJF）
   - 最高响应比优先算法（HRRN）

 > 就绪队列的排队依据：到达时间、进程预期执行时间、按响应比R=(w+s)/s

 > 算法特征：平均等待时间、等待时间方差、资源利用率

2. 为什么短进程优先算法的平均周转时间最优？

 > 可以证明知进程优先算法的调度顺序的平均周转时间是最短的。

3. 如何调试你的调度算法？

 > 监控调度算法的执行状态、调度结果、算法特征等信息。

### 15.4 时间片轮转、多级反馈队列、公平共享调度算法和ucore调度框架

1. 尝试描述下列处理机调度算法的工作原理和算法优缺点。
   - 时间片轮转算法（RR）
   - 多级反馈队列算法（MLFQ）
   - 公平共享调度算法（FSS）

 > 让出CPU的条件：进程执行时间

 > 就绪队列排队依据：多队列、多种排队依据、依据进程特征调整所在队列

 > 算法特征：响应时间、平均周转时间、不同类型进程的调度差异

2. RR算法选择时间片长度的依据有哪些？

 > 进程切换开销

 > 进程当前操作占用的CPU时间

 > 切换开销与响应时间的折中权衡

3. 请描述在MLFQ中，如何提升或降低进程的优先级？

 > I/O操作后提升优先级

 > 时间片用完后降低优先级

4. 尝试跟踪ucore中进程切换和调度算法的就绪进程选择过程。

 > 中断响应、进程的中断现场保存、中断处理、调度判断、进程切换、新进程的中断现场恢复、新进程的继续执行

5. 为什么要引入调度框架？定义调度算法接口需要考虑哪些因素？

 > 引入调度框架的目的：分离调度操作和调度策略，以支持多种调度算法；

 > 定义调度算法接口的考虑因素：调度算法需要的调度操作类型、调度算法在各调度操作中的体现方式；

6. 通过观察FIFO、SJF和RR调度算法的[模拟程序](https://github.com/chyyuu/os_tutorial_lab/blob/master/ostep/ostep8-scheduler.py)运行结果及其[描述文档](https://github.com/chyyuu/os_tutorial_lab/blob/master/ostep/ostep8-scheduler.md)，理解其工作原理和算法特征。

7. 通过观察MLFQ调度算法的[模拟程序](https://github.com/chyyuu/os_tutorial_lab/blob/master/ostep/ostep9-mlfq.py)运行结果及其[描述文档](https://github.com/chyyuu/os_tutorial_lab/blob/master/ostep/ostep9-mlfq.md)，理解其工作原理和算法特征。

8. 通过观察彩票调度算法([Lottery scheduling](https://en.wikipedia.org/wiki/Lottery_scheduling))的[模拟程序](https://github.com/chyyuu/os_tutorial_lab/blob/master/ostep/ostep10-lottery.py)运行结果及其[描述文档](https://github.com/chyyuu/os_tutorial_lab/blob/master/ostep/ostep10-lottery.md)，理解其工作原理和算法特征。

### 15.5 实时调度和多处理器调度

1. 什么是实时操作系统？

 > 实时操作系统是保证在一定时间限制内完成特定功能的操作系统。

 > 参考： http://baike.baidu.com/item/%E5%AE%9E%E6%97%B6%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F

2. 什么是可调度性？

 > 可调度表示一个实时操作系统能够满足任务时限要求

3. 尝试描述下列处理机调度算法的工作原理和算法优缺点。
   - 速率单调调度算法(RM, Rate Monotonic) 
   - 最早截止时间优先算法 (EDF, Earliest Deadline First) 

 > 任务优先级定义：按周期大小排队、按截止时间先后排队

 > [RM](https://en.wikipedia.org/wiki/Rate-monotonic_scheduling)的可调度条件：CPU利用率小于ln2时，是可调度的。

 > [EDF](https://en.wikipedia.org/wiki/Earliest_deadline_first_scheduling)的可调度条件：CPU利用率小于100%。

4. 有兴趣的同学，请阅读下面论文，然后说明实时调度面临的主要困难是什么？
   - [Buttazzo, “Rate monotonic vs. EDF: Judgement Day”, EMSOFT 2003.](http://www.eecs.umich.edu/courses/eecs571/reading/rm-vs-edf.pdf)
   - [单调速率及其扩展算法的可调度性判定](http://www.jos.org.cn/ch/reader/create_pdf.aspx?file_no=20040602)

5. 多处理机调度中每个处理机一个就绪队列与整个系统一个就绪队列有什么不同？

 > 区别：调度开销、负载均衡程度

### 15.6 优先级反置

1. 什么是优先级反置现象？

 > 操作系统中出现高优先级进程长时间等待低优先级进程所占用资源的现象

2. 什么是优先级继承(Priority Inheritance)和优先级天花板协议(priority ceiling protocol)？它们的区别是什么？

 > 优先级继承：占用资源的低优先级进程继承申请资源的高优先级进程的优先级

 > 优先级天花板协议：占用资源进程的优先级和所有可能申请该资源的进程的最高优先级相同

 > 区别：提升占用资源的低优先级进程的优先级的时机不同

3. 在单CPU情况下，基于以前的OS知识，能否设计一个更简单的方法（也许执行效率会低一些）解决优先级反置现象？

4. ppt中“优先级继承”页里的图示有误，你能指出来吗？

## 小组练习与思考题

参考往届同学的处理机调度算法实现练习，从下列8个算法中选择一个你感兴趣的调度算法，对其实现进行完善，并分析算法特征。
 - [2016春季-第十五讲 课堂思考题回答-向勇班](https://piazza.com/class/i5j09fnsl7k5x0?cid=803)
 - [陈渝班-2016春季-第15讲 课堂思考题回答](https://piazza.com/class/i5j09fnsl7k5x0?cid=806)

### (1)(spoc) 理解并实现FIFO调度算法

可基于“python, ruby, C, C++，LISP、JavaScript”等语言模拟实现，并给出测试，在试验报告写出设计思路和测试结果分析。

请参考[scheduler-homework.py](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab6/scheduler-homework.py)代码或独自实现。
最后统计采用不同调度算法的每个任务的相关时间和总体的平均时间：

　- turnaround time　周转时间
　- response time 响应时间
　- wait time　等待时间

#### 对模拟环境的抽象
- 任务/进程，及其执行时间
  Job 0 (length = 1)
  Job 1 (length = 4)
  Job 2 (length = 7)

 - 何时切换？
 - 如何统计？

#### 执行结果

采用FIFO调度算法

```
 ./scheduler-homework.py  -p FIFO
ARG policy FIFO
ARG jobs 3
ARG maxlen 10
ARG seed 0

Here is the job list, with the run time of each job: 
  Job 0 ( length = 9 )
  Job 1 ( length = 8 )
  Job 2 ( length = 5 )


** Solutions **

Execution trace:
  [ time   0 ] Run job 0 for 9.00 secs ( DONE at 9.00 )
  [ time   9 ] Run job 1 for 8.00 secs ( DONE at 17.00 )
  [ time  17 ] Run job 2 for 5.00 secs ( DONE at 22.00 )

Final statistics:
  Job   0 -- Response: 0.00  Turnaround 9.00  Wait 0.00
  Job   1 -- Response: 9.00  Turnaround 17.00  Wait 9.00
  Job   2 -- Response: 17.00  Turnaround 22.00  Wait 17.00

  Average -- Response: 8.67  Turnaround 16.00  Wait 8.67

```
### (2)(spoc) 理解并实现SJF调度算法

可基于“python, ruby, C, C++，LISP、JavaScript”等语言模拟实现，并给出测试，在试验报告写出设计思路和测试结果分析。

#### 执行结果

采用SJF调度算法
```
 ./scheduler-homework.py  -p SJF
ARG policy SJF
ARG jobs 3
ARG maxlen 10
ARG seed 0

Here is the job list, with the run time of each job: 
  Job 0 ( length = 9 )
  Job 1 ( length = 8 )
  Job 2 ( length = 5 )


** Solutions **

Execution trace:
  [ time   0 ] Run job 2 for 5.00 secs ( DONE at 5.00 )
  [ time   5 ] Run job 1 for 8.00 secs ( DONE at 13.00 )
  [ time  13 ] Run job 0 for 9.00 secs ( DONE at 22.00 )

Final statistics:
  Job   2 -- Response: 0.00  Turnaround 5.00  Wait 0.00
  Job   1 -- Response: 5.00  Turnaround 13.00  Wait 5.00
  Job   0 -- Response: 13.00  Turnaround 22.00  Wait 13.00

  Average -- Response: 6.00  Turnaround 13.33  Wait 6.00
```
### (3)(spoc) 理解并实现RR调度算法

可基于“python, ruby, C, C++，LISP、JavaScript”等语言模拟实现，并给出测试，在试验报告写出设计思路和测试结果分析。

#### 执行结果


采用RR调度算法
```
 ./scheduler-homework.py  -p RR
ARG policy RR
ARG jobs 3
ARG maxlen 10
ARG seed 0

Here is the job list, with the run time of each job: 
  Job 0 ( length = 9 )
  Job 1 ( length = 8 )
  Job 2 ( length = 5 )


** Solutions **

Execution trace:
  [ time   0 ] Run job   0 for 1.00 secs
  [ time   1 ] Run job   1 for 1.00 secs
  [ time   2 ] Run job   2 for 1.00 secs
  [ time   3 ] Run job   0 for 1.00 secs
  [ time   4 ] Run job   1 for 1.00 secs
  [ time   5 ] Run job   2 for 1.00 secs
  [ time   6 ] Run job   0 for 1.00 secs
  [ time   7 ] Run job   1 for 1.00 secs
  [ time   8 ] Run job   2 for 1.00 secs
  [ time   9 ] Run job   0 for 1.00 secs
  [ time  10 ] Run job   1 for 1.00 secs
  [ time  11 ] Run job   2 for 1.00 secs
  [ time  12 ] Run job   0 for 1.00 secs
  [ time  13 ] Run job   1 for 1.00 secs
  [ time  14 ] Run job   2 for 1.00 secs ( DONE at 15.00 )
  [ time  15 ] Run job   0 for 1.00 secs
  [ time  16 ] Run job   1 for 1.00 secs
  [ time  17 ] Run job   0 for 1.00 secs
  [ time  18 ] Run job   1 for 1.00 secs
  [ time  19 ] Run job   0 for 1.00 secs
  [ time  20 ] Run job   1 for 1.00 secs ( DONE at 21.00 )
  [ time  21 ] Run job   0 for 1.00 secs ( DONE at 22.00 )

Final statistics:
  Job   0 -- Response: 0.00  Turnaround 22.00  Wait 13.00
  Job   1 -- Response: 1.00  Turnaround 21.00  Wait 13.00
  Job   2 -- Response: 2.00  Turnaround 15.00  Wait 10.00

  Average -- Response: 1.00  Turnaround 19.33  Wait 12.00

```

### (4)理解并实现MLFQ调度算法

可基于“python, ruby, C, C++，LISP、JavaScript”等语言模拟实现，并给出测试，在试验报告写出设计思路和测试结果分析。

### (5)理解并实现stride调度算法

可基于“python, ruby, C, C++，LISP、JavaScript”等语言模拟实现，并给出测试，在试验报告写出设计思路和测试结果分析。

### (6)理解并实现EDF实时调度算法

可基于“python, ruby, C, C++，LISP、JavaScript”等语言模拟实现，并给出测试，在试验报告写出设计思路和测试结果分析。

### (7)理解并实现RM实时调度算法

可基于“python, ruby, C, C++，LISP、JavaScript”等语言模拟实现，并给出测试，在试验报告写出设计思路和测试结果分析。

### (8)理解并实现优先级反置方法

可基于“python, ruby, C, C++，LISP、JavaScript”等语言模拟实现，并给出测试，在试验报告写出设计思路和测试结果分析。

## 问答题

### 单处理机调度

#### Q1：[基础] 非抢占式的调度算法，以及抢占式的调度算法，他们的优点各是什么？

A:非抢占式：只有必要的进程切换，减少花费在切换上的时间（更高吞吐量）；抢占式：响应速度更短，用户体验更高（更短响应时间）；

#### Q2：[进阶] Linux 2.6 引入了 “内核抢占” 技术。简单的说，如果运行内核态代码时发生时钟中断，支持内核抢占的系统在处理完时钟中断后，允许发生进程切换。引入内核抢占有什么好处？又有什么挑战？（高阶）ucore / rcore 现在是可抢占的吗？

A:
好处：提升系统响应时间，防止长时间执行的内核线程因为内核不可抢占而让系统响应时间变长。例如，耗时很长的 syscall 不会阻塞掉。
坏处（可能需要后面同步互斥的知识）：增添系统复杂性，曾经单核机器运行的内核代码很少考虑并发。引入内核抢占以后，即使单核运行的内核代码也可能出现并发问题。所以需要各种加锁。
注意内核执行过程中，不一定关中断。完成启动过程，开始执行用户程序以后，只有两种方法进入内核：interrupt / exception（exception 包括 syscall）。（例如 ucore 在 x86 下）通过 interrupt / exception 进入内核，那么内核执行过程中关中断；通过 syscall 进入内核，内核执行过程中不关中断。虽然这意味着内核执行 syscall 的时候可能被中断打断，但通常处理中断的代码和 syscall 代码不会共享太多数据，所以不会有并发问题。引入内核抢占后，内核执行 syscall 过程中，可能发生时钟中断然后内核抢占，导致内核切换到另一个内核线程。假设另一个内核线程在被打断前也在执行 syscall，那么这两个并发执行 syscall 的线程就可能导致并发问题。
ucore 没有内核抢占，代码在 trap.c：if (!in_kernel) { ... if (current->need_resched) { schedule(); } }。rcore 有内核抢占，因为 rust 本身无论你单线程还是多线程都会要求全局变量通过锁访问。

#### Q3：[基础] 假设我们简单的将任务分为两种：前台交互（要求短时延）、后台计算（任务量大）。下列任务分别是前台还是后台？a) make 编译 linux; b) vim 光标移动; c) firefox 下载影片; d) 某游戏处理玩家点击鼠标开枪; e) 播放令人愉悦的歌曲; f) 转码十分有意思的视频。除此以外，想想你日常的任务，它们哪些是前台，哪些是后台的？

A:前台：b) d) e)；后台：a) c) f)

#### Q4：[基础] RR 算法的时间片长短有什么影响？

A:短时间片减少进程响应时间；长时间片减少进程切换代价，时间片越长 RR 越像 FCFS

#### Q5：[进阶] MLFQ 算法并不公平，恶意的用户程序可以愚弄 MLFQ 算法，大幅挤占其他进程的时间。（MLFQ 的规则：“如果一个进程，时间片用完了它还在执行用户计算，那么 MLFQ 下调它的优先级”）你能举出一个例子，使得你的用户程序能够挤占其他进程的时间吗？

A:首先我们的目的就是，让进程停留在高优先级；其次我们进程每次运行不能耗时太短，不然无法挤占其他进程的时间。所以：进程需要预先知道高优先级的时间片长度 T。每次调度运行它，它先执行计算。快到 T 的时候，赶紧发起一个 I/O 让自己停留在高优先级。

#### Q6：[基础] 课上讲的 MLFQ 只提到了降低优先级，这会导致什么问题？如何解决？

A:没有考虑到进程的特点（是前台还是后台）可能随着时间变化。比如某 IDE 启动需要 30 秒，其中全是初始化的计算，但启动完成后它就变成了交互进程。按照课程 MLFQ，30 秒后 IDE 进程会是低优先级而且不能提升，导致后面用户交互体验极差。
为了解决这个问题，需要一种机制允许进程的优先级提升。最暴力的就是周期性的把所有进程优先级拉到最高。更细致的如使用 token bucket / leaky bucket（网络原理讲过），桶里面的 token 表示时间，进程执行消耗 token，token 周期性增加直到桶满。桶满则提高优先级、桶空则降低优先级
(本小题参考了 OSTEP 的 MLFQ 一章。)

#### Q7：[进阶] Linux 内核代码中，持有资源（例如 spinlock，不过具体是什么资源不重要）的时候会关闭内核抢占。仅从本节课的内容来说，请问这是为了避免什么现象？

A:避免优先级反转。这种做法相当于天花板协议，持有资源后优先级拉到最高（就等价与关闭抢占）。

#### Q8：[进阶] SMP 调度中，将进程动态指派到各个处理器上，效率方面的好处是使得各个处理器负载均衡。那么这样做，效率方面不好的地方是什么？

A:进程从一个 CPU 迁移到另一个 CPU 时，cache miss 会变多。
把一个进程 "pin" 到某个处理器上的操作叫 cpu affinity。Linux 下可以使用 taskset 来设置 cpu affinity。

### 多处理机调度

**多核处理器**

#### Q1：[基础] 对称多处理（SMP）和非一致内存访问（NUMA）最主要的区别是什么？它们分别适用于哪些场景？你面前的这台设备使用的是那种？

A:
SMP 中每个核共享同一个总线访问内存，访问内存的时间相同
NUMA 中每个核都有距离它最近的本地内存，访问本地内存比访问其它内存要快
SMP 适用于核数比较少的情况，例如 PC、手机
NUMA 适用于众核处理器，例如服务器，有些主板上能放两块 CPU
目前大部分个人设备应该都是 SMP 架构

#### Q2：[进阶] 考虑两台计算机，A 有一个 4 核 8 线程 CPU，B 有两个 4 核 4 线程的 CPU。假设每个核的性能大致相同。那么对于计算密集型和访存密集型的多线程程序，你觉得分别在哪台计算机上跑得更快一些？

A:
【参考回答，不一定正确】
计算密集型 B 更快。4 个物理核的算力肯定比超线程出来的 4 个虚拟核要强。程序对访存要求不高，放在哪儿跑都差不多。
~~访存密集型 A 更快。SMP 集中访存一般要比 NUMA 跨核访存快。超线程可以共享Cache。

**多核调度**

#### Q3：[基础] 多核执行和调度引入了哪些新的问题和挑战？

A:
亲和性问题：程序应尽量在同一个核上运行，以降低缓存失效的开销。
共享内存竞争问题：多核并发访问同一个数据结构会引入同步互斥的巨大开销。
远程控制问题：假如进程 A 要杀掉进程 B，而 B 此时正在另一个核上运行，则要通过 IPI 的方式通知那个核停止运行。
缓存一致性问题：假如同一个进程的两个线程正在两个核上同时运行，其中一个线程修改了页表，则要通知另一个核刷新相应的 TLB。

#### Q4：[基础] 工作窃取技术要解决什么问题？简述它的做法。

A:
解决多队列调度中的负载不均衡问题。
做法：负载较低的处理器去其它核的队列里“窃取”任务。

**O(1) 调度**

#### Q5：[基础] 为什么它叫 O(1)调度器？我们知道常见的数据结构（如平衡树、Hash 表、数组、链表），都无法做到插入、删除、查找复杂度均为 O(1)。那么 O(1)调度器是如何做到这一点的？

A:
它设置了 140 个优先级，每个优先级对应一个 FIFO。相当于 Hash 表+链表。
这样显然插入和删除都是 O(1)的。
而对于查找，它并不需要找到某个特定进程，而只需找到优先级最高的即可。
因此它用一个 bitmap 维护 140 个优先级是否存在进程，然后借助 CPU 的特殊指令，查找一个整数中最高为 1 的位（x86 中是`bsr`指令），由此实现 O(1)的分配。

**CFS 调度**

#### Q6：[基础] CFS 的全称是什么？它为了解决 O(1)调度的哪些问题？算法如何体现其名字蕴含的特点？

A:
完全公平调度。
O(1)调度是基于优先级的，各个进程不能公平地占有 CPU 时间。（？）
CFS 为每个进程设置权重，按照权重比例分配 CPU 时间。

#### Q7：[基础] CFS 使用红黑树的原因是什么？为什么 O(logN)的算法会比 O(1)效果好？

A:
CFS 每次需要找到 vruntime 值最小的进程，并且这个值还会动态增长，因此需要一个平衡树来维护。而红黑树是 Linux 内核中广泛使用的数据结构。
算法的效果不能单纯地从复杂度上分析，要看内核和用户程序的整体运行时间。
在同样具有大量进程数的情况下，CFS 可能在维护红黑树上的开销比 O(1)略大，但它能够更合理地调度进程，降低用户态切换的开销，整体运行效果更好。

#### Q8：[进阶] vruntime 的意义是什么？对于休眠进程 vruntime 要如何处理？

A:
vruntime 可以理解为是 真实运行时间/权重，归一化出来的虚拟运行时间。所有进程统一使用 vruntime 进行比较。

#### Q9：[进阶] 你是否认同 CFS 的做法达到了完全公平的目标？为什么？

A:
随便说。
有人认为 CFS 的策略更偏好休眠进程，做不到完全公平。

**BFS 调度**

#### Q10：[基础] BFS 的全称是什么？它为了解决 CFS 调度的哪些问题？算法如何体现其名字蕴含的特点？

A:
「“脑残调度器”。这古怪的名字有多重含义，比较容易被接受的一个说法为：它如此简单，却如此出色，这会让人对自己的思维能力产生怀疑。」
CFS 是一个复杂而通用的调度器，但它在需要快速响应的交互性程序上表现比较糟糕，使得桌面环境显得十分卡顿：「事情源于一些 Linux 用户，他们发现 Linux 虽然号称能够充分发挥 4096 颗 CPU 系统的计算能力，但在普通的 laptop 上却无法流畅地播放 Youtube 视频。」
BFS 使用单一队列，没有负载均衡，实现非常简单。这使得它代码足迹更短，延迟更小，让桌面环境变得非常流畅。

#### Q11： [扩展] BFS 蕴含了哪些哲学思想？体现了计算机系统领域的哪些特点？

A:
解决方法往往越简单效果越好。
没有银弹。在特定领域需要有专门的解决方案。

#### 关于多核调度算法的思考
我们只是简单地讨论了三个不同的 scheduler，便发现它们的实现根据其业务需求有不同的 pattern 或 trade-off。在考虑一个 scheduler 时，我们要考虑这些问题：
* 任务如何组织？是所有的资源共享一个任务的 runqueue，调度器调度时通过加锁来保证互斥，还是针对每个资源，我们都为其设置一个  runqueue，以减少锁带来的损耗？那么问题又来了，如果某个资源上的任务列表空了，资源是就此闲置，还是可以去别的资源的 runqueue  上「偷」任务给自己执行？
* 资源如何利用？是 run to completion，还是 time slicing？run to completion  对于计算密集，且在意 latency 的场景非常有价值，因而在网络设备中，除 traffic shaping 的需求外，报文的处理大多采用  run to completion。time slicing 则适用于 I/O 密集，或者在意系统总体的利用率的场景。
* 用什么数据结构组织 runqueue？是 FIFO queue (linked list)，还是 rb tree，还是 bitmap + FIFO queue，各种结构的优劣如何？
* 使用什么算法？是用一个 while loop 去 O(N) 地遍历来寻找最合适的任务，还是对于任意任务 round  robin（weighted round robin）？Linux 2.6 O(1) scheduler 使用的是什么样的算法？CFS  (Completely Fair Scheduler) 为何又将其取而代之？对于目前大集群下的 cluster  scheduling，scheduler 如何处理，borg / mesos /omega 这些先后名噪江湖的算法是怎么回事，该怎么选用？