

关于VUE切换成virtual dom的树的流程

简单举例子：
``` xml
const demo = new Vue({
	data: {
    	text:"before",
    }
    render(h) {
    	return h('div',{},[
        	h('span',{},[this.__toString__(this.text)])
        ])
    }
})
setTimeout(function(){
	demo.text = "afer"
},3000)
```
对应的虚拟dom的变化;;before->after的变化
``` xml
<div><span>before</span></div>
<div><span>after</span></div>
```

监听data下的所有属性的变化,data的属性变化，如何触发对应的函数；；
ES5提供了新的方法，Object.defineProperty。根据这个方法，可以自定getter和setter函数，那么在获取对象属性后设置对象属性就能直接执行响应的回调函数；；

语法Object.defineProperty方法会直接定义在一个新的属性上，或者修改一个对象的现有属性，并返回这个对象。
Object.defineProperty(obj,prop,description);
obj定义对象的属性；；prop 定义或者修改的属性  description 被定义和描述

底层Vue类的定义：
``` xml

class Vue {
  constructor(options) {
    this.$options = options
    this._data = options.data
    observer(options.data, this._update.bind(this))
    this._update()
  }
  _update(){
    this.$options.render()
  }
}

function observer(obj, cb) {
  Object.keys(obj).forEach((key) => {
    defineReactive(obj, key, obj[key], cb)
  })
}

改造后的方案：
function observer(obj, cb) {
  Object.keys(obj).forEach((key) => {
  	if(typeof obj[key] === 'object') {
    	new obser
    }
    defineReactive(obj, key, obj[key], cb)
  })
}


function defineReactive(obj, key, val, cb) {
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: () => {
      console.log('你访问了' + key)
      return val
    },
    set: newVal => {
      if (newVal === val)
        return
      console.log('你设置了' + key)
      console.log('新的' + key + ' = ' + newVal)
      val = newVal
      cb()
    }
  })
}

var demo1 = new Vue({
  el: '#demo',
  data: {
    text: "before"
  },
  render(){
    console.log("我要render了")
  }
})

var demo2 = new Vue({
	el:'#demo2',
    data: {
    	text: 'before',
        o: {
        	text:'o-before'
        }
    },
    render() {
    	console.log('我要render')
    }
})

```

添加代理
``` xml

_proxy(key) {
  const self = this
  Object.defineProperty(self, key, {
    configurable: true,
    enumerable: true,
    get: function proxyGetter() {
      return self._data[key]
    },
    set: function proxySetter(val) {
      self._data[key] = val
    }
  })
}

在构造函数中添加如下代码

Object.keys(options.data).forEach(key => this._proxy(key))

```

#### 转换响应式工作完工
1. 只要 data 的属性发生变化，就会触发 render 函数
2. 这也是为什么只有 data 中的属性是响应式的，而其他地方声明的值不是的原因

但是这里有个问题 ？即触发 render 函数的准确度问题？

### 第二步,解决准确度问题，引出虚拟dom
思考下边的 demo
``` xml
new Vue({
  template: `
    <div>
      <span>name:</span> { {name} }
    <div>`,
  data: {
    name: 'js',
    age: 24
  }
})
setTimeout(function(){
  demo.age = 25
}, 3000)
```


### 引入虚拟demo

``` xml
vue的两种写法
// template模板写法（最常用的）
new Vue({
  data: {
    text: "before",
  },
  template: `
    <div>
      <span>text:</span>
    </div>`
})
// render函数写法，类似react的jsx写法
new Vue({
  data: {
    text: "before",
  },
  render (h) {
    return (
      <div>
        <span>text:</span>
      </div>
    )
  }
})
```
``` xml
转化成下面的格式
new Vue({
  data: {
    text: "before",
  },
  render(){
    return this.__h__('div', {}, [
      this.__h__('span', {}, [this.__toString__(this.text)])
    ])
  }
})
```

