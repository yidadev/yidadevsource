
#### 关于swift中的知识点

swift中的所有关键字：

用作声明的关键字:
``` xml
class、deinit、enum、extension、func、import、init、let、protocol、static、struct、subscript、typealias、var
```

用作语句的关键字
``` xml
break、case、continue、default、break、case、continue、default、do、else、fallthrough、if、in、for、return、switch、where、while
do、else、fallthrough、if、in、for、return、switch、where、while
```

用作表达和类型的关键字：
``` xml
as、dynamicType、new、is、super、self、Self、Type、__COLUMN__、__FILE__、__FUNCTION__、__LINE__
```

特定上下文中被保留的关键字:
``` xml
associativity、didset、get、infix、inout、left、mutating、none、nonmutating、operator、override、postfix、precedence、prefix、rightset、unowned、unowned(sale)、unowned(unsafe)、weak、willset
```

常见的关键字的处理：

class : 用来声明一个类
enum : 用来声明一个枚举
init : 相对于类的释构方法的修饰。
deinit : 相对于类的释构方法的修饰。
对于类的构造和释构在swift 中需要使用关键词来修饰，而很多高级语言并不需要特别的指定，便C++ 只需要类名与构造函数名相同就可以，不需要额外的关键词。

extension:扩展(类似于OC的categories)，
1. Swift 中的可以扩展以下几个：
2. 添加计算型属性和计算静态属性
3. 定义实例方法和类型方法
4. 提供新的构造器
5. 定义下标
6. 定义和使用新的嵌套类型
7. 使一个已有类型符合某个接口

let : 声明一个常量. 类似于const
protocol : 协议.也可以叫接口.这个往往在很多高级语言中不能多重继承的情况下使用协议是一个比较好的多态方式。
static : 声明静态变量或者函数
struct : 声明定义一个结构体
subscript : 下标索引修饰.可以让class、struct、以及enum使用下标访问内部的值
typealias : 为此类型声明一个别名.和 typedef类似.
break : 跳出循环.一般在控制流中使用,比如 for . while switch等语句
case : switch的选择分支.
continue : 跳过本次循环,继续执行后面的循环.
in : 范围或集合操作,多用于遍历.
fallthrough : swift语是执行完当前case,继续执行下面的case.类似于其它语言中省去break里，会继续往后一个c言特性switch语句的break可以忽略不写,满足条件时直接跳出循环.fallthrough的作用就是执行完当前case,继续执行下面的case.类似于其它语言中省去break里，会继续往后一个case跑，直到碰到break或default才完成的效果.

where : 用于条件判断,和数据库查询时的where 'id > 10'这样功能. swift语言的特性.OC中并没有.
is & as : is一般用于对一些变量的类型做判断.类似于OC中的isKindClass. as 与强制转换含义雷同.
as的使用场合，从派生类向基类转化。
as！向下转类型，由于是强制类型转化，如果转化失败，会报runtime的错误。
as?使用的场合，as？和as!操作符的转化规则是完全一致的，但as？如果转化不成功的时候便会返回一个nil对象，成功的话，返回可选择的类型的值，需要拆包使用，并且as？及时出现错误了，也不会报错，如果对于转化需要确保100%的话，可以使用as！，需要做细节说明。

没看懂是怎么使用的，暂时先忽略的关键字
dynamicType:获取对象的动态类型,即运行时的实际类型,而非代码指定或编译器看到的类型

__COLUMN__ 列号
__FILE__ 路径
__FUNCTION__ 函数
__LINE__ 行号

运算符的结合性： associativity
inout: inout作为函数声明，引用传值的关键字，但是在调用的时候引用的地址，所以在引用的时候要加上&
```
func test(inout a : Int ,inout b : Int) {
}
var num1 = 3
var num2 = 4
test(&num1,&num2)
```

willSet和didSet:willSet和didSet的作用是对赋值过程前后附加额外的操作
可以看做是捕获状态然后做操作,在将要赋值的时候和已经赋值的时候做相关操作

mutating:作用:写在func前面,以便于让func可以修改struct和protocol的extension中的成员的值。 如果func前面不加mutating,struct和protocol的extension中的成员的值便被保护起来,不能修改

class var:在swift中对于enum和struct来说支持用static关键字来标示静态变量，
但是对于class成员来说，只能以class var的方式返回一个只读值

convenience : convenience用来进行方便的初始化，就相当于构造函数重载。
对于class来讲，默认或指定的初始化方法作为所谓的Designated初始化。
若重载的初始化需要调用Designated初始化则将它作为convenience初始化，在方法前要加上convenience关键字
``` xml
class Figure{
       var name:String!
       var nikname:String?
       init(){
          name = "John"
       }
      convenience init(name:String!,nikname:String!) {
           self.init() self.name = name self.nikname = nikname
      }
    }
```

precedence : 运算的优先级，越高的话优先进行计算。swift 中乘法和除法的优先级是 150 ，加法和减法的优先级是 140 ，这里我们定义点积的优先级为 160 ，就是说应该早于普通的乘除进行运算。
unowned, unowned(safe), unowned(unsafe):无宿主引用。
infix: 表示要定义的是一个中位操作符，即前后都是输入
defer: 用来包裹一段代码，这个代码块将会在当前作用域结束的时候被调用。这通常被用来对当前的代码进行一些清理工作，比如关闭打开的文件等。
可以在同一个作用域中指定多个 defer
代码块，在当前作用域结束时，它们会以相反的顺序被调用，即先定义的后执行，后定义的先执行。
guard : 当某些条件不满足的情况下，跳出作用域. 色值