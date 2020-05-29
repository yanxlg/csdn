---
title: vue
categories: vue
cover: /images/top/vue.jpeg
top_img: /images/cover/vue.png
top: true
keywords:
  - Vue原理分析
  - Vue双向绑定
  - Vue状态更新
description:
  - Vue原理分析
  - Vue双向绑定
  - Vue状态更新
abbrlink: c0add594
date: 2020-05-27 16:24:32
---
## 前言
Vue 作为近两年跟React几乎并驾齐驱的前端框架，其在前端开发中的地位是尤其重要，其轻量、易用、灵活、入门简单的特点收到很多前端开发者的推崇，本文主要揭秘Vue的架构、响应式原理、状态更新及vue@3重大更新。

## 架构
Vue是`MVVM`架构的最佳实践,是一个`JavaScript MVVM`库，是一套构建用户界面的渐进式框架。专注于 `MVVM` 中的 `ViewModel`，不仅做到了数据双向绑定，而且也是一款相对比较轻量级的JS 库。因此说到Vue架构不得不说`MVVM`。`MVVM`是目前前端主流架构之一，其从`MVC`演变而来，演变过程为`MVC->MVP->MVVM`。`MVVM`模式图如下：
![MVVM](./mvvm.png)
1. `MVVM` 由 `Model`,`View`,`ViewModel` 三部分构成，`Model` 层代表数据模型，也可以在Model中定义数据修改和操作的业务逻辑；`View` 代表UI 组件，它负责将数据模型转化成UI 展现出来，`ViewModel` `是一个同步View` 和 `Model`的对象。
2. 在`MVVM`架构下，`View` 和 `Model` 之间并没有直接的联系，而是通过`ViewModel`进行交互，`Model` 和 `ViewModel` 之间的交互是双向的， 因此`View` 数据的变化会同步到`Model`中，而`Model` 数据的变化也会立即反应到`View` 上。
3. `ViewModel` 通过双向数据绑定把 `View` 层和 `Model` 层连接了起来，而`View` 和 `Model` 之间的同步工作完全是自动的，无需人为干涉，因此开发者只需关注业务逻辑，不需要手动操作DOM, 不需要关注数据状态的同步问题，复杂的数据状态维护完全由 `MVVM` 来统一管理。
4. 特点：
  - 双向数据绑定
  - 数据与视图层分离

## Vue原理
Vue原理相对于React来说比较简单易懂，源码也没有React那么复杂，整体调试过程和阅读过程比较清晰，具体原理图如下：
![Vue原理图](./design.png)

1. `Vue` 是采用 `Object.defineProperty` 的 `getter` 和 `setter`，并结合`发布订阅模式`来实现数据绑定的。当把一个普通 Javascript 对象传给 Vue 实例来作为它的 data 选项时，Vue 会递归遍历它的属性，用 `Object.defineProperty` 将它们转为 `getter/setter`。用户看不到 `getter/setter`，但是在内部它们让 Vue 追踪依赖，在属性被访问和修改时通知变化。
2. `Observer`： 数据监听器，能够对数据对象的所有属性进行监听，如有变动可拿到最新值并通知订阅者，内部采用Object.defineProperty的getter和setter来实现。
3. `Compile`： 指令解析器，它的作用对每个元素节点的指令进行扫描和解析，根据指令模板替换数据，以及绑定相应的更新函数。
4. `Watcher`： 订阅者， 作为连接 Observer 和 Compile 的桥梁，能够订阅并收到每个属性变动的通知，执行指令绑定的相应回调函数。
5. `Dep`： 消息订阅器，内部维护了一个数组，用来收集订阅者（Watcher），数据变动触发notify 函数，再调用订阅者的 update 方法。

Vue原理中比较重要的就是双向绑定实现、响应式实现、发布订阅模式、nextTick、updateQueue，下面我们分别进行介绍。

