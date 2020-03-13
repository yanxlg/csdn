---
title: rn-base
abbrlink: 140ab620
date: 2020-02-27 10:05:22
tags: 随笔
categories: 随笔
---

## 前言
大型RN项目中离不开拆包优化，通常拆包将bundle拆分成base和module包，base包中主要是将通用的第三方库及通用组件编译成一个独立的包供RN Context初始化时做全局加载，在页面加载时只需要加载并执行对应的module包即可。

## 环境依赖
- npm
- git

## vv-rn base包开发方式
1. clone 项目到本地
```shell
git clone https://g.gitvv.com/frontend/vv-rn.git
# or
git clone ssh://git@git.gitvv.com:38022/frontend/vv-rn.git
```
如无项目权限，请联系相关负责人授予项目权限

2. 前端项目初始化
```shell
npm config set registry http://npm.gitvv.com/  # 设置镜像源
npm login                                     # 登录
npm install                                   # 安装依赖
```

3. 修改相关代码
代码可以直接修改，不需要修改配置即可发布生效

4. 添加组件及工具类等公共文件
* 组件在`components`目录中添加，并需要在`components/index.ts`中`export`导出
* 工具类同样，在`utils`中添加，并需要在`utils/index.ts`中`export`导出
* 其他目录可自行添加，只要确保在`src/index.tsx`中`export`列表里包含添加的内容即可

5. 添加外部依赖模块
* package.json 中添加需要的模块并使用`npm install`或`yarn install` 安装，package.json详解见文章{% post_link node/package.json package.json%}
* 在`src/index.tsx`中`import`引入需要打到base 中的模块

6. 如何将node_modules中模块打进module包
* 在`业务项目`根目录创建`bundle.json`文件，具体格式如下
```json
{
    "^__prelude__$": false,
    "/node_modules/": false
}
```
>false：表示不编译进`module`包 
>true：表示编译进`module`包
>注意：编译进`modules`包中的模块需要确定`base`包中不存在

7. 编译配置
* `/build/metro.base.config.js`
业务项目编译`base`包的通用配置，一般不需要进行修改或者外部重新定义，因为里面集成了svg模块代码，需要了解相关配置才可外部重新定义

* `/build/metro.bundle.config.js`
业务项目编译`module`包的通用配置，一般不需要进行修改或者外部重新定义，因为里面集成了svg模块代码，需要了解相关配置才可外部重新定义，并根据编译时间生成`module`名，确保不同页面使用同一个文件，会将改文件编译成两个不同的模块进行引用，防止页面冲突

8. 发布
- 做完以上工作后，可以进行`vv-rn`的发布，发布前首先需要修改`package.json`中的`version`版本号，否则会发布失败
- 修改完版本号后，使用下面命令进行发布
```shell
npm run pub
```
pub命令对应文件在`/publish.js`中，如需了解，可以阅读。