
关于多线程的理解：

pthread的特点：
一套通用的多线程api，适用于Unix，Linux，Windows等系统
跨平台\可移植，使用难度大，C语言，使用的频率，几乎不用
线程生命周期： 由程序员进行管理

NSThread：
使用更加面向对象 简单易用，可以直接操作线程对象，使用的OC语言进行封装，使用的频率：偶尔使用。线程的生命周期： 由程序员进行管理

GCD：
旨在代替NSThread等线程技术，充分利用设备的多核（自动），使用语言是C语言，使用频率：经常使用，线程生命周期：自动管理

NSOperation：
基于GCD，比CGD多了一些更加简单实用的功能，使用面向对象，使用语言：OC语言，使用频率：经常使用，生命周期管理：自动管理

多线程的原理：
同一时间，CPU只能处理1条线程，只有一条线程在工作，
多线程并发（同时）执行，其实是CPU快速地在多条线程之间调度（切换）
如果CPU调度线程的时间足够快，就造成了都线程并发执行的假象

思考点？？
如果线程非常非常多，会发生什么情况？
CPU会在N多个线程之前调度，CPU会累死，消耗大量的CPU资源。
每条线程被调度执行的频次会降低（线程的执行效率降低）

多线程的优点：
能适当提高程序的执行效率，能适当提高资源的利用率（CPU，内存利用率）

多线程的缺点：
开启线程需要占用一定的内存空间（默认情况下，主线程占用1M， 子线程占用512kb，如果开启大量的线程，会占用大量的内存空间，降低程序的性能）
线程越多，CPU在调度线程上开销就越大
程序设计更加复杂：比如线程之间的通信，多线程的数据共享

更倾向于哪一种通信方式：
GCD技术是一个轻量的，底层实现隐藏的神奇技术，我们能够通过GCD和block轻松的实现多线程编程，有时候，CGD相比其他系统提供的多线程方法更加有效，当然，有时候GCD不是最佳选择，另一个多线程编程的技术

NSOperationQueue让我们能够将后台线程以队列方式依序执行，并提供更多操作的入口，这和GCD的实现有些类似。

CGD和NSOperationQueue的区别：
CGD是底层的C语言构成的API，而NSOperationQueue及相关对象是Objc的对象，在GCD中，在队列中执行的是由block构成的任务，这是一个轻量级的数据结构，而Operation作为一个对象，为我们提供了更多的选择。
在NSOperationQueue中，我们可以随时取消已经设定要准备的任务，（当然，已经开始的任务就无法阻止了，）而GCD没法停止已经加入Queue的block（其实是有的，但是需要需要更多复杂的代码实现）
NSOperation能够方便的设置依赖关系，我们可以让一个Operation依赖另一个Operation，这样的话，尽管两个NSOperation处于同一个并行队列中，但前者会到后者执行完毕后再执行。
我们将KVO应用在NSOperation中，可以监听一个Operation是否完成或者取消，这样子能比GCD更加有效的掌控我们执行的后台任务。
NSOperation可以设置NSOperation的priority优先级，能够使用同一个队列中的任务区分先后地执行，而在GCD中，我们只能区分不同的任务队列的优先级，如果要区分block任务的优先级，也需要大量的复杂代码。
我们能够对NSOperation进行继承，在这之上添加变量和成员方法，提高整个代码的复用度，这比简单的将block任务排入执行队列更有自由度，能够在其之上添加更多自定制的功能。
总的来说，OperationQueue提供了更多你在编写程序时需要的功能，并隐藏了许多线程调度，线程取消与线程优先级的复杂代码，为我们提供简单的API入口，从编程原则来说，一般我们需要尽可能的使用高等级，封装完美的API，在必须时才使用底层API，但如果能以更简单的底层代码实现时，CGD确实是更好的选择。

使用NSOperation的情况： 各个操作之间有依赖关系，操作需要取消暂停，并发管理 ，控制操作之间的优先级，限制同时能执行的线程数量，让线程在某个时刻停止或者继续等。

使用GCD的情况： 一般的需求很简单的多线程操作，用GCD都可以，简单高效


GCD的执行原理：
GCD有一个底层线程池，这个池中存放的是一个个的线程。之所以称为“池”，很容易理解出这个“池”中的线程是可以重用的，当一段时间后这个线程没有被调用胡话，这个线程就会被销毁。注意：开多少条线程是由底层线程池决定的（线程建议控制再3~5条），池是系统自动来维护，不需要我们程序员来维护（看到这句话是不是很开心？）