## 双向数据绑定
Vue中双向绑定是通过`defineProperty`实现的，下面我们介绍简单的双向绑定实现：
首先，我们需要了解Object上属性劫持方法：Object.`defineProperty`和Object.`defineProperties`。这两个方法可以实现对象中属性`getter`和`setter`方法定义，从而实现属性的劫持，例如：
```typescript
const obj = {};
Object.defineProperty(obj,'hello',{
  get:function(){
    //我们在这里拦截到了数据
    console.log("get方法被调用");
  },
  set:function(newValue){
    //改变数据的值，拦截下来
    console.log("set方法被调用");
  }
});
obj.hello//输出为“get方法被调用”，输出了值。
obj.hello = 'new Hello';//输出为set方法被调用，修改了新值
```
输出结果如下：
```console
get方法被调用
set方法被调用
```
可以从这里看到，这是在对更底层的对象属性进行编程。简单地说，也就是我们对其更底层对象属性的修改或获取的阶段进行了拦截（也可以称之为对象属性`钩子`）。
在这数据拦截的基础上，我们可以手写数据的双向绑定：
```javascript
const obj = {};
Object.defineProperty(obj,'hello',{
  get:function(){
    console.log("get方法被调用");
  },
  set:function(newValue){
    document.getElementById('test').value = newValue;
    document.getElementById('test1').innerHTML = newValue;
  }
});
document.getElementById('test').addEventListener('input',function(e){
  obj.hello = e.target.value;//触发它的set方法
})
function reset(){
  obj.hello="reset";
}
```
```html
<div id="mvvm">
    <input v-model="text" id="test"></input>
    <button onclick="reset()">重置</button>
    <div id="test1" style="display:inline-block"></div>
</div>
```
在线演示demo1：
<div id="mvvm">
    <input v-model="text" id="test"></input>
    <button onclick="reset()">重置</button>
    <div id="test1" style="display:inline-block"></div>
</div>
<script>
    const obj = {};
    Object.defineProperty(obj, 'hello', {
        get: function () {
            console.log("get方法被调用");
        },
        set: function (newValue) {
            document.getElementById('test').value = newValue;
            document.getElementById('test1').innerHTML = newValue;
        }
    });
    document.getElementById('test').addEventListener('input', function (e) {
        obj.hello = e.target.value;//触发它的set方法
    })
    function reset(){
        obj.hello="reset";
    }
</script>

根据上面我们简单的实现了一个双向绑定功能。这是Vue的基础，Vue中的响应式本质上就是跟上面一样，下面简单实现Vue中的响应式代码。

## 响应式实现
1. 定义Vue的data的属性响应式
```javascript
function defineReactive (obj, key, value){
  Object.defineProperty(obj,key,{
    get:function(){
      console.log("get了值"+value);
      return value;//获取到了值
    },
    set:function(newValue){
      if(newValue === value){
        return;//如果值没变化，不用触发新值改变
      }
      value = newValue;//改变了值
      console.log("set了最新值"+value);
    }
  })
}
```
这里的obj我们这定义为vm实例或者vm实例里面的data属性。
>`defineProperty`这个方法，不仅可以定义obj的直接属性，比如obj.hello这个属性。也可以间接定义属性比如：obj.middle.hello。这里导致的效果就是两者的hello属性都被定义成响应式了。

2. `observe`方法循环调用响应式方法。
```javascript
function observe (obj,vm){
  Object.keys(obj).forEach(function(key){
    defineReactive(vm,key,obj[key]);
  })
}
```
3. Vue初始化实例
```javascript
function Vue(options){
  this.data = options.data;
  var data = this.data;

  observe(data,this);//这里调用定义响应式方法

  var id = options.el;
  var dom = nodeContainer(document.getElementById(id),this);
  document.getElementById(id).appendChild(dom); //把虚拟dom渲染上去 
}
```
4. 在Compile方法中处理`v-model`指令：
```javascript
function compile(node, vm){
  var reg = /\{\{(.*)\}\}/g;
  if(node.nodeType === 1){
    var attr = node.attributes;
    //解析节点的属性
    for(var i = 0;i < attr.length; i++){
      if(attr[i].nodeName == 'v-model'){
        
        var name = attr[i].nodeValue;
     
        node.addEventListener('input',function(e){
          console.log(vm[name]);
          vm[name] = e.target.value;//改变实例里面的值
        });

        node.value = vm[name];//讲实例中的data数据赋值给节点
      }
    }
  }
}
```


在线演示demo2：
<div id="mvvm1">
    <input v-model="text" id="test1">&#123;&#123;text&#125;&#125;
    <div>&#123;&#123;text&#125;&#125;</div>
