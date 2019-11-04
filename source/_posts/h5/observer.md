---
title: Observer API
abbrlink: e4278974
date: 2019-11-04 12:02:24
categories:
    - h5
    - API
---
## 前言
前端项目中经常需要用到异常监控和埋点功能，虽然大部分情况下我们都会直接接入第三方平台，因为第三方平台相对简单、稳定，通过第三方平台提供的API便可以简单实现服务注册及上报；很少我们会自己去开发一个自己的系统，但是，了解如何去开发异常监控和埋点功能还是非常有必要的。本文不针对方案进行详细介绍，如果有兴趣，可以转到{% post_link h5/monitor 前端异常监控 %}和{% post_link h5/buryingPoint 前端埋点实现 %}详细了解。
在思考方案过程中，我们想到了{% post_link h5/designPatterns Javascript中的设计模式 %}，观察者模式中提到浏览器自带的观察者如下：
{% note %}
1、Intersection Observer，交叉观察者。
2、Mutation Observer，变动观察者。
3、Resize Observer，视图观察者。
4、Performance Observer，性能观察者。
{% endnote %}


## 简介
&emsp;&emsp; | Intersection Observer |  Mutation Observer | Resize Observer | Performance Observer
-|-|-|-|-
作用 | 观察一个元素是否在viewport中可视 | 观察DOM中的变化 | 观察DOM元素大小的变化 | 检测性能度量事件
方法 | observe()<br>disconnect()<br>takeRecords()<br>unobserve() | observe()<br>disconnect()<br>takeRecords()<br>unobserve() | observe()<br>disconnect()<br>unobserve() | observe()<br>disconnect()<br>takeRecords()
替代 | DOM Mutation events | getBoundingRect()返回元素大小及其相对于viewport的位置<br>Scroll和Resize事件 | Resize事件 | Performance接口
场景 | 1.无限滚动<br>2.图片懒加载<br>3.兴趣埋点<br>4.`控制动画、视频执行`（性能优化） | 1.更高性能的数据绑定和响应<br>2.实现视觉差滚动<br>3.图片预加载<br>4.富文本编辑器实现 | 1.更智能的响应式布局（取代@media）<br>2.响应式组件 | 1.更细颗粒的性能检测<br>2.分析性能对业务的影响

## 详细介绍
### Intersection Observer：交叉观察者
> `IntersectionObserver`接口，提供了一种异步观察目标元素与其祖先元素或顶级文档视窗(`viewport`)交叉状态的方法，祖先元素与视窗(`viewport`)被称为根(`root`)

#### 意义：
想要实时计算Web页面元素的位置，并根据位置进行交互控制，从而实现前端性能优化，这方面的实现非常依赖于`DOM`状态的显示查询，但这些查询是`同步`的，会导致昂贵的样式计算开销（重绘和回流），且不停轮询会导致大量的性能浪费。早起针对此问题有以下几种方案：
 + 定时器轮询计算每个元素的实时可见性
 + 基于`享元模式`实现数据绑定的高性能滚动列表，列表中呈现的是数据集的子集
 + 通过onscroll事件实时计算元素的可见性
这些方案存在以下共同点：
{% note %}
1. 都是查询各个元素相对与某些元素（全局视口）的“被动查询”。
2. 过度增长的CPU使用，牺牲CPU提升性能。
{% endnote %}

#### 优势：
`Intersection Observer API`通过为开发人员提供一种新方法来`异步`查询元素相对于其他元素或全局视口的位置，从而解决了上述问题:
- 异步处理消除了昂贵的DOM和样式查询，连续轮询以及使用自定义scroll插件的需求。
- 使应用程序显着降低CPU，GPU和资源成本。