``` xml

虚拟dom的生成
new Vue({
  data: {
    text: "before",
  },
  render(){
    return this.__h__('div', {}, [
      this.__h__('span', {}, [this.__toString__(this.text)])
    ])
  }
})

function VNode(tag, data, children, text) {
  return {
    tag: tag,
    data: data,
    children: children,
    text: text
  }
}

class Vue {
  constructor(options) {
    this.$options = options
    const vdom = this._update()
    console.log(vdom)
  }
  _update() {
    return this._render.call(this)
  }
  _render() {
    const vnode = this.$options.render.call(this)
    return vnode
  }
  __h__(tag, attr, children) {
    return VNode(tag, attr, children.map((child)=>{
      if(typeof child === 'string'){
        return VNode(undefined, undefined, undefined, child)
      }else{
        return child
      }
    }))
  }
  __toString__(val) {
    return val == null ? '' : typeof val === 'object' ? JSON.stringify(val, null, 2) : String(val);
  }
}
```

回头看问题
我们需要知道 render 函数中依赖了 data 中的哪些属性，只有这些属性变化，才需要去触发 render 函数


### 第三步，依赖收集，准确渲染
思路： 在这之前，我们已经把 data 中的属性改成响应式了，当去获取或者修改这些变量时便能够触发相应函数。那这里就可以利用这个相应的函数做些手脚了。
当声明一个 vue 对象时，在执行 render 函数获取虚拟dom的这个过程中，已经对 render 中依赖的 data 属性进行了一次获取操作，这次获取操作便可以拿到所有依赖。
其实不仅是 render，任何一个变量的改别，是因为别的变量改变引起(观察者模式)，都可以用上述方法，也就是 computed 和 watch 的原理


依赖收集
首先需要写一个依赖收集的类，每一个data中的属性都有可能被依赖，因此每个属性在响应式转化(defineReactive)的时候，就初始化它。
``` xml
class Dep {
  constructor() {
    this.subs = []
  }
  add(cb) {
    this.subs.push(cb)
  }
  notify() {
    console.log(this.subs)
    this.subs.forEach((cb) => cb())
  }
}

function defineReactive(obj, key, val, cb) {
  const dep = new Dep()
  Object.defineProperty(obj, key, {
    // 省略
  })
}
class Dep {
  constructor() {
    this.subs = []
  }
  add(cb) {
    this.subs.push(cb)
  }
  notify() {
    console.log(this.subs)
    this.subs.forEach((cb) => cb())
  }
```

执行过程
当执行render函数的时候，依赖到的变量的get就会被执行，然后就把这个 render函数加到subs里面去。
当set的时候,就执行notify，将所有的subs数组里的函数执行，其中就包含render的执行。
代码中有一个Dep.target值，这个值时用来区分是普通的get还是收集依赖时的get
完整代码： https://github.com/SirM2z/assets/blob/master/vue-demo.js

vue react响应式简单对比
综上发现： 用Object.defineProperty这个特性可以精确的写出订阅发布模式，从这点来说，vue是优于react的，在没经过优化之前，vue的渲染机制一定是比react更加准确的，为了验证这一说法，我用两个框架同时写了两个相同的简单项目进行对比。

没有对比就没有伤害
react项目地址：http://sirm2z.github.io/react-vue-test/react/index.html
vue项目地址：http://sirm2z.github.io/react-vue-test/vue/index.html
通过对比发现，react 在正常使用的过程中产生了多余的渲染，在移动端或者组件嵌套非常深的情况下会产生非常大的性能消耗，因此在使用 react 的过程中，写好 react 生命周期中的 shouldComponentUpdate 是非常重要的！

所有资源
完整代码： https://github.com/SirM2z/assets/blob/master/vue-demo.js
PPT网址： http://sirm2z.github.io/vue-demo-ppt/index.html
博客文章网址： http://www.jianshu.com/p/7374c4e91891
react项目地址： http://sirm2z.github.io/react-vue-test/react/index.html
vue项目地址： http://sirm2z.github.io/react-vue-test/vue/index.html












