# 同步互斥(lec 19 lab7) spoc 思考题

- 有"spoc"标记的题是要求拿清华学分的同学在实体课上完成的，对于学堂在线的选课同学是可选题目。

## 视频相关思考题

### 19.1 总体介绍

1. 同步机制、处理机调度、等待队列和定时器有些什么相互影响？

 > 底层支持：中断禁止、定时器、等待队列
 
 > 同步机制：信号量、管程（条件变量）

 > 同步管理：处理机调度

### 19.2 底层支撑

1. 操作系统内核如何利用定时器实现sleep系统？在定时队列中保存的时间格式是什么？

 > 定时器的数据结构（timer_t）：所有当前还未触发的定时设置形成一个队列（timer_list）；

 > 定时器的设置函数do_sleep()
 
 > 定时器的触发流程：trap_dispatch()会检查定时器设置是否触发，并在触发时唤醒相应线程；

2. 中断屏蔽控制位在哪个寄存器中？如何修改中断屏蔽控制位？

 > EFLAGS寄存器

 > 屏蔽中断指令：STI、CLI

 > local_intr_save(x); local_intr_restore(x);

3. 在ucore中有多少种等待队列？

 > static list_entry_t timer_list;

 > 包含wait_queue_t字段的数据结构通常都会对应有一类等待队列

4. 等待队列的两个基本操作down()和up()对线程状态和等待队列有什么影响？

 > 等待队列的数据结构、等待队列操作（等待down()、唤醒up()）

 > 关键的等待队列操作实现函数：wait_current_set(), wakeup_wait()

 > proc.h中定义了若干种等待状态（如WT_TIMER），通常一种等待状态会对应有一类等待队列

### 19.3 信号量设计实现

1. ucore中的信号量实现能实现是按FIFO获取信号量资源吗？给出你的理由。

 > semaphore_t

 > 由于线程唤醒原因可能与等待标志可能不一致，可能出现不按FIFO排队的情况；

 > 信号量的实现用到了“屏蔽中断”机制；使用wait_current_set()把当前线程放入等待队列；使用wait_current_del()把等待线程唤醒；

2. ucore中的信号量实现用到自旋锁(spinlock)吗？为什么？

 > 信号量的实现用到了“屏蔽中断”机制；使用wait_current_set()把当前线程放入等待队列；使用wait_current_del()把等待线程唤醒；

 > spinlock只在多处理机环境下有需求，单处理机环境下仅需要“屏蔽中断”；

3. 为什么down()要加一个_down()函数来进行实质的处理？类似情况还出现在up()和_up()。

> 参考回答：有多个有小差异的类似函数要使用相同的核心功能实现。类似情况还出现在do_fork()函数。

### 19.4 管程和条件变量设计实现

1. 管程与信号量是等价的。如何理解？

 > 等价，原因是可以基于一个机制来实现另一个机制；

2. 基于信号量的管程中在什么地方用到信号量？

 > 入口队列、唤醒队列和条件变量对应的等待队列等队列的实现中都用到了信号量；

3. 管程实现中的monitor.next的作用是什么？

 > monitor.next就是唤醒队列，对应三种管程模型中的hoare管程；

4. 分析管程实现中PV操作的配对关系，并解释PV操作的目的。

 > monitor.mutex

 > monitor.next

 > condvar.sem

5. 基于视频中对管程的17个状态或操作的分析，尝试分析管程在入口队列和条件变量等待队列间的等待和唤醒关系。注意分析各队列在什么情况下会有线程进入等待，在什么时候会被唤醒，以及这个等待和唤醒的依赖关系。

 > 线程A占用管程（0），A进入条件变量等待（1），A唤醒在入口等待队列中等待的线程B（3）

 > B占用管程（5），B唤醒在条件变量等待队列等待的A(6)，B进入唤醒队列等待（9）

 > A占用管程（11），A在结束时唤醒在唤醒队列等待的B（13），B占用管程并执行到结束（15）

6. ucore中的管程实现是Hoare管程吗？为什么？

 > 是的。占用管程的线程在唤醒条件等待队列中的线程后立即进行唤醒队列等待。

### 19.5 哲学家就餐问题

1. 哲学家就餐问题的管程实现中的外部操作成员函数有哪几个？

 > pickup();

 > putdown();

2. 哲学家就餐问题的管程实现中用了几个条件变量？每个条件变量的作用是什么？

 > 5个条件变量，每个条件代表一个哲学家；
 
## 小组思考题

1. (扩展练习) 每人用ucore中的信号量和条件变量两种手段分别实现40个同步问题中的一题。向勇老师的班级从前往后，陈渝老师的班级从后往前。请先理解与采用python threading 机制实现的异同点。

2. （扩展练习）请在lab7-answer中分析
  -  cvp->count含义是什么？cvp->count是否可能<0, 是否可能>1？请举例或说明原因。
  -  cvp->owner->next_count含义是什么？cvp->owner->next_count是否可能<0, 是否可能>1？请举例或说明原因。
  -  目前的lab7-answer中管程的实现是Hansen管程类型还是Hoare管程类型？请在lab7-answer中实现另外一种类型的管程。
  -  现在的管程（条件变量）实现是否有bug?

