#### weak实现原理：

Runtime维护了一个weak表，用于存储指向某个对象的所有weak指针。weak表其实是一个hash（哈希）表，Key是所指对象的地址，Value是weak指针的地址（这个地址的值是所指对象的地址）数组。
1、初始化时：runtime会调用objc_initWeak函数，初始化一个新的weak指针指向对象的地址。
2、添加引用时：objc_initWeak函数会调用 objc_storeWeak() 函数， objc_storeWeak() 的作用是更新指针指向，创建对应的弱引用表。
3、释放时，调用clearDeallocating函数。clearDeallocating函数首先根据对象地址获取所有weak指针地址的数组，然后遍历这个数组把其中的数据设为nil，最后把这个entry从weak表中删除，最后清理对象的记录。

追问的问题一：实现weak后，为什么对象释放后会自动为nil？
runtime对注册的类， 会进行布局，对于weak对象会放入一个hash表中。 用weak指向的对象内存地址作为key，当此对象的引用计数为0的时候会dealloc，假如weak指向的对象内存地址是a，那么就会以a为键， 在这个weak表中搜索，找到所有以a为键的weak对象，从而设置为nil。

追问的问题二：当weak引用指向的对象被释放时，又是如何去处理weak指针的呢？
1、调用objc_release
2、因为对象的引用计数为0，所以执行dealloc
3、在dealloc中，调用了_objc_rootDealloc函数
4、在_objc_rootDealloc中，调用了object_dispose函数
5、调用objc_destructInstance
6、最后调用objc_clear_deallocating,详细过程如下：
a. 从weak表中获取废弃对象的地址为键值的记录
b. 将包含在记录中的所有附有 weak修饰符变量的地址，赋值为 nil
c. 将weak表中该记录删除
d. 从引用计数表中删除废弃对象的地址为键值的记录