#### 基本使用：
 1. 创建观察者：
 ```typescript
    const options = {
        root: document.querySelector('.scrollContainer'),
        rootMargin: '0px',
        threshold: [0.3, 0.5, 0.8, 1] 
    };
    const observer = new IntersectionObserver(handler, options)
 ```
 配置参数的含义：
    `root`：指定根元素即观察区域元素  
    `rootMargin`：通过类似css margin的值设置观察区域相对于root大小的扩展  
    `threshold`：阈值，`number`或`Array<number>`，指定当目标元素在root指定区域内，当可见度等于阈值时触发调度函数
    {% asset_img 640.webp 对比图 %}
    
 2. 定义回调事件
 当目标与根元素通过阈值相交时，就会触发回调函数。
 ```typescript
    interface IntersectionObserverEntry {
        readonly boundingClientRect: ClientRect | DOMRect;
        readonly intersectionRatio: number;
        readonly intersectionRect: ClientRect | DOMRect;
        readonly isIntersecting: boolean;
        readonly rootBounds: ClientRect | DOMRect | null;
        readonly target: Element;
        readonly time: number;
    }
    function handler (entries:IntersectionObserverEntry[], observer) { 
        entries.forEach(entry => { 
        // 每个成员都是一个IntersectionObserverEntry对象。
        // 举例来说，如果同时有两个被观察的对象的可见性发生变化，entries数组就会有两个成员。
        // entry.boundingClientRect 
        // entry.intersectionRatio 
        // entry.intersectionRect 
        // entry.isIntersecting 
        // entry.rootBounds 
        // entry.target 
        // entry.time 
        }); 
    }
 ```
 `time`：返回交叉被触发的时间的时间戳，可见性发生变化的时间，是一个高精度时间戳，单位为毫秒
 `rootBounds`：返回包含根元素的矩形区域的信息
 `boundingClientRect`：返回包含目标元素的矩形区域的信息，边界的计算方式与 `getBoundingClientRect()` 相同
 `intersectionRect`： 返回根元素和目标元素的交叉区域的信息
 `intersectionRatio`： 返回目标元素的可见比例,也就是 intersectionRect 占 boundingClientRect 的比例值，见下图
 `target`：返回目标元素的 dom 节点对象
 `isIntersecting`：目标元素是否与根元素相交
 {% asset_img 641.webp 对比图 %}
 
 3. 定义观察目标对象
 任何目标元素都可以通过调用`.observer(target)`方法来观察。
 ```typescript
    const target = document.querySelector(".targetBox"); 
    observer.observe(target);
 ```
 此外，还有两个停止监听的方法方法：
 ```typescript
    observer.unobserve(target);// 停止对某目标的监听
    observer.disconnect();// 终止对所有目标的监听
 ```

 4. 示例1：图片懒加载
 ```html
    <img src="placeholder.png" data-src="img-1.jpg">
    <img src="placeholder.png" data-src="img-2.jpg">
    <img src="placeholder.png" data-src="img-3.jpg">
    <!-- more images -->
 ```
 ```typescript
    const callback = (entries, observer)=> {
        entries.forEach(entry => {
            /* 替换属性 */
            entry.target.src = entry.target.dataset.src;
            observer.unobserve(entry.target);
        });
    };
    let observer = new IntersectionObserver(callback,{rootMargin: "0px 0px -200px 0px"});
    document.querySelectorAll('img').forEach(img => { observer.observe(img) });
 ```
 html中通过js遍历或模板方式输出一批图片元素，其src属性指定为固定的logo或者占位图，并设置其data-src属性为真实图片地址，在js中通过观察器观察，当图片进入窗口区域底部200px位置时开始替换属性加载真实图片并且回收该元素监听。当然由于API兼容性问题，真实的图片懒加载不可能只有这么多代码，如要考虑在API不兼容的情况下，通过scroll事件和节流消抖来轮询检测元素位置，具体参考{% post_link h5/lib/lazyload 图片懒加载 %}。
 
 5. 示例2：兴趣埋点
 关于兴趣埋点，一个比较通用的方案是：
 > 来自：《超好用的API之IntersectionObserver》
 ```typescript
    const boxList = [...document.querySelectorAll('.box')];
    
    var io = new IntersectionObserver((entries) =>{
      entries.forEach(item => {
        // intersectionRatio === 1说明该元素完全暴露出来，符合业务需求
        if (item.intersectionRatio === 1) {
          // TODO 埋点曝光代码
          io.unobserve(item.target)
        }
      })
    }, {
      root: null,
      threshold: 1, // 阀值设为1，当只有比例达到1时才触发回调函数
    });
    
    // observe遍历监听所有box节点
    boxList.forEach(box => io.observe(box));
  ```
  至于怎样评断用户是否感兴趣，方案就多种多样了，此处不作叙述，需要可以查看{% post_link h5/buryingPoint 埋点 %}
  
 6. 示例3：控制动画视频执行
  这是一个比较常见的场景，如果h5列表元素中包含动画或视频，此优化是必须的，经常在面试中也会问到此类问题，此处介绍视频的控制：
  ```html
    <video src="OSRO-animation.mp4" controls=""></video>
  ```
  ```typescript
    let video = document.querySelector('video');
    let isPaused = false; /* Flag for auto-paused video */
    let observer = new IntersectionObserver((entries, observer) => { 
      entries.forEach(entry => {
        if(entry.intersectionRatio!=1  && !video.paused){
          video.pause(); isPaused = true;
        }
        else if(isPaused) {video.play(); isPaused=false}
      });
    }, {threshold: 1});
    observer.observe(video);
  ```
  这只是简单的实现，当然页面中有多个视频组成视频列表时还需要控制什么情况下才开始播放，播放中的视频仅能有一个，否则出现混乱，影响用户体验。
      
