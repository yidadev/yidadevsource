
如果一个类，我们发现这个类不存在某个方法时，比如Person类，我要使用它的run方法，但我发现自己的类并没有实现run的方法，那么我怎么操作，可以使得当前的app不会发生crash。

第一种方式 需要实现run的method
``` xml
void run(id self,SEL sel) {
    NSLog(@"执行。。。。");
}

+ (BOOL)resolveInstanceMethod:(SEL)sel {
    if(sel == @selector(run)) {
        class_addMethod(self.class,sel,(IMP)run,"v@:");
        return YES;
    }
    return [super resolveInstanceMethod:sel];
}
```

第二种方案实现执行函数的方式
```xml
- (id)forwardingTargetForSelector:(SEL)aSelector {
    NSLog(@"执行改方法的事件");
    //传递给下一个方法去接收当前的方法
    return nil;
}
```

第三种方案进行 签名的方式
``` xml
- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector {
    NSString *sel = NSStringFromSelector(aSelector);
    if([sel isEqualToString:@"run"]) {
        return [NSMethodSignature signatureWithObjCTypes:"v@:"];
    }
    return [super methodSignatureForSelector:aSelector];
}

针对"v@:" 代表默认的两个参数，一个是self，一个_cmd
- (void)forwardInvocation:(NSInvocation *)anInvocation {
    SEL selector = [anInvocation selector];
    Person *person = [[Person alloc] init];
    if([person respondsToSelector:@selector(selector)]) {
        [anInvocation invokeWithTarget:person];
    }
}
```