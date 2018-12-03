iOS启动优化

App启动过程：

①解析Info.plist
加载相关信息，例如闪屏
沙箱建立、权限检查
②Mach-O加载
如果是胖二进制文件，寻找合适当前CPU架构的部分
加载所有依赖的Mach-O文件（递归调用Mach-O加载的方法）
定位内部、外部指针引用，例如字符串、函数等
执行声明为__attribute__((constructor))的C函数
加载类扩展（Category）中的方法
C++静态对象加载、调用ObjC的 +load 函数
③程序执行
调用main()
调用UIApplicationMain()
调用applicationWillFinishLaunching

App启动，系统首先加载可执行文件（自身App的所有.o文件的集合）
加载动态连接器dyld，dyld专门用来加载动态链接库的库，执行从dyld开始，dyld从可执行文件的依赖开始，递归加载所有的依赖动态链接库（动态链接库，framework，加载OCruntime的libobjc,系统级别的libSystem，）