<hr>

### Mutation Observer：变动观察者
> 接口提供了监视对DOM树所做更改的能力。它被设计为旧的MutationEvents功能的替代品，该功能是DOM3 Events规范的一部分。

#### 意义
归根究底，是MutationEvents的功能不尽人意：
{% note %}
1. 在`MDN`中也写到了，是由于`DOM Event`承认在API上有缺陷，反对使用。
2. 核心缺陷是：性能问题和跨浏览器支持。
3. 为DOM添加 `mutation` 监听器极度降低进一步修改DOM文档的性能（慢1.5 - 7倍），此外, 移除监听器不会逆转的损害。
{% endnote %}
MutationEvents的原理：通过绑定事件监听DOM，支持的事件大致如下，不同浏览器存在兼容性差异，不需要去记录他们
```code
  DOMAttributeNameChanged
  DOMCharacterDataModified
  DOMElementNameChanged
  DOMNodeInserted
  DOMNodeInsertedIntoDocument
  DOMNodeRemoved
  DOMNodeRemovedFromDocument
  DOMSubtreeModified
```

#### 优势
1. MutationEvents事件是同步触发，也就是说，DOM 的变动立刻会触发相应的事件；
2. Mutation Observer 则是异步触发，DOM 的变动并不会马上触发，而是要等到当前所有 DOM 操作都结束才触发。
3. 可以通过配置项，监听目标DOM下子元素的变更记录
    
#### 基本使用
1. 创建观察者
```typescript
let observer = new MutationObserver(callback);
```
2. 定义回调函数
上面代码中的回调函数，会在每次 DOM 变动后调用。该回调函数接受两个参数，第一个是变动数组，第二个是观察器实例，下面是一个例子：
```typescript
interface MutationRecord {
    readonly addedNodes: NodeList;
    readonly attributeName: string | null;
    readonly attributeNamespace: string | null;
    readonly nextSibling: Node | null;
    readonly oldValue: string | null;
    readonly previousSibling: Node | null;
    readonly removedNodes: NodeList;
    readonly target: Node;
    readonly type: MutationRecordType;
}
function callback (mutations:MutationRecord[], observer) {
  mutations.forEach(function(mutation) {
    console.log(mutation);
  });
}
```
`MutationRecord`对象中各属性含义如下：

    属性 | 意义
    -|-
    type | 观察的变动类型（`attribute`、`characterData`或者`childList`）
    target | 发生变动的`DOM`节点
    addedNodes | 新增的`DOM`节点
    removedNodes | 删除的`DOM`节点
    previousSibling | 前一个同级节点，如果没有则返回null
    nextSibling | 下一个同级节点，如果没有则返回null
    attributeName | 发生变动的属性。如果设置了`attributeFilter`，则只返回预先指定的属性
    oldValue | 变动前的值。这个属性只对`attribute`和`characterData`变动有效，如果发生`childList`变动，则返回null
    
