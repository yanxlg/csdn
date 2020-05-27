---
abbrlink: dee17c46
title: React
tags: react
categories: react
date: 2019-9-20 13:20:41
cover: /images/top/react.jpeg
top_img: /images/cover/react.png
top: true
keywords:
  - React原理分析
  - Fiber
  - Fiber算法
  - Fiber优先级
  - Fiber调度
  - Fiber任务循环
  - React状态更新
description: 
  - React原理分析
  - Fiber
  - Fiber算法
  - Fiber优先级
  - Fiber调度
  - Fiber任务循环
  - React状态更新
---
## 前言
React 近两年更新也相对较大，从节点树到跟新机制，到数据结构都有了很大的变化，本文主要介绍16.4+以后的React源码分析及原理，之前的相关内容已删除，不做保留。

## 简介
&emsp;&emsp;学习React就要学习它的架构设计和原理，众所周知，React是前端`MV*`框架的主流，其实这种说法每个人有不同的理解，有人说React是MAC，有人说是MVVM，有人说从React将Router和DOM分离出去后React只能算VM，加上其全家桶才能组成MVVM，更有人说React仅仅是一个library，不是MVVM，因为不满足MVVM双向绑定的特性，这些说法其实都有依据，不能说错，但也不能说对，React设计初期是跟前端MVVM框架几乎同时出现，设计者的初衷还是基于MVVM去开发框架的，个人比较偏向与`从React将Router和DOM分离出去后React只能算VM，加上其全家桶才能组成MVVM`这种说法，但也不完全认同，React单个库中有单独的状态管理，有MVVM实现，因此从全局看React不是一个完整的MVVM框架，但局部不缺MVVM的设计和使用。
&emsp;&emsp;React原理在前端框架中也是独树一帜的，Vue都大量借鉴，而新版本中的内容更是`花里胡哨`，有意思的很，下面主要介绍新版React中数据流、节点结构、任务调度、分片算法、状态更新等，涉及到具体的更详细的内容不在本文中介绍，请到专项文章中了解，例如`虚拟Dom Diff算法`、`Redux`等等。

## 节点树
React16-之前的虚拟DOM树或者节点树大部分人都有所了解，本质上是一个大的JSON对象，从根节点开始children存放所有子节点数组这种层层嵌套的结构，如下图所示：
{% asset_img img-left node_list_old.png 旧版节点树 %}
从上图可以看出树结构就是通过每个节点的children组成父子关系的json结构，当发生更新时从触发节点向下通过深度递归的方式进行遍历节点更新；优先完成子节点更新。其遍历都是通过数组索引进行。然而在现在，前端对于性能要求越来越高，这种设计结构已经不满足用户需求了，当树结构嵌套过深时，DOM树更新会阻塞用户行为，例如输入卡顿，动画卡顿等等，主要原因还是因为js是同步的，DOM树的Diff及更新都是在js线程中执行的，当一次更新任务耗时较长时明显会阻塞js其他行为。因此在React16开始，对于DOM树结构做了改变，由原来的JSON树变成了`链表树`，其结构图如下：  
{% asset_img img-left node_list_new.png 旧版节点树 %}
其结构变成了链表树，父节点只跟第一个子节点有直接关系，其他子节点都是右序节点，更新遍历是通过树的深度优先算法进行遍历
> Fiber节点中的`return`和`stateNode`属性是个私有属性，默认情况下不希望开发者使用的，因此typing文件中并没有对外开放，但是部分场景下我们可以通过这两个属性实现获取父组件属性的需求，例如`嵌套权限控制`等，想要了解可以通过控制台输出查看即可，获取当前Fiber节点的办法：
> - 类组件中：this._reactInternalFiber 
> - 函数式组件中：JSX变量的`_owner`属性，例如：
> ```typescript
>    const pageJSX = <div>1111</div>;
>    console.log(pageJSX);
>    return pageJSX;
> ```

数据结构改变的优点如下：
- 采用游标遍历和深度优先算法遍历，可以提升遍历效率同时可以做到`中断控制`及`恢复`
- 节点新增，删除效率提升，不需要从数组中插入删除，数组中插入删除涉及到元素前移后移，链表直接新增节点，断开节点即可

## React是一个单向数据流的库，状态驱动视图。

