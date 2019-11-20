---
title: Call setState when unMounted
date: 2019-11-20 13:20:41
tags:
    -react
categories:
    -react
---
## 前言
React开发过程中经常涉及到异步修改状态的问题，然而开发过程中会发现，某些操作会导致控制台提示`当前组件已经卸载，不应该继续调用setState修改其状态`，本文关于该问题点进行讨论。

## 官方
[官方讨论](https://reactjs.org/blog/2015/12/16/ismounted-antipattern.html)

1. React中每个组件提供了`isMounted()`方法去判断该组件是否被渲染，因此第一种方案可以为
```typescript
if (this.isMounted()) { // This is bad.
  this.setState({...});
}
```
但是这种写法会使项目代码变得非常冗余且违背了警告的目的，官方也不建议使用

2. `_isMounted`变量控制
在`componentDidMount`中将`isMounted`设为`true`，在`componentWillUnmount`中将其设置为`false`，调用`setState`时通过该变量进行判断，自己跟踪装载状态是为了兼容后续react升级，即使`isMounted`不可用也不会受到影响

3. 将Api调用封装成可以`cancel`的对象
在`componentWillUnmount`中将接口调用cancel掉，但是官方的cancel只是简单的cancel，并不能直接cancel接口调用，仍然会消耗流量

## 方法推荐
1. 重写`Component`基类，改写`setState`方法
```typescript
// 仅供简单参考，具体使用根据个人框架理解去封装
class BaseComponent extends React.Component{
    constructor(){
        super();
        const setState = this.setState;
        this.setState=(data)=>{
            if(this.isMounted()){
                setState(data);
            }
        }
    }
}
```

2. 重写`Component`基类，重置`setState`方法
```typescript
// 仅供简单参考，具体使用根据个人框架理解去封装
class BaseComponent extends React.Component{
    componentWillUnmount(){
        this.setState=()=>{};
    }
}
```

3. Cancel接口
```typescript
// 仅供简单参考，具体使用根据个人框架理解去封装
class BaseComponent extends React.Component{
    constructor(){
        super();
        this.fetch=new Fetch();
    }
    // this.fetch.post()
    componentWillUnmount(){
        this.fetch.cancel();// cancel掉当前所有请求
    }
}
```