3. 定义要观察的目标
 ```typescript
    interface MutationObserverInit {
        attributeFilter?: string[];
        attributeOldValue?: boolean;
        attributes?: boolean;
        characterData?: boolean;
        characterDataOldValue?: boolean;
        childList?: boolean;
        subtree?: boolean;
    }
    mutationObserver.observe(content, {
        attributes: true, // Boolean - 观察目标属性的改变
        characterData: true, // Boolean - 观察目标数据的改变(改变前的数据/值)
        childList: true, // Boolean - 观察目标子节点的变化，比如添加或者删除目标子节点，不包括修改子节点以及子节点后代的变化
        subtree: true, // Boolean - 目标以及目标的后代改变都会观察
        attributeOldValue: true, // Boolean - 表示需要记录改变前的目标属性值
        characterDataOldValue: true, // Boolean - 设置了characterDataOldValue可以省略characterData设置
        // attributeFilter: ['src', 'class'] // Array - 观察指定属性
    });
 ```
优先级如下：
{% note %}
1、attributeFilter/attributeOldValue > attributes
2、characterDataOldValue > characterData
3、attributes/characterData/childList（或更高级特定项）至少有一项为true；
4、特定项存在, 对应选项可以忽略或必须为true
{% endnote %}
此外，还有两个停止观察的方法：
```typescript
    mutationObserver.unobserve(target);// 取消观察某个元素
    mutationObserver.disconnect();// 全部取消观察
    mutationObserver.takeRecords();// 清除变动记录。即不再处理未处理的变动。该方法返回变动记录的数组，注意，该方法立即生效。
```

4. 例子1：监听文本变化
```typescript
const target = document.getElementById('target-id');
const observer = new MutationObserver(records => {
  // 输入变更记录
});
// 开始观察
observer.observe(target, {
  characterData: true
})
```
这里可以有几种处理。
    1. 聊天的气泡框彩蛋，检测文本中的指定字符串/表情包，触发类似微信聊天的表情落下动画。
    2. 输入框的热点话题搜索，当输入"#"号时，启动搜索框预检文本或高亮话题。

 有个`Vue`的小型插件就是这么实现的：
 
 > 来自：《vue-hashtag-textarea》

{% asset_img 642.gif %}
    
    
### ResizeObserver，视图观察者
`ResizeObserver API`是一个新的JavaScript API，与`IntersectionObserver API`非常相似，它们都允许我们去监听某个元素的变化。

#### 意义：
* 开发过程当中经常遇到的一个问题就是如何监听一个 div 的尺寸变化。
* 但众所周知，为了监听 div 的尺寸变化，都将侦听器附加到 window 中的 resize 事件。
* 但这很容易导致性能问题，因为大量的触发事件。
* 换句话说，使用`window.resize` 通常是浪费的，因为它告诉我们每个视窗大小的变化，而不仅仅是当一个元素的大小发生变化。
* 而且`resize`事件会在一秒内触发将近60次，很容易在改变窗口大小时导致性能问题，比如说，你要调整一个元素的大小，那就需要在 `resize` 的回调函数 callback() 中调用 `getBoundingClientRect` 或 `getComputerStyle`。不过你要是不小心处理所有的读和写操作，就会导致布局混乱。

#### 优势
* 细颗粒度的`DOM元素`观察，而不是window，当然是用`resize`也可以实现单个元素的监听，具体方案参考{% post_link h5/domOnResize DOM resize listener %}
* 没有额外的性能开销，只会在绘制前或布局后触发调用

#### 基本使用
1. 创建观察者
```javascript
let observer = new ResizeObserver(callback);
```

2. 定义回调函数
```typescript
 interface ResizeObserverEntry {
      readonly contentRect: DOMRectReadOnly;
      readonly target: Element;
  }
const callback = (entries:ResizeObserverEntry[]) => {
    entries.forEach(entry => {

    })
}
```

3. 定义要观察的目标
```typescript
observer.observe(document.body)
```
此外，取消观察的api有：
```typescript
observer.unobserve(document.body);// 取消特定元素
observer.disconnect();// 全部取消
```

4. 例子1：响应式组件
前端响应式组件通常是使用`@media`去实现的，但是`@media`仅能监控窗口大小，不能监控元素本身大小，如果布局发生改变，`@media`的响应式会存在一些问题，完美的响应式组件是监控元素本身大小，例如`vue-responsive-components`库的实现，就是通过ResizeObserver观察元素本身大小变化从而实现响应式。
    
### `PerformanceObserver`：性能观察者
这是一个浏览器和Node.js 里都存在的API，采用相同W3C的Performance Timeline规范
* 在浏览器中，我们可以使用 window 对象取得window.performance和 window.PerformanceObserver 。
* 而在 Node.js 程序中需要perf_hooks 取得性能对象，如下：`const { PerformanceObserver, performance } = require('perf_hooks');`

