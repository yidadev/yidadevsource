### 关于javaCore


一个JSVirtualMachine的实例就是一个完整独立的JavaScript的执行环境，为JavaScript的执行提供底层资源。

这个类主要用来做两件事情：

实现并发的JavaScript执行
JavaScript和Objective-C桥接对象的内存管理


每一个JavaScript上下文（JSContext对象）都归属于一个虚拟机（JSVirtualMachine）。每个虚拟机可以包含多个不同的上下文，并允许在这些不同的上下文之间传值（JSValue对象）。

然而，每个虚拟机都是完整且独立的，有其独立的堆空间和垃圾回收器（garbage collector ），GC无法处理别的虚拟机堆中的对象，因此你不能把一个虚拟机中创建的值传给另一个虚拟机。

线程和JavaScript的并发执行

JavaScriptCore API都是线程安全的。你可以在任意线程创建JSValue或者执行JS代码，然而，所有其他想要使用该虚拟机的线程都要等待。

如果想并发执行JS，需要使用多个不同的虚拟机来实现。
可以在子线程中执行JS代码。

三个线程分别异步执行每秒1次的js log，首先会休眠1秒。

在context上执行一个休眠5秒的JS函数。

首先执行的应该是休眠5秒的JS函数，在此期间，context所处的虚拟机上的其他调用都会处于等待状态，因此tick和tick_2在前5秒都不会有执行。

而context1所处的虚拟机仍然可以正常执行tick_1。

休眠5秒结束后，tick和tick_2才会开始执行（不保证先后顺序）