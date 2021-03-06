---
title: Page Components Design
abbrlink: 210ae7d9
date: 2020-12-02 19:09:14
tags:
    - design
    - react
password: yanxianliang
message: 本文暂不开放，需要密码才可阅读.
wrong_pass_message: 密码错误，请输入正确的密码.
---
## 需求
为了实现快速生成多元化、多样化的网站，实现站点风格、元素自定义，从而解决UI相似，第三方平台通过相似度算法进行封站等问题。

## 设计
- 考虑点
    1. 组件化设计
    2. 性能问题
    3. 页面编辑器设计
    4. 框架选型
    5. 附加服务
    6. 窗口数据、状态同步
- 技术设计
1. 项目架构
{% asset_img 1.png 架构图 %}
+ 全局配置管理：用于管理站点全局配置，页面共享配置，网站公共api，站点数据管理层。
+ Pages：页面层，Page组件，用于组装页面组件列表，调用组件api，获取初始数据，实现页面服务端渲染。
+ 组件：`业务`组件库，用于互相组合形成页面。
+ 数据：每个组件拥有自己的api，进行异步调用，通过h2复用及reconnect解决创建链接问题，同时解决服务端同步取数据造成的性能问题。
+ 底层支撑：整个项目将使用到React、Preact、Nextjs、Nestjs、Jest、Typescript。PC端使用React，移动端使用Preact，通过plugin自动转换，实现开发过程无差别化，开发过程基于React开发，移动端使用Preact代替React能有效减少lib的体积；Nestjs 用于实现服务端渲染；Nestjs 作为Node环境框架，配合Nextjs实现服务端渲染系统，同时提供部分附加服务；Jest用于开发组件`UI Test` 及 `Unit Test`代码，组件及lib测试覆盖率必须达到`85%`以上；Typescript作为开发语言，严格限制代码规范，不符合规范的代码不允许提交。

2. 系统结构
{% asset_img 4.png 系统结构图 %}
    1. AppContext、Page、PageContext、PageContent 层都属于系统封装，不提供开发人员直接修改。PageContent支持开发人员处理声明周期事件，允许开发人员在其不同生命周期中实现打点、preload等逻辑。
    2. 组件在组件库中开发、发布、Unit Test、Ui Test、在h5和PC项目中根据配置进行动态倒入组件
    3. 组件拥有版本号属性，进行版本控制，结构如下：
    ```shell
        compponentA/
        ├── version1.0
        │   ├── .less
        │   ├── .tsx
        │   ├── .test.tsx
        │   └── .yaml
    ```
    - .less：组件样式文件
    - .tsx：组件内容文件
    - .test.tsx：Unit Test、UI Test文件，执行Jest时会遍历每个组件下该文件，进行检测并计算覆盖率
    - .yaml：组件属性及配置定义，用于编辑系统中生成JsonForm
    - 组件需要支持`QTP`标记，为自动化测试提供基础

3. ssr流程
{% asset_img 3.png ssr流程图 %}
    - Node服务获取网站组件静态配置，通过api获取会比较慢，发生修改需要通知Node进行缓存（memory或redis）,调用缓存中的配置进行服务端渲染，并通过`注水`完成客户端同构
    - 遍历组件列表，调用组件`getServerSideProps`方法获取initData，生成html，并脱水后返回给浏览器
    - 客户端注水，完成框架同构
    - 路由切换由客户端处理渲染逻辑，根据下一页面组件配置数据生成骨架屏页面，并由客户端发起api请求
4. 完整流程
{% asset_img 2.png 完整流程 %}
    + ssr需要根据国家、语言、货币等key进行缓存，如商品列表页面添加缓存的同时需要确保列表支持`商品重复检测`，不显示重复商品，因缓存数据可能更新不及时，加载下一页时数据可能会重复。
    + PWA在PC和h5中都需要使用，最基本用于实现强缓存框架类文件。组件及业务代码也可缓存，编译时如文件发生变化，须改变生成文件名，文件名生成通过文件内容hash8生成
   
## 优化分析
{% asset_img 5.jpg 优化分析 %}
性能问题及相关解决方案基本参照上述思维导图即可进行优化，在开发过程中需要严格按照相应要求进行开发。

## 附加工程化
1. components-loader：webpack loader，项目中引入时自动引入所有组件，并以dynamic方式注册，结局schema解析时通过componentName和version追溯到组件实现的问题。
2. editor-loader：webpack loader，编辑器项目中引入组件库时自动转换成Landing Page源码能识别的组件json格式。