#### 意义：
- 可以获取到当前页面中与性能相关的信息。它是 `High Resolution Time API` 的一部分，同时也融合了 `Performance Timeline API`、`Navigation Timing API`、 `User Timing API` 和 `Resource Timing API`
- `Performance API` 是大家熟悉的一个接口，他记录着几种性能指数的庞大对象集合
```typescript
  interface PerformanceTiming {
      readonly connectEnd: number;
      readonly connectStart: number;
      readonly domComplete: number;
      readonly domContentLoadedEventEnd: number;
      readonly domContentLoadedEventStart: number;
      readonly domInteractive: number;
      readonly domLoading: number;
      readonly domainLookupEnd: number;
      readonly domainLookupStart: number;
      readonly fetchStart: number;
      readonly loadEventEnd: number;
      readonly loadEventStart: number;
      readonly navigationStart: number;
      readonly redirectEnd: number;
      readonly redirectStart: number;
      readonly requestStart: number;
      readonly responseEnd: number;
      readonly responseStart: number;
      readonly secureConnectionStart: number;
      readonly unloadEventEnd: number;
      readonly unloadEventStart: number;
      toJSON(): any;
  }
  interface Performance extends EventTarget {
      /** @deprecated */
      readonly navigation: PerformanceNavigation;
      onresourcetimingbufferfull: ((this: Performance, ev: Event) => any) | null;
      readonly timeOrigin: number;
      /** @deprecated */
      readonly timing: PerformanceTiming;
      clearMarks(markName?: string): void;
      clearMeasures(measureName?: string): void;
      clearResourceTimings(): void;
      getEntries(): PerformanceEntryList;
      getEntriesByName(name: string, type?: string): PerformanceEntryList;
      getEntriesByType(type: string): PerformanceEntryList;
      mark(markName: string): void;
      measure(measureName: string, startMark?: string, endMark?: string): void;
      now(): number;
      setResourceTimingBufferSize(maxSize: number): void;
      toJSON(): any;
      addEventListener<K extends keyof PerformanceEventMap>(type: K, listener: (this: Performance, ev: PerformanceEventMap[K]) => any, options?: boolean | AddEventListenerOptions): void;
      addEventListener(type: string, listener: EventListenerOrEventListenerObject, options?: boolean | AddEventListenerOptions): void;
      removeEventListener<K extends keyof PerformanceEventMap>(type: K, listener: (this: Performance, ev: PerformanceEventMap[K]) => any, options?: boolean | EventListenerOptions): void;
      removeEventListener(type: string, listener: EventListenerOrEventListenerObject, options?: boolean | EventListenerOptions): void;
  }
```
- `Performance API`若想获得某项页面加载性能记录，就需要调用performance.getEntries或者performance.getEntriesByName来获得。
- `Performance API`而获得执行效率，也只能通过performance.now来计算。

#### 优势
`PerformanceObserver`是浏览器内部对`Performance`实现的观察者模式，也是现代浏览器支持的几个 Observer 之一。
>来自：《你了解 Performance Timeline Level 2 吗？》

它解决了以下3点问题：
{% note %}
1. 避免不知道性能事件啥时候会发生，需要重复轮训timeline获取记录。
2. 避免产生重复的逻辑去获取不同的性能数据指标
3. 避免其他资源需要操作浏览器性能缓冲区时产生竞态关系。
{% endnote %}
{% note warning%}
`W3C`官网文档鼓励开发人员尽可能使用`PerformanceObserver`，而不是通过`Performance`获取性能参数及指标。
{% endnote%}

