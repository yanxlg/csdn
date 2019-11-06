---
title: performance
abbrlink: 82d79681
date: 2019-11-06 15:11:51
tags:
    - h5
    - performance
---
## 前言
H5 项目中经常涉及到性能优化，此时需要知道页面的性能指数，并上报统计，如何知道页面的性能指数对于很多前端开发人员来说比较迷茫，本文提供一段统一的代码，统计并打印出该页面所有耗时。

## 代码
```typescript
setTimeout(function() {
    const performance = window.performance;
    if (performance) {
        var e = performance.getEntriesByType('navigation')[0], r = 0
        e || (r = (e = performance.timing).navigationStart)
        var n = [{
            key: 'Redirect',
            desc: '\u7f51\u9875\u91cd\u5b9a\u5411\u7684\u8017\u65f6',
            value: e.redirectEnd - e.redirectStart
        }, {
            key: 'AppCache',
            desc: '\u68c0\u67e5\u672c\u5730\u7f13\u5b58\u7684\u8017\u65f6',
            value: e.domainLookupStart - e.fetchStart
        }, {
            key: 'DNS',
            desc: 'DNS\u67e5\u8be2\u7684\u8017\u65f6',
            value: e.domainLookupEnd - e.domainLookupStart
        }, {
            key: 'TCP',
            desc: 'TCP\u8fde\u63a5\u7684\u8017\u65f6',
            value: e.connectEnd - e.connectStart
        }, {
            key: 'Waiting(TTFB)',
            desc: '\u4ece\u5ba2\u6237\u7aef\u53d1\u8d77\u8bf7\u6c42\u5230\u63a5\u6536\u5230\u54cd\u5e94\u7684\u65f6\u95f4 / Time To First Byte',
            value: e.responseStart - e.requestStart
        }, {
            key: 'Content Download',
            desc: '\u4e0b\u8f7d\u670d\u52a1\u7aef\u8fd4\u56de\u6570\u636e\u7684\u65f6\u95f4',
            value: e.responseEnd - e.responseStart
        }, {
            key: 'HTTP Total Time',
            desc: 'http\u8bf7\u6c42\u603b\u8017\u65f6',
            value: e.responseEnd - e.requestStart
        }, {
            key: 'DOMContentLoaded',
            desc: 'dom\u52a0\u8f7d\u5b8c\u6210\u7684\u65f6\u95f4',
            value: e.domContentLoadedEventEnd - r
        }, { 
            key: 'Loaded',
            desc: '\u9875\u9762load\u7684\u603b\u8017\u65f6', 
            value: e.loadEventEnd - r 
        }];
        console && console.table && console.table(n)
    }
}, 0)
```