## 任务调度、分片
### Fiber 任务拆分
React实现可以粗划为两部分：`reconciliation`（diff阶段）和 `commit`(操作DOM阶段)。在 v16 之前，`reconciliation` 简单说就是一个自顶向下递归算法，产出需要对当前DOM进行更新或替换的操作列表，一旦开始，会持续占用主线程，中断操作却不容易实现。当JS长时间执行（如大量计算等），会阻塞样式计算、绘制等工作，出现页面脱帧现象。所以，v16 进行了一次重写，迎来了代号为Fiber的异步渲染架构。

> Fiber核心是实现了一个基于优先级和requestIdleCallback的循环任务调度算法。它包含以下特性：(参考：fiber-reconciler)
> - reconciliation阶段可以把任务拆分成多个小任务
> - reconciliation阶段可随时中止或恢复任务
> - 可以根据优先级不同来选择优先执行任务

从其特性可看出，Fiber核心是更换了reconciliation阶段的运作。那么：
- 为什么对reconciliation阶段进行拆分，而不在commit阶段处理：
reconciliation阶段包含的主要工作是对current tree 和 new tree 做diff计算，找出变化部分。进行遍历、对比等是可以中断，延迟后继续执行的。
commit阶段是对上一阶段获取到的变化部分应用到真实的DOM树中，是一系列的DOM操作。不仅要维护更复杂的DOM状态，而且中断后再继续，会对用户体验造成影响。在普遍的应用场景下，此阶段的耗时比diff计算等耗时相对短。
所以，Fiber选择在reconciliation阶段拆分

