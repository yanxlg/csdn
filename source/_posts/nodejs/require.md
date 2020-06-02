---
title: require
tags:
  - node
  - require
abbrlink: '41266748'
date: 2018-11-16 14:14:11
---
## 前言
在node开发或者前端工程化开发共，经常会需要用到`require`这个方法去加载一个文件，我们所认识的`require`似乎仅仅只有这一个功能，然而`nodejs`真的仅仅如此吗？`require`似乎并没有那么简单，只是我们了解的简单而已。

## 详细介绍
- 定义
我们先来看下关于`require`的接口定义
```typescript
interface NodeRequireFunction {
    (id: string): any;
}

interface NodeRequire extends NodeRequireFunction {
    resolve: RequireResolve;
    cache: any;
    /**
     * @deprecated
     */
    extensions: NodeExtensions;
    main: NodeModule | undefined;
}

interface RequireResolve {
    (id: string, options?: { paths?: string[]; }): string;
    paths(request: string): string[] | null;
}

interface NodeExtensions {
    '.js': (m: NodeModule, filename: string) => any;
    '.json': (m: NodeModule, filename: string) => any;
    '.node': (m: NodeModule, filename: string) => any;
    [ext: string]: (m: NodeModule, filename: string) => any;
}

declare var require: NodeRequire;

interface NodeModule {
    exports: any;
    require: NodeRequireFunction;
    id: string;
    filename: string;
    loaded: boolean;
    parent: NodeModule | null;
    children: NodeModule[];
    paths: string[];
}
```
从定义中看，我们会发现`require`除了可以用作方法外，还有四个额外的属性:
   + extensions：require支持的ext列表
   + cache：require缓存，多次require同一个文件会从cache中读取，并不会重新读取，因此在某些情况下并不会重新创建实例，如果想要清除cache，需要使用delete去删除cache中的key，直接置空会失效
   + main：可以当做入口文件的对应模块
    - filename：入口文件名
    - `其他一般不会使用，用到时再进行备注`
   + resolve：返回引用的（`模块的入口文件`或者`文件`）的绝对路径，通常我们需要用到fs读取模块中的某个文件，却不清楚模块安装位置时（可能全局安装、可能在`/node_modules`中，也可能在某个模块的子`node_modules`中），可以使用该方式去获取
