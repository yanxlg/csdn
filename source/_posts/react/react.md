---
abbrlink: dee17c46
title: React
tags: react
categories: react
date: 2019-9-20 13:20:41
password: mikemessi
message: 本文暂不开放，需要密码才可阅读.
wrong_pass_message: 密码错误，请输入正确的密码.
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
Fiber之前React状态更新是通过`isBatchingUpdates`状态机来控制是立即执行`performWork`进行更新还是合并成`事务`进行一次行`commit`，而目前由于Fiber异步渲染思想，React的更新阶段分为`reconciliation`和`commit`两个。
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
首先，我们可以通过 A Cartoon Intro to Fiber中的一张图来看：



更新：scheduler  

## 状态跟新
setState，React没有vue那种观察者模式，React主张一切有源，数据源主懂修改，修改时从当前节点创建更新任务

setState前后发生了什么，怎么走更新事务的，什么时候创建任务


1. setState合并更新策略：isBatchingUpdates：
在 React 的 setState 函数实现中，会根据一个变量 isBatchingUpdates 判断是直接更新 this.state 还是放到队列中延时更新，而 isBatchingUpdates 默认是 false，表示 setState 会同步更新 this.state；但是，有一个函数 batchedUpdates，该函数会把 isBatchingUpdates 修改为 true，而当 React 在调用事件处理函数之前就会先调用这个 batchedUpdates将isBatchingUpdates修改为true，这样由 React 控制的事件处理过程 setState 不会同步更新 this.state。

## Context源码