## 使用
1. 创建观察者
```typescript
let observer = new PerformanceObserver(callback); 
```
2. 定义回调函数
```typescript
interface PerformanceObserverEntryList {
    getEntries(): PerformanceEntryList;
    getEntriesByName(name: string, type?: string): PerformanceEntryList;
    getEntriesByType(type: string): PerformanceEntryList;
}
interface PerformanceEntry {
    readonly duration: number;
    readonly entryType: string;
    readonly name: string;
    readonly startTime: number;
    toJSON(): any;
}
const callback = (list:PerformanceObserverEntryList, observer) => {
    const entries = list.getEntries();
    entries.forEach((entry:PerformanceEntry) => {
    });
}
```
3. 定义要观察的目标对象
```typescript
interface PerformanceObserverInit {
    buffered?: boolean;
    entryTypes?: string[];
    type?: string;
}
observer.observe({entryTypes: ["entryTypes"]} as PerformanceObserverInit);
```
`observer.observe(...)`方法接受可以观察到的有效的入口类型。这些输入类型可能属于各种性能API，比如`User Tming`或`Navigation Timing API`。有效的`entryType`值：
    
  属性 | 别名 | 类型 | 描述 
  -|-|-|- 
  frame<br>navigation | `PerformanceFrameTiming`<br>`PerformanceNavigationTiming` | URL | 文件的地址
  resource | `PerformanceResourceTiming` | URL |所请求资源的解析URL
  mark | `PerformanceMark` | DOMString | 通过调用创建标记时使用的名称performance.mark()
  measure | `PerformanceMeasure` | DOMString | 通过调用创建度量时使用的名称performance.measure()
  paint | `PerformancePaintTiming` | DOMString | 无论是`first-paint`或`first-contentful-paint`
  longtask | `PerformanceLongTaskTiming` | DOMString | 报告长任务的实例

4. 例子：静态资源监控
>来自：《资源监控》
```typescript
  function filterTime(a, b) {
    return (a > 0 && b > 0 && (a - b) >= 0) ? (a - b) : undefined;
  }
  
  let resolvePerformanceTiming = (timing) => {
    let o = {
      initiatorType: timing.initiatorType,
      name: timing.name,
      duration: parseInt(timing.duration),
      redirect: filterTime(timing.redirectEnd, timing.redirectStart), // 重定向
      dns: filterTime(timing.domainLookupEnd, timing.domainLookupStart), // DNS解析
      connect: filterTime(timing.connectEnd, timing.connectStart), // TCP建连
      network: filterTime(timing.connectEnd, timing.startTime), // 网络总耗时
  
      send: filterTime(timing.responseStart, timing.requestStart), // 发送开始到接受第一个返回
      receive: filterTime(timing.responseEnd, timing.responseStart), // 接收总时间
      request: filterTime(timing.responseEnd, timing.requestStart), // 总时间
  
      ttfb: filterTime(timing.responseStart, timing.requestStart), // 首字节时间
    };
  
    return o;
  };
  
  let resolveEntries = (entries) => entries.map(item => resolvePerformanceTiming(item));
  
  let resources = {
    init: (cb) => {
      let performance = window.performance || window.mozPerformance || window.msPerformance || window.webkitPerformance;
      if (!performance || !performance.getEntries) {
        return void 0;
      }
  
      if (window.PerformanceObserver) {
        let observer = new window.PerformanceObserver((list) => {
          try {
            let entries = list.getEntries();
            cb(resolveEntries(entries));
          } catch (e) {
            console.error(e);
          }
        });
        observer.observe({
          entryTypes: ['resource']
        })
      } else {
          window.addEventListener('load', () => {
          let entries = performance.getEntriesByType('resource');
          cb(resolveEntries(entries));
        });
      }
    },
  };
```

## 参考文章
  >* 资源监控
  >* Media Queries Based on Element Width with MutationObserver
  >* 以用户为中心的性能指标
  >* A Few Functional Uses for Intersection Observer to Know When an Element is in View
  >* Getting To Know The MutationObserver API
  >* Different Types Of Observers Supported By Modern Browsers
  >* THE RESIZE OBSERVER EXPLAINED
  >* A Look at the Resize Observer JavaScript API 
  
  
## 兼容性
[Intersection Observer](https://caniuse.com/#search=Intersection%20Observer) | [Mutation Observer](https://caniuse.com/#search=Mutation%20Observer) | [Resize Observer](https://caniuse.com/#search=Resize%20Observer) | [Performance Observer](https://caniuse.com/#search=Performance%20Observer)

## 总结
这四个观察者，都非常适合集成到监控系统。虽然其兼容性有一定的限制，但是都有对应的polyfills：
Intersection Observer：intersection-observer
Mutation Observer：mutationobserver-shim
Resize Observer：resize-observer-polyfill
Performance Observer：@fastly/performance-observer-polyfill
