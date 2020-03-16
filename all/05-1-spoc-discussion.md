#lec11: 进程／线程概念spoc练习

## 视频相关思考题

### 11.1 进程的概念

1. 什么是程序？什么是进程？


2. 进程有哪些组成部分？


3. 请举例说明进程的独立性和制约性的含义。


4. 程序和进程联系和区别是什么？


### 11.2 进程控制块

1. 进程控制块的功能是什么？


2. 进程控制块中包括什么信息？


3. ucore的进展控制块数据结构定义中哪些字段？有什么作用？


### 11.3 进程状态

1. 进程生命周期中的相关事件有些什么？它们对应的进程状态变化是什么？


### 11.4 三状态进程模型

1. 运行、就绪和等待三种状态的含义？7个状态转换事件的触发条件是什么？


### 11.5 挂起进程模型

1. 引入挂起状态的目的是什么？

2. 引入挂起状态后，状态转换事件和触发条件有什么变化？

3. 内存中的什么内容放到外存中，就算是挂起状态？

### 11.6 线程的概念

1. 引入线程的目的是什么？

2. 什么是线程？

3. 进程与线程的联系和区别是什么？

### 11.7 用户线程

1. 什么是用户线程？

2. 用户线程的线程控制块保存在用户地址空间还是在内核地址空间？


### 11.8 内核线程

1. 用户线程与内核线程的区别是什么？

2. 同一进程内的不同线程可以共用一个相同的内核栈吗？

3. 同一进程内的不同线程可以共用一个相同的用户栈吗？


## 选做题
1. 请尝试描述用户线程堆栈的可能维护方法。