</div>
<script> 
function nodeContainer(node, vm, flag){
  var flag = flag || document.createDocumentFragment();
  var child;
  while(child = node.firstChild){
    compile(child, vm);
    flag.appendChild(child);
    if(child.firstChild){
      // flag.appendChild(nodeContainer(child,vm));
      nodeContainer(child, vm, flag);
    }
  }
  return flag;
}
//编译
function compile(node, vm){
  var reg = /\{\{(.*)\}\}/g;
  if(node.nodeType === 1){
    var attr = node.attributes;
    //解析节点的属性
    for(var i = 0;i < attr.length; i++){
      if(attr[i].nodeName == 'v-model'){
        var name = attr[i].nodeValue;
        node.addEventListener('input',function(e){
          vm[name] = e.target.value;
        });
        node.value = vm[name];//讲实例中的data数据赋值给节点
        node.removeAttribute('v-model');
      }
    }
  }
  //如果节点类型为text
  if(node.nodeType === 3){
    if(reg.test(node.nodeValue)){
      // console.dir(node);
      var name = RegExp.$1;//获取匹配到的字符串
      name = name.trim();
      node.nodeValue = vm[name];
    }
  }
}
function defineReactive (obj, key, value){
  Object.defineProperty(obj,key,{
    get:function(){
      console.log("get了值"+value);
      return value;
    },
    set:function(newValue){
      if(newValue === value){
        return;
      }
      value = newValue;
      console.log("set了最新值"+value);
    }
  })
}
function observe (obj,vm){
  Object.keys(obj).forEach(function(key){
    defineReactive(vm,key,obj[key]);
  })
}
function Vue(options){
  this.data = options.data;
  var data = this.data;
  observe(data,this);
  var id = options.el;
  var dom = nodeContainer(document.getElementById(id),this);
  document.getElementById(id).appendChild(dom);  
}
var Demo = new Vue({
  el:'mvvm1',
  data:{
    text:'HelloWorld',
    d:'123'
  }
})
 </script> 

实现效果（观察控制台）：

```console
get了值HelloWorld
get了值HelloWorld
get了值HelloWorld

get了值HelloWorld
HelloWorld
set了最新值HelloWorldj
```

通过上面代码我们实现了，在输入框里面输入，同时触发getter&setter，去改变vm实例中data的值。也就是说Vue原理的图例中经过getter&setter已经成功了。上述仅是抽象出的简化代码，真实的Vue代码中还有Dep，Observer以类的形式存在并递归属性等等...，此处不做完全代码拷贝。接下去就是订阅——发布者模式。

## 发布订阅模式
发布订阅模式在前端开发中用处比较广泛，大部分前端框架中都会涉及到该设计模式，尤其是实时通信框架；那么什么是发布订阅模式？
简单点说：例如微信里面经常会订阅一些公众号，一旦这些公众号发布新消息了。那么他就会通知你，告诉你：我发布了新东西，快来看。
这种场景下，你就是`订阅者`，公众号就是`发布者`。
所以我们要模拟这种情景，我们需要
1. 声明订阅者：
```javascript
var sub1 = {
  update:function(){
    console.log(1);
  }
}
var sub2 = {
  update:function(){
    console.log(2);
  }
}
var sub3 = {
  update:function(){
    console.log(3);
  }
}
```
每个订阅者对象内部声明一个update方法来触发订阅属性。
2. 声明一个发布者，去触发发布消息，通知的方法：
```javascript
function Dep(){
  this.subs = [sub1,sub2,sub3];//把三个订阅者加进去
}
Dep.prototype.notify = function(){//在原型上声明“发布消息”方法
  this.subs.forEach(function(sub){
    sub.update();
  })
}
var dep = new Dep();
dep.notify();
```
3. 声明另外一个中间对象作为调度中心
```javascript
var dep = new Dep();
var pub = {
  publish:function(){
    dep.notify();
  }
}
pub.publish();//这里的结果是跟上面一样的
```
实现效果：
```console
1
2
3
```

到这，我们已经实现了：
1. 修改输入框内容 => 触发修改vm实例里的属性值 => 触发set&get方法
2. 订阅成功 => 发布者发出通知notify() => 触发订阅者的update()方法
接下来重点要实现的是介绍Vue是如何更新的。

## 观察者模式
上面介绍了发布订阅模式，在前端中说到发布订阅模式就会联想到观察者模式，两者的界限比较小，容易混淆，但两者之间确实存在明显区别，观察者模式是直接观察数据源，数据元更新时观察者直接收到通知；而发布订阅模式中间会存在一个名叫`调度中心`的中间层，观察者的监听事件都是注册到`调度中心`，发布者更新后会通知`调度中心`，`调度中心`根据数据源变化通知相应的观察者，同时在`调度中心`里可以进行一些派发检测等逻辑处理，例如实现部分消息不下发

## 更新
从原理图中可以看出，Vue的更新分为异步和同步，核心代码在queueWatcher方法中
```typescript
export function queueWatcher (watcher: Watcher) {
  const id = watcher.id
  if (has[id] == null) {
    has[id] = true
    if (!flushing) {
      queue.push(watcher)
    } else {
      // if already flushing, splice the watcher based on its id
      // if already past its id, it will be run next immediately.
      let i = queue.length - 1
      while (i > index && queue[i].id > watcher.id) {
        i--
      }
      queue.splice(i + 1, 0, watcher)
    }
    // queue the flush
    if (!waiting) {
      waiting = true

      if (process.env.NODE_ENV !== 'production' && !config.async) {
        flushSchedulerQueue()
        return
      }
      nextTick(flushSchedulerQueue)
    }
  }
}
```
当为同步更新时立即执行flush，输出更新，当为异步时，延迟更新，等待下一帧再输出更新，同步异步与React中类似，事件模型和生命钩子中都是异步，即进行状态更新合并后更新视图，而在定时器，Promise等异步操作中则立即执行视图更新，视图更新过程主要时Virtual DOM diff，当然与React不同的是在diff阶段Vue会去做一层校验，判断哪些是不需要更新的，而React中需要自己通过`shouldComponentUpdate`或者`memo`实现

