#lec12: 进程／线程控制spoc练习

## 视频相关思考题
### 12.1 进程切换

1. 进程切换的可能时机有哪些？

2. 分析ucore的进程切换代码，说明ucore的进程切换触发时机和进程切换的判断时机都有哪些。

3. ucore的进程控制块数据结构是如何组织的？主要字段分别表示什么？

### 12.2 进程创建

1. fork()的返回值是唯一的吗？父进程和子进程的返回值是不同的。请找到相应的赋值代码。

2. 新进程创建时的进程标识是如何设置的？请指明相关代码。

3. 请通过fork()的例子中进程标识的赋值顺序说明进程的执行顺序。

4. 请在ucore启动时显示空闲进程（idleproc）和初始进程（initproc）的进程标识。

5. 请在ucore启动时显示空闲线程（idleproc）和初始进程(initproc)的进程控制块中的“pde_t *pgdir”的内容。它们是否一致？为什么？

### 12.3 进程加载

1. 加载进程后，新进程进入就绪状态，它开始执行时的第一条指令的位置，在elf中保存在什么地方？在加载后，保存在什么地方？
2. 第一个用户进程执行的代码在哪里？它是什么时候加载到内存中的？

### 12.4 进程等待与退出

1. 试分析wait()和exit()的结果放在什么地方？exit()是在什么时候放进去的？wait()在什么地方取到出的？

2. 试分析ucore操作系统内核是如何把子进程exit()的返回值传递给父进程wait()的？

3. 什么是僵尸进程和孤儿进程？

4. 试分析sleep()系统调用的实现。在什么地方设置的定时器？它对应的等待队列是哪个？它的唤醒操作在什么地方？

5. 通常的函数调用和函数返回都是一一对应的。有不是一一对应的例外情况？如果有，请举例说明。

## 小组思考题

(1) (spoc)设计一个简化的进程管理子系统，可以管理并调度支持“就绪”和“等待”状态的简化进程。给出了[参考代码](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab5/process-cpuio-homework.py)，请理解代码，并完成＂YOUR CODE"部分的内容．　可２个人一组

### 进程的状态 
```
 - RUNNING - 进程正在使用CPU
 - READY   - 进程可使用CPU
 - WAIT    - 进程等待I/O完成
 - DONE    - 进程结束
```

### 进程的行为
```
 - 使用CPU, 
 - 发出YIELD请求,放弃使用CPU
 - 发出I/O操作请求,放弃使用CPU
```

### 进程调度
 - 使用FIFO/FCFS：先来先服务, 只有进程done, yield, io时才会执行切换
   - 先查找位于proc_info队列的curr_proc元素(当前进程)之后的进程(curr_proc+1..end)是否处于READY态，
   - 再查找位于proc_info队列的curr_proc元素(当前进程)之前的进程(begin..curr_proc-1)是否处于READY态
   - 如都没有，继续执行curr_proc直到结束

### 关键模拟变量
 - io_length : IO操作的执行时间
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
   Ｙ是执行yield指令（进程放弃CPU,进入READY状态）的比例(0..100) 
   Ｚ是执行I/O请求指令（进程放弃CPU,进入WAIT状态）的比例(0..100)
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

#### 例1
```
$./process-simulation.py  -l 5:30:30,5:40:30 -c
Produce a trace of what would happen when you run these processes:
Process 0
  io
  io
  yld
  cpu
  yld

Process 1
  yld
  io
  yld
  yld
  yld

Important behaviors:
  System will switch when the current process is FINISHED or ISSUES AN YIELD or IO
Time     PID: 0     PID: 1        CPU        IOs 
  1      RUN:io      READY          1            
  2     WAITING    RUN:yld          1          1 
  3     WAITING     RUN:io          1          1 
  4     WAITING    WAITING                     2 
  5     WAITING    WAITING                     2 
  6*     RUN:io    WAITING          1          1 
  7     WAITING    WAITING                     2 
  8*    WAITING    RUN:yld          1          1 
  9     WAITING    RUN:yld          1          1 
 10     WAITING    RUN:yld          1          1 
 11*    RUN:yld       DONE          1            
 12     RUN:cpu       DONE          1            
 13     RUN:yld       DONE          1            
```
## 问答题

#### Q1：[基础] 上下文切换需要保存哪些东西

A:通用寄存器；PC（RIP）、SP；（可能）浮点寄存器

#### Q2：[基础] 上下文切换为什么需要用汇编语言实现

A:上下文切换需要保证寄存器被正确地保存和恢复；用 C 语言编写很容易引入额外的指令使得上下文切换读/写了错误的信息

#### Q3：[进阶] 针对上下文切换，对于不需要浮点指令的程序，如何优化上下文切换的性能

A:
优化的必要性：浮点寄存器很多，保存和恢复较慢
优化方法：程序刚运行的时候默认关闭浮点指令支持，上下文切换时不处理浮点寄存器；当程序运行到浮点指令的时候，会产生异常进入内核，内核读取指令，发现是一个浮点指令，于是打开浮点指令支持（RISC-V 的 FS bit）；在此之后，针对此程序进行浮点寄存器的保存和恢复。更进一步，可以根据是否有浮点寄存器的更新来决定是否进行保存。

#### Q4：[基础] fork 为什么需要在父进程和子进程提供不同的返回值

A:让父进程可以知道自己创建的子进程的 pid；让子进程可以知道自己是子进程，父进程的 pid 可以通过 getppid 获得

#### Q5：[进阶] 可以怎样在 fork 里面实现上述功能

A:在创建新进程完成的时候，分别修改父进程和子进程的上下文，修改返回值对应的寄存器

#### Q6：[进阶] 为什么子进程退出后一定需要父进程对它进行 wait 才能被完全销毁？

A:
1. 因为子进程的 exit code 等信息需要提供给父进程
2. 父进程对子进程知道的只有 pid，所以即使子进程退出了，这个 pid 也暂时不能用到其他进程
3. 在父进程处理了 exit code 之后，才可以释放子进程和它的 pid

#### Q7：进程切换的 时机/原因/诱因/发生条件 可能是什么？

A:
1. 进程等待子进程返回(wait)或者等待 I/O 结束，主动交出 CPU 资源
2. 进程时间片用完，被内核调度到其他进程
3. 进程通过 exit 主动退出 / 被迫 (例如 kill) 结束
4. 被高优先级进程抢占（可能涉及到调度的知识、提提概念即可）

#### Q8：上下文切换的 时机/原因/诱因/发生条件 可能是什么？

A:除了进程切换以外，如中断/syscall处理

#### Q9：[基础] 僵尸进程是什么？

A:
1. 已经退出（包括 exit 和被 kill），但是还没有被父进程 wait 的进程
2. 无伤大雅，只不过僵尸进程的部分资源（尤其是 PCB）不能释放，因为父进程可能 wait

#### Q10：[进阶] 孤儿进程是什么？

A:
1. 如果父进程退出（exit 或 kill），那么子进程不会被终止掉，而是继续执行
2. 但是其子进程的父进程会变成 init 进程（linux） / initproc（ucore）