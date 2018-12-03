

基本信息：
关于class和struct的区别（网易的考题）


介绍下设计模式：
注意点：回答了NSNotificationCenter的可以直接pass

首先回答设计模式为了解决什么问题
其次是通过什么方案来解决这些问题
最后才是当前体系下的具体的实现方案


https链接的网站里面，输入账号密码点击登录后，到服务器返回这个请求前，中间经历了什么。

TCP的协议，
对于整个网络连接模型的理解可以看出基本功


关于UI的链式，在一个app中间有一个button，在你的手触摸屏幕点击后，到这个button收到点击事件，中间发生了什么。
runloop和响应链需要说清楚
随便问问UIResponse ， UIControl , UIView之间的关系


组件化的解耦

蘑菇街的组件间通信，采用的是URL跳转模式，理论上页面之间的跳转只需要open一个URL即可。所以对一个组件来说，只要定义【支持哪些URL】即可，比如详情页面，

``` xml
//注册进入到指定页面， 页面级别的路由和需要传入的参数
[MGJRouter registerURLPattern:@"mgl://detail?id=id" toHandler:^(Dictionary *routerParam) {
	//获取指定的value的值
}];

组件A调用组件B的，比如商品详情页面要调用展示购物车的商品数量，就涉及到想购物车组件拿数据

[MGJRouter registerURLPattern:@"mgj://cart/orderCount" toObjectHandler:^id(NSDictionary *routerParamters) {

	//参数解析，然后获取对应的value的值
}]

使用时，
NSNumber *orderCount = [MGJRouter obejctForURL:@"mgj://catd/ordercount"]就这样，可以获取购物车里面的商品数量。

稍微复杂但更加通用性的是使用协议【协议】 ->  [类]绑定的方式，还是以购物车为例，购物车组件可以提供一个协议protocol

@protocol MGJCart<NSObecjt>
@end


可以看到通过协议可以直接指定返回的数据类型，然后在购物车组件内新建个类实现这个协议，假设这个类名MGJCartImpl ,接着就可以把它与协议关联起来。

[ModuleManager registerClass:MGJCartImpl forProtocol:@protocol(MGJCart)];

需要调用[ModuleManagerClassForProtocol:@protocol(MGJCart)]，然后调用orderCount的就可以了。

//入口方式，

通过文件加载出所有的module的方式 然后通过调用module里面的application的协议事件来实现module的需要进行初始化的的事件，

在app启动时，从入口调用

```