而我们程序员需要关心的是什么呢？我们只关心的是向队列中添加任务，队列调度即可。

如果队列中存放的是同步任务，则任务出队后，底层线程池中会提供一条线程供这个任务执行，任务执行完毕后这条线程再回到线程池。这样队列中的任务反复调度，因为是同步的，所以当我们用currentThread打印的时候，就是同一条线程。

如果队列中存放的是异步的任务，（注意异步可以开线程），当任务出队后，底层线程池会提供一个线程供任务执行，因为是异步执行，队列中的任务不需等待当前任务执行完毕就可以调度下一个任务，这时底层线程池中会再次提供一个线程供第二个任务执行，执行完毕后再回到底层线程池中。

这样就对线程完成一个复用，而不需要每一个任务执行都开启新的线程，也就从而节约的系统的开销，提高了效率。在iOS7.0的时候，使用GCD系统通常只能开5至8条线程，iOS8.0以后，系统可以开启很多条线程，但是实在开发应用中，建议开启线程条数：3至5条最为合理。


关于gcd的原理的执行案例：
``` xmL
NSLog(@("1"));
dispatch_sync(dispatch_get_main_queue(),^{
	NSLog(@"2");
});
NSLog(@"3");
执行的结果是：1

NSLog(@"1");
dispatch_sync(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH,0)^{
	NSLog(@"2");
});
NSLog(@("3"));
执行顺序： 1 2 3


dispath_queue_t queue = dispatch_queue_create("com.demo.serialQueue",DISPATCH_QUEUE_SERIAL);
NSLog(@"1");
dispatch_aync(queue,^{
	NSLog(@"2");
    dispatch_sync(queue,^{
    	NSLog("3");//任务3
    }
    NSLog(""4);//任务4
})；
NSLog(@"5");//任务5
执行的顺序： 1， 2 ，5  或者  1， 5， 2

NSLog(@"1");
dispatch_async(dispatch_get_global_queue(0,0),^{
	NSLog(@"2");
    dispatch_sync(dispatch_get_main_queue(),^{
    	NSLog(@"3");
    }
    NSLog(@"4");
})
NSLog(@"5");
输出结果： 1， 2， 5, 3, 4

关于主线程出现死循环的情况
dispatch_aynsc(dispatch_get_global_queue(0,0),^{
	NSLog(@"1");
    dispatch_sync(dispatch_get_main_queue(),^{
    	NSLog(@"2");
    })
    NSLog(@"3");
})
NSLog(@"4");
while(1) {

}
NSLog(@"5");

输出结果： 1，

```
iOS8之前队列优先级：
``` xml
DISPATCH_QUEUE_PRIORITY_HIGH 2高优先级
DISPATCH_QUEUE_PRIORITY_DEFAULT 0默认优先级
DISPATCH_QUEUE_PRIORITY_LOW (-2)低优先级
DISPATCH_QUEUE_PRIORITY_BACKGROUND INT16_MIN后台优先级
iOS8+之后：
QOS_CLASS_USER_INTERACTIVE 0x21, 用户交互(希望尽快完成，不要放太耗时操作)
QOS_CLASS_USER_INITIATED 0x19, 用户期望(不要放太耗时操作)
QOS_CLASS_DEFAULT 0x15, 默认(用来重置对列使用的)
QOS_CLASS_UTILITY 0x11, 实用工具(耗时操作，可以使用这个选项)
QOS_CLASS_BACKGROUND 0x09, 后台
QOS_CLASS_UNSPECIFIED 0x00, 未指定

```

NSOperation是苹果提供给我们的一套多线程解决方案。实际上NSOperation是基于GCD更高一层的封装，但是比GCD更简单易用、代码可读性也更高。

NSOperation需要配合NSOperationQueue来实现多线程。因为默认情况下，NSOperation单独使用时系统同步执行操作，并没有开辟新线程的能力，只有配合NSOperationQueue才能实现异步执行。

因为NSOperation是基于GCD的，那么使用起来也和GCD差不多，其中，NSOperation相当于GCD中的任务，而NSOperationQueue则相当于GCD中的队列。NSOperation实现多线程的使用步骤分为三步：

创建任务：先将需要执行的操作封装到一个NSOperation对象中。
创建队列：创建NSOperationQueue对象。
将任务加入到队列中：然后将NSOperation对象添加到NSOperationQueue中。
之后呢，系统就会自动将NSOperationQueue中的NSOperation取出来，在新线程中执行操作。