## 小组思考题
(1) 熟悉和理解下面的简化进程管理系统中的进程状态变化情况。
 - [简化的三状态进程管理子系统使用帮助](https://github.com/chyyuu/os_tutorial_lab/blob/master/ostep/ostep7-process-run.md)
 - [简化的三状态进程管理子系统实现脚本](https://github.com/chyyuu/os_tutorial_lab/blob/master/ostep/ostep7-process-run.py)

(2) (spoc)设计一个简化的进程管理子系统，可以管理并调度如下简化进程。在理解[参考代码](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab4/process-concept-homework.py)的基础上，完成＂YOUR CODE"部分的内容。然后通过测试用例和比较自己的实现与往届同学的结果，评价自己的实现是否正确。可２个人一组。

### 进程的状态 

 - RUNNING - 进程正在使用CPU
 - READY   - 进程可使用CPU
 - DONE    - 进程结束

### 进程的行为
 - 使用CPU, 
 - 发出YIELD请求,放弃使用CPU


### 进程调度
 - 使用FIFO/FCFS：先来先服务,
   - 先查找位于proc_info队列的curr_proc元素(当前进程)之后的进程(curr_proc+1..end)是否处于READY态，
   - 再查找位于proc_info队列的curr_proc元素(当前进程)之前的进程(begin..curr_proc-1)是否处于READY态
   - 如都没有，继续执行curr_proc直到结束

### 关键模拟变量
 - 进程控制块
```
PROC_CODE = 'code_'
PROC_PC = 'pc_'
PROC_ID = 'pid_'
PROC_STATE = 'proc_state_'
```
 - 当前进程 curr_proc 
 - 进程列表：proc_info是就绪进程的队列（list），
 - 在命令行（如下所示）需要说明每进程的行为特征：（１）使用CPU ;(2)等待I/O
```
   -l PROCESS_LIST, --processlist= X1:Y1,X2:Y2,...
   X 是进程的执行指令数; 
   Ｙ是执行CPU的比例(0..100) ，如果是100，表示不会发出yield操作
```
 - 进程切换行为：系统决定何时(when)切换进程:进程结束或进程发出yield请求

### 进程执行
```
instruction_to_execute = self.proc_info[self.curr_proc][PROC_CODE].pop(0)
```

### 关键函数
 - 系统执行过程：run
 - 执行状态切换函数:　move_to_ready/running/done　
 - 调度函数：next_proc

### 执行实例

#### 例１
```
$./process-simulation.py -l 5:50
Process 0
  yld
  yld
  cpu
  cpu
  yld

Important behaviors:
  System will switch when the current process is FINISHED or ISSUES AN YIELD
Time     PID: 0 
  1     RUN:yld 
  2     RUN:yld 
  3     RUN:cpu 
  4     RUN:cpu 
  5     RUN:yld 

```


#### 例２
```
$./process-simulation.py  -l 5:50,5:50
Produce a trace of what would happen when you run these processes:
Process 0
  yld
  yld
  cpu
  cpu
  yld

Process 1
  cpu
  yld
  cpu
  cpu
  yld

Important behaviors:
  System will switch when the current process is FINISHED or ISSUES AN YIELD
Time     PID: 0     PID: 1 
  1     RUN:yld      READY 
  2       READY    RUN:cpu 
  3       READY    RUN:yld 
  4     RUN:yld      READY 
  5       READY    RUN:cpu 
  6       READY    RUN:cpu 
  7       READY    RUN:yld 
  8     RUN:cpu      READY 
  9     RUN:cpu      READY 
 10     RUN:yld      READY 
 11     RUNNING       DONE 
```

## 问答题

#### Q1：[基础] 如何看到你正在使用的操作系统里的进程

A:命令行：tasklist、ps 、htop 等

UI：任务管理器、Activity Monitor 等

#### Q2：[基础] 简单描述一下进程地址空间都有哪些数据和代码

A:

* 数据：初始化数据、堆、栈
* 代码：程序代码、共享库（包括动态加载库、动态链接库，如 libc）
* Linux 特殊：vdso（代码） vvar（数据）
* Linux 下用 cat /proc/self/maps 看地址空间的映射
* 为什么一个文件有多个映射 -- 用 readelf -e /usr/bin/cat 看下面的 LOAD

#### Q3：[基础，开放] 进程控制块可能保存哪些内容

A:

* 寄存器
* Process ID、User ID、Group ID、Parent Process ID
* 内核栈地址
* 调度优先级
* 页表
* Exit Code、Signal
* 统计信息：运行时间、内存占用
* 进程内打开的文件描述符列表(描述文件系统中的文件、块设备、字符设备、套接字等)

#### Q4：[基础] 考虑三状态进程模型下，空闲的 Web 服务器、Sleep啥也不干型和计算密集型程序一般来说分别在什么阶段停留的时间长（不考虑退出后的时间）

A:前两个：等待；第三个：运行

#### Q5：[进阶] 主流操作系统系统中用于存储处于挂起状态的进程镜像的地方是什么

A:*nix：swap ，Windows：Pagefile

#### Q6：[进阶，开放] 操作系统怎么判断是否应该挂起一个程序

A:

* 长时间等待一个事件（网络、sleep）
* 进程内存使用
* 总内存剩余

#### Q7：[基础] 线程和进程的区别

A:

* 线程共享地址空间（页表）
* 每个线程对应自己的寄存器和栈

#### Q8：[基础，进阶] 用户线程和内核线程的关系和区别

A:

* 用户线程对操作系统透明，用户线程对内核线程可以是 M:1 或者 M:N
* 操作系统负责调度内核线程，用户程序中会有一部分代码（runtime）负责用户线程的调度
* 用户线程一样需要维护自己的寄存器和栈
* 用户线程需要针对阻塞的系统调用进行特殊处理

#### Q9：[开放] 你都了解哪些侧信道攻击和对应的解决方案

A:

* Meltdown 等：针对 Cache 访问时间的侧信道攻击
  * 解决方案：KPTI、CPU 微码更新、控制计时函数精度
* 加密算法：如果可以构造密文使得加解密的时间或者功耗与密钥的某一个位有关系（比如出现 if else 分支），可以通过统计足够次数的数据得到密钥的信息

#### Q10：在等待 I/O 的时候，从运行状态转换到等待状态，相比在程序中直接循环等待 I/O 结束有什么优缺点？

A:优点：对于 I/O 速度远远低于 CPU 频率的情况，可以更为有效地利用 CPU 资源；

对于 I/O 速度接近 CPU 频率的情况，反而直接循环等待效率比较高，因为无需上下文切换的开销。

#### Q11：不同的用户线程是否使用相同的内核栈？使用用户线程的效率是否更高？

A:硬件线程(hardware thread)：指 CPU 中一个独立的指令执行流的单位，包含有自己的寄存器状态与 PC。比如经典的 8 核 16 线程就是指硬件线程，这里核数不等于硬件线程数是因为超线程技术。 

软件线程包含内核线程(OS thread)以及用户线程(green thread) 。在现代 OS 设计中，每个进程可以包含多个内核线程。每个内核线程为进程中一条独立的指令执行流，不仅需要占用一个硬件线程(一部分 CPU 资源与寄存器)，还需要在进程地址空间中分配一个线程独立的栈。该栈按照访问特权级分为两部分：用户栈和内核栈，其中用户栈在跑用户态代码的时候使用，在产生中断/系统调用的时候则切换到内核栈。内核线程作为内核内部进行 CPU 资源的调度单位而存在。 

内核向应用程序提供了若干与内核线程有关的系统调用，如创建内核线程、等待内核线程运行结束等等。然而，为了编写一个多线程应用程序来合理利用多核资源，我们不一定直接通过系统调用直接使用内核线程模型，还可以通过用户线程库在用户态自己完成线程的管理与调度。这种用户线程库(通常是一个 runtime 或者 VM)可以通过封装内核线程的接口来实现，甚至不需要内核对于线程的支持全部自己实现。如玩具项目 [gthreads](https://github.com/mpu/gthreads) 就没有使用任何内核线程支持，完全在用户态实现了线程切换与管理；而 Golang 则是将每个内核线程复用为多个轻量级线程 goroutine。

 因此，不同的用户线程不一定使用相同的内核栈；而使用用户线程库还是直接用内核线程效率更高则取决于实际情况。