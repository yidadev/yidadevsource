
---
title: 常用的锁机制
date: 2018-11-18 22:13:29
tags: iOS
------


### 锁的性能的排序

![](http://images.bestswifter.com/lock_benchmark.png)

可以看到的是，目前来说自旋锁（OSSPinLock）的性能是最好的，我们常用的synchronized的性能是最差的；


### OSSPinLock（自旋锁）

上述文章中已经介绍了 OSSpinLock 不再安全，主要原因发生在低优先级线程拿到锁时，高优先级线程进入忙等(busy-wait)状态，消耗大量 CPU 时间，从而导致低优先级线程拿不到 CPU 时间，也就无法完成任务并释放锁。这种问题被称为优先级反转。

为什么忙等会导致低优先级线程拿不到时间片？这还得从操作系统的线程调度说起。

现代操作系统在管理普通线程时，通常采用时间片轮转算法(Round Robin，简称 RR)。每个线程会被分配一段时间片(quantum)，通常在 10-100 毫秒左右。当线程用完属于自己的时间片以后，就会被操作系统挂起，放入等待队列中，直到下一次被分配时间片。

自旋锁的实现原理

自旋锁的目的是为了确保临界区只有一个线程可以访问，它的使用可以用下面这段伪代码来描述:

``` xml

do {
    Acquire Lock
        Critical section  // 临界区
    Release Lock
        Reminder section // 不需要锁保护的代码
}

在Acquire Lock这一步，我们申请加锁，目的是为了保护临界区Critical section中的代码不会被多个线程执行， 自选锁实现的思路很简单，理论上来说只要定义一个全局的变量，用来表示锁的可用情况即可，伪代码如下

//狭义的原子操作
bool test_and_set (bool *target) {
    bool rv = *target;
    *target = TRUE;
    return rv;
}

bool lock = false; // 一开始没有锁上，任何线程都可以申请锁
do {
    while(test_and_set(&lock); // test_and_set 是一个原子操作
        Critical section  // 临界区
    lock = false; // 相当于释放锁，这样别的线程可以进入临界区
        Reminder section // 不需要锁保护的代码
}

```

如果临界区的执行时间过长，使用自旋锁不是个好主意。之前我们介绍过时间片轮转算法，线程在多种情况下会退出自己的时间片。其中一种是用完了时间片的时间，被操作系统强制抢占。除此以外，当线程进行 I/O 操作，或进入睡眠状态时，都会主动让出时间片。显然在 while 循环中，线程处于忙等状态，白白浪费 CPU 时间，最终因为超时被操作系统抢占时间片。如果临界区执行时间较长，比如是文件读写，这种忙等是毫无必要的。


自旋锁的问题：

系统维护了 5 个不同的线程优先级/QoS: background，utility，default，user-initiated，user-interactive。高优先级线程始终会在低优先级线程前执行，一个线程不会受到比它更低优先级线程的干扰。这种线程调度算法会产生潜在的优先级反转问题，从而破坏了 spin lock。

具体来说，如果一个低优先级的线程获得锁并访问共享资源，这时一个高优先级的线程也尝试获得这个锁，它会处于 spin lock 的忙等状态从而占用大量 CPU。此时低优先级线程无法与高优先级线程争夺 CPU 时间，从而导致任务迟迟完不成、无法释放 lock。这并不只是理论上的问题，libobjc 已经遇到了很多次这个问题了，于是苹果的工程师停用了 OSSpinLock。

苹果工程师 Greg Parker 提到，对于这个问题，一种解决方案是用 truly unbounded backoff 算法，这能避免 livelock 问题，但如果系统负载高时，它仍有可能将高优先级的线程阻塞数十秒之久；另一种方案是使用 handoff lock 算法，这也是 libobjc 目前正在使用的。锁的持有者会把线程 ID 保存到锁内部，锁的等待者会临时贡献出它的优先级来避免优先级反转的问题。理论上这种模式会在比较复杂的多锁条件下产生问题，但实践上目前还一切都好。

关于OSSpinLock的使用和基础实现

``` xml
#import OSSpinLock lock = OS_SPINLOCK_INIT;
OSSpinLockLock(&lock);
//需要执行的代码
OSSpinLockUnlock(&lock);
//OSSPINLOCK_DEPRECATED_REPLACE_WITH(os_unfair_lock)
//苹果在OSSpinLock注释表示被废弃，改用不安全的锁替代

```


### 信号量的锁：

``` xml
针对线程非安全的情况
#define INIT(...) self = super.init; \
if (!self) return nil; \
__VA_ARGS__; \
if (!_arr) return nil; \
_lock = dispatch_semaphore_create(1); \
return self;


#define LOCK(...) dispatch_semaphore_wait(_lock, DISPATCH_TIME_FOREVER); \
__VA_ARGS__; \
dispatch_semaphore_signal(_lock);

```


### 互斥锁