三类封装类型：
使用子类NSInvocationOperation
``` xml
// 1.创建NSInvocationOperation对象
NSInvocationOperation *op = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(run) object:nil];
// 2.调用start方法开始执行操作
[op start];
- (void)run
{
    NSLog(@"------%@", [NSThread currentThread]);
}
在没有使用NSOperationQueue，单独使用NSInvocationOperation的情况下，NSInvocationOperation在主线程中执行操作，并没有开启新线程。
```
使用子类NSBlockOperation
``` xml
NSBlockOperation *op = [NSBlockOperation blockOperationWithBlock:^{
    // 在主线程
    NSLog(@"------%@", [NSThread currentThread]);
}];
[op start];
在没有使用NSOperationQueue，单独使用NSInvocationOperation的情况下，NSInvocationOperation在主线程中执行操作，并没有开启新线程。

// 添加额外的任务(在子线程执行)
    [op addExecutionBlock:^{
        NSLog(@"2------%@", [NSThread currentThread]);
    }];
```
定义继承自NSOperation的子类，通过实现内部相应的方法来封装任务。

创建队列
NSOperationQueue一共有两种队列：主队列和其他队列，其中包含了串行，并发功能，
主队列：
NSOperationQueue *queue = [NSOperationQueue mainQueue];

其他队列：（非主队列）
添加到这种队列的任务,（NSOperation），就会自动放到子线程中执行，同时包含了串行和并发功能。
NSOperationQueue *queue = [[NSOperationQueue alloc] init];

将任务加到队列中：
创建任务，然后将创建好的任务加入到创建好的队列中。
``` xml

// 1.创建队列
    NSOperationQueue *queue = [[NSOperationQueue alloc] init];
    // 2. 创建操作
    // 创建NSInvocationOperation
    NSInvocationOperation *op1 = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(run) object:nil];
    // 创建NSBlockOperation
    NSBlockOperation *op2 = [NSBlockOperation blockOperationWithBlock:^{
        for (int i = 0; i < 2; ++i) {
            NSLog(@"1-----%@", [NSThread currentThread]);
        }
    }];
    // 3. 添加操作到队列中：addOperation:
    [queue addOperation:op1]; // [op1 start]
    [queue addOperation:op2]; // [op2 start]
    
    // 1. 创建队列
    NSOperationQueue *queue = [[NSOperationQueue alloc] init];
    // 2. 添加操作到队列中：addOperationWithBlock:
    [queue addOperationWithBlock:^{
        for (int i = 0; i < 2; ++i) {
            NSLog(@"-----%@", [NSThread currentThread]);
        }
    }];
    可以看出，addOperationWithBlock和NSOperationQueue能够开启新线程，进行并发执行。

```

控制串执行和并行执行的关键：
关键参数，maxConcurrentOperationCount, 最大并发数。
maxConcurrentOperationCount默认情况是-1，表示不进行限制，默认认为并发执行。
maxConcurrentOperationCount为1时，进行串行执行，
maxConcurrentOperationCount大于1时，指不能超过系统的限制，即使设置了，系统也会自动做调整。


操作依赖：
NSOperation和NSOperationQueue最吸引人的地方是它能添加操作之间的依赖关系。比如说有A、B两个操作，其中A执行完操作，B才能执行操作，那么就需要让B依赖于A。

```xml
	NSOperationQueue *queue = [[NSOperationQueue alloc] init];
    NSBlockOperation *op1 = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"1-----%@", [NSThread  currentThread]);
    }];
    NSBlockOperation *op2 = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"2-----%@", [NSThread  currentThread]);
    }];
    [op2 addDependency:op1];    // 让op2 依赖于 op1，则先执行op1，在执行op2

    [queue addOperation:op1];
    [queue addOperation:op2];

```
其他的要点接口:
``` xml
- (void)cancel; NSOperation提供的方法，可取消单个操作

- (void)cancelAllOperations; NSOperationQueue提供的方法，可以取消队列的所有操作

- (void)setSuspended:(BOOL)b; 可设置任务的暂停和恢复，YES代表暂停队列，NO代表恢复队列

- (BOOL)isSuspended; 判断暂停状态

```

注意：

这里的暂停和取消并不代表可以将当前的操作立即取消，而是当当前的操作执行完毕之后不再执行新的操作。
暂停和取消的区别就在于：暂停操作之后还可以恢复操作，继续向下执行；而取消操作之后，所有的操作就清空了，无法再接着执行剩下的操作。