## nextTick原理
首先看Vue中的源码：
```javascript
/* @flow */
/* globals MutationObserver */

import { noop } from 'shared/util'
import { handleError } from './error'
import { isIE, isIOS, isNative } from './env'

export let isUsingMicroTask = false

const callbacks = []
let pending = false

function flushCallbacks () {
  pending = false
  const copies = callbacks.slice(0)
  callbacks.length = 0
  for (let i = 0; i < copies.length; i++) {
    copies[i]()
  }
}

// Here we have async deferring wrappers using microtasks.
// In 2.5 we used (macro) tasks (in combination with microtasks).
// However, it has subtle problems when state is changed right before repaint
// (e.g. #6813, out-in transitions).
// Also, using (macro) tasks in event handler would cause some weird behaviors
// that cannot be circumvented (e.g. #7109, #7153, #7546, #7834, #8109).
// So we now use microtasks everywhere, again.
// A major drawback of this tradeoff is that there are some scenarios
// where microtasks have too high a priority and fire in between supposedly
// sequential events (e.g. #4521, #6690, which have workarounds)
// or even between bubbling of the same event (#6566).
let timerFunc

// The nextTick behavior leverages the microtask queue, which can be accessed
// via either native Promise.then or MutationObserver.
// MutationObserver has wider support, however it is seriously bugged in
// UIWebView in iOS >= 9.3.3 when triggered in touch event handlers. It
// completely stops working after triggering a few times... so, if native
// Promise is available, we will use it:
/* istanbul ignore next, $flow-disable-line */
if (typeof Promise !== 'undefined' && isNative(Promise)) {
  const p = Promise.resolve()
  timerFunc = () => {
    p.then(flushCallbacks)
    // In problematic UIWebViews, Promise.then doesn't completely break, but
    // it can get stuck in a weird state where callbacks are pushed into the
    // microtask queue but the queue isn't being flushed, until the browser
    // needs to do some other work, e.g. handle a timer. Therefore we can
    // "force" the microtask queue to be flushed by adding an empty timer.
    if (isIOS) setTimeout(noop)
  }
  isUsingMicroTask = true
} else if (!isIE && typeof MutationObserver !== 'undefined' && (
  isNative(MutationObserver) ||
  // PhantomJS and iOS 7.x
  MutationObserver.toString() === '[object MutationObserverConstructor]'
)) {
  // Use MutationObserver where native Promise is not available,
  // e.g. PhantomJS, iOS7, Android 4.4
  // (#6466 MutationObserver is unreliable in IE11)
  let counter = 1
  const observer = new MutationObserver(flushCallbacks)
  const textNode = document.createTextNode(String(counter))
  observer.observe(textNode, {
    characterData: true
  })
  timerFunc = () => {
    counter = (counter + 1) % 2
    textNode.data = String(counter)
  }
  isUsingMicroTask = true
} else if (typeof setImmediate !== 'undefined' && isNative(setImmediate)) {
  // Fallback to setImmediate.
  // Technically it leverages the (macro) task queue,
  // but it is still a better choice than setTimeout.
  timerFunc = () => {
    setImmediate(flushCallbacks)
  }
} else {
  // Fallback to setTimeout.
  timerFunc = () => {
    setTimeout(flushCallbacks, 0)
  }
}

export function nextTick (cb?: Function, ctx?: Object) {
  let _resolve
  callbacks.push(() => {
    if (cb) {
      try {
        cb.call(ctx)
      } catch (e) {
        handleError(e, ctx, 'nextTick')
      }
    } else if (_resolve) {
      _resolve(ctx)
    }
  })
  if (!pending) {
    pending = true
    timerFunc()
  }
  // $flow-disable-line
  if (!cb && typeof Promise !== 'undefined') {
    return new Promise(resolve => {
      _resolve = resolve
    })
  }
}
```
通过上面源码可以发现，nextTick本质上就是创建一个异步任务，通过创建下一轮任务队列执行的任务来延迟函数调用，通过`Promise.then`、`setImmediate`、`setTimeout 0`等手段实现，具体js中任务队列相关可以阅读[js 事件驱动](/h5/eventloop.html)

## Vue3.0 探索
待补充