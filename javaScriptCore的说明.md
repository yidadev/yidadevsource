关于javaScriptCore

JSContext是JS代码的执行环境
JSContext为JS代码的执行提供了上下文环境，通过JSCore执行的js代码都得通过JSContext来执行

JSContext对应于一个JS中的全局对象


JSValue是对JS值的包装，JS值到OC中不能直接使用，JSValue就是对JS值的包装，
![](https://upload-images.jianshu.io/upload_images/762048-c669c91024c9308b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/491)

JSValue存在于JSValue是不能独立存在的，它必须被存在某一个Context中，JSValue对应的JS值和其所属的JSContext的对象都是强弱引用的关系。

``` xml
 context[@"makeNSColor"] = ^(NSDictionary *rgb){
        float r = [rgb[@"red"] floatValue];
        float g = [rgb[@"green"] floatValue];
        float b = [rgb[@"blue"] floatValue];
        return [UIColor
                colorWithRed:(r/255.f)
                green:(g/255.f) blue:(b/255.f)
                alpha:1.0];
    };
    JSValue *value1 = [context evaluateScript:@"makeNSColor({red:12,green:23,blue:67})"];
```

使用Block暴露方法很方便，但是有2个坑需要注意一下：

1.不要在Block中直接使用JSValue
2.不要在Block中直接使用JSContext

因为Block会强引用它里面用到的外部变量，如果直接在Block中使用JSValue的话，那么这个JSvalue就会被这个Block强引用，而每个JSValue都是强引用着它所属的那个JSContext的，这是前面说过的，而这个Block又是注入到这个Context中，所以这个Block会被context强引用，这样会造成循环引用，导致内存泄露。不能直接使用JSContext的原因同理。


针对第一点，建议把JSValue当做参数传到Block中，而不是直接在Block内部使用，这样Block就不会强引用JSValue了。

针对第二点，可以使用[JSContext currentContext] 方法来获取当前的Context。

``` xml
@protocol MyPointExports <JSExport>

声明一个自定义的协议并继承自JSExport协议。然后当你把实现这个自定义协议的对象暴露给JS时，JS就能像使用原生对象一样使用OC对象了，也就是前面说的API目标之高保真。

```
![](https://upload-images.jianshu.io/upload_images/762048-3ae18cd8c1e4bdfc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/497)

当你声明一个继承自JSExport的自定义协议时，就是在告诉JSCore，这个自定义协议中声明的属性，实例方法和类方法需要被暴露给JS使用。（不在这个协议中的方法不会被暴露出去。）

当你把实现这个协议的类的对象暴露给JS时，JS中会生成一个对应的JS对象，然后，JSCore会按照这个协议中声明的内容，去遍历实现这个协议的类，把协议中声明的属性，转换成JS 对象中的属性，实质上是转换成getter 和 setter 方法，转换方法和之前说的block类似，创建一个JS方法包装着OC中的方法，然后协议中声明的实例方法，转换成JS对象上的实例方法，类方法转换成JS中某个全局对象上的方法。

![](https://upload-images.jianshu.io/upload_images/762048-be5d18cdb7bb7df5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/599)

我们有一个MyPoint类的对象point，当我们用JSExport协议将这个OC对象暴露给JS时，JSCore首先会在JS上下文环境中为该类生成一个对应的原型对象和构造函数，然后JSCore会扫描这个类，把其中在JSExport协议中声明的内容暴露给JS，属性(即getter和setter方法)会被添加到原型对象上，而类方法会被添加到到这个构造函数上，这个放的位置，就正好对应了OC中的类和元类。


不要在JS中给OC对象增加成员变量，这句话的意思就是说，当我们将一个OC对象暴露给JS后，就像前面说的使用JSExport协议，我们能想操纵JS对象一样操纵OC对象，但是这时候，不要在JS中给这个OC对象添加成员变量，因为这个动作产生的后果就是，只会在JS中为这个OC对象增加一个额外的成员变量，但是OC中并不会同步增加。所以说这样做并没有什么意义，还有可能造成一些奇怪的内存管理问题。

OC对象不要直接强引用JSValue对象，这句话再说直白点，就是不要直接将一个JSValue类型的对象当成属性或者成员变量保存在一个OC对象中，尤其是这个OC对象还暴露给JS的时候。这样会造成循环引用。

用JSManagedValue来解决强引用的问题：
，用JSValue创建一个JSManagedValue对象，JSManagedValue里面其实就是包着一个JSValue对象，可以通过它里面一个只读的value属性取到，这一步其实是添加一个对JSValue的弱引用。


说多线程之前得先说下另一个类 JSVirtualMachine, 它为JavaScript的运行提供了底层资源，有自己独立的堆栈以及垃圾回收机制。

JSVirtualMachine还是JSContext的容器，可以包含若干个JSContext，在一个进程中，你可以有多个JSVirtualMachine，里面包含着若干个JSContext，而JSContext中又有若干个JSValue，他们的包含关系如下图：

![](https://upload-images.jianshu.io/upload_images/762048-8b482ac389babfc2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

