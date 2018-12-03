1.关于底层的属性说明
属性和成员变量不同，
@property = ivar + getter  + setter；
定义的property会在objc_ivar_list中添加一个成员变量的描述，然后再methodList中添加setter和getter方法，objc_property_t是声明的属性类型，是一个指向objc_property结构体指针；

总结：
object_property_t * propertyList = class_copyPropertyList(class, &count);
foreach就可以查询所有的property的数据信息

``` xml
typedef struct {
    const char * _Nonnull name;           /**< The name of the attribute */
    const char * _Nonnull value;          /**< The value of the attribute (usually empty) */
} objc_property_attribute_t;
```

成员变量
Ivar的实例变量类型，是一个指向objc_ivar的结构体的指针
typedef struct objc_ivar *ivar;
结构体指针的组成如下：
``` xml
- struct objc_ivar {
    char * _Nullable ivar_name;
    char * _Nullable ivar_type;
    int ivar_offset;基地址的偏移量
#ifdef __LP64__
    int space;
#endif
}
```
获取整个成员变量列表
Ivar *class_copyIvarList（Class cls, unsigned int *outCount）;
我们可以直接通过 ivar_getName(ivar);
ivar_getTypeEncoding(ivar);
释放指针，
注意的一些细节点是：记得释放free(ivars);

定义成员变量的列表；
``` xml
在objc_class中，所有的成员变量，属性的信息是放在链表ivars中的，ivars是一个数组，数组中的每个元素指向Ivar变量指针的地址。
struct objc_ivar_list {
    int ivar_count；
#ifdef __LP64__
    int space；
#endif
    /* variable length structure */
    struct objc_ivar ivar_list[1]；
}
```
关于存储objc_method的记录
``` xml
struct objc_method {
    SEL _Nonnull method_name;
    char * _Nullable method_types; 存储方法的参数类类和返回值类型
    IMP _Nonnull method_imp;
}
```
关于selector的定义，其实底层并没有这个定义；；从相对的总结可以看出，selector其实就是一个char类型的指针；

关于IMP：就是函数指针 ，指向方法实现的首地址
``` xml
/// A pointer to the function of a method implementation.
#if !OBJC_OLD_DISPATCH_PROTOTYPES
typedef void (*IMP)(void /* id, SEL, ... */ );
#else
typedef id _Nullable (*IMP)(id _Nonnull, SEL _Nonnull, ...);
#endif
```
``` xml
struct objc_cache {
    unsigned int mask /* total = mask + 1 */  ;
    一个整数，指定分配的缓存bucket的总数。在方法查找过程中，Objective-C runtime使用这个字段来确定开始线性查找数组的索引位置。指向方法selector的指针与该字段做一个AND位操作(index = (mask & selector))。这可以作为一个简单的hash散列算法
    unsigned int occupied;一个整数，指定实际占用的缓存bucket的总数。
    Method _Nullable buckets[1];指向Method数据结构指针的数组。这个数组可能包含不超过mask+1个元素。需要注意的是，指针可能是NULL，表示这个缓存bucket没有被占用，另外被占用的bucket可能是不连续的。这个数组可能会随着时间而增长。
};
```

``` xml
objc_protocol_list
struct objc_protocol_list {
    struct objc_protocol_list * _Nullable next;
    long count;
    __unsafe_unretained Protocol * _Nullable list[1];
};
```

方法调用的api
方法调用流程
检查 selector 是否需要忽略
检查 target 是否为 nil，如果是 nil 就直接 cleanup，然后 return
在 target 的 Class 中根据 selector 去找 IMP

寻找 IMP 的过程:
在当前 class 的方法缓存里寻找（cache methodLists）
找到了跳到对应的方法实现，没找到继续往下执行
从当前 class 的 方法列表里查找（methodLists），找到了添加到缓存列表里，然后跳转到对应的方法实现；没找到继续往下执行
从 superClass 的缓存列表和方法列表里查找，直到找到基类为止
以上步骤还找不到 IMP，则进入消息动态处理和消息转发流程

runtime的api的使用：
``` xml
//获取属性列表
objc_property_t *propertyList = class_copyPropertyList([self class], &count);
//获取方法列表
Method *methodList = class_copyMethodList([self class], &count);
//获取成员列表
Ivar *ivarList = class_copyIvarList([self class], &count);
//获取协议列表
__unsafe_unretained Protocol **protocolList = class_copyProtocolList([self class], &count);
```

```
动态创建类
Class cls = objc_allocateClassPair(MyClass.class, "MySubClass", 0);
class_addMethod(cls, @selector(submethod1), (IMP)imp_submethod1, "v@:");
class_replaceMethod(cls, @selector(method1), (IMP)imp_submethod1, "v@:");
class_addIvar(cls, "_ivar1", sizeof(NSString *), log(sizeof(NSString *)), "i");

objc_property_attribute_t type = {"T", "@"NSString""};
objc_property_attribute_t ownership = { "C", "" };
objc_property_attribute_t backingivar = { "V", "_ivar1"};
objc_property_attribute_t attrs[] = {type, ownership, backingivar};

class_addProperty(cls, "property2", attrs, 3);
objc_registerClassPair(cls);

id instance = [[cls alloc] init];
[instance performSelector:@selector(submethod1)];
[instance performSelector:@selector(method1)];
```

``` xml
转化关系
id obj = [[NSObject alloc] init];
void *p = (__bridge void *)obj;
id o = (__bridge id)p;

关于这个转换可以了解更多：ARC 类型转换：显示转换 id 和 void *
```