- 如何拆分：
{% note primary %}
1. 用户调用ReactDOM.render传入组件，React创建Element树;
2. 在第一次渲染时，创建vdom树，用来维护组件状态和dom节点的信息（如List/Button/Item等）。当后续操作如render或setState时需要更新，通过diff算出变化的部分；
3. 根据变化的部分更新vdom树、调用组件生命周期函数等，同步应用到真实的DOM节点中。
{% endnote %}
在第二阶段，Fiber是把render/update分片，拆解成多个小任务来执行，每次只检查树上部分节点，做完此部分后，若当前一帧（16ms）内还有足够的时间就继续做下一个小任务，时间不够就停止操作，等主线程空闲时再恢复。
这种停止/恢复操作，需要记录上下文信息。而当前只记录单一dom节点的vDom tree 是无法完成的，
Fiber Tree节点结构大致如下：
```typescript
export type Fiber = {
    tag: TypeOfWork, // 类型
    type: 'div',
    return: Fiber|null, // 父节点
    child: Fiber|null, // 子节点
    sibling: Fiber|null, // 兄弟节点
    alternate: Fiber|null, //diff出的变化记录在这个fiber上
    .....
};
```
完整的结构可以参考源码[ReactFiber](https://github.com/facebook/react/blob/master/packages/react-reconciler/src/ReactInternalTypes.js#L66)
所以，Fiber是根据一个fiber节点（VDOM节点）来拆分，以fiber node为一个任务单元，一个组件实例都是一个任务单元。任务循环中，每处理完一个fiber node，可以中断/挂起/恢复。

### Fiber调度算法
#### requestIdleCallback
```typescript
window.requestIdleCallback(callback[, options:{timeout?:number}]);
//timeout：如果指定了timeout并具有一个正值，并且尚未通过超时毫秒数调用回调，那么回调会在下一次空闲时期被强制执行，尽管这样很可能会对性能造成负面影响。
```
浏览器提供的`requestIdleCallback` API可以让浏览器在空闲时间执行回调（开发者传入的方法），在回调参数中可以获取到当前帧（16ms）剩余的时间。利用这个信息可以合理的安排当前帧需要做的事情，如果时间足够，那继续做下一个任务，如果时间不够就歇一歇，挂起现有任务，再次调用requestIdleCallback来探知主线程不忙的时候，再继续恢复任务。
React 为了兼容大部分浏览器，自己实现了`requestIdleCallback` API,利用`Message`事件与`requestAnimationFrame`实现，具体可以查看源码[Scheduler](https://github.com/facebook/react/blob/master/packages/scheduler/src/Scheduler.js)

#### 优先级任务调度 
不同的任务分配不同的优先级，Fiber根据任务的优先级来动态调整任务调度，会优先执行高优先级的任务
```typescript
export const ImmediatePriority = 1;// 最紧急的任务
export const UserBlockingPriority = 2;// 用户交互反馈
export const NormalPriority = 3;// 默认任务
export const LowPriority = 4;// 低优先级任务，数据的更新
export const IdlePriority = 5;// 预估未来需要显示的任务，浏览器空闲时处理的任务
```
对应过期时间
```typescript
var maxSigned31BitInt = 1073741823;
//	过期时间
var IMMEDIATE_PRIORITY_TIMEOUT = -1;
var USER_BLOCKING_PRIORITY = 250;
var NORMAL_PRIORITY_TIMEOUT = 5000;
var LOW_PRIORITY_TIMEOUT = 10000;
var IDLE_PRIORITY = maxSigned31BitInt;
```
最终优先级是根据`update`的`expirationTime`属性来管理先后的，计算expirationTime返回值是各个公式`MAGIC_NUMBER_OFFSET - ((ms / UNIT_SIZE) | 0)`，其中`MAGIC_NUMBER_OFFSET`是个常量值为`1073741821`,ms是window.performance.now()（这个时间一般是从地址栏输入地址回车后到执行这个方法中间间隔的毫秒数），也就是ms越小，优先级应该越高，当ms越小时，expirationTime就越大，自然expirationTime越大，优先级越高；此`expirationTime`属性与Task的`expirationTime`属性不同，Task的`expirationTime`属性表示超时时间，等同于`requestIdleCallback`的tmeout，越小优先级越高。早期这两个之间是等同的，都是越小越高，后来有所改动，后续规则可能会继续发生变化。
关于优先级划分，React一直以来没有固定的标准，其计算规则当前任务队列中的任务数有关，目前并没有详细文档可以了解，等待后续研究补充。

#### 任务执行流程
创建任务的唯一接口为 `unstable_scheduleCallback(priorityLevel, callback, options)` 函数。任务分为两种，同步任务或异步延时任务。同步任务在 `unstable_scheduleCallback` 调用期间就会添加到 `taskQueue` 队列，且通过 `requestHostCallback` 函数立即执行；异步延时任务会添加到 `timerQueue` 队列，且通过 `requestHostTimeout` 函数延后执行。`unstable_scheduleCallback` 的执行流程图如下：
{% asset_img img-left task.png 任务执行流程 %}
在上图中，以 `workLoop` 方式循环调度 `taskQueue` 队列或以 `handleTimeout` 递归调度 `timerQueue` 队列这两种方式，只有一个在激活状态，也即 `requestHostCallback`、`requestHostTimeout` 只有一个在调用周期中。因为，`taskQueue` 队列调度完毕，会调用 `requestHostTimeout` 处理 `timerQueue` 队列；`timerQueue` 队列有一个任务进入 `taskQueue`，又会调用 `requestHostCallback` 处理 `taskQueue` 队列。

另外，`workLoop` 循环也使任务具有可并发执行的特性。任务若返回函数，这个函数将作为 `currentTask.callback` 执行内容，即在 `workLoop` 循环中保证任务的回调被立即执行。

#### 任务的暂停、中止等
我们再来看看 `requestHostCallback`、`requestHostTimeout`。在浏览器环境中，react 通过 `MessageChannel` 发送消息的方式触发任务的真正执行，详情可参看 `requestHostCallback` 函数 的实现，效果是在渲染后执行任务。至于 `requestHostTimeout`，react 则使用 setTimeout 方法实现。

#### 任务的暂停与恢复
需要指出的是，在单次 `workLoop` 循环中，如果 `taskQueue` 队列未被清空（比如任务被暂停了），react 会基于 `MessageChannel` 再次 发送消息，以处理 taskQueue 队列。为此，scheduler 对外提供 `unstable_pauseExecution、unstable_continueExecution` 接口用于暂停、恢复任务，这就是任务暂停与恢复。

#### 任务的中止
至于任务的中止，scheduler 提供 `unstable_getFirstCallbackNode` 函数 用于获取 `taskQueue` 的首个任务，然后就可以使用 `unstable_cancelCallback` 接口销毁任务了。

#### 任务循环的过程如下
1. 找到高优先级的待处理的节点
2. 检查当前节点是否需要更新，不需要的话，直接clone子节点，直接到5
3. 打个tag标记，更新自己（组件更新props，context等，DOM节点记下DOM change）
4. 通过render获取子节点，生成子节点的workInProgress节点
5. 若没有产生子节点，归并diff出的不同部分effect list（包含DOM change）到父节点
6. 把子节点或兄弟节点作为待处理任务，准备进入下一个任务循环。若已经回到了workInProgress tree的根节点，则任务循环结束

#### 任务循环过程
在调用render或setState后，会克隆出一个镜像fiber，存储在当前节点`alternate`属性中，diff产生出的变化会标记在镜像fiber上。而alternate就是链接当前fiber tree和镜像fiber tree, 用于断点恢复。当前fiber节点的alternate属性指向workInProgress节点，对应workInProgress节点的alternate属性指向当前fiber节点。
通过下面一段代码我们看看如何实现任务循环：
```typescript
function Count({count}:{count:number}){
    return count;
}
function Sibling(){
    return "sibling"
}
function Test() {
    const [count,setCount] = useState(0);
    const onClick = useCallback(()=>{
        setCount((count)=>count+1);
    },[]);

    return (
        <div style={{width:200,height:200,backgroundColor:'red'}} onClick={onClick}>
            <Count count=count/>
        </div>
    );
}

const App=()=> {
  return (
    <div className="App">
      <Test/>
      <div><Sibling/></div>
    </div>
  );
};

ReactDOM.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
  document.getElementById('root')
);
```
以它为例创建fiber tree：
{% asset_img img-left tree.png FiberTree %}
再次setState或render时，构建workInProgress tree:
{% asset_img img-left clone.png Clone FiberTree %}

{% note info%}
1. 从Root节点向下遍历，查找相关节点Test
2. 先把根节点 Test克隆出来，并用child属性指向它在fiber tree中的子节点div.style
3. 把div.style从fiber tree中克隆出来，需要更新的话，加一个tag标志
4. 更新div.style节点的状态，属性的指向等，并通过render获取到它的新子节点
5. 尽量复用旧子节点Count来创建新子节点的workInProgress结构
6. 把新节点Count做为下一个任务，调用requestIdleCallback获取当前帧所剩余的时间，如果还足够，就继续Count的任务，重复2-4，否则，就等主线程空闲后再开始循环
7. 当子节点Count中没有child属性的指向，表示已到达底部。会把diff时产生的effect list会merge回return属性指向的父节点div.style上
8. 当父节点div.style没有兄弟节点时，会一直向上return回根节点，并把每个节点上产生的effect-list合并到根节点上。任务循环结束
{% endnote %}

## 状态跟新
React状态更新并没有vue那种跟踪机制，vue中是通过发布订阅模式去控制相关节点渲染的，核心使用的是`Proxy`或者`Getter Setter`，从而实现状态更新监听，并通过listener回调更新每个节点，相对来说，静态化内存，vue比React高，因为一个应用vue中会创建大量的订阅者，这也是大家说`React适合大项目，Vue适合中小项目`的原因之一。React中状态更新都是从跟节点root出发，进行链表遍历（遍历结果会缓存，并不是每次都需要从头遍历），生成需要更新的`alternate`(等同于`workInProgress` tree)子树，创建更新任务，并按照异步任务执行过程执行更新。

1. setState怎么实现合并的？
setState在`合成事件`和`钩子函数`中同步调用是会进行合并的，因为commit是在`合成事件`和`钩子函数`的调用顺序之后，因此才形成了合并更新，在旧版本中是通过变量`isBatchingUpdates`控制是否当前正在更新，后续更新操作是否放在消费队列中执行实现的，如果已经在执行更新则放在异步事务中，否则添加到当前消费队列，等待commit。默认是false，表示会立即更新，但是在`合成事件`和`钩子函数`的前置钩子中会调用`batchedUpdates`，该函数会把 `isBatchingUpdates` 修改为 true ,这样由 React 控制的事件处理过程 setState 不会同步更新 this.state。最新代码中已经删除了`isBatchingUpdates`，在Fiber任务队列中做了处理，一次commit之后会把前面的update Task合并执行，因为都是即时任务，超时都是-1，会被立即执行。
2. 同步异步
setState 只在`合成事件`和`钩子函数`中是“异步”的，在`原生事件`和`setTimeout`等中都是同步的。异步同步并不是说setState就是异步方法，其本身执行的过程和代码都是同步的，只是在React封装的事件机制`合成事件`和`钩子函数`中，commit操作是函数执行完成后执行的，而在其他场景，调用setState之后就会执行commit，因此出现了异步同步的现象，当然，同步并不是表示在调用setState之后可以立即获取到最新的state，最新的state在更新时才能获取，而更新是异步的，因此setState之后不管什么场景都无法直接获取到最新的state。`可以通过第二个参数callback来获取到更新后的结果setState(partialState, callback)`，callback会在组件更新之后执行，等同于`componentDidUpdate`

## Context源码（临时记录）
从跟节点通过alternate遍历出依赖的子节点树，进行更新，感觉每次都是从父节点遍历，遍历后进行记忆，与Vue使用数据拦截不同

## 总结
以上分析仅个人针对源码理解，如有错误，欢迎提出交流，无法保证与最新版本完全符合，目前React在快速迭代中。