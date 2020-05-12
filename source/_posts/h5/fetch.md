---
title: Fetch
categories:
  - h5
tags:
  - h5
  - ajax
abbrlink: 43284c47
date: 2019-02-06 15:01:13
---
## 前言
前端开发中经常涉及到接口请求，而在h5中常用的接口请求框架有`fetch`、`axios`、`umi-request`，`fetch`功能比较简单，能够满足正常的api调用需要，`axios`和`umi-request`功能比较丰富，支持取消及拦截器操作。本文主要介绍`fetch`的功能扩展。

## fetch 请求取消
早期，fetch没有提供cancel操作是受到Promise的影响，计划中Promise是会提供cancel方法，fetch是打算使用Promise的cancel逻辑，但是Promise一直没有提供，因此造成fetch在使用方面不如`axios`和`umi-request`，但最近，fetch支持了cancel功能。其依赖于AbortController，低版本浏览器不支持AbortController的需要使用polyfill。
具体demo如下：
```typescript
const Ajax = (input: RequestInfo, init?: IIRequestInit):IFetchPromise<any>=>{
    const {timeout,signal} = init||{};
    let _fetch:IFetchPromise<any>;
    const controller = new AbortController();
    const _signal = controller.signal;
    const __fetch = fetch(input,{...init,signal:_signal}).then(response=>{
        return response.json()
    }).then((json={})=>{
        const {code,msg} = json;
        if(code===0){
            return json;
        }
        throw json;
    });
    signal?.addEventListener('abort', ()=>{
        controller.abort();
    });
    if(timeout === void 0){
        // @ts-ignore
        _fetch=__fetch
    }else{
        // @ts-ignore
        _fetch =Promise.race([
            __fetch,
            new Promise(function (_, reject) {
                setTimeout(function () {
                    reject(new TypeError(`Network request failed,timeout of ${timeout} ms exceeded.`));
                    controller.abort();
                }, timeout);
            })
        ]);
    }
    _fetch.cancel=()=>{
        controller.abort();
    };
    return _fetch;
};
```

## fetch 超时
fetch超时一直以来也没有提供参数设置，可以参照取消请求的方式实现超时功能，当超时时取消请求并抛出超时异常

## RN中优化
RN中需要给每个Page组件提供一个`fetch`实例，当页面卸载时，调用该实例的`cancel`回收所有未完成的请求任务，释放资